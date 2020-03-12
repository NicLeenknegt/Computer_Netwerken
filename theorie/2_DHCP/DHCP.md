# 2 DHCP

Een IP-adress en de default gateway worden vaak ingesteld op het moment van installatie.
Dit is niet mogelijk bij apparaten die vaak van netwerk verwisselen en apparaten die niet veel met het internet verbinden.

Hiervoor werd **DHCP** (Dynamic Host Configuration protocol) uitgevonden.
Een manier om TCP/IP-configuratie te automatiseren en te centraliseren.
Computers ontvangen hun configuratie bij opstarten.
Naast **DHCP** bestaat er ook **RARP** (Reverse Adress Resolution Protocol) en **BOOTP** (Bootstrap Protocol).

**BOOTP**:

* Heeft alle MAC-adressen nodig van de verbonden ethernetkaarten.
* Kan geen tijdelijke IP's uitdelen.
* Minder flexibel als **DHCP**.

**DHCP**:

* Client-server model.
* Houd een lijst van configuratie bij voor specifieke geregistreerde apparaten.
    * Vaak apparaten met een vaste identificatie naar de buitenwereld: WWW-servers, mail-servers,...
* Deelt dynamisch adressen uit aan niet-geristreerde apparaten.
    * Haalt deze adressen uit de **DHCP-scope** een reeks adressen die vrij worden uitgedeeld.
    * Id van dynamisch gealloceerd apparaat wijzigt van sessie tot sessie.

**DHCP-lease**: levensduur van dynamisch gealloceerde IP-adressen. Word om de zoveel tijd vernieuwd. Er kunnen meerder DHCP adressen actief zijn dan dat er verkrijgbaar zijn, zolang alle apperaten niet tegelijk actief zijn.

**DHCP voordelen**:

* Minder complexiteit
* Vermijd configuratie fouten
* Automatische configuratie
* Vermakkelijkt verwisselen van subnetwerk
* Besturingsysteem onafhankelijk

## 2.1.1 opties

### DHCP header:

Dit is het begin van de DHCP-wagon.

* Exact 300 bytes lang.
* Identiek aan BOOTP header.
* bevat configuraties van server naar client.

<table>
    <tbody>
        <tr>
            <td>Opcode</td>
            <td>Hardware type</td>
            <td>Hardware adress length</td>
            <td>Hop count</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Transaction ID </td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Client IP-adress</td>
        </tr>
         <tr>
            <td colspan="4" align="center" >"your" IP-adress</td>
        </tr>
         <tr>
            <td colspan="4" align="center" >server IP-adress</td>
        </tr>
         <tr>
            <td colspan="4" align="center" >Gateway IP-adress</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Client Hardware adress</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Server host name</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Boot file name</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Options</td>
        </tr>
    </tbody>
</table>

**"your" ip-adress**: adres dat de DHCP server uitleent aan de client.

**transaction ID**: Maakt duidelijk aan de client op welke vraag de server heeft geantwoord.

**Options**:

| BOOTP header | 99.130.83.99 | id | length | value |
| --- | ------- | --- | --- | --- |

* 99.130.83.99 is de magic cookie, identificeert formaat van DHCP-bericht
* alles na optie 255 word genegeerd
* 80 verschillende opties
    * Een windows client kan alleen optie 1, 3, 6, 15, 31, 33, 51, 53, 54, 58 aan.
    * Op windows moet de naam statisch worden geconfigureerd. Er bestaat hiervoor geen optie.

## 2.1.2 Het lease-proces

2 verschillende lease-processen voor clients: initialiseren en vernieuwen.

8 verschillen berichttypes, worden verstuurt met optie 53 in de header:

1. DHCP-discovery
2. DHCP-offer
3. DHCP-request
4. DHCP-decline
5. positief DHCP-acknowledgement
6. negatief DHCP-acknowledgement
7. DHCP-release
8. DHCP-inform

UDP-poort 67 is de destination port voor de server en 68 voor de client.

### Intisialisatie

| | server | | | client|
|---| --- | --- | --- | --- |
|1| | <= | DHCP-DISCOVERY | |
|2| | DHCP-OFFER | => | |
|3| | <= | DHCP-REQUEST | |
|4| | DHCP-ACK | => | |
|5| | <= | DHCP-DECLINE | |
|6| | <= | DHCP-INFORM | |
|7| | <= | DHCP-RELEASE | |

1. DHCP-DISCOVERY:
    * Broadcast over lokaal subnet
    * Optie 51: Gewenste lease-tijd (Niet altijd geaccepteert door server)
    * optie 55: reeks van geordende opties
    * Client-adress: 0.0.0.0
2. DHCP-OFFER:
    * Geeft de aangevraagde opties in dezelfde volgorde van de client.
    * Server reserveert het adres voorlopig.
    * clients krijgen vaak hetzelfde adres over meerdere sessies.
    * Optie 54: serveridentificatie, zodat clients het verschil zien tussem meerdere offers.
    * ZOnder offer kan e client niet initialiseren. In dit geval:
        * Stuurt de client om 2+, 4+, 8+ en 16+ seconden een nieuw bericht. de + is een willekeuring getal tussen 0 en 1.
        * Na deze 5 pogingen stuurt de client een bericht om de 5 minuten.
3. DHCP-REQUEST:
    * Client selecteert 1 van de offers
    * Optie 54: identificatie van gekozen server.
    * Er is geen systeem voor het kiezen van een server
    * Vaak kiest de client de eerste server die aan alle aanbiedingen voldoet.
4. DHCP-ACKNOWLEDGEMENT:
    * Broadcast zodat niet geselecteerde servers weten dat ze hun gereserveerde adressen mogen loslaten
    * positief:
        * Confirmatie van lease
        * DHCP-opties
    * negatief:
        * als een client een ongeldig of reeds toegekend adres vraagt.
        * de client komt uit een subnet die buiten de **DHCP-scope** valt.
        * De client die dit bericht moet het gehele proces opnieuw starten.
5. DHCP-DECLINE:
    * Als de client fouten in de configuratie merkt
    * Client begint weer van stap 1
6. DHCP-Inform:
    * De client vraagt on aanvullende informatie.
7. DHCP-release:
    * Als de lease niet meer nodig is
    * Bijvoorbeeld bij een shutdown

### vernieuwing

* T1: 1/2 originele Lease tijd, tenzij optie 58 is ingesteld in de server
* T2: 7/8 lease tijd, tenzij optie 59 is ingesteld in de server

| Time | | server | | | client|
| --- |---| --- | --- | --- | --- |
| T1 |1| | <= | DHCP-REQUEST | |
| T1 |2| | DHCP-OFFER | => | |
| T2 |3| | <= | DHCP-REQUEST | |
| T2 |4| | DHCP-ACK | => | |
||5| | | INITIALISATION | |

1. DHCP-REQUEST:
    * rechtstreeks naar de server
    * aanvraag om adreslease te verlengen
2. DHCP-OFFER:
    * verlengd lease
    * er worden ook DHCP-opties doorgestuurd, als in de tussentijd opties zijn veranderd.
    * Indien de client geen ACK krijgt stuurt hij om 2+, 4+, 8+ en 16+ seconden een nieuwe REQ.
3. DHCP-REQUEST:
    * Als de client geen communicatie kan vastleggen met de server wacht die tot T2.
    * De client zit nu in REBINDING status.
    * Dan probeer de client de huidige lease te vernieuwen bij een andere willekeurige server.
4. DHCP-ACKNOWLEDGEMENT:
    * Als er een server reageert kan de client weer verder gaan
    * anders wacht de client 2+, 4+, 8+ en 16+ seconden
5. INITIALISATION:
    * De lease is verlopen en de client heeft gee server bereiikt
    * Een server heeft gereageerd met een negatieve ACK.
    * De client beeindigt zijn adreslease, TCP/IP word beeindigt.
    * De client herbegint aan intialisation

## 2.1.3 DHCP relay agents

Routers sturen geen broadcasts, dus ook geen **DHCP** Discovery berichten ( of andere **DHCP** berichten). Leden van een subnet kunnen dus hun **DHCP** niet doorsturen naar een centrale **DHCP**-server op een ander netwerk. Dus om een netwerk correct te laten functioneren zou elk subnet een aparte **DHCP**-server nodig hebben.

=> Dit is onpraktisch. Als oplossing werd er **DHCP-relay** software uitgevonden.

### DHCP-relay

**Praktisch**:

* Het is een klein programma
* Stuurt DHCP-berichten van het ene naar het andere subnet
* Werkt voor zowel linux als windows
* Als een router niet als **relay-agent** kan werken kan een computer in het subnet dienen als **relay-agent**. Een subnet zonder **relay-agent** heeft een **DHCP**-server nodig

**BOOTP**:

* Meeste hardware routers met recente software en firmware ondersteunen ook de functionaliteit een van **BOOTP relay-agent**
* DHCP berichten word via dezelfde UDP poorten (Client: 68, Server: 67) als BOOTP berichten verstuurt
* DHCP en BOOTP hebben dezelfde berichtstructuur

=> Een BOOTP relay-agent kan ook DHCP berichten ontvangen en versturen

**Werking**:

* **BOOTP header**:
    * **Hop count**: Word verhoogd met 1 als een dhcp-relay het bericht opnieuw broadcast. Windows servers hebben hieropp een max van 4. Dit veld dient vooral om het aantal intensieve broadcast berichten laag te houden.
    * **Gateway IP-adress**: is 0.0.0.0 als het pakket/wagonnetje door een client word verstuurd. Een DHCP-relay die dit bericht ontvangt en doorstuurt wijzigt dit veld naar zijn eigen adres.

<table>
    <tbody>
        <tr>
            <td>Opcode</td>
            <td>Hardware type</td>
            <td>Hardware adress length</td>
            <td style="text-decoration: underline" >Hop count</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Transaction ID </td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Client IP-adress</td>
        </tr>
         <tr>
            <td colspan="4" align="center" >"your" IP-adress</td>
        </tr>
         <tr>
            <td colspan="4" align="center" >server IP-adress</td>
        </tr>
         <tr>
            <td colspan="4" align="center" style="text-decoration: underline"  >Gateway IP-adress</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Client Hardware adress</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Server host name</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Boot file name</td>
        </tr>
        <tr>
            <td colspan="4" align="center" >Options</td>
        </tr>
    </tbody>
</table>

* **DHCP relay-agent**:

    * =*>: broadcast
    * =>: point to point

    1. Client stuurt broadcast over subnet.
        * Er zit geen DHCP-server op dit subnet
    2. Relay agent ontvangt een broadcast op 1 van zijn interfaces.
        * Het gateway adress in het bericht word omgezet naar de ip van de relay agent
        * Als de relay agent via een andere interface met een dhcp server is verbonden zal die via point to point het bericht doorsturen naar de server.
        * Als de interfaces van de relay agent niet rechtstreeks met een dhcp server is verbonden stuurt die het client bericht via broadcast door over de andere interfaces van de relay agent.
            * De hop count in het bericht word verhoogd met 1.
    3. Uiteindelijk bereikt het bericht de DHCP-server.
        * Hij controleert het Gateway-IP adress, deze toont tot welke scope de client toebehoord.
        * Als de server geen adressen heeft voor deze scope mag hij hierop niet reageren. Ook al heeft hij vrije adressen in andere scopes.
        * De server stuurt een respons rechtstreeks via point to point naar de relay agent.
            * Dit is het voordeel van hop & gateway, De server moet geen broadcast uitsturen.
    4. Uiteindelijk ontvangt de relay agent het bericht van de server.
        * Dit bericht word over het subnet van de client gebroadcast
        * De relay agent weet niet exact voor welke client het bericht dient.

| | client | | relay |  | server|
|---| --- | --- | --- | --- |--- |
| 1| DHCP-DISCOVERY | =*> |   | | |
| 2| |  |  DHCP-DISCOVERY  | =>  | |
| 3| |  |  | <=  | DHCP-offer |
| 4| |  <*= | DHCP-offer |  | |

**Weken met meerdere DHCP-servers**:

DHCP servers kunnnen niet onderling met elkander communiceren.

* **Probleem**: Een centrale DHCP-server is een single point of failure.
    * **Oplossing**: DHCP-servers worden in verschillende subnets geplaatst
        * Deze servers mogen niet dezelfde scopes bevatten want DHCP servers kunnen niet met elkander communiceren over welke adressen ze al hebben geleased, dit kan tot duplicatie leiden.
        * elke server hun scope is verantwoordelijk voor 1 of meerdere subnet.
* **Probleem**: Als 1 server wegvalt dan zijn de adressen voor een reeks subnetten niet bereikbaar.
    * **oplossing**: Een DHCP-server heeft 80% van het adreisbereik van zijn eigen subnetten, de overige 20% zit in andere DHCP-servers.
        * Als een server uitvalt zijn er dus nog alternatieven.
* **probleem**: servers ontvangen ongezeste hoeveelheden dhcp berichten veroorzaakt door willekeurig geselecteerde paden
    * **oplossing**: vertragingsintervallen in relay agents.

**Configuratie van relay agents**:

Zeer eenvoudig.

Linux:

```bash
dhcrelay <DHCP server Ip adress> <DHCP server Ip adress> <DHCP server Ip adress>
```

Windows:

1. open rrasmgmt.msc
2. Selecteer New routing protocol
3. Selecteer DHCP Relay Agent
4. 1 of meerdere interfaces toevoegen
5. hop-count threshold en boot threshold (vertragingsinterval) instellen
6. DHCP-server IP-adressen toevoegen.
