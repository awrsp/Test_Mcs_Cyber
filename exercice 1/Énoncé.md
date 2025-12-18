# Exercice  01 : Packet Tracer

Création d’un « miniLab » : vous devez configurer le réseau de manière à respecter les contraintes suivantes et tester la connectivité entre les différents VLAN et l’accès Internet.

## Vous disposez des équipements suivants pour la mise en place du réseau :
- 1 routeur Cisco 1941
- 3 switch PT
- 3 points d’accès Wi-Fi PT-AC
- 3 PC portables
- 6 PC fixes
- 3 téléphones IP Cisco 7960 (Vous n’avez a pas gérer l’ipbx)




##  Chaque switch doit être configuré avec les VLAN suivants :

| Port | Affectation | VLAN |
|---|---|---|
| Port 8 | Administration | VLAN 30 |
| Ports 6-7 | PC fixes | VLAN 20 |
| Ports 4-5 | Points d’accès Wi-Fi | VLAN 10 |
| Ports 2-3 | Téléphones IP (VoIP) | VLAN 1 |
| Ports 1 et 9 | Uplink (Ethernet/Fibre) | TRUNK |


## Plan d’adressage et configuration DHCP :


Le routeur 1941 servira de passerelle Internet, de serveur DHCP et de gestionnaire de VLAN. Les réseaux sont configurés comme suit :

| VLAN | Usage | Réseau | Plage DHCP |
|---|---|---|---|
| VLAN 1 | VoIP | 192.168.0.0/24 | 192.168.0.10 - 192.168.0.50 |
| VLAN 10 | PC fixes | 192.168.10.0/24 | 192.168.10.10 - 192.168.10.50 |
| VLAN 20 | Wi-Fi | 192.168.20.0/24 | 192.168.20.10 - 192.168.20.50 |
| VLAN 30 | Administration | 192.168.30.0/24 | 192.168.30.10 - 192.168.30.50 |




## Répartition dans les bureaux :
Chaque bureau doit contenir :
- 1 switch
- 1 point d’accès Wi-Fi
- 1 PC portable
- 2 PC fixes
- 1 téléphone IP
