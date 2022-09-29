# TP1 Réseau - MAXIM DOUBLAIT B1B

## <u> I) Exploration locale en solo </u>

### 1) Affichage d'informations sur la pile TCP/IP locale

Affichage des infos des cartes réseau : <br> <br>
<u> Interface WiFi : </u>

Ligne de commande tapée : ```ifconfig```

nom = ```en0``` <br>
addresse MAC = ```3c:06:30:33:47:ed``` <br>
adresse IP = ```10.33.19.89```

Screenshot du résultat : 
![](Screenshot%202022-09-27%20at%2012.20.33.png)

<u> Interface Ethernet : </u>

Ligne de commande tapée : ```ifconfig```

nom = ```en1``` <br>
addresse MAC = ```36:68:2f:f5:ac:00``` <br>
adresse IP = Aucune, pas de ports ethernet sur ma machine (macbook pro M1)

Screenshot du résulat : 
![](Screenshot%202022-09-27%20at%2012.28.03.png)

Affichage de la paserelle <br>

Ligne de commande tapée : ```netstat -nr | grep default```

adresse IP = ```10.33.19.254```

Screenshot du résultat :
![](Screenshot%202022-09-27%20at%2012.33.33.png)


Affichage des informations sur une carte IP 

Pour voir ces informations sur MacOs, depuis le menu principal cliquer sur la Pomme, puis sur "System Preferences" > "Network" > "Advanced".

Adresse IP pour l'interface wifi en GUI : <br> <br>
![](Screenshot%202022-09-27%20at%2012.36.27.png) 
<br> <br>
Adresse MAC pour l'interface wifi en GUI : <br> <br>
![](Screenshot%202022-09-27%20at%2012.37.02.png)

Sur MacOS, on ne peut pas voir la paserelle pour l'interface wifi en GUI, on peut seulement le voir en ligne de commande.


La gateway dans le réseau d'Ynov sert à relier deux réseaux informatiques qui cherche à se connecter. Pour faire simple il permet de nous connecter à internet.

### 2) Modification des informations

A) Modification d'addresse IP 

Adresse IP de la carte wifi avec DHCP : 
![](Screenshot%202022-09-27%20at%2013.46.29.png)
<br> <br>
Adresse IP choisie manuellement : 
![](Screenshot%202022-09-27%20at%2013.48.49.png)


Il est possible de perdre accès à internet en changeant l'addresse IP manuellement dans le cas ou l'on change son IP à une IP qui existe déjà. 


## <u> II) Exploration locale en duo </u>

Partie II) effectué sur Windows (pas de ports ethernet sur mac)

3) Modification d'addresse IP

On modifie les addresses IP sur les deux machines et on vérifie avec la commande ```ipconfig /all``` que les deux machines ont bien eu leurs addresses IP modifiées.

Adresse IP PC Audran Latorre : 192.168.203.1
Adresse IP PC Personnel : 192.168.203.2

Test de ping entre les 2 machines réussi : <br> <br>
![](unknown%20(2).png)

Affichage de la table arp avec la commande ```arp -a``` : <br> <br> 
![](unknown%20(3).png)

4) Utilisation d'un des deux comme gateway 

On a désactivé l'interface internet du PC d'Audran et il a ensuite réussi à se connecter à mon PC Personnel et à pu ping des DNS connus. <br>

<u> Tests de la connectivité à internet depuis le PC d'Audran : <br> </u>

Screenshot du ping DNS réussi avec ma machine comme paserelle pour Audran :
![](unknown%20(4).png)

Screenshot pour suivre le chemin emprunté par le PC d'Audran pour se connecter à internet :
![](unknown%20(5).png)

Pour avoir accès à internet depuis google.com par exemple cela ne fonctionnera pas si l'on n'ajoute pas l'adresse IP du DNS (ex = 8.8.8.8) manuellement sur la machine. 

5) Petit chat privé <br>

La communication entre les deux machines avec netcat fonctionne :

![](unknown%20(7).png)

![](unknown%20(8).png)

6) Firewall 

Pour autoriser les ping, il faut taper la commande ```netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:0,any dir=in action=allow``` dans le terminal en tant qu'administrateur. Ceci va rajouter une règle dans le firewall pour autoriser les ping. 

Après avoir entrée cette commande, le terminal nous retourne "OK", la règle a bien été ajoutée.
![](unknown%20(11).png)

Pour autoriser le traffic sur le port qu'utilise netcat, il faut aller dans le firewall de windows et autoriser le traffic sur le port 8888 (le port que l'on a decidé d'utiliser).
Pour faire cela sur Windows 11 depuis le menu principal, il faut cliquer sur le logo Windows > Panneau de configuration > Système et sécurité > Pare-feu Windows Defender > Paramètres avancés > Règles de traffic entrant > Actions > Nouvelle Règle > choisir le type de règle "Port" > choisir le type de port "TCP" > choisir "Ports locaux spécifiques" et taper le port "8888" > choisir l'action "Autoriser la connexion" > choisir les profil "Domaine", "Privé" et "Public" > choisir le nom de cette nouvelle règle > cliquer sur "Terminer" pour valider la règle.

On peut maintenant communiquer avec netcat en ayant le firewall activé comme on peut le voir sur le screenshot ci-dessous : : 
![](unknown%20(10).png)

## <u> III) Manipulations d'autres outils/protocoles côté client </u> 

DHCP <br>

Ligne de commande tapée : ```ipconfig getpacket en0``` <br>

Screenshot de l'IP du serveur DHCP du réseau WiFi YNOV : 
![](Screenshot%202022-09-28%20at%2008.52.39.png)

Sur MacOS, on ne peut pas voir la date d'expiration de l'IP du serveur DHCP du réseau WiFi YNOV. On trouve une ligne avec le lease time mais ce n'est pas très parlant. 
Screenshot du lease time du DHCP sur MacOS : 
![](Screenshot%202022-09-28%20at%2008.59.09.png)

Voici a quoi ressemble le DHCP lease sur Windows :
![](unknown%20(12).png)

On peut voir quand l'IP a été attribuée et quand elle expire.

DNS <br>

Ligne de commande tapée : ```cat /etc/resolv.conf``` 
La commande ```scutil --dns``` fonctionne aussi. 

Screenshot des IP connus du serveur DNS (en cas de panne de l'un, l'autre prend le relais) :
![](Screenshot%202022-09-28%20at%2009.50.21.png)

```dig google.com``` 
Une des addresses IP de google est 216.58.215.46. 
![](Screenshot%202022-09-28%20at%2009.53.12.png)

```dig ynov.com``` 
Les adresses IP de ynov sont les suivantes : 
![](Screenshot%202022-09-28%20at%2009.55.42.png)

Interprétation des résultats : <br>
Cette commande nous donne la ou les addresses IP des noms de domaines existants ainsi que la date et l'heure de quand on a tapé cette  commande dans le terminal ainsi que la taille du message reçu.   

Reverse lookup <br>

```dig -x 78.74.21.21``` <br>
Screenshot du nom de domaine associé à l'IP :
![](Screenshot%202022-09-28%20at%2009.58.00.png)

```dig -x 92.146.54.88``` <br>
Screenshot du résultat de la commande : 
![](Screenshot%202022-09-28%20at%2010.06.26.png)

Interprétation des résultats : <br>
La première addresse IP a un nom de domaine associé. Par contre il n'y a pas de noms de domaine associés à la deuxième IP, c'est pour cela que la commande nous renvoie un "IN SOA" au lieu d'un "IN PTR".

## <u> IV) Wireshark </u> 

On retape la commande ```netstat -nr | grep default``` pour avoir l'adresse IP de la passerelle par défaut. <br>

Puis on ping cette addresse IP (172.20.10.1) et on observe les trames suivantes : 
![](Screenshot%202022-09-28%20at%2019.29.11.png)

Sur Wireshark, on filtre les trames avec comme IP source ou comme IP destination l'adresse IP de la machine d'Audran.
Screenshot des trames entre les 2 machines avec netcat (entre les IP 192.168.203.1 et ...2) :
![](unknown%20(13).png) 

On tape ensuite la commande ```scutil --dns``` afin de voir l'addresse IP du serveur DNS (192.168.1.254). <br>
![](Screenshot%202022-09-28%20at%2020.10.52.png)

Puis on décide de faire une requête DNS vers google.com donc pour connaître l'addresse IP de google.com on écrit la commande ```dig google.com```. Ensuite on fait un ping de l'addresse IP 142.250.179.110 <br>
![](Screenshot%202022-09-28%20at%2020.11.17.png)

On observe ensuite les trames sur Wireshark :
![](Screenshot%202022-09-28%20at%2020.10.33.png)

On observe que l'addresse IP de google (142.250.179.110) communique avec le serveur DNS (192.168.1.254) en passant par l'IP 192.168.1.117. Visuellement c'est simple de repèrer les requêtes DNS car elles sont en bleus et il est inscrit 'DNS' dans la colonne "Protocol". 