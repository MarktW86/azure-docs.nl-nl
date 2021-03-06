---
title: 'Met een computer verbinding maken met een virtueel Azure-netwerk met behulp van punt-naar-site: klassieke Azure-portal | Microsoft Docs'
description: U maakt een beveiligde verbinding met uw klassieke Azure Virtual Network door een punt-naar-site-VPN-gatewayverbinding te maken met Azure Portal.
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-service-management
ms.assetid: 65e14579-86cf-4d29-a6ac-547ccbd743bd
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/27/2017
ms.author: cherylmc
ms.translationtype: Human Translation
ms.sourcegitcommit: 857267f46f6a2d545fc402ebf3a12f21c62ecd21
ms.openlocfilehash: f048e344026b0fd930569c949b23a42c3c30fffe
ms.contentlocale: nl-nl
ms.lasthandoff: 06/28/2017


---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-the-azure-portal-classic"></a>Een punt-naar-site-verbinding met een VNet configureren met Azure Portal (klassiek)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

In dit artikel wordt beschreven hoe u een VNet met een punt-naar-site-verbinding maakt in het klassieke implementatiemodel met behulp van Azure Portal. U kunt deze configuratie ook maken met een ander implementatiehulpprogramma of een ander implementatiemodel door in de volgende lijst een andere optie te selecteren:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)
> * [Azure Portal (klassiek)](vpn-gateway-howto-point-to-site-classic-azure-portal.md)
>

Met een punt-naar-site-configuratie (P2S) kunt u een beveiligde verbinding maken tussen een afzonderlijke clientcomputer en een virtueel netwerk. Punt-naar-site-verbindingen zijn handig als u verbinding wilt maken met uw VNet vanaf een externe locatie, zoals vanaf thuis of een conferentie, of wanneer u slechts enkele clients hebt die verbinding moeten maken met een virtueel netwerk. De P2S-VPN-verbinding wordt door de clientcomputer met de systeemeigen Windows VPN-client gestart. Clienten verbinden met certificaten om te verifiëren. 


![Punt-naar-site-diagram](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/point-to-site-connection-diagram.png)

Punt-naar-site-verbindingen hebben geen VPN-apparaat of openbaar IP-adres nodig. P2S maakt de VPN-verbinding via SSTP (Secure Socket Tunneling Protocol). Wij ondersteunen SSTP versies 1.0, 1.1 en 1.2 aan de serverzijde. De client besluit welke versie moet worden gebruikt. Voor Windows 8.1 en hoger, gebruikt SSTP standaard 1.2. Voor meer informatie over punt-naar-site-verbindingen leest u de [Veelgestelde vragen over punt-naar-site](#faq) onder aan dit artikel.

Voor P2S-verbindingen is het volgende vereist:

* Een dynamische VPN-gateway.
* De openbare sleutel (CER-bestand) voor een basiscertificaat, geüpload naar Azure. Dit wordt beschouwd als een vertrouwd certificaat en wordt gebruikt voor verificatie.
* Op basis van het basiscertificaat wordt een clientcertificaat gegenereerd voor installatie op elke clientcomputer die wordt verbonden. Dit certificaat wordt gebruikt voor clientverificatie.
* Op elke clientcomputer die wordt verbonden, moet bovendien een VPN-clientconfiguratiepakket worden gegenereerd en geïnstalleerd. Het clientconfiguratiepakket configureert de systeemeigen VPN-client die zich al op het systeem bevindt, met de benodigde informatie om verbinding te maken met het VNet.

### <a name="example-settings"></a>Voorbeeldinstellingen

U kunt de volgende waarden gebruiken om een testomgeving te maken of ze raadplegen om meer inzicht te krijgen in de voorbeelden in dit artikel:

* **Naam: VNet1**
* **Adresruimte: 192.168.0.0/16**<br>In dit voorbeeld gebruiken we slechts één adresruimte. U kunt meer dan één adresruimte voor uw VNet hebben.
* **Subnetnaam: FrontEnd**
* **Subnetadresbereik: 192.168.1.0/24**
* **Abonnement:** controleer of u het juiste abonnement hebt in het geval u er meer dan één hebt.
* **Resourcegroep: TestRG**
* **Locatie: VS - oost**
* **Verbindingstype: punt-naar-site-verbinding**
* **Clientadresruimte: 172.16.201.0/24**. VPN-clients die verbinding maken met het VNet via deze punt-naar-site-verbinding, ontvangen een IP-adres van de opgegeven pool.
* **GatewaySubnet: 192.168.200.0/24**. Het gatewaysubnet moet de naam GatewaySubnet gebruiken.
* **Grootte:** selecteer de gateway-SKU die u wilt gebruiken.
* **Routeringstype: dynamisch**

## <a name="vnetvpn"></a>Sectie 1: Een virtueel netwerk en een VPN-gateway maken

Controleer eerst of u een Azure-abonnement hebt. Als u nog geen Azure-abonnement hebt, kunt u [uw voordelen als MSDN-abonnee activeren](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) of [u aanmelden voor een gratis account](https://azure.microsoft.com/pricing/free-trial).

### <a name="createvnet"></a>Deel 1: Een virtueel netwerk maken

Als u nog geen virtueel netwerk hebt, maakt u er een. De schermafbeeldingen dienen alleen als voorbeeld. Zorg dat u de waarden vervangt door die van uzelf. Volg de volgende stappen om een VNet te maken met behulp van Azure Portal:

1. Navigeer via een browser naar de [Azure Portal](http://portal.azure.com) en log, indien nodig, in met uw Azure-account.
2. Klik op **Nieuw**. In het veld **Marketplace doorzoeken** typt u 'virtueel netwerk'. Zoek **virtueel netwerk** in de geretourneerde lijst en klik hierop om de resourceblade **Virtueel netwerk** te openen.

  ![Een virtuele netwerkblade zoeken](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/newvnetportal700.png)
3. Kies uit de lijst **Selecteer een implementatiemodel** (aan de onderzijde van de blade Virtueel netwerk) de optie **Klassiek** en klik vervolgens op **Maken**.

  ![Een implementatiemodel selecteren](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/selectmodel.png)
4. Configureer de VNet-instellingen op de blade **virtueel netwerk aanmaken**. Voeg uw eerste adresruimte en één adresbereik voor het subnet toe aan deze blade. Nadat u het VNet hebt aangemaakt, kunt u terugkeren en aanvullende subnets en adresruimten toevoegen.

  ![Een virtuele netwerkblade aanmaken](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/vnet125.png)
5. Controleer of het **Abonnement** het juiste is. U kunt abonnementen wijzigen met behulp van de vervolgkeuzelijst.
6. Klik op **Resourcegroep** en selecteer een bestaande resourcegroep of maak een nieuwe aan door een naam in te voeren voor uw nieuwe resourcegroep. Geef de resourcegroep een naam die aansluit bij de geplande configuratiewaarden wanneer u een nieuwe resourcegroep aanmaakt. Zie voor meer informatie over resourcegroepen [Overzicht van Azure Resource Manager](../azure-resource-manager/resource-group-overview.md#resource-groups).
7. Selecteer vervolgens de **Locatie**-instellingen voor uw VNet. De locatie bepaalt waar de resources die u naar dit VNet implementeert, worden opgeslagen.
8. Selecteer **Vastmaken aan dashboard** als u uw VNet gemakkelijk wilt terugvinden op het dashboard en klik vervolgens op **Aanmaken**.

  ![Vastmaken aan dashboard](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/pintodashboard150.png)
9. Nadat u op Aanmaken hebt geklikt, verschijnt er een tegel op uw dashboard die de voortgang van uw VNet aangeeft. De tegel wordt gewijzigd wanneer het VNet wordt gemaakt.

  ![Tegel Virtueel netwerk maken](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/deploying150.png)
10. Wanneer het virtuele netwerk is gemaakt, wordt op de pagina met netwerken in de klassieke Azure-portal **Gemaakt** vermeld onder **Status**.
11. Voeg een DNS-server toe (optioneel). Nadat u uw virtuele netwerk hebt gemaakt, kunt u het IP-adres van een DNS-server voor naamomzetting toevoegen. De DNS-server die u opgeeft moet een DNS-server zijn die de namen voor de resources in uw VNet kan omzetten.<br>Open de instellingen voor het virtuele netwerk, klik op de DNS-servers en voeg het IP-adres toe van de DNS-server die u wilt gebruiken om een DNS-server toe te voegen. Het clientconfiguratiepakket dat u in een latere fase maakt, bevat de IP-adressen van de DNS-servers die u in deze instelling opgeeft. Als u de lijst met DNS-servers in de toekomst moet bijwerken, kunt u nieuwe VPN-clientconfiguratiepakketten genereren die overeenkomen met de bijgewerkte lijst.

### <a name="gateway"></a>Deel 2: Een gatewaysubnet en een gateway voor dynamische routering maken

In deze stap maakt u een gatewaysubnet en een gateway voor dynamische routering. Als u in Azure Portal het gatewaysubnet en de gateway maakt voor het klassieke implementatiemodel, kunt u dit doen via dezelfde configuratieblades.

1. Navigeer in de portal naar het virtuele netwerk waarvoor u een gateway wil maken.
2. Ga op de blade voor het virtuele netwerk naar de blade **Overzicht** en klik in de sectie VPN-verbindingen op **Gateway**.

  ![Klik om een gateway te maken](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/beforegw125.png)
3. Selecteer op de blade **Nieuwe VPN-verbinding** de optie **Punt-naar-site**.

  ![Verbindingstype punt-naar-site](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/newvpnconnect.png)
4. Voeg voor **Clientadresruimte** het IP-adresbereik toe. Dit is het bereik van waaruit de VPN-clients een IP-adres ontvangen wanneer er verbinding wordt gemaakt. Gebruik een privé-IP-adresbereik dat niet overlapt met de on-premises locatie waarvanaf u verbinding maakt of met het VNet waarmee u verbinding wilt maken. U kunt het automatisch ingevulde bereik verwijderen en daarna het privé-IP-adresbereik toevoegen dat u wilt gebruiken.

  ![Clientadresruimte](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clientaddress.png)
5. Selecteer het selectievakje **Gateway onmiddellijk maken**.

  ![Gateway onmiddellijk maken](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/creategwimm.png)
6. Klik op **Optionele gatewayconfiguratie** om de blade **Gatewayconfiguratie** te openen.

  ![Klik op Optionele gatewayconfiguratie](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/optsubnet125.png)
7. Klik op **Vereiste subnetinstellingen configureren** om het **Gatewaysubnet** toe te voegen. Het is mogelijk om een klein gatewaysubnet van /29 te maken, maar we raden u aan een groter subnet met meer adressen te maken door ten minste /28 of /27 te selecteren. Hierdoor hebt u genoeg adressen voor mogelijke aanvullende toekomstige configuraties. Als u met gatewaysubnetten werkt, vermijd dan om een netwerkbeveiligingsgroep (NSG) te koppelen aan het gatewaysubnet. Een koppeling van een netwerkbeveiligingsgroep naar dit subnet zorgt er mogelijk voor dat uw VPN-gateway niet meer werkt zoals verwacht.

  ![GatewaySubnet toevoegen](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/gwsubnet125.png)
8. Selecteer de gateway **Grootte**. De grootte is de gateway-SKU voor de gateway van uw virtuele netwerk. In de portal is de standaard-SKU **Basic**. Zie [Informatie over VPN-gatewayinstellingen](vpn-gateway-about-vpn-gateway-settings.md#gwsku) voor informatie over gateway-SKU's.

  ![Grootte van de gateway](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/gwsize125.png)
9. Selecteer het **Routeringstype** voor uw gateway. P2S-configuraties vereisen een **Dynamisch** routeringstype. Klik op **OK** wanneer u klaar bent met het configureren van deze blade.

  ![Routeringstype configureren](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/routingtype125.png)
10. Klik op de blade **Nieuwe VPN-verbinding** op **OK** (onder aan de blade) om te beginnen met het maken van uw virtuele netwerkgateway. Een VPN-gateway wordt binnen maximaal 45 minuten voltooid. De daadwerkelijke instelduur hangt af van de gateway-SKU die u selecteert.

## <a name="generatecerts"></a>Sectie 2 - Certificaten maken

Certificaten worden door Azure gebruikt om VPN-clients voor punt-naar-site-VPN's te verifiëren. U uploadt de informatie van de openbare sleutel over het basiscertificaat naar Azure. De openbare sleutel wordt dan beschouwd als 'vertrouwd'. Clientcertificaten moeten worden gegenereerd op basis van het vertrouwde basiscertificaat en geïnstalleerd op elke clientcomputer. Dit gebeurt in het certificaatarchief Certificates-Current user/Personal. Het certificaat wordt gebruikt om de client te verifiëren bij het maken van verbinding met het VNet. 

Als u zelfondertekende certificaten gebruikt, moeten ze worden gemaakt met behulp van specifieke parameters. U kunt een zelfondertekend certificaat maken met behulp van de instructies voor [PowerShell en Windows 10](vpn-gateway-certificates-point-to-site.md) of [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Het is belangrijk dat u de stappen volgt in deze instructies bij het werken met zelfondertekende basiscertificaten en het genereren van clientcertificaten uit het zelfondertekende basiscertificaat. Anders zijn de certificaten die u maakt niet compatibel met P2S-verbindingen en zal er een verbindingsfout optreden.

### <a name="cer"></a>Deel 1: De openbare sleutel (.cer) voor het basiscertificaat verkrijgen

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-rootcert-include.md)]

### <a name="genclientcert"></a>Deel 2: Een clientcertificaat genereren

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="upload"></a>Sectie 3: Het CER-bestand van het basiscertificaat uploaden

Wanneer de gateway is gemaakt, uploadt u het CER-bestand (dat de informatie over de openbare sleutel bevat) voor een vertrouwd basiscertificaat naar Azure. U uploadt de persoonlijke sleutel voor het basiscertificaat niet naar Azure. Nadat het CER-bestand is geüpload, kan Azure daarmee clients met een geïnstalleerd clientcertificaat (gemaakt op basis van het vertrouwde basiscertificaat) verifiëren. Indien nodig kunt u later aanvullende vertrouwde basiscertificaatbestanden uploaden (maximaal 20).  

1. Klik in het gedeelte **VPN-verbindingen** van de blade voor uw VNet op de afbeelding **clients** om de blade **Punt-naar-site VPN-verbinding** te openen.

  ![Clients](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clients125.png)
2. Klik op de blade **Punt-naar-site-verbinding** op **Certificaten beheren** om de blade **Certificaten** te openen.<br>

  ![Blade Certificaten](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/ptsmanage.png)<br><br>
3. Klik op de blade **Certificaten** op **Uploaden** om de blade **Certificaat uploaden** te openen.<br>

    ![Blade Certificaten uploaden](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/uploadcerts.png)<br>
4. Klik op de afbeelding van een map om het .cer-bestand te zoeken. Selecteer het bestand en klik vervolgens op **OK**. Vernieuw de pagina om het geüploade certificaat weer te geven op de blade **Certificaten**.

  ![Certificaat uploaden](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/upload.png)<br>

## <a name="vpnclientconfig"></a>Sectie 4: De client configureren

Als u verbinding wilt maken met een VNet met behulp van een punt-naar-site-VPN, moet op elke client een pakket worden geïnstalleerd om de systeemeigen Windows VPN-client te configureren. Het configuratiepakket configureert de systeemeigen Windows VPN-client met de instellingen die nodig zijn om verbinding te maken met het virtuele netwerk, en als u een DNS-server voor uw VNet hebt opgegeven, bevat het pakket het IP-adres van de DNS-server die de client gaat gebruiken voor naamomzetting. Als u de opgegeven DNS-server later wijzigt, na het genereren van het clientconfiguratiepakket, moet u een nieuw clientconfiguratiepakket genereren om op uw clientcomputers te installeren.

U kunt hetzelfde configuratiepakket voor de VPN-client gebruiken op elke clientcomputer, mits de versie overeenkomt met de architectuur van de client. Zie voor de lijst met ondersteunde clientbesturingssystemen de [Veelgestelde vragen over punt-naar-site-verbindingen](#faq) onderaan dit artikel.

### <a name="part-1-generate-and-install-the-vpn-client-configuration-package"></a>Deel 1: Het configuratiepakket voor de VPN-client downloaden en installeren

1. Klik in Azure Portal op de blade **Overzicht** voor uw VNet in **VPN-verbindingen** op de clientafbeelding om de blade **Punt-naar-site VPN-verbinding** te openen.
2. Klik boven aan de blade **Punt-naar-site VPN-verbinding** op het downloadpakket dat overeenkomt met het besturingssysteem van de client waarop het wordt geïnstalleerd:

  * Selecteer voor 64-bits clients **VPN-clients (64-bits)**.
  * Selecteer voor 32-bits clients **VPN-clients (32-bits)**.

  ![Het configuratiepakket voor de VPN-client downloaden](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/dlclient.png)<br>
3. Nadat het pakket is gegenereerd, downloadt en installeert u dit op de clientcomputer. Als u een SmartScreen-melding ziet, klikt u op **Meer info** en vervolgens op **Toch uitvoeren**. U kunt het pakket ook opslaan en op andere clientcomputers installeren.

### <a name="part-2-install-the-client-certificate"></a>Deel 2: Het clientcertificaat installeren

Als u een P2S-verbinding wilt maken vanaf een andere clientcomputer dan de computer die u gebruikt om de clientcertificaten te genereren, moet u een clientcertificaat installeren. Wanneer u een clientcertificaat installeert, hebt u het wachtwoord nodig dat is gemaakt tijdens het exporteren van het clientcertificaat. Dit is meestal alleen een kwestie van dubbelklikken op het certificaat en het installeren. Zie [Een geëxporteerd clientcertificaat installeren](vpn-gateway-certificates-point-to-site.md#install) voor meer informatie.

## <a name="connect"></a>Sectie 5: Verbinding maken met Azure

### <a name="connect-to-your-vnet"></a>Verbinding maken met uw VNet

1. Als u met uw VNet wilt verbinden, gaat u op de clientcomputer naar de VPN-verbindingen en zoekt u de VPN-verbinding die u hebt gemaakt. Deze heeft dezelfde naam als het virtuele netwerk. Klik op **Verbinden**. Er verschijnt mogelijk een pop-upbericht dat verwijst naar het certificaat. Klik in dat geval op **Doorgaan** om verhoogde bevoegdheden te gebruiken.
2. Klik op de pagina **Verbindingsstatus** op **Verbinden** om de verbinding te starten. Als het scherm **Certificaat selecteren** wordt geopend, controleert u of het weergegeven clientcertificaat het certificaat is dat u voor de verbinding wilt gebruiken. Als dat niet het geval is, gebruikt u de pijl-omlaag om het juiste certificaat te selecteren en klikt u op **OK**.

  ![VPN-clientverbinding](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clientconnect.png)
3. De verbinding is tot stand gebracht.

  ![Tot stand gebrachte verbinding](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/connected.png)

Als u problemen hebt met het maken van een verbinding, controleert u het volgende:

- Open **Gebruikerscertificaten beheren** en navigeer naar **Vertrouwde basiscertificeringsinstanties\Certificaten**. Controleer of het basiscertificaat wordt weergegeven. Verificatie werkt alleen als het basiscertificaat aanwezig is. Wanneer u een PFX-bestand van het clientcertificaat exporteert met de standaardwaarde 'Indien mogelijk alle certificaten in het certificeringspad opnemen', wordt ook de basiscertificaatinformatie geëxporteerd. Wanneer u het clientcertificaat installeert, wordt vervolgens ook het basiscertificaat geïnstalleerd op de clientcomputer. 

- Als u een certificaat gebruikt dat is verleend met behulp van een oplossing voor een CA voor ondernemingen en problemen ondervindt bij de verificatie, controleert u de verificatievolgorde op het clientcertificaat. U kunt de volgorde van de verificatielijst controleren door op het clientcertificaat te dubbelklikken en naar **Details > Uitgebreid sleutelgebruik** te gaan. Controleer of Clientverificatie als het eerste item in de lijst wordt weergegeven. Als dat niet het geval is, moet u een clientcertificaat op basis van de gebruikerssjabloon verlenen waarin clientverificatie als het eerste item in de lijst wordt weergegeven. 

### <a name="verify-the-vpn-connection"></a>De VPN-verbinding controleren

1. Als u wilt controleren of uw VPN-verbinding actief is, opent u een opdrachtprompt met verhoogde bevoegdheid en voert u *ipconfig/all* in.
2. Bekijk de resultaten. Het IP-adres dat u hebt ontvangen, is een van de adressen binnen het adresbereik van de punt-naar-site-verbinding dat u hebt opgegeven tijdens het maken van het VNet. De resultaten moeten er ongeveer als volgt uitzien:

Voorbeeld:

    PPP adapter VNet1:
        Connection-specific DNS Suffix .:
        Description.....................: VNet1
        Physical Address................:
        DHCP Enabled....................: No
        Autoconfiguration Enabled.......: Yes
        IPv4 Address....................: 192.168.130.2(Preferred)
        Subnet Mask.....................: 255.255.255.255
        Default Gateway.................:
        NetBIOS over Tcpip..............: Enabled

 
 Als u geen verbinding met een virtuele machine kunt maken via P2S, gebruikt u 'ipconfig' om het IPv4-adres te controleren dat is toegewezen aan de ethernetadapter op de computer waarvanaf u de verbinding maakt. Als het IP-adres zich binnen het adresbereik bevindt van het VNet waarmee u verbinding maakt of binnen het adresbereik van uw VPNClientAddressPool, wordt dit een overlappende adresruimte genoemd. Als uw adresruimte op deze manier overlapt, kan het netwerkverkeer Azure niet bereiken en blijft het in het lokale netwerk. Als uw netwerkadresruimte niet overlapt en u nog steeds geen verbinding kunt maken met de virtuele machine, raadpleegt u [Troubleshoot Remote Desktop connections to a VM](../virtual-machines/windows/troubleshoot-rdp-connection.md) (Problemen met extern-bureaubladverbindingen met virtuele machines oplossen).

## <a name="connectVM"></a>Verbinding maken met een virtuele machine

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-classic-include.md)]

## <a name="add"></a>Vertrouwde basiscertificaten toevoegen of verwijderen

U kunt vertrouwde basiscertificaat toevoegen in en verwijderen uit Azure. Wanneer u een basiscertificaat verwijdert, kunnen clients met een certificaat dat is gegenereerd op basis van dat basiscertificaat niet worden geverifieerd, en kunnen ze dus geen verbinding maken. Als u wilt dat clients kunnen worden geverifieerd en verbinding kunnen maken, moet u een nieuw clientcertificaat installeren dat is gegenereerd op basis van een basiscertificaat dat wordt vertrouwd (is geüpload) in Azure.

### <a name="to-add-a-trusted-root-certificate"></a>Een vertrouwd basiscertificaat toevoegen

U kunt maximaal 20 vertrouwde .cer-basiscertificaatbestanden toevoegen aan Azure. Zie [Sectie 3: Het CER-bestand van het basiscertificaat uploaden](#upload) voor instructies.

### <a name="to-remove-a-trusted-root-certificate"></a>Een vertrouwd basiscertificaat verwijderen

1. Klik in het gedeelte **VPN-verbindingen** van de blade voor uw VNet op de afbeelding **clients** om de blade **Punt-naar-site VPN-verbinding** te openen.

  ![Clients](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/clients125.png)
2. Klik op de blade **Punt-naar-site-verbinding** op **Certificaten beheren** om de blade **Certificaten** te openen.<br>

  ![Blade Certificaten](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/ptsmanage.png)<br><br>
3. Op de blade **Certificaten** klikt u op de knop met weglatingstekens naast het certificaat dat u wilt verwijderen. Klik vervolgens op **Verwijderen**.

  ![Basiscertificaat verwijderen](./media/vpn-gateway-howto-point-to-site-classic-azure-portal/deleteroot.png)<br>

## <a name="revokeclient"></a>Een clientcertificaat intrekken

U kunt clientcertificaten intrekken. Met de certificaatintrekkingslijst kunt u selectief punt-naar-site-verbinding weigeren op basis van afzonderlijke clientcertificaten. Dit wijkt af van het verwijderen van een vertrouwd basiscertificaat. Als u een vertrouwd .cer-basiscertificaat uit Azure verwijdert, wordt de toegang ingetrokken voor alle clientcertificaten die zijn gegenereerd/ondertekend door het ingetrokken basiscertificaat. Als u in plaats van een basiscertificaat een clientcertificaat intrekt, kunt u de certificaten die zijn gegenereerd op basis van het basiscertificaat, blijven gebruiken voor verificatie van de punt-naar-site-verbinding.

De algemene procedure is het basiscertificaat te gebruiken om de toegang te beheren op het team- of organisatieniveau, terwijl u ingetrokken clientcertificaten gebruikt voor nauwkeuriger toegangsbeheer bij afzonderlijke gebruikers.

### <a name="to-revoke-a-client-certificate"></a>Een clientcertificaat intrekken

U kunt een clientcertificaat intrekken door de vingerafdruk toe te voegen aan de intrekkingslijst.

1. Haal de vingerafdruk voor het clientcertificaat op. Zie voor meer informatie [De vingerafdruk van een certificaat ophalen](https://msdn.microsoft.com/library/ms734695.aspx).
2. Kopieer de gegevens naar een teksteditor en verwijder alle spaties, zodat u een doorlopende tekenreeks overhoudt.
3. Navigeer naar de blade **'klassieke virtuele-netwerknaam' > Punt-naar-site VPN-verbinding > Certificaten** en klik vervolgens op **Intrekkingslijst** om de blade Intrekkingslijst te openen. 
4. Op de blade **Intrekkingslijst** klikt u op **+Certificaat toevoegen** om de blade **Certificaat toevoegen aan intrekkingslijst** te openen.
5. Op de blade **Certificaat toevoegen aan intrekkingslijst** plakt u de vingerafdruk van het certificaat als een doorlopende regel tekst, zonder spaties. Klik op **OK** onder aan de blade.
6. Wanneer het bijwerken is voltooid, kan het certificaat niet langer worden gebruikt om verbinding te maken. Clients die verbinding proberen te maken met het certificaat, ontvangen een bericht waarin wordt gemeld dat het certificaat niet meer geldig is.

## <a name="faq"></a>Veelgestelde vragen over punt-naar-site

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-point-to-site-faq-include.md)]

## <a name="next-steps"></a>Volgende stappen
Wanneer de verbinding is voltooid, kunt u virtuele machines aan uw virtuele netwerken toevoegen. Zie [Virtuele machines](https://docs.microsoft.com/azure/#pivot=services&panel=Compute) voor meer informatie. Zie [Azure and Linux VM Network Overview](../virtual-machines/linux/azure-vm-network-overview.md) (Overzicht van Azure- en Linux-VM-netwerken) voor meer informatie over netwerken en virtuele machines.

