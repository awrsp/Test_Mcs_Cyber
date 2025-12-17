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
exit
write memory
exit
```

## Attribution des ports aux VLAN


# Ports 2-3 : VLAN 2 (VoIP)

```
enable
configure terminal
interface range GigabitEthernet1/1 , GigabitEthernet2/1
 switchport mode access
 switchport access vlan 2
 spanning-tree portfast
exit
```


# Ports 4-5 : VLAN 10 (Wi-Fi)
```
enable
configure terminal
interface range GigabitEthernet3/1 , GigabitEthernet4/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
```


# Ports 6-7 : VLAN 20 (PC)
```
enable
configure terminal
interface range GigabitEthernet5/1 , GigabitEthernet6/1
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit
```


# Port 8 : VLAN 30 (Administration)
```
enable
configure terminal
interface GigabitEthernet7/1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
```


## Configuration des trunks sur les commutateurs

```
enable
configure terminal
! Ports 1 et 9 : trunks (uplinks)
interface range GigabitEthernet0/1 , GigabitEthernet0/9
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 2,10,20,30
exit

hostname SW1
end
write memory

```
Répétez la même configuration sur les autres commutateurs en adaptant le `hostname`.