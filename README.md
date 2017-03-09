# SSI2017
## Adresses IP des machines
* NETFILTER : 192.168.49.253
* Réseau host only : VMNET4
* Réseau bridge : VMNET2

## Initialisation de la machine virtuelle
Faire un snapshot de l'état initial de la machine

  $addgroup debian sudo

Déloger / loger

  $sudo apt-get install wireshark

Choisir "Yes" pour utilisateur normal

```
    $sudo addgroup debian wireshark
    $sudo apt-get install openconnect
    $sudo apt-get install freerdp-x11
    $sudo openconnect dcloud-lon-anyconnect.cisco.com
    $sudo routel

```

Dans une autre console :

```
    $ping  198.19.10.200
    $xfreerdp /u:administrator /p:C1sco12345 /v:198.19.10.1

```

## Présentation générale de l'infrastructure
- Plan du réseau
- Différents éléments

## Liste des machines

Machines virtuelles présentes sur le réseau :
- Client : machine client pour réaliser les tests
- IPTable : firewall IPTable
- ProxySyslog : machine "à tout faire" utilisée comme proxy et pour la gestion des logs (+ éventuellement DHCP et DNS)
- Pfsense : Firewall PFSense
- Onion : machine embarquant security onion


Machines de base utilisées pour monter les machines du réseau :
- DebianSSI-IMIE2017 : machine de référence utilisée en linked clone par Client + utilisée pour faire les tests dcloud
- DebianSSI-IMIE-NoX : machine de référence utilisée en linkedclone par IPtable et ProxySyslog

## Réseau
- Création d'un réseau Custom : vmnet4 / 192.168.49.0/24
- Les machines IPTable et Pfsense ont une interface sur vmnet4 et une interface bridge sur le réseau externe
- Les autres machines ont une interface sur vmnet4

Le réseau interne (VMNET4) est en 192.168.49.0/24.

La plage DHCP est en 192.168.49.0/29
- Netmask:   255.255.255.224 = 27
- Wildcard:  0.0.0.31
- Network:   192.168.49.0/27
- Broadcast: 192.168.49.31
- HostMin:   192.168.49.1
- HostMax:   192.168.49.30
- Hosts/Net: 30             


Les machines serveurs (IP fixes) sont dans le 192.168.49.224/27
- Address:   192.168.49.224
- Netmask:   255.255.255.224 = 27
- Wildcard:  0.0.0.31   
- Broadcast: 192.168.49.255
- HostMin:   192.168.49.225
- HostMax:   192.168.49.254
- Hosts/Net: 30

Cf : http://jodies.de/ipcalc 

## Firewall NETFILTER
Il comporte 3 modules principaux :
- IPRoute2 : embarque les fonctions de routage
- Netfilter/IPtable : le firewall à proprement parler
- L7Filter : les fonctions de filtrage aux niveaux applicatifs

Les 4 modules interagissent dans une machine Linux pour produire un firewall relativement complet.

### Configuration du firewall NETFILTER
#### Etape 1 : poser la machine sur le réseau
```
    $sudo addgroup <useraccount> sudo
    $apt-get install vim
    $vim /etc/network/interfaces
    allow-hotplug eth0
    iface eth0 inet static
        address 192.168.49.253
        netmask 255.255.255.0
    $cat /etc/resolv.conf
    $vim /etc/hostname
    NETFILTER
    $vim /etc/hosts
    127.0.1.1 NETFILTER
    $sudo service networking restart
 ```
Tester ensuite le ping vers l'extérieur et le ping depuis le client se trouvant sur le réseau host only.
Les deux doivent être OK.

#### Etape 2 : activer le routage, mettre en place le NAT
Activer le routage :
```
  $sudo vim /etc/sysctl.conf
  net.ipv4.ip_forward=1
```
Activer le NAT
* eth1 : interface WAN
* eth0 : interface LAN
```
$sudo iptables -A FORWARD -i eth0 -j ACCEPT
$sudo iptables -A FORWARD -o eth0 -j ACCEPT
$sudo iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```
Vérification
* Pinger depuis la station présente sur le réseau interne vers une IP externe
* Lister les règles sur le firewal
```
$sudo iptables -L -v
$sudo iptables -t nat -L -v

```



  

