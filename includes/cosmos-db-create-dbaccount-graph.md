1. Meld u in een nieuw venster aan bij [Azure Portal](https://portal.azure.com/).
2. Klik in het deelvenster links op **Nieuw**, klik op **Databases** en klik vervolgens op **Azure Cosmos DB**.
   
   ![Het deelvenster Databases in Azure Portal](./media/cosmos-db-create-dbaccount-graph/create-nosql-db-databases-json-tutorial-1.png)

3. Geef op de blade **Nieuw account** de gewenste configuratie op voor het Azure Cosmos DB-account. 

    Met Azure Cosmos DB kunt u een van de vier programmeermodellen kiezen: Gremlin (Graph), MongoDB, SQL (DocumentDB) en Tabel (sleutelwaarde).  
       
    In dit artikel programmeren we met de Graph API. Daarom kiest u bij het invullen van het formulier voor **Gremlin (graaf)**. Als u documentgegevens uit een catalogus-app, sleutelwaardegegevens (tabelgegevens) of gegevens die zijn gemigreerd uit een MongoDB-app hebt, moet u er rekening mee houden dat Azure Cosmos DB een zeer beschikbaar, globaal gedistribueerd databaseserviceplatform kan bieden voor alle bedrijfskritische toepassingen.

    Vul de velden op de blade **Nieuw account** in en gebruik de gegevens in de volgende schermopname hierbij als richtlijn. Omdat uw eigen waarden niet overeenkomen met die in de schermopname, kiest u unieke waarden wanneer u het account instelt. 
 
    ![De blade Azure Cosmos DB](./media/cosmos-db-create-dbaccount-graph/create-nosql-db-databases-json-tutorial-2.png)

    Instelling|Voorgestelde waarde|Beschrijving
    ---|---|---
    Id|*Unieke waarde*|Een unieke naam die u kiest om het Azure Cosmos DB-account aan te duiden. Omdat *documents.azure.com* is toegevoegd aan de id die u hebt opgegeven om uw URI te maken, gebruikt u een unieke maar identificeerbare id. De id mag alleen kleine letters, cijfers en het koppelteken (-) bevatten en moet 3 tot 50 tekens lang zijn.
    API|Gremlin (Graph)|Verderop in dit artikel gaan we programmeren met de [Graph API](../articles/cosmos-db/graph-introduction.md).|
    Abonnement|*Uw abonnement*|Het Azure-abonnement dat u wilt gebruiken voor het Azure Cosmos DB-account. 
    Resourcegroep|*Dezelfde waarde als id*|De nieuwe resourcegroepnaam voor het account. Gebruik dezelfde naam als uw id om het uzelf gemakkelijk te maken. 
    Locatie|*De regio het dichtst bij uw gebruikers*|De geografische locatie waar u het Azure Cosmos DB-account gaat hosten. Kies de locatie die zich het dichtst bij uw gebruikers bevindt, zodat ze de snelst mogelijke toegang tot de gegevens hebben.

4. Klik op **Maken** om het account te maken.
5. Klik op de werkbalk op **Meldingen** om het implementatieproces te bewaken.

    ![De melding Implementatie is gestart](./media/cosmos-db-create-dbaccount-graph/azure-documentdb-nosql-notification.png)

6.  Als de implementatie is voltooid, opent u het nieuwe account via de tegel **Alle resources**. 

    ![DocumentDB-account op de tegel Alle resources](./media/cosmos-db-create-dbaccount-graph/azure-documentdb-all-resources.png)