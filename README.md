# SSI2017
## Initialisation de la machine virtuelle
Faire un snapshot de l'état initial de la machine

$addgroup debian sudo

Déloger / loger

$sudo apt-get install wireshark

Choisir "Yes" pour utilisateur normal

$sudo addgroup debian wireshark

$sudo apt-get install openconnect

$sudo apt-get install freerdp-x11

$sudo xfreerdp dcloud-lon-anyconnect.cisco.com

Autre console :
$ping  198.19.10.200


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

## Firewall IPTable
Il comporte 3 modules principaux :
- IPRoute2 : embarque les fonctions de routage
- Netfilter/IPtable : le firewall à proprement parler
- L7Filter : les fonctions de filtrage aux niveaux applicatifs

Les 4 modules interagissent dans une machine Linux pour produire un firewall relativement complet.

