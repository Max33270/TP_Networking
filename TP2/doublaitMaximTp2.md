# TP2 : Ethernet, IP, et ARP - Doublait Maxim (PC Windows 11)

## I. Setup IP

Addresse IP Audran Latorre : 192.168.1.132 / 26 <br>
Addresse IP PC Personnel : 192.168.1.133 / 26 <br>
Adresse Réseau : 192.168.1.128 / 26 <br>
Adresse de Broadcast : 192.168.1.191 / 26 

Voici la ligne de commande pour changer d'addresse IP : ```New-NetIPAddress -InterfaceIndex "24" -IPAddress 192.168.1.133 -PrefixLength 26```

On ping la machine du binôme pour tester la connectivité : ```ping 192.168.1.132``` 

Les types de paquets IMCP sont de type : <br>
- ```Echo Request``` (ping envoyé) 
-  ```Echo Reply``` (ping reçu) 

## II. ARP

CLI pour montrer la table arp : ```arp -a``` <br> 

Voici la table ARP : 
```
Interface : 192.168.195.1 --- 0x7
  Adresse Internet      Adresse physique      Type
  192.168.195.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.33.18.144 --- 0xd
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  10.33.19.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.0.3.2 --- 0x13
  Adresse Internet      Adresse physique      Type
  10.0.3.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.240.1 --- 0x17
  Adresse Internet      Adresse physique      Type
  192.168.240.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.1.133 --- 0x18
  Adresse Internet      Adresse physique      Type
  192.168.1.132         d8-bb-c1-1f-04-67     dynamique
  192.168.1.191         ff-ff-ff-ff-ff-ff     statique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

La ligne contenant l'IP et la MAC de mon binôme est : <br> ```192.168.1.132     d8-bb-c1-1f-04-67   dynamique``` <br> <br>
La ligne contenant l'addresse MAC de la gateway d'Ynov est : <br> ```Passerelle par défaut. . . . . . . . . : 10.33.19.254```
<br>

Ligne de commande pour vider la table arp : ```arp -d``` <br>
La ligne de commande pour afficher la table arp actualiser est : ```arp -a```
<br> <br>
 
Comme on peut voir sur la table ci-dessous après avoir rapidement tapé la commande ```arp -a``` suivi de ```arp -d``` qu'elle est beaucoup plus petite que celle au-dessus :

``` 
Interface : 192.168.195.1 --- 0x7
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.18.144 --- 0xd
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.0.3.2 --- 0x13
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.240.1 --- 0x17
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.1.133 --- 0x18
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  ``` 

## IV) DHCP


On commence par taper cette commande : ```Get-NetIPAddress```

```
IPAddress         : 192.168.1.195
InterfaceIndex    : 13
InterfaceAlias    : Wi-Fi
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
```

Maintenant que l'on a toutes les informations qu'il nous faut on tape cette ligne de commande pour changer l'addresse IP sur l'interface Wi-Fi.  

```New-NetIPAddress -InterfaceIndex "13" -IPAddress 192.168.1.203 -PrefixLength 24```

L'addresse IP a bien été modifié. On tape ensuite la commande suivante : ```Get-NetAdapter```

Résultat de la commande : 
```
Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
----                      --------------------                    ------- ------       ----------             ---------
Ethernet                  Realtek PCIe GbE Family Controller           24 Disconnected 10-62-E5-E2-90-3D          0 bps
VMware Network Adapte...1 VMware Virtual Ethernet Adapter for ...      23 Up           00-50-56-C0-00-01       100 Mbps
FreedomeVPNConnection     Freedome Wintun Userspace Tunnel             21 Disconnected                         100 Gbps
VirtualBox Host-Only ...2 VirtualBox Host-Only Ethernet Adap...#2      19 Up           0A-00-27-00-00-13         1 Gbps
Bluetooth Network Conn... Bluetooth Device (Personal Area Netw...      16 Disconnected D0-C5-D3-1D-9F-66         3 Mbps
Wi-Fi                     Realtek RTL8723DE 802.11b/g/n PCIe A...      13 Up           D0-C5-D3-1D-9F-67      72.2 Mbps
VMware Network Adapte...8 VMware Virtual Ethernet Adapter for ...       7 Up           00-50-56-C0-00-08       100 Mbps
```

Après avoir tapé la commande ```Disable-NetAdapter -Name "Wi-Fi" -Confirm:$false``` on remarque que le pour l'interface Wi-fi, le status passe de Up à disabled.

On tape ensuite la commande ```netsh interface ip set address "Wi-Fi" dhcp``` pour que le serveur dhcp nous donne une nouvelle addresse IP. 

On tape ensuite la commande ``Enable-NetAdapter -Name "Wi-Fi" -Confirm:$false`` pour se reconnecter au réseau. 

Après avoir tapé toutes ces commandes l'addresse IP 192.168.1.203 est devenue 192.168.1.195 : <br>
```
IPAddress         : 192.168.1.195
InterfaceIndex    : 13
InterfaceAlias    : Wi-Fi
``` 

4 trames DHCP lors d'un échange DHCP : <br>
- Discover : addresse source = ``0.0.0.0`` / addresse destination = ``255.255.255.255`` <br>
- Offer : addresse source = ``10.33.19.254`` / addresse destination = ``10.33.18.144`` <br>
- Request : addresse source = ``0.0.0.0`` / addresse destination = ``255.255.255.255`` <br> 
- Acknowledge : ``10.33.19.254`` / addresse destination = ``10.33.18.144`` <br> <br>

L'addresse IP à utiliser : ``10.33.18.144`` <br>
L'addresse IP de la paserelle du réseau : ``10.33.19.254`` <br>
L'addresse du serveur DNS joignable depuis ce réseau : ``0.0.0.0``

## TCP/UDP

Pour regarder une vidéo youtube mon PC se connecte à l'IP ```2a00:1450:4007:9::7``` sur le port 443 (le port https)