# TP3 : Routage - Doublait Maxim (PC Windows 11)

On lance la commande ``ipconfig /all `` pour s'assurer que les deux réseaux host only sont bien configurés. Tout est bon! <br>
``` 
Carte Ethernet VirtualBox Host-Only Network :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : VirtualBox Host-Only Ethernet Adapter
   Adresse physique . . . . . . . . . . . : 0A-00-27-00-00-05
   DHCP activé. . . . . . . . . . . . . . : Non
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::ed07:abd5:de31:c871%5(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.3.1.3(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . :
   IAID DHCPv6 . . . . . . . . . . . : 319422503
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-28-08-E2-D4-10-62-E5-E2-90-3D
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé

Carte Ethernet VirtualBox Host-Only Network #2 :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : VirtualBox Host-Only Ethernet Adapter #2
   Adresse physique . . . . . . . . . . . : 0A-00-27-00-00-16
   DHCP activé. . . . . . . . . . . . . . : Non
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::548e:16dd:4d04:95d5%22(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.3.2.3(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Passerelle par défaut. . . . . . . . . :
   IAID DHCPv6 . . . . . . . . . . . : 1107951655
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-28-08-E2-D4-10-62-E5-E2-90-3D
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé

``` 

## I. ARP 

### 1) Echange ARP

On réussit à ping les 2 machines correctement : <br>

Depuis la VM 10.3.1.11 : 
```
[max@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.560 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=1.25 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=1.48 ms
``` 
<br> Depuis la VM 10.3.1.12 : 
```
[max@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.382 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=1.22 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=1.21 ms
```
<br>

On observe la table ARP de 10.3.1.11 : 
```
[max@localhost network-scripts]$ ip neigh show
10.3.1.12 dev enp0s3 lladdr 08:00:27:b8:2a:59 STALE
10.3.1.3 dev enp0s3 lladdr 0a:00:27:00:00:05 REACHA
```

Pour voir la MAC de Marcel dans la table ARP de John on tape la commande suivante : 
```
[max@localhost network-scripts]$ ip neigh show 10.3.1.12
10.3.1.12 dev enp0s3 lladdr 08:00:27:b8:2a:59 STALE
```

Pour voir la MAC de Marcel depuis la table ARP de Marcel on tape la commande ``ip a`` et on voit que la MAC de marcel est bien ``08:00:27:b8:2a:59`` : 

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b8:2a:59 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s3
```

### 2) Analyse de trames

On ouvre 2 Windows Powershell et on se connecte en ssh aux 2 machines : ``ssh max@10.3.1.11`` et ``ssh max@10.3.1.12`` 

Ensuite, on tape la commande ``sudo tcpdump -i enp0s3 -w tp2_arp.pcapng``. <br>
On vide les tables ARP des 2 machines avec la commande ``sudo ip neigh flush all``. On vérifie à l'aide d'un ``ip neigh show`` que la commande a bien fonctionné. 
On fait un ping entre les 2 machines, on attend un peu et on met fin à la commande tcpdump. 

On exporte ensuite le fichier tp2_arp.pcapng en tapant la commande suivante en local : <br>
``scp max@10.3.1.11:/home/tp2_arp.pcapng ./``

On sélectionne 2 paquets : un ARP Request et un Reply et on les mets dans le fichier ``arpReplyRequest.pcap``

## II) Routage

Création de la 3ème VM et configuration : <br> 

Interface enp0s3 : <br> <br>
``` sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3```

``` 
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.254
NETMASK=255.255.255.0 
``` 

Interface enp0s8 : <br> <br>
``` sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8```

``` 
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.2.254
NETMASK=255.255.255.0 
``` 

### 1) Mise en place du routage

A) Activer le routage sur le noeud router 

On tape les commandes suivantes dans la VM "Router" : 

```
sudo firewall-cmd --add-masquerade --zone=public
sudo firewall-cmd --add-masquerade --zone=public --permanent
```

B) Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping

Pour la VM de John (10.3.1.11) : <br>
``` sudo nano /etc/sysconfig/network-scripts/route-enp0s3 ```  <br>

Dans ce fichier route-enp0s3, on rajoute la route : <br>
``10.3.2.12/24 via 10.3.1.254 dev enp0s3`` <br>

Pour la VM de Marcel (10.3.2.12) : <br>
``` sudo nano /etc/sysconfig/network-scripts/route-enp0s3 ```  <br>

Dans ce fichier route-enp0s3, on rajoute la route : <br>
``10.3.1.11/24 via 10.3.2.254 dev enp0s3``

On fait un test de ping pour vérifier que John puisse joindre Marcel : 
```
[max@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=2.24 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.28 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.44 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.37 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=1.47 ms
``` 

Et on vérifie aussi que Marcel puisse joindre John : 
```
[max@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=1.26 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=2.71 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=3.31 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=63 time=2.73 ms
64 bytes from 10.3.1.11: icmp_seq=5 ttl=63 time=2.58 ms
```

### 2) Analyse de trames

Pour vider la tabme ARP de John, Marcel et du Routeur : <br>
``sudo ip neigh flush all`` 

```
[max@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.14 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=2.07 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=2.31 ms
```

Table ARP de John après le ping : 
```
[max@localhost ~]$ ip neigh show
10.3.1.3 dev enp0s3 lladdr 0a:00:27:00:00:05 REACHABLE
10.3.1.254 dev enp0s3 lladdr 08:00:27:b8:2a:59 REACHABLE
```

Table ARP du routeur après le ping : 
```
[max@localhost ~]$ ip neigh show
10.3.2.3 dev enp0s8 lladdr 0a:00:27:00:00:16 REACHABLE
10.3.2.12 dev enp0s8 lladdr 08:00:27:b8:2a:59 STALE
10.3.1.11 dev enp0s3 lladdr 08:00:27:b8:2a:59 REACHABLE
10.3.1.3 dev enp0s3 lladdr 0a:00:27:00:00:05 REACHABLE
```

Table ARP de Marcel après le ping : 
```
10.3.2.3 dev enp0s3 lladdr 0a:00:27:00:00:16 DELAY
10.3.2.254 dev enp0s3 lladdr 08:00:27:7a:7d:ee STALE
```

Déduction des échanges ARP qui ont lieu : 
- John diffuse une requête ARP en broadcast et demande qui se trouve à l'IP 10.3.2.12 
- Il passe par le routeur et le routeur enregistre John dans sa table ARP
- Marcel reçoit la requête de John via le routeur et ajoute le routeur dans sa table ARP
- Marcel renvoie une réponse et indique que c'est lui qui se trouve à l'IP demandé 
- Le routeur reçoit le message et le transmet vers John et rajoute Marcel dans sa table ARP
- John reçoit le message et rajoute le routeur dans sa table ARP 


On lance la commande ``sudo ip neigh flush all`` sur les 3 VM. <br> 

On lance ensuite la commande sudo tcpdump -i enp0s3 -w tp2_routage_marcel.pcapng
Puis on fait un ping :  ``ping 10.3.2.12`` depuis le PC de John. 


| ordre | type  trame    | IP  source | MAC  source              | IP destination | MAC  destination         |
|-------|----------------|------------|--------------------------|----------------|--------------------------|
| 1     | Requête ARP    | 10.3.1.11  | John 08:00:27:b8:2a:59   | 10.3.2.254     | Broadcast FF:FF:FF:FF    |
| 2     | Réponse  ARP   | 10.3.2.12  | Marcel 08:00:27:b8:2a:59 | 10.3.1.11      | John 08:00:27:b8:2a:59   |
| 3     | Ping (request) | 10.3.2.254 | Router 08:00:27:7a:7d:ee | 10.3.2.12      | Marcel 08:00:27:b8:2a:59 |
| 4     | Pong (reply)   | 10.3.2.12  | Marcel 08:00:27:b8:2a:59 | 10.3.2.254     | Router 08:00:27:7a:7d:ee |
| 5     | Requête  ARP   | 10.3.2.12  | Marcel 08:00:27:b8:2a:59 | 10.3.2.254     | Router 08:00:27:7a:7d:ee |
| 6     | Réponse  ARP   | 10.3.2.254 | Router 08:00:27:7a:7d:ee | 10.3.2.12      | Marcel 08:00:27:b8:2a:59 |

Ensuite on exporte le fichier avec la commande : <br> ``scp max@10.3.2.12:/home/tp2_routage_marcel.pcapng ./``

### 3) Accès internet 

On ajoute une carte NAT sur le routeur et on vérifie qu'il ai accès à internet : 
```
[max@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=13.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=119 time=12.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=119 time=13.8 ms
```

On rajoute ensuite les routes pour que John et Marcel puisse avoir accès à internet : 

Pour John : 

``sudo nano /etc/sysconfig/network``
```
# Created by anaconda
GATEWAY=10.3.1.254
``` 

On vérifie que la route a bien été ajouté avec la commande ``sudo ip r s `` : 

```
[max@localhost ~]$ sudo ip r s
[sudo] password for max:
default via 10.3.1.254 dev enp0s3 proto static metric 100
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.11 metric 100
10.3.2.0/24 via 10.3.1.254 dev enp0s3 proto static metric 100
```

Pour Marcel : 

``sudo nano /etc/sysconfig/network``
```
# Created by anaconda
GATEWAY=10.3.2.254
``` 

On vérifie que la route a bien été ajouté avec la commande ``sudo ip r s `` : 
```
[max@localhost ~]$ sudo ip r s
[sudo] password for max:
default via 10.3.2.254 dev enp0s3 proto static metric 100
10.3.1.11 via 10.3.2.254 dev enp0s3 proto static metric 100
10.3.2.0/24 dev enp0s3 proto kernel scope link src 10.3.2.12 metric 100
```

#### John

Ping d'une addresse IP (Google) : 

```
[max@localhost ~]$ ping 142.250.75.238
PING 142.250.75.238 (142.250.75.238) 56(84) bytes of data.
64 bytes from 142.250.75.238: icmp_seq=1 ttl=118 time=21.2 ms
64 bytes from 142.250.75.238: icmp_seq=2 ttl=118 time=17.2 ms
64 bytes from 142.250.75.238: icmp_seq=3 ttl=118 time=17.7 ms
64 bytes from 142.250.75.238: icmp_seq=4 ttl=118 time=17.7 ms
```

On modifie le fichier resolv.conf avec la commande ``sudo nano /etc/resolv.conf`` : 

```
# Generated by NetworkManager
nameserver 8.8.8.8
nameserver 1.1.1.1
```

On vérifie le bon fonctionnement de la résolution de noms : 
```
[max@localhost ~]$ dig google.com

; <<>> DiG 9.16.23-RH <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46389
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             236     IN      A       142.250.179.110

;; Query time: 29 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sun Oct 09 14:55:55 CEST 2022
;; MSG SIZE  rcvd: 55
```

Ping d'un nom de domaine : 
```
[max@localhost ~]$ ping google.com
PING google.com (142.250.75.238) 56(84) bytes of data.
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=1 ttl=118 time=14.9 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=118 time=17.0 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=3 ttl=118 time=19.1 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=4 ttl=118 time=16.4 ms
```


#### Marcel

Ping d'une addresse IP (Ynov) : 

```
[max@localhost ~]$ ping 104.26.11.233
PING 104.26.11.233 (104.26.11.233) 56(84) bytes of data.
64 bytes from 104.26.11.233: icmp_seq=1 ttl=55 time=16.1 ms
64 bytes from 104.26.11.233: icmp_seq=2 ttl=55 time=18.4 ms
64 bytes from 104.26.11.233: icmp_seq=3 ttl=55 time=19.3 ms
64 bytes from 104.26.11.233: icmp_seq=4 ttl=55 time=18.4 ms
```

On modifie le fichier resolv.conf avec la commande ``sudo nano /etc/resolv.conf`` : 

```
# Generated by NetworkManager
nameserver 8.8.8.8
nameserver 1.1.1.1
```

On vérifie le bon fonctionnement de la résolution de noms : 
```
[max@localhost ~]$ dig ynov.com

; <<>> DiG 9.16.23-RH <<>> ynov.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1483
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;ynov.com.                      IN      A

;; ANSWER SECTION:
ynov.com.               300     IN      A       104.26.10.233
ynov.com.               300     IN      A       104.26.11.233
ynov.com.               300     IN      A       172.67.74.226

;; Query time: 30 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sun Oct 09 15:02:11 CEST 2022
;; MSG SIZE  rcvd: 85
```

Ping d'un nom de domaine : 
```
[max@localhost ~]$ ping ynov.com
PING ynov.com (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=55 time=20.1 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=2 ttl=55 time=32.3 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=3 ttl=55 time=31.8 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=4 ttl=55 time=19.7 ms
```

#### Analyse de trames 

On ping 8.8.8.8 depuis John et on lance la commande ``sudo tcpdump -i enp0s3 -w tp3_routage_internet.pcapng``

```
[max@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=26.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=56.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=25.7 ms
```


Wireshark donne la même mac aux 2 IP : 

| ordre | type trame | IP  source | MAC  source              | IP destination | MAC  destination         |
|-------|------------|------------|--------------------------|----------------|--------------------------|
| 1     | ping       | 10.3.1.11  | John 08:00:27:b8:2a:59   | 8.8.8.8    | 08:00:27:b8:2a:59  |
| 2     | pong       | 8.8.8.8  |  08:00:27:b8:2a:59 | 10.3.1.11      | John 08:00:27:b8:2a:59   |


## III) DHCP

Pour l'installation du serveur dhcp sur John on tape les commandes suivantes : 
- ``sudo dnf install dhcp-server``
- ``sudo nano /etc/dhcp/dhcpd.conf``

Fichier dhcpd.conf : 
```
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.1.0 netmask 255.255.255.0 {
  range 10.3.1.12 10.3.1.253;
  option routers 10.3.1.254; 
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8;
}
```

- ``sudo systemctl enable --now dhcpd``
- ``sudo systemctl status dhcpd ``

```
Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
Active: active (running) since Sun 2022-10-09 21:19:44 CEST; 17min ago
```

Définir une IP dynamique pour Bob : 
- sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

```
NAME=enp0s8
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes

```

- sudo systemctl restart NetworkManager 

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b8:2a:59 brd ff:ff:ff:ff:ff:ff
    inet 10.3.2.12/24 brd 10.3.2.255 scope global noprefixroute enp0s3
```

#### Améliorer la configuration du DHCP : 

Fichier dhcpd.conf : 

```
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.1.0 netmask 255.255.255.0 {
  range 10.3.1.12 10.3.1.253;
  option routers 10.3.1.254; 
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8;
}
```

On récupère une nouvelle ip pour Bob : 
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b8:2a:59 brd ff:ff:ff:ff:ff:ff
    inet 10.3.2.12/24 brd 10.3.2.255 scope global noprefixroute enp0s3
```


Bob peut ping sa paserelle : 
```
[max@localhost ~]$ ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.294 ms
64 bytes from 10.3.1.254: icmp_seq=1 ttl=63 time=0.412 ms
```

Bob a une route par défaut : 
```
[max@localhost ~]$ ip r s
default via 10.3.1.254 dev enp0s3 proto dhcp src 10.3.1.12 metric 100
```

Après avoir fait la commande ``dig facebook.com``, on fait un ping vers cette IP : 
```
[max@localhost ~]$ ping 157.240.21.35
PING 157.240.21.35 (157.240.21.35) 56(84) bytes of data.
64 bytes from 157.240.21.35: icmp_seq=1 ttl=56 time=273 ms
64 bytes from 157.240.21.35: icmp_seq=1 ttl=55 time=274 ms (DUP!)
64 bytes from 157.240.21.35: icmp_seq=1 ttl=55 time=274 ms (DUP!)
```

La commande dig fonctionne : 

```
[max@localhost ~]$ dig twitter.com

; <<>> DiG 9.16.23-RH <<>> twitter.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7706
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;twitter.com.                   IN      A

;; ANSWER SECTION:
twitter.com.            196     IN      A       104.244.42.193
twitter.com.            196     IN      A       104.244.42.1

;; Query time: 43 msec
;; SERVER: 192.168.1.254#53(192.168.1.254)
;; WHEN: Sun Oct 09 23:19:48 CEST 2022
;; MSG SIZE  rcvd: 72
```

Bob peut aussi ping un nom de domaine : 
```
[max@localhost ~]$ ping google.com
PING google.com (216.58.198.206) 56(84) bytes of data.
64 bytes from par10s27-in-f14.1e100.net (216.58.198.206): icmp_seq=1 ttl=118 time=26.0 ms
64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=1 ttl=117 time=26.4 ms
```

### 2) Analyse de trames 

On lance la commande ``sudo tcpdump -i enp0s3 -w tp2_dhcp.pcapng``

Pour demander une nouvelle ip on supprime l'ancienne avec ``sudo dhclient -r``
Puis on en redemande une nouvelle avec ``sudo dhclient``

On exporte ensuite le fichier en local avec la commande ```scp max@10.3.1.12:/home/tp2_dhcp.pcapng ./Desktop/```