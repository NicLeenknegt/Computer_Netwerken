# DHCP cliënts

## Linux

dhcp clients voor linux:

* dhclient
* dhcpcd
* pump

=> wijzigt naam van machine, IP-adres, subnet masker, routetabel en resolver config (etc/resolv.conf)

### dhclient

```bash
dhclient eth0
```

start de dhclient service, de ip-informatie van eth0 word dan ingesteld door een dhcp-server.

Dhclient neemt controle over de interface en zal altijd een nieuwe lease zoeken als de vorige is verlopen.

* eth0: vaak de dhcp client interface
* eth1, eth2,...: De andere interfaces speken vaak een speciale rol en worden dus manueel ingesteld.

```bash
dhclient eth0 -r
```

Lost de lease van eth0.

```bash
dhclient eth0 -d
```

Toont DHCP verkeer op de console.

```bash
dhclient -1 -d eth0
```

* -1: De client gaat 1 keer een lease voor eth0 zoeken. De interface komt niet onder beheer van dhclient na deze eerste poging.
* -d: Forceert de client om in de voorgrond te draaien. Toont dan alle debug informatie ?

```bash
kill 'cat /var/run/dhclient-eth0.pid' >/dev/null 2>&1
```

Dood het dhclient proces dat beheerd heeft over eth0.

```bash
cat /var/lib/dhcp/dhcpclient.leases
lease {
  interface "enp0s3";
  fixed-address 192.168.16.32;
  option subnet-mask 255.255.255.0;
  option dhcp-lease-time 43200;
  option routers 192.168.16.1;
  option dhcp-message-type 5;
  option dhcp-server-identifier 192.168.16.16;
  option domain-name-servers 192.168.16.16,192.168.16.13;
  option dhcp-renewal-time 21600;
  option dhcp-rebinding-time 37800;
  option domain-name "iii.hogent.be";
  renew 3 2015/01/07 16:55:12;
  rebind 3 2015/01/07 22:35:37;
  expire 4 2015/01/08 00:05:37;
}
```

Op de Besturing_Systemen_II VM was het /var/lib/dhclient/dhclient.leases
Dit is het voorbeeld van mijn macbook.

Dit commando toont de info die dhclient bijhoud over zijn leases. Zo weet dhclient bij opstarten aan welke server die om een adres moet vragen.

```bash
vim /etc/dhclient.conf
```

Bevat alle dhclient configuraties zoals:

* Gewenste leasetijd
* Hoeveel keer een anvraag opnieuw mag proberen
* of /etc/resolv.conf mag worden gewijzigd

### dhcpd

```bash
dhcpd -n eth0
```

Vraag een lease aan.

```bash
dhcp -k eth0
```

Laat lease los.

```bash
dhcpd -H -D -R eth0
```

* -H en -D: Bepalen of de opties 12 en 15 worden verwerkt.
    * 12: Naam voor de host
    * 15: DNS suffix
* -R: verbied aanpassen van /etc/resolv.conf

```bash
cat /etc/dhcpc/dhcpd-eth0.info
cat /etc/dhcpc/dhcpd-eth0.cache
```

Toont informatie over de leases van eth0

### pump

```bash
pump -i eth0
```

Vraagt IP-informatie aan voor eth0.

```bash
pump -i eth0 -r
```

Laat lease van eth0 los.

```bash
pump -i eth0 -s
```

Toont status van eth0.

### Opstart script

```bash
vim /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
DEVICE=eth0
NM_CONTROLLED=no
BOOTPROTO=dhcp # BOOTPROTO moet gelijk zijn aan dhcp
ONBOOT=yes
```

ipv. manueel pump, dhcpd of dhclient te gebruiken kan men bovenstaande optie dhcp automatisch laten configureren bij opstarten.ß

Met ifup eth0 kan men dan een lease aanvragen.
ifdown eth0 verwijderd de lease niet maar sluit de interface wel.