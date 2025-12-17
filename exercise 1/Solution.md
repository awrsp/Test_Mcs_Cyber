# Matériel

Premièrement, mise en place du matériel physique dans la salle.


Ils ont été installés dans chaque bureau, comme l'exige l'énoncé.


Le routeur a été placé dans la salle réseau pour faciliter le câblage.


J'ai ajouté le module HWIC-4EW pour permettre le branchement RJ45 aux commutateurs dans chaque salle.


Un panneau de brassage dans la salle réseau et des prises murales en cuivre ont été ajoutés pour faciliter les branchements.


Trois salles ont été choisies comme bureaux, puis les équipements ont été installés.


Les commutateurs ont reçu 8 modules PT-SWITCH-NM1CGE pour normaliser les ports. Nous n'avons pas pu ajouter de ports fibre optique sur les commutateurs pour les reconnecter au routeur, car le routeur n'est pas compatible avec 4 ports fibre optique ; il dispose seulement d'un module RJ45 4 ports.





# Logiciel
## Configuration des VLAN sur les commutateurs

~~Premièrement, il nous est demandé de créer 4 VLAN.~~


```
enable
configure terminal

! Création des VLAN
vlan 1
 name VOIP
vlan 10
 name WIFI
vlan 20
 name PC
vlan 30
 name ADMIN
exit

```


Cependant, il est généralement recommandé de ne pas utiliser le VLAN 1. 


- Networkpulse FR — [Ne laissez pas votre VLAN 1 être un risque : guide de sécurisation rapide](https://networkpulse.fr/ne-laissez-pas-votre-vlan-1-etre-un-risque-guide-de-securisation-rapide/)


- Netseccloud EN — [What is VLAN 1 and How Does It Work?](https://netseccloud.com/what-is-vlan-1-and-how-does-it-work)

Donc, je préfère créer le VLAN 2 pour la VoIP et laisser le VLAN 1 comme VLAN natif ; on essaiera de ne pas l'utiliser.

```
enable
configure terminal
vlan 2
 name VOIP
vlan 10
 name WIFI
vlan 20
 name PC
vlan 30
 name ADMIN
exit
exit
write memory
exit
```

## Attribution des ports aux VLAN


### Ports 2-3 : VLAN 2 (VoIP)

```
enable
configure terminal
interface range GigabitEthernet1/1 , GigabitEthernet2/1
 switchport mode access
 switchport access vlan 2
 spanning-tree portfast
exit
exit
```


### Ports 4-5 : VLAN 10 (Wi-Fi)
```
enable
configure terminal
interface range GigabitEthernet3/1 , GigabitEthernet4/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
exit
```


### Ports 6-7 : VLAN 20 (PC)
```
enable
configure terminal
interface range GigabitEthernet5/1 , GigabitEthernet6/1
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit
exit
```


### Port 8 : VLAN 30 (Administration)
```
enable
configure terminal
interface GigabitEthernet7/1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
exit
```
### Fermeture des ports inutilisés
il est de [bonne pratique](https://learn.pivitglobal.com/securing-unused-ports) de desactiver les port inutilisee et ou de rediriger le trafic vers un vlan vide  

```
enable
configure terminal
interface GigabitEthernet9/1
shutdown
exit
exit
```

## Configuration des trunks sur les commutateurs
### Ports 1 et 9 : trunks (uplinks)
```
enable
configure terminal
interface range GigabitEthernet0/1 , GigabitEthernet8/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 2,10,20,30
exit

hostname SW1
exit
write memory

```
Répétez la même configuration sur les autres commutateurs en adaptant le `hostname`.

# Configuration du routeur

## Configuration des sous-interfaces pour les VLAN
### Fermeture des ports inutilisés
```
enable 
configure terminal
interface range FastEthernet0/0/0 - 2 , FastEthernet0/0/3
shutdown
exit
```



### Configuration des sous-interfaces pour les VLAN par trunk
```
enable
conf terminal
!
! Physical trunk interface
interface GigabitEthernet0/0
 description Trunk vers SW1
 no shutdown
!
! VLAN 2 - VoIP 192.168.2.0/24 GW 192.168.2.1
interface GigabitEthernet0/0.1
 encapsulation dot1q 2
 ip address 192.168.2.1 255.255.255.0
 no shutdown
!
! VLAN 10 - PC fixes 192.168.10.0/24
interface GigabitEthernet0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
! VLAN 20 - Wi-Fi 192.168.20.0/24
interface GigabitEthernet0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
!
! VLAN 30 - Administration 192.168.30.0/24
interface GigabitEthernet0/0.30
 encapsulation dot1q 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
end

2. Router 1941 – DHCP for each VLAN

Keep the DHCP ranges .10–.50 by excluding .1–.9.

​

text
conf t
! Excluded addresses
ip dhcp included-address 192.168.2.10 192.168.0.50
ip dhcp included-address 192.168.10.10 192.168.10.50
ip dhcp included-address 192.168.20.10 192.168.20.50
ip dhcp included-address 192.168.30.10 192.168.30.50
!
! VLAN 2 - VoIP
ip dhcp pool VLAN2-VOIP
 network 192.168.0.0 255.255.255.0
 default-router 192.168.2.1
! dns-server 8.8.8.8      ! optional
!
! VLAN 10 - PC fixes
ip dhcp pool VLAN10-PC
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
!
! VLAN 20 - Wi-Fi
ip dhcp pool VLAN20-WIFI
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
!
! VLAN 30 - Admin
ip dhcp pool VLAN30-ADMIN
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
!
end

3. Switches – trunk to the router, access ports for VLANs

On each switch, use one trunk uplink to the router and keep your VLANs already created. Example for the port toward the router (GigabitEthernet0/1):

​

text
conf t
! Uplink to router
interface GigabitEthernet0/1
 description Trunk vers routeur 1941
 switchport mode trunk
 switchport trunk allowed vlan 1,10,20,30
 no shutdown
!
! Example access ports
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
!
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
!
interface FastEthernet0/4
 switchport mode access
 switchport access vlan 30
end

Every switch that has a trunk to the router should have its uplink configured like this (same VLAN list), so all VLANs can reach the router and get DHCP via the subinterfaces.
​