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
- Attention aux adresses réservées par VMWARE (gateway, host, dhcp, .1, .2, .254)

Le réseau interne (VMNET4) est en 192.168.49.0/24.

La plage DHCP est en 192.168.49.16/28
- Address:   192.168.49.16
- Netmask:   255.255.255.240 = 28
- Wildcard:  0.0.0.15
- Network:   192.168.49.16/28
- Broadcast: 192.168.49.31
- HostMin:   192.168.49.17
- HostMax:   192.168.49.30
- Hosts/Net: 14
             


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

### Etape 3 : protéger le firewall
Bloquer l'ensemble du trafic destiné au firewall, sauf SSH et PING
```
$sudo iptables -t filter -A OUTPUT -p icmp -j ACCEPT           
$sudo iptables -t filter -A INPUT -p icmp -j ACCEPT                       
$sudo iptables -t filter -A OUTPUT -p TCP --sport 22 -j ACCEPT            
$sudo iptables -t filter -A INPUT -p TCP --dport 22 -j ACCEPT
$sudo iptables -A OUTPUT -o lo -j ACCEPT
$sudo iptables -A INPUT -i lo -j ACCEPT
$sudo iptables -t filter -P INPUT DROP
$sudo iptables -t filter -P OUTPUT DROP
$sudo iptables -t filter -P FORWARD DROP
```

## Firewall PFSENSE
- IP LAN : 192.168.49.252
- Plage DHCP : 192.168.49.16/28

### Définir la machine
- FreeBSD64
- 2 interfaces réseau, une bridge, l'autre host only
- 2 cores
- 2048Mo de mémoire
- Boot CDROM sur l'iso de Pfsense

### Poser le système sur la machine
Booter la VM sur l'ISO d'installation

1. Boot multi user : enter
2. I
3. Change keymap to FR/ISO => accept settings
4. Quick and easy install : enter
5. Enter (standard)
6. Reboot

### Booter sur le disque de la machine + configurer
7. Enter
8. Vérifier l'attribution des interfaces réseau LAN et WAN
9. Si interfaces réseau mal attribuées, choix 1) puis réattribuer les interfaces
10. Choix 2) enter
11. Laisser l'interface WAN en DHCP
12. Configurer l'interface LAN :
  * IP : 192.168.49.252
  * Mask : 24
  * Enter (pas de gateway)
  * Enter (pas d'IPV6)
  * Enable DHCP : y
  * DHCP : 192.168.49.17 / 192.168.49.31
  * Revert to http : si y => choix entre http et https, si n => https uniquement : n

Fin de la configuration réseau minimale, ouvrir un browser sur le réseau host only et poursuivre en allant à l'adresse : https://192.168.49.252
* Login : admin
* Password : pfsense

### Configurer le firewall depuis l'interface Web
- Hostname : PFSENSE
- Domain : local
- Primary DNS Server : vide (conf DHCP WAN)
- Secondary DNS Server : vide

Ecran suivant

- NTP server: inchangé (on reste sur celui de pfsense pour la démonstration)
- Time zone :  Paris

Ecran suivant

- WAN : on ne change rien

Ecran suivant 

- LAN : on ne change rien

Ecran suivant
- cliquer sur le lien pour accéder à la configuration Web => affichage d'un dashboard system

On arrête la machine et on fait un snapshot en préparation des étapes suivantes.

### Gestion des flux
#### Contrôle de la configuration de base
- Désactiver le DHCP intégré à l'outil de virtualisation (VMWARE)
- Démarrer le PFSENSE
- Démarrer un client sur le réseau host-only
- Mettre le client en IP dynamique
  * Supprimer la configuration de eth0 dans /etc/network/interfaces
  * reboot (ou redémarrage ntwk et ntwk manager)
  * /sbin/ifconfig => ok lease dhcp 192.168.49.17
  * ping adresse IP externe => ok
  * traceroute adresse IP externe => le flux passe bien par le pfsense
  
