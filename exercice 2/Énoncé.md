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



# Script PowerShell Active Directory - Création utilisateurs laplateforme.io
# Exécuter en tant qu'Administrateur avec module ActiveDirectory installé
# Import-Module ActiveDirectory (si pas déjà chargé)

# Configuration
```
$Domain = "laplateforme.io"
$OU = "OU=Utilisateurs,DC=laplateforme,DC=io"  # Adapter selon votre OU
$Password = ConvertTo-SecureString "Azerty_2025!" -AsPlainText -Force
$UsersCSV = @"
Username,Mail,Groups,Mot de passe,Changer au 1er login
alexandre.marcelline@laplateforme.io,alexandre.marcelline@laplateforme.io,Animation,Azerty_2025!,Oui
aragon.isabelle@laplateforme.io,aragon.isabelle@laplateforme.io,Animation,Azerty_2025!,Oui
avaro.marina@laplateforme.io,avaro.marina@laplateforme.io,As|Médical,Azerty_2025!,Oui
bernard.isabelle@laplateforme.io,bernard.isabelle@laplateforme.io,As|Médical,Azerty_2025!,Oui
bouffier.stephane@laplateforme.io,bouffier.stephane@laplateforme.io,Hébergement|ASH,Azerty_2025!,Oui
bouziane.fatiha@laplateforme.io,bouziane.fatiha@laplateforme.io,Cadres|Médical|cadre de santé,Azerty_2025!,Oui
carlier.chantal@laplateforme.io,carlier.chantal@laplateforme.io,Comptable|Administratif,Azerty_2025!,Oui
thillot.marc@laplateforme.io,thillot.marc@laplateforme.io,Cadres|Directeur|Animation|Hébergement|Technique|Administratif,Azerty_2025!,Oui
gallien.carole@laplateforme.io,gallien.carole@laplateforme.io,Cadres|Hébergement|Maîtresse de Maison,Azerty_2025!,Oui
griveaux.patricia@laplateforme.io,griveaux.patricia@laplateforme.io,Cadres|Médecin|Médical,Azerty_2025!,Oui
larguier.silvania@laplateforme.io,larguier.silvania@laplateforme.io,Cadres|Psychologue|Médical,Azerty_2025!,Oui
malaure.ophylanadra@laplateforme.io,malaure.ophylanadra@laplateforme.io,Hébergement|ASH,Azerty_2025!,Oui
pratabuy.myriam@laplateforme.io,pratabuy.myriam@laplateforme.io,Secrétaire|Administratif,Azerty_2025!,Oui
sahtit.oifaa@laplateforme.io,sahtit.oifaa@laplateforme.io,IDE|Médical,Azerty_2025!,Oui
salvador.gladys@laplateforme.io,salvador.gladys@laplateforme.io,IDE|Médical,Azerty_2025!,Oui
schneider.emile@laplateforme.io,schneider.emile@laplateforme.io,Cadres|Technique,Azerty_2025!,Oui
vignolo.veronique@laplateforme.io,vignolo.veronique@laplateforme.io,Cadres|Animation|Hébergement|Administratif,Azerty_2025!,Oui
"@

# Traitement des utilisateurs
$Users = $UsersCSV | ConvertFrom-Csv
$Results = @()

foreach ($User in $Users) {
    $SamAccountName = ($User.Username -split '@')[0]
    $UPN = $User.Username
    $Groups = $User.Groups -split '\|' | Where-Object { $_ -ne "" }
    
    try {
        # Vérifier si utilisateur existe déjà
        if (Get-ADUser -Filter {SamAccountName -eq $SamAccountName} -ErrorAction SilentlyContinue) {
            $Result = [PSCustomObject]@{
                Username = $SamAccountName
                Status = "EXISTS"
                Error = "Utilisateur déjà présent"
            }
        } else {
            # Créer l'utilisateur
            $NewUserParams = @{
                Name = $SamAccountName
                SamAccountName = $SamAccountName
                UserPrincipalName = $UPN
                EmailAddress = $UPN
                AccountPassword = $Password
                Enabled = $true
                ChangePasswordAtLogon = $true
                Path = $OU
            }
            $NewUser = New-ADUser @NewUserParams
            
            # Ajouter aux groupes (créer si n'existent pas)
            foreach ($GroupName in $Groups) {
                if (!(Get-ADGroup -Filter {Name -eq $GroupName} -ErrorAction SilentlyContinue)) {
                    New-ADGroup -Name $GroupName -GroupScope Global -GroupCategory Security -Path $OU
                }
                Add-ADGroupMember -Identity $GroupName -Members $NewUser
            }
            
            $Result = [PSCustomObject]@{
                Username = $SamAccountName
                Status = "CREATED"
                Groups = ($Groups -join ', ')
                Error = ""
            }
        }
        $Results += $Result
        Write-Host "✓ $($Result.Status): $SamAccountName" -ForegroundColor Green
    }
    catch {
        $Results += [PSCustomObject]@{
            Username = $SamAccountName
            Status = "ERROR"
            Groups = ($Groups -join ', ')
            Error = $_.Exception.Message
        }
        Write-Host "✗ ERREUR: $SamAccountName - $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Résumé
Write-Host "`n=== RÉSUMÉ ===" -ForegroundColor Cyan
$Results | Format-Table -AutoSize
$Results | Export-Csv -Path "AD_Users_Creation_Report_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv" -NoTypeInformation -Encoding UTF8

Write-Host "`nRapport exporté. Tous les utilisateurs ont mot de passe 'Azerty_2025!' à changer au 1er login." -ForegroundColor Yellow
```