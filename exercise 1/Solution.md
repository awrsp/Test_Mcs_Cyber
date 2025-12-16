# Hardware


Premièrement, la mise en place du matériel physique dans la salle.


Ils ont été mis en place dans chaque bureau comme nous l'a demandé l'énoncé.


Le routeur a été placé dans la salle réseau pour faciliter le câblage.


J'ai ajouté le module HWIC-4EW pour rendre possible le branchement en RJ au switch dans chaque salle.


Un patch panel dans la salle reseau et des copper wall mount ont été ajouté pour faciliter les branchements.


3 salles ont été choisies comme bureau, puis les équipements ont été placés.


Les switchs ont reçu 8 modules PT-SWITCH-NM1CGE pour normaliser les ports. Nous n'avons pas pu mettre de ports fibres sur les switchs pour les reconnecter au routeur car le routeur n'est pas compatible avec 4 ports fibre; il n'a que disponible un module RJ45 a 4 ports.





# Software
##  Configuration des VLAN sur les switchs

~~premierement ils nous est demmande de creer 4 vlan~~


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


mais probleme il est generalement demmande de ne pas utiliser le vlan 1 


`Networkpulse FR`– [Ne laissez pas votre VLAN 1 être un risque : guide de sécurisation rapide](https://networkpulse.fr/ne-laissez-pas-votre-vlan-1-etre-un-risque-guide-de-securisation-rapide/)


`Netseccloud EN`– [What is VLAN 1 and How Does It Work?](https://netseccloud.com/what-is-vlan-1-and-how-does-it-work)

donc je prefere creer le vlan 2 pour le voip et laisser le vlan1 etre le vlan natif Et on va essayer de ne pas l'utiliser.

```
enable
configure terminal

! Création des VLAN
vlan 2
 name VOIP
vlan 10
 name WIFI
vlan 20
 name PC
vlan 30
 name ADMIN
exit
```

## Attribution des ports aux VLANs

```
enable
configure terminal

! Ports 2-3 : VLAN 2 (VoIP)
interface range GigabitEthernet0/2 - 3
 switchport mode access
 switchport access vlan 2
 spanning-tree portfast
exit

! Ports 4-5 : VLAN 10 (Wi-Fi)
interface range GigabitEthernet0/4 - 5
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

! Ports 6-7 : VLAN 20 (PC)
interface range GigabitEthernet0/6 - 7
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

! Port 8 : VLAN 30 (Administration)
interface GigabitEthernet0/8
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
```


## Configuration des trunks sur les switchs

```
enable
configure terminal
! Ports 1 and 9 : trunks (uplinks)
interface range GigabitEthernet0/1 , GigabitEthernet0/9
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1,10,20,30
exit

hostname SW1
end
write memory

```
Répéter la même configuration sur les autres switchs en adaptant le hostname.