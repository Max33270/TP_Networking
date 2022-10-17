# TP4 : TCP, UDP et services réseau - Doublait Maxim

## I. First steps


1. Discord 

``IP = 162.159.130.235  ;  Port = 443  ;   Port Local = 49734 `` 

2. Netflix

``IP = 92.90.254.205 ; Port = 443  ;  Port Local = 49180``

3. Youtube

``IP = 91.68.245.144 ; Port = 443  ;  Port Local = 61598``

4. Spotify 

``IP = 2.21.34.10 ; Port = 443  ;  Port Local = 49465``

5. Prime Video 

``IP = 185.178.53.19 ; Port = 443  ;  Port Local = 49435``

<br> 

Pour confirmer avec l'OS on tape la commande ``netstat -n -b `` : <br>

1. Discord 
```
 [chrome.exe]
  TCP    10.33.19.55:49734      162.159.130.235:443    ESTABLISHED
```

2. Netflix 
```
 [chrome.exe]
  TCP    10.33.19.55:49180      92.90.254.205:443      ESTABLISHED
``` 

3. Youtube 
```
 [chrome.exe]
  TCP    10.33.19.55:61598      91.68.245.144:443    ESTABLISHED
```

4. Spotify 
```
 [chrome.exe]
  TCP    10.33.19.55:49465      2.21.34.10:443         ESTABLISHED
```

5. Prime Video 
```
 [chrome.exe]
  TCP    10.33.19.55:49435      185.178.53.19:443    ESTABLISHED
```

## II. Mise en place

### 1. SSH 

SSH utilise TCP, en effet pour une connection secure dans un shell c'est mieux que ce soit fiable et un peu lent que non-fiable et rapide.

Pour voir la connexion SSH depuis ma machine, on tape la command ``netstat -a -b`` : <br>
```
TCP    10.3.1.3:50194         10.3.1.11:ssh          ESTABLISHED
```
Pour voir cette connexion depuis la VM, on tape la commande : ``ss | grep -i ssh``
```
tcp   ESTAB  0      52                       10.3.1.11:ssh       10.3.1.3:50134
```

### 2. NFS 

Serveur : 

``` sudo dnf -y install nfs-utils ``` <br>
``` 
sudo nano /etc/idmapd.conf  
Domain = srv.world
```

``` 
sudo nano /etc/exports  
# create new
# for example, set [/home/nfsshare] as NFS share
/home/nfsshare 10.3.2.12/24(rw,no_root_squash)
```

```  mkdir /home/nfsshare ``` <br>

``` systemctl enable --now rpcbind nfs-server ``` 

``` firewall-cmd --add-service=nfs ```

``` firewall-cmd --add-service={nfs3,mountd,rpc-bind} ``` 

``` firewall-cmd --runtime-to-permanent ```

Client : 

``` sudo dnf -y install nfs-utils ``` <br>
``` 
sudo nano /etc/idmapd.conf  
Domain = srv.world
```

``` sudo mount -t nfs dlp.srv.world:/home/nfsshare /mnt ``` <br>
Access Denied for mountfs <br>
```  sudo mount -t nfs -o vers=3 dlp.srv.world:/home/nfsshare /mnt ``` <br>
Access Denied for mountfs <br>


### 3. DNS 

IP : 8.8.8.8  ; Port  : 37281 