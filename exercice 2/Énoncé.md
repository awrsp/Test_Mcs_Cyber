# Exercice 02 : Active Directory

Sur un Windows Serveur 20xx fraîchement installé, vous devez en Powershell :
- Créer l'AD "laplateforme.io"
- Peupler l'AD depuis le fichier csv suivant : (un utilisateur peut appartenir à plusieurs groupes) :
```csv
nom,prénom,groupe1,groupe2,groupe3,groupe4,groupe5,groupe6
ALEXANDRE,MARCELLINE,Animation,,,,,
ARAGON,ISABELLE,Animation,,,,,
AVARO,MARINA,As,Médical,,,,
BERNARD,ISABELLE,As,Médical,,,,
BOUFFIER,STEPHANE,ASH,Hébergement,,,,
BOUZIANE,FATIHA,cadre de santé,Cadres,Médical,,,
CARLIER,CHANTAL,Comptable,Administratif,,,,
THILLOT,MARC,Directeur,Cadres,Hébergement,Technique,Administratif,Animation
GALLIEN,CAROLE,Maîtresse de Maison,Cadres,Hébergement,,,
GRIVEAUX,PATRICIA,Médecin,Cadres,Médical,,,
LARGUIER,SILVANIA,Psychologue,Cadres,Médical,,,
MALAURE,OPHYLANADRA,ASH,Hébergement,,,,
PRATABUY,MYRIAM,Secrétaire,Administratif,,,,
SAHTIT,OIFAA,IDE,Médical,,,,
SALVADOR,GLADYS,IDE,Médical,,,,
SCHNEIDER,EMILE,Technique,Cadres,,,,
VIGNOLO,VERONIQUE,Animation,Cadres,Hébergement,Administratif,,
```

- Définir comme mot de passe "Azerty_2025!" à tous les utilisateurs et forcer le changement à la première connexion

Mettre dans votre Github :
- un fichier Readme.md pour expliquer votre process 
- les scripts Powershell