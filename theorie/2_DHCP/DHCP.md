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

**transaction ID**: Maakt duidelijk aan de client op welke vraag de server heeft geantwoord.

***Options**:

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