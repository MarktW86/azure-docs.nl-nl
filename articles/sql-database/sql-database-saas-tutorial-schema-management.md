---
title: Een Azure SQL Database-schema beheren in een app met meerdere tenants | Microsoft Docs
description: Het schema voor meerdere tenants beheren in een toepassing met meerdere tenants die gebruikmaakt van Azure SQL Database
keywords: zelfstudie sql-database
services: sql-database
documentationcenter: 
author: stevestein
manager: jhubbard
editor: 
ms.assetid: 
ms.service: sql-database
ms.custom: tutorial
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 05/10/2017
ms.author: billgib; sstein
ms.translationtype: Human Translation
ms.sourcegitcommit: fc4172b27b93a49c613eb915252895e845b96892
ms.openlocfilehash: 19d02229781186053a0063af1c7e1a3280179f46
ms.contentlocale: nl-nl
ms.lasthandoff: 05/12/2017


---
# <a name="manage-schema-for-multiple-tenants-in-the-wtp-saas-application"></a>Het schema voor meerdere tenants beheren in de WTP SaaS-toepassing

In de zelfstudie met een inleiding tot de WTP-toepassing ziet u hoe de WTP-app een tenantdatabase kan inrichten met zijn basisschema en deze database kan registreren in de database. Net als elke toepassing verandert de WTP-app na verloop van tijd, en moeten er af en toe wijzigingen worden aangebracht aan de database. Dit kunnen bijvoorbeeld wijzigingen zijn aan het schema en de referentiegegevens, of er kan routineus database-onderhoud worden uitgevoerd om ervoor te zorgen dat de app optimaal presteert. Bij een SaaS-toepassing moeten deze wijzigingen op een gecoördineerde wijze worden geïmplementeerd in een mogelijk enorme reeks tenantdatabases. De wijzigingen moeten ook worden opgenomen in het inrichtingsproces voor toekomstige tenantdatabases.

In deze zelfstudie worden twee scenario's beschreven: referentiegegevensupdates implementeren voor alle tenants en een index retourneren van de tabel met de referentiegegevens. De functie [Elastische taken](sql-database-elastic-jobs-overview.md) wordt gebruikt om deze bewerkingen voor alle tenants uit te voeren. Er wordt een *basis*tenantdatabase gebruikt als sjabloon voor nieuwe databases.

In deze zelfstudie leert u het volgende:

> [!div class="checklist"]

> * Een taakaccount maken om query's uit te voeren voor meerdere tenants
> * Gegevens in alle tenantdatabases bijwerken
> * Een index in een tabel maken in alle tenantdatabases


Voor het voltooien van deze zelfstudie moet u ervoor zorgen dat aan de volgende vereisten wordt voldaan:

* De WTP-app is geïmplementeerd. Zie [De WTP SaaS-toepassing implementeren en verkennen](sql-database-saas-tutorial.md) om dit in minder dan vijf minuten te doen
* Azure PowerShell is geïnstalleerd. Zie [Aan de slag met Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) voor meer informatie.
* De laatste versie van SQL Server Management Studio (SSMS) moet zijn geïnstalleerd. [SSMS downloaden en installeren](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)

*In deze zelfstudie wordt gebruikgemaakt van de functies van de SQL Database-service die deel uitmaken van een beperkte preview (elastische databasetaken). Als u deze zelfstudie wilt volgen, stuurt u uw abonnements-id naar SaaSFeedback@microsoft.com, met als onderwerp 'Elastic Jobs Preview'. Wanneer u de bevestiging hebt ontvangen dat uw abonnement is ingeschakeld, [downloadt en installeert u de taak-cmdlets van de voorlopige versie](https://github.com/jaredmoo/azure-powershell/releases). Omdat dit een beperkte preview is, neemt u contact op met SaaSFeedback@microsoft.com bij vragen of voor ondersteuning.*


## <a name="introduction-to-saas-schema-management-patterns"></a>Kennismaking met patronen voor SaaS-schemabeheer

Het SaaS-patroon met één tenant per database is enorm handig door de gegevensisolatie die hierdoor ontstaat, maar tegelijkertijd ontstaat er extra complexiteit omdat er vele databases moeten worden onderhouden en beheerd. [Elastische taken](sql-database-elastic-jobs-overview.md) vereenvoudigt het onderhoud en het beheer van de SQL-gegevenslaag. Met Elastische taken kunt u veilig en betrouwbaar taken uitvoeren (T-SQL-scripts) voor een groep databases, onafhankelijk van de interactie met of invoer van gebruikers. Deze methode kan worden gebruikt om wijzigingen in het schema en in de algemene referentiegegevens te implementeren voor alle tenants in een toepassing. Elastische taken kan ook worden gebruikt om een *basis*exemplaar van de gebruikte database op te slaan voor het maken van nieuwe tenants. Op die manier beschikken deze altijd over het nieuwste schema en de meest recente referentiegegevens.

![scherm](media/sql-database-saas-tutorial-schema-management/schema-management.png)


## <a name="elastic-jobs-limited-preview"></a>Beperkte preview voor Elastische taken

Er is een nieuwe versie beschikbaar van Elastische taken; dit is nu een ingebouwde functie in Azure SQL Database (waarvoor geen extra services of onderdelen zijn vereist). Van deze nieuwe versie van Elastische taken is momenteel een beperkte preview beschikbaar. Deze beperkte preview biedt ondersteuning voor PowerShell voor het maken van taakaccounts en voor T-SQL voor het maken en beheren van taken.

> [!NOTE]
> *In deze zelfstudie wordt gebruikgemaakt van de functies van de SQL Database-service die deel uitmaken van een beperkte preview (elastische databasetaken). Als u deze zelfstudie wilt volgen, stuurt u uw abonnements-id naar SaaSFeedback@microsoft.com, met als onderwerp 'Elastic Jobs Preview'. Wanneer u de bevestiging hebt ontvangen dat uw abonnement is ingeschakeld, [downloadt en installeert u de taak-cmdlets van de voorlopige versie](https://github.com/jaredmoo/azure-powershell/releases). Omdat dit een beperkte preview is, neemt u contact op met SaaSFeedback@microsoft.com bij vragen of voor ondersteuning.*

## <a name="get-the-wingtip-application-scripts"></a>De scripts van de Wingtip-toepassing downloaden

De scripts en broncode van de Wingtip Tickets-toepassing zijn beschikbaar in de GitHub-opslagplaats [WingtipSaaS](https://github.com/Microsoft/WingtipSaaS). U vindt de scriptbestanden in de map [Learning Modules](https://github.com/Microsoft/WingtipSaaS/tree/master/Learning%20Modules). Download de map **Learning Modules** naar de lokale computer en behoud de mapstructuur.

## <a name="create-a-job-account-database-and-new-job-account"></a>Een taakaccountdatabase en een nieuw taakaccount maken

In deze zelfstudie moet u PowerShell gebruiken om de taakaccountdatabase en het taakaccount te maken. Net als MSDB en SQL Agent maakt Elastische taken gebruik van een Azure SQL-database om taakdefinities, de taakstatus en de geschiedenis op te slaan. Wanneer het taakaccount is gemaakt, kunt u direct aan de slag met het maken en bewaken van taken.

1. Open... \\Learning Modules\\Schemabeheer\\*Demo-SchemaManagement.ps1* in de **PowerShell ISE**.
1. Druk op **F5** om het script uit te voeren.

Met het script *Demo-SchemaManagement.ps1* wordt het script *Deploy-SchemaManagement.ps1* aangeroepen om op de catalogusserver een *S2*-database te maken met de naam **jobaccount**. Het taakaccount wordt dan gemaakt. De database van dat account wordt doorgegeven als parameter bij de aanroep voor het maken van het taakaccount.

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>Een taak maken om nieuwe referentiegegevens te implementeren voor alle tenants

Elke tenantdatabase bevat een reeks locatietypen waarmee wordt gedefinieerd welke soorten evenementen bij die locatie worden georganiseerd. In deze oefening implementeert u een update voor alle tenantdatabases waarbij twee nieuwe locatietypen worden toegevoegd: *Motorcycle Racing* en *Swimming Club*. Deze locatietypen sluiten aan op de achtergrondafbeelding die u in de app voor tenantevenementen ziet.

Klik op het vervolgkeuzemenu Locatietype en controleer of er slechts tien locatietypen beschikbaar zijn. Bekijk ook of Motorcycle Racing en Swimming Club niet in de lijst staan.

U gaat nu een taak maken om de tabel *VenueTypes* in alle tenantdatabases bij te werken met de nieuwe locatietypen.

Voor het maken van een nieuwe taak gebruikt u een reeks in het systeem opgeslagen procedures uit de jobaccount-database die is gemaakt tijdens het maken van het taakaccount.

1. Open SSMS en maak verbinding met deze catalogusserver: catalog-\<user\>.database.windows.net
1. Maak ook verbinding met deze tenantserver: tenants1-\<user\>.database.windows.net
1. Blader naar de *contosoconcerthall*-database op de *tenants1*-server en voer een query uit voor de *VenueTypes*-tabel om te controleren of *Motorcycle Racing* en *Swimming Club* **in de resultatenlijst** staan.
1. Open het bestand …\\Learning Modules\\Schemabeheer\\DeployReferenceData.sql
1. Bewerk de \<gebruiker\>: gebruik de gebruikersnaam die u hebt gebruikt bij het implementeren van de WTP-app op alle drie de locaties in het script
1. Zorg dat u verbinding hebt met de jobaccount-database en druk op **F5** om het script uit te voeren

* Met **sp\_add\_target\_group** maakt u de doelgroepnaam DemoServerGroup. Nu gaat u de beoogde leden toevoegen.
* Met **sp\_add\_target\_group\_member** voegt u een *server*doellidtype toe, waarbij ervan uit wordt gegaan dat alle databases op de server (dit is de customer1-&lt;WtpUser&gt;-server met de tenantdatabases) tijdens het uitvoeren van de taak deel uitmaken van de taak. Ten tweede wordt een *database*doellidtype toegevoegd, de basisdatabase (baseTenantDB), die op de catalog-&lt;WtpUser&gt;-server staat. Tot slot wordt een ander *database*doelgroeplidtype toegevoegd zodat ook de adhocanalytics-database wordt toegevoegd die in een latere zelfstudie wordt gebruikt.
* Met **sp\_add\_job** maakt u een taak die ook wel 'Implementatie van referentiegegevens' heet
* Met **sp\_add\_jobstep** maakt u de taakstap met de T-SQL-opdracht om de referentietabel VenueTypes bij te werken
* De resterende weergaven in het script tonen het bestaan van de objecten en controleren de taakuitvoering. Bekijk de statuswaarde in de kolom **lifecycle**. De taak is voltooid op alle tenantdatabases en de twee andere databases die de referentietabel bevatten.

1. Blader in SSMS naar de *contosoconcerthall*-database op de *tenants1*-server en voer een query uit voor de *VenueTypes*-tabel om te controleren of *Motorcycle Racing* en *Swimming Club* **in de resultatenlijst** staan.


## <a name="create-a-job-to-manage-the-reference-table-index"></a>Een taak maken voor het beheren van de referentietabelindex

Net als bij de vorige oefening, wordt bij deze oefening een taak gemaakt om de index op de primaire sleutel van de referentietabel opnieuw op te bouwen. Dit is een typische databasebeheerbewerking die een beheerder zal moeten uitvoeren bij het laden van veel gegevens in een tabel.

Maak een taak met dezelfde in het systeem opgeslagen procedures.

1. Open SSMS en maak verbinding met de server catalog-&lt;WtpUser&gt;.database.windows.net
1. Open het bestand …\\Learning Modules\\Schemabeheer\\OnlineReindex.sql
1. Klik met de rechtermuisknop, selecteer Verbinding en maak verbinding met de catalog-&lt;WtpUser&gt;.database.windows.net-server, als deze verbinding nog niet is gemaakt
1. Zorg dat u verbinding hebt met de jobaccount-database en druk op F5 om het script uit te voeren

* Met sp\_add\_job maakt u een nieuwe taak met de naam Online Reindex PK\_\_VenueTyp\_\_265E44FD7FD4C885
* Met sp\_add\_jobstep maakt u de taakstap met de T-SQL-opdracht om de index bij te werken




## <a name="next-steps"></a>Volgende stappen

In deze zelfstudie hebt u het volgende geleerd:

> [!div class="checklist"]

> * Een taakaccount maken om query's uit te voeren voor meerdere tenants
> * Gegevens in alle tenantdatabases bijwerken
> * Een index in een tabel maken in alle tenantdatabases

[Zelfstudie over ad-hocanalyses](sql-database-saas-tutorial-adhoc-analytics.md)


## <a name="additional-resources"></a>Aanvullende bronnen

* [Aanvullende zelfstudies die voortbouwen op de eerste implementatie van WTP (Wingtip Tickets Platform)](sql-database-wtp-overview.md#sql-database-wtp-saas-tutorials)
* [Uitgeschaalde clouddatabases beheren](sql-database-elastic-jobs-overview.md)
* [Uitgeschaalde clouddatabases maken en beheren](sql-database-elastic-jobs-create-and-manage.md)
