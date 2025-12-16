Hardware
Premièrement, la mise en place du matériel physique dans la salle.
Ils ont été mis en place dans chaque bureau comme nous l'a demandé l'énoncé.
Le routeur a été placé dans la salle réseau pour faciliter le câblage.
J'ai ajouté le module HWIC-4EW pour rendre possible le branchement en RJ au switch dans chaque salle.
Un patch panel dans la salle reseau et des copper wall mount ont été ajouté pour faciliter les branchements.
3 salles ont été choisies comme bureau, puis les équipements ont été placés.
Les switchs ont reçu 8 modules PT-SWITCH-NM1CGE pour normaliser les ports. Nous n'avons pas pu mettre de ports fibres sur les switchs pour les reconnecter au routeur car le routeur n'est pas compatible avec 4 ports fibre; il n'a que disponible un module RJ45 a 4 ports.





Software
premierement ils nous est demmande de creer 4 vlan 

```
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
Cisco FR– Bonnes pratiques VLAN et sécurité (FR)

​
https://www.cisco.com/c/fr_ca/support/docs/smb/routers/cisco-rv-series-small-business-routers/1778-tz-VLAN-Best-Practices-and-Security-Tips.html

Article FR « Ne laissez pas votre VLAN 1 être un risque »

​
https://networkpulse.fr/ne-laissez-pas-votre-vlan-1-etre-un-risque-guide-de-securisation-rapide/

Vidéo FR « Pourquoi faut éviter d’utiliser Vlan 1 »
​

https://www.youtube.com/watch?v=vFOg2uaDnuc