---
title: Beveiliging bewaken in Azure Security Center | Microsoft Docs
description: Dit artikel helpt u aan de slag te gaan met de bewakingsmogelijkheden in Azure Security Center.
services: security-center
documentationcenter: na
author: YuriDio
manager: mbaldwin
editor: 
ms.assetid: 3bd5b122-1695-495f-ad9a-7c2a4cd1c808
ms.service: security-center
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/09/2017
ms.author: yurid
ms.translationtype: Human Translation
ms.sourcegitcommit: 2db2ba16c06f49fd851581a1088df21f5a87a911
ms.openlocfilehash: f7f50c305a5ae6ad7bba4e793f4cb6a0735b80b4
ms.contentlocale: nl-nl
ms.lasthandoff: 05/08/2017


---
# <a name="security-health-monitoring-in-azure-security-center"></a>Beveiligingsstatus bewaken in Azure Security Center
Dit artikel bevat informatie over het gebruik van de bewakingsmogelijkheden in Azure Security Center om naleving van het beleid te controleren.

## <a name="what-is-security-health-monitoring"></a>Wat houdt de bewaking van de beveiligingsstatus in?
Bij het woord bewaking wordt misschien gesuggereerd dat er wordt gewacht tot een gebeurtenis plaatsvindt en dat er dan op de situatie wordt gereageerd. Bij beveiligingsbewaking is er echter sprake van een proactieve strategie waarbij uw resources worden gecontroleerd om systemen op te sporen die niet voldoen aan de standaarden of aanbevolen procedures van de organisatie.

## <a name="monitoring-security-health"></a>Beveiligingsstatus bewaken
Nadat u een [beveiligingsbeleid](security-center-policies.md) voor de resources van een abonnement hebt ingeschakeld, analyseert Security Center de beveiliging van uw resources om mogelijke beveiligingsproblemen op te sporen. Informatie over uw netwerkconfiguratie is onmiddellijk beschikbaar. Het kan echter wel een uur of langer duren voordat er informatie over de configuratie van een virtuele machine, zoals de status van een beveiligingsupdate en configuratie van het besturingssysteem, beschikbaar komt. U kunt de beveiligingsstatus van uw resources en eventuele problemen bekijken in het gedeelte **Preventie**. U kunt ook een overzicht van die problemen op de tegel **Aanbevelingen** bekijken.

Zie [Beveiligingsaanbevelingen implementeren in Azure Security Center](security-center-recommendations.md) voor meer informatie over het toepassen van aanbevelingen.

In het gedeelte **Preventie** kunt u de beveiligingsstatus van uw resources bewaken. In het volgende voorbeeld ziet u dat in de tegel van elke resource (Berekenen, Netwerken, Opslag en gegevens, en Toepassing) het totale aantal problemen dat is geïdentificeerd wordt weergegeven.

![De tegel Beveiligingsstatus van de resource](./media/security-center-monitoring/security-center-monitoring-fig1-newUI-2017.png)


### <a name="monitor-compute"></a>Berekenen controleren
Als u op de tegel **Berekenen** klikt, wordt de blade **Berekenen** geopend met daarop drie tabbladen:

- **Overzicht**: aanbevelingen voor controle en virtuele machines.
- **Virtuele machines**: lijst met alle virtuele machines en de bijbehorende actuele beveiligingsstatussen.
- **Cloudservices**: lijst met alle web- en worker-rollen gecontroleerd met het beveiligingscentrum.

![Ontbrekende systeemupdate per virtuele machine](./media/security-center-monitoring/security-center-monitoring-fig1-new002-2017.png)

Op elk tabblad kunt u meerdere sectie hebben en in elke sectie kunt u een afzonderlijke optie selecteren voor meer informatie over de aanbevolen stappen voor het oplossen van het specifieke probleem. 

#### <a name="monitoring-recommendations"></a>Aanbevelingen ten aanzien van controle
In deze sectie wordt het totale aantal virtuele machines weergegeven dat is geïnitialiseerd voor het verzamelen van gegevens en de huidige status. Nadat voor alle virtuele machines gegevensverzameling is geïnitialiseerd, zijn ze klaar om Security Center-beveiligingsbeleid te ontvangen. Als u op deze vermelding klikt, wordt de blade **VM-agent ontbreekt of reageert niet** geopend. 

![Ontbrekende systeemupdate per virtuele machine](./media/security-center-monitoring/security-center-monitoring-fig1-new003-2017.png)


#### <a name="virtual-machine-recommendations"></a>Aanbevelingen voor virtuele machines
Dit gedeelte bevat een reeks [aanbevelingen voor elke virtuele machine](security-center-virtual-machine-recommendations.md) die wordt bewaakt door Azure Security Center. De eerste kolom bevat de aanbeveling. De tweede kolom geeft het totale aantal virtuele machines dat wordt beïnvloed door deze aanbeveling. De derde kolom geeft de ernst van het probleem weer, zoals wordt geïllustreerd op de volgende schermafbeelding.

![Aanbevelingen voor virtuele machines](./media/security-center-monitoring/security-center-monitoring-fig1-new004-2017.png)

> [!NOTE]
> Alleen virtuele machines met ten minste één openbaar eindpunt worden weergegeven op de blade **Netwerkstatus** in de lijst **Netwerktopologie**.
>
>

Elke aanbeveling heeft een set acties die kunnen worden uitgevoerd wanneer u erop klikt. Als u bijvoorbeeld op **Ontbrekende systeemupdates** klikt, wordt de blade **Ontbrekende systeemupdates** geopend. U ziet hier de virtuele machines waarvoor patches ontbreken, en de ernst van de ontbrekende update, zoals wordt weergegeven op de volgende schermafbeelding.

![Ontbrekende systeemupdates voor virtuele machines](./media/security-center-monitoring/security-center-monitoring-fig5-ga.png)

Op de blade **Ontbrekende systeemupdates** wordt een tabel met de volgende informatie weergegeven:

* **VIRTUELE MACHINE**: de naam van de virtuele machine waarvoor updates ontbreken.
* **SYSTEEMUPDATES**: het aantal systeemupdates dat ontbreekt.
* **TIJD VAN DE LAATSTE SCAN**: het tijdstip waarop de virtuele machine voor het laatst door Security Center op updates is gecontroleerd.
* **STATUS**: de huidige status van de aanbeveling:
  * **Open**: de aanbeveling is nog niet opgelost.
  * **Wordt uitgevoerd**: de aanbeveling wordt momenteel toegepast op deze resources en u hoeft geen actie te ondernemen.
  * **Opgelost**: de aanbeveling is al voltooid. (Als het probleem is opgelost, wordt de vermelding grijs).
* **ERNST**: hiermee wordt de ernst van deze bepaalde aanbeveling beschreven:
  * **Hoog**: er bestaat een beveiligingsprobleem voor een belangrijke resource (toepassing, virtuele machine, netwerkbeveiligingsgroep) en dit probleem vereist uw aandacht.
  * **Gemiddeld**: er zijn niet-kritieke of extra stappen nodig om een proces te voltooien of een beveiligingsprobleem op te lossen.
  * **Laag**: een beveiligingsprobleem moet worden opgelost, maar dit vereist niet uw onmiddellijke aandacht. (Aanbevelingen met de ernstaanduiding Laag worden niet standaard weergegeven, maar u kunt hierop filteren als u deze aanbevelingen wilt bekijken.)

Klik op de naam van de virtuele machine om meer informatie voor de aanbeveling weer te geven. Er wordt een nieuwe blade voor die virtuele machine geopend met de lijst met updates zoals weergegeven op de volgende schermafbeelding.

![Ontbrekende systeemupdates voor een specifieke virtuele machine](./media/security-center-monitoring/security-center-monitoring-fig6-ga.png)

> [!NOTE]
> Dit zijn dezelfde beveiligingsaanbevelingen als die op de blade **Aanbevelingen**. Zie het artikel [Beveiligingsaanbevelingen implementeren in Azure Security Center](security-center-recommendations.md) voor meer informatie over het toepassen van aanbevelingen. Dit is niet alleen van toepassing voor virtuele machines, maar ook voor alle resources die beschikbaar zijn in de tegel **Resourcestatus**.
>
>

#### <a name="virtual-machines-section"></a>Sectie voor virtuele machines
De sectie voor virtuele machines geeft een overzicht van alle virtuele machines en aanbevelingen. Elke kolom vertegenwoordigt een reeks aanbevelingen, zoals te zien is op de volgende schermafbeelding:

![Overzicht van alle virtuele machines en aanbevelingen](./media/security-center-monitoring/security-center-monitoring-fig1-new005-2017.png)

U kunt met het pictogram dat wordt weergegeven onder elke aanbeveling, snel zien welke virtuele machines aandacht vereisen en wat het type aanbeveling is.

In het vorige voorbeeld heeft één virtuele machine een cruciale aanbeveling met betrekking tot eindpuntbeveiliging. Klik op de virtuele machine als u meer informatie over de virtuele machine wilt bekijken. Er wordt een nieuwe blade geopend over deze virtuele machine, zoals wordt weergegeven op de volgende schermafbeelding.

![Beveiligingsdetails van virtuele machines](./media/security-center-monitoring/security-center-monitoring-fig8-ga.png)

Deze blade bevat de beveiligingsdetails voor de virtuele machine. Onder in deze blade ziet u de aanbevolen actie en de ernst van elk probleem.

#### <a name="cloud-services-section"></a>Veelgestelde vragen over cloudservices
Voor cloudservices wordt een aanbeveling gemaakt wanneer de versie van het besturingssysteem verouderd is, zoals u ziet in de volgende schermafbeelding:

![Status van cloudservices](./media/security-center-monitoring/security-center-monitoring-fig1-new006-2017.png)

Bij een scenario waarin u een aanbeveling krijgt (wat in het vorige voorbeeld niet het geval is), volgt u de stappen in de aanbeveling om de versie van het besturingssysteem bij te werken. Wanneer een update beschikbaar is, ontvangt u een waarschuwing (rood of oranje, afhankelijk van de ernst van het probleem). Als u klikt op deze waarschuwing in de WebRole1 (Windows Server wordt uitgevoerd met uw web-app automatisch geïmplementeerd in IIS) of WorkerRole1 (Windows Server wordt uitgevoerd met uw web-app automatisch geïmplementeerd in IIS), wordt een nieuwe blade geopend met meer informatie over deze aanbeveling. Dit wordt weergegeven op de volgende schermafbeelding:

![Details van de cloudservice](./media/security-center-monitoring/security-center-monitoring-fig8-new3.png)

Als u een meer beschrijvende uitleg wilt zien over deze aanbeveling, klikt u op **Besturingssysteemversie bijwerken** onder de kolom **BESCHRIJVING**. De blade **Besturingssysteemversie bijwerken (preview)** wordt geopend met meer informatie.

![Aanbevelingen ten aanzien van cloudservices](./media/security-center-monitoring/security-center-monitoring-fig8-new4.png)  

### <a name="monitor-virtual-networks"></a>Virtuele netwerken bewaken
Wanneer u op de tegel **Netwerken** klikt, wordt de blade **Netwerken** geopend met meer details, zoals op de volgende schermafbeelding wordt weergegeven:

![De blade Netwerken](./media/security-center-monitoring/security-center-monitoring-fig9-new3.png)

#### <a name="networking-recommendations"></a>Aanbevelingen voor netwerken
Net als bij de informatie over de resourcestatus van virtuele machines biedt deze blade bovenin een overzicht van de problemen en onderin een lijst met bewaakte netwerken.

In het gedeelte met een uitsplitsing van de netwerkstatussen vindt u de mogelijke beveiligingsproblemen en [aanbevelingen](security-center-network-recommendations.md). Mogelijke aandachtspunten zijn:

* NGFW (Next Generation Firewall) is niet geïnstalleerd
* De netwerkbeveiligingsgroepen op subnetten zijn niet ingeschakeld
* De netwerkbeveiligingsgroepen op virtuele machines zijn niet ingeschakeld
* Externe toegang via openbaar extern eindpunt beperken
* Status van internetgerichte eindpunten in orde

Wanneer u op een aanbeveling klikt, wordt een nieuwe blade geopend met meer informatie over de aanbeveling, zoals weergegeven in het volgende voorbeeld.

![Details voor een aanbeveling op de blade Netwerken](./media/security-center-monitoring/security-center-monitoring-fig9-ga.png)

In dit voorbeeld bevat de blade **Ontbrekende netwerkbeveiligingsgroepen voor subnetten configureren** een lijst met subnetten en virtuele machines zonder beveiliging van netwerkbeveiligingsgroepen. Als u op het subnet klikt waarop u de netwerkbeveiligingsgroep wilt toepassen, wordt er een andere blade geopend.

Selecteer op de blade **Netwerkbeveiligingsgroep kiezen** de meest geschikte netwerkbeveiligingsgroep voor het subnet of maak een nieuwe netwerkbeveiligingsgroep.

#### <a name="internet-facing-endpoints-section"></a>Sectie Internetgerichte eindpunten
De sectie **Internetgerichte eindpunten** bevat de virtuele machines die momenteel zijn geconfigureerd met een internetgericht eindpunt en de huidige status.

![Virtuele machines die zijn geconfigureerd met een internetgericht eindpunt en status](./media/security-center-monitoring/security-center-monitoring-fig10-ga.png)

Deze tabel bevat de eindpuntnaam die de virtuele machine vertegenwoordigt, het internetgerichte IP-adres en de huidige status van de ernst van de netwerkbeveiligingsgroep en de NGFW. De tabel is gesorteerd op ernst:

* Rood (bovenaan): hoge prioriteit en moet onmiddellijk worden opgelost
* Oranje: gemiddelde prioriteit en moet zo snel mogelijk worden opgelost
* Groen (laatste): integriteitsstatus

#### <a name="networking-topology-section"></a>Sectie Netwerktopologie
In de sectie **Netwerktopologie** ziet u een hiërarchische weergave van de resources, zoals te zien is op de onderstaande schermafbeelding:

![Hiërarchische weergave van resources in de sectie Netwerktopologie](./media/security-center-monitoring/security-center-monitoring-fig121-new4.png)

Deze tabel is gesorteerd (virtuele machines en subnetten) op ernst:

* Rood (bovenaan): hoge prioriteit en moet onmiddellijk worden opgelost
* Oranje: gemiddelde prioriteit en moet zo snel mogelijk worden opgelost
* Groen (laatste): integriteitsstatus

In deze topologieweergave bevat het eerste niveau [virtuele netwerken](../virtual-network/virtual-networks-overview.md), [virtuele netwerkgateways](/vpn-gateway/vpn-gateway-site-to-site-create.md) en [virtuele netwerken (klassiek)](/virtual-network/virtual-networks-create-vnet-classic-pportal.md). Het tweede niveau bevat subnetten en het derde niveau bevat de virtuele machines die horen bij deze subnetten. In de rechterkolom bevindt zich de huidige status van de netwerkbeveiligingsgroep voor deze resources, zoals in het volgende voorbeeld wordt weergegeven:

![Status van de netwerkbeveiligingsgroep in de sectie Netwerktopologie](./media/security-center-monitoring/security-center-monitoring-fig12-ga.png)

Het onderste gedeelte van deze blade bevat de aanbevelingen voor deze virtuele machine, vergelijkbaar met wat eerder is beschreven. U kunt op een aanbeveling klikken voor meer informatie of om het benodigde beveiligingsbeheer of de benodigde beveiligingsconfiguratie toe te passen.

### <a name="monitor-storage--data"></a>Opslag en gegevens controleren

Wanneer u klikt op **Opslag en gegevens** in het gedeelte **Preventie**, wordt de blade **Gegevensbronnen** geopend met aanbevelingen voor SQL en Storage. Ook bevat de blade [aanbevelingen](security-center-sql-service-recommendations.md) voor de algemene integriteitsstatus van de database. Lees voor meer informatie over de versleuteling van opslag [Versleuteling inschakelen voor een Azure-opslagaccount in Azure Security Center](security-center-enable-encryption-for-storage-account.md).

![Gegevensbronnen](./media/security-center-monitoring/security-center-monitoring-fig13-newUI-2017.png)

U kunt onder **SQL-aanbevelingen** op elke aanbeveling klikken voor meer informatie over verdere acties die u kunt ondernemen om een probleem te verhelpen. In het volgende voorbeeld is de aanbeveling **Databasecontrole en detectie van bedreigingen in SQL-databases** uitgevouwen.

![Details over een SQL-aanbeveling](./media/security-center-monitoring/security-center-monitoring-fig14-ga-new.png)

De blade **Controle en detectie van bedreigingen in SQL-databases inschakelen** bevat de volgende informatie:

* Een lijst met SQL-databases
* De server waarop deze zich bevinden
* Informatie over of deze instelling is overgenomen van de server of dat deze uniek is in deze database
* De huidige status
* De ernst van het probleem

Als u op de database klikt om deze aanbeveling op te volgen, wordt de blade **Controle en detectie van bedreigingen** geopend, zoals wordt weergegeven in het volgende scherm.

![De blade Controle en detectie van bedreigingen](./media/security-center-monitoring/security-center-monitoring-fig15-ga.png)

Als u controle wilt inschakelen, hoeft u alleen maar **AAN** te selecteren onder de optie **Controle**.

### <a name="monitor-applications"></a>Toepassingen bewaken

Als uw Azure-workload toepassingen in [virtuele machines (gemaakt met Azure Resource Manager)](../azure-resource-manager/resource-manager-deployment-model.md) heeft met ontsloten webpoorten (TCP-poorten 80 en 443), kunnen deze in Security Center worden bewaakt om mogelijke beveiligingsproblemen op te sporen en stappen voor herstel aan te bevelen. Wanneer u op de tegel **Toepassingen** klikt, wordt de blade **Toepassingen** geopend met een reeks aanbevelingen in de sectie met **Aanbevelingen voor toepassingen**. U ziet ook de uitsplitsing van de toepassingen per host/virtueel IP-adres, zoals wordt weergegeven op de volgende schermafbeelding.

![Beveiligingsstatus van toepassingen](./media/security-center-monitoring/security-center-monitoring-fig16-ga.png)

Net als bij de andere aanbevelingen kunt u op een aanbeveling klikken om meer informatie over het probleem en mogelijke oplossingen weer te geven. Het voorbeeld in de onderstaande afbeelding is een toepassing die als onbeveiligde webtoepassing is geïdentificeerd. Wanneer u de toepassing selecteert die als onveilig wordt beschouwd, wordt een andere blade geopend met de volgende optie:

![Meer informatie over een app die is niet beveiligd](./media/security-center-monitoring/security-center-monitoring-fig17-ga.png)

In deze blade vindt u een lijst met alle aanbevelingen voor deze toepassing. Als u klikt op de aanbeveling **Een Web Application Firewall toevoegen**, wordt de blade **Een Web Application Firewall toevoegen** met opties geopend, zodat u een WAF (Web Application Firewall) van een partner kunt installeren, zoals wordt weergegeven op de volgende schermafbeelding.

![Het dialoogvenster Web Application Firewall toevoegen](./media/security-center-monitoring/security-center-monitoring-fig18-ga.png)

## <a name="see-also"></a>Zie ook
In dit artikel hebt u kunnen lezen hoe u de bewakingsmogelijkheden in Azure Security Center gebruikt. Zie de volgende onderwerpen voor meer informatie over Azure Security Center:

* [Beveiligingsbeleid instellen in Azure Security Center](security-center-policies.md): leer hoe u beveiligingsinstellingen configureert in Azure Security Center.
* [Beveiligingswaarschuwingen beheren en erop reageren in Azure Security Center](security-center-managing-and-responding-alerts.md): ontdek hoe u beveiligingswaarschuwingen kunt beheren en erop kunt reageren.
* [Partneroplossingen controleren met Azure Security Center](security-center-partner-solutions.md): leer hoe u de integriteitsstatus van uw partneroplossingen kunt controleren.
* [Azure Security Center FAQ](security-center-faq.md) (Veelgestelde vragen over Azure Security Center): raadpleeg veelgestelde vragen over het gebruik van de service.
* [Azure-beveiligingsblog](http://blogs.msdn.com/b/azuresecurity/): lees blogberichten over de beveiliging en naleving van Azure.

