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
# Script PowerShell AD COMPLET - Crée OU + Utilisateurs + Groupes
# Exécuter en Admin domaine | Import-Module ActiveDirectory

# Configuration automatique
$Domain = (Get-ADDomain).DNSRoot
$BaseDN = (Get-ADDomain).DistinguishedName
$OUPath = "OU=Utilisateurs,$BaseDN"

Write-Host "Domaine détecté: $Domain" -ForegroundColor Green
Write-Host "OU cible: $OUPath" -ForegroundColor Yellow

# 1. Créer OU si manquante
if (!(Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$OUPath'" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit -Name "Utilisateurs" -Path $BaseDN
    Write-Host "✓ OU 'Utilisateurs' créée" -ForegroundColor Green
} else {
    Write-Host "✓ OU existe déjà" -ForegroundColor Green
}

# 2. Mot de passe commun
$Password = ConvertTo-SecureString "Azerty_2025!" -AsPlainText -Force

# 3. Liste utilisateurs (format simplifié)
$Users = @(
    @{Sam="alexandre.marcelline"; Groups=@("Animation")}
    @{Sam="aragon.isabelle"; Groups=@("Animation")}
    @{Sam="avaro.marina"; Groups=@("As","Médical")}
    @{Sam="bernard.isabelle"; Groups=@("As","Médical")}
    @{Sam="bouffier.stephane"; Groups=@("ASH","Hébergement")}
    @{Sam="bouziane.fatiha"; Groups=@("cadre de santé","Cadres","Médical")}
    @{Sam="carlier.chantal"; Groups=@("Comptable","Administratif")}
    @{Sam="thillot.marc"; Groups=@("Directeur","Cadres","Hébergement","Technique","Administratif","Animation")}
    @{Sam="gallien.carole"; Groups=@("Maîtresse de Maison","Cadres","Hébergement")}
    @{Sam="griveaux.patricia"; Groups=@("Médecin","Cadres","Médical")}
    @{Sam="larguier.silvania"; Groups=@("Psychologue","Cadres","Médical")}
    @{Sam="malaure.ophylanadra"; Groups=@("ASH","Hébergement")}
    @{Sam="pratabuy.myriam"; Groups=@("Secrétaire","Administratif")}
    @{Sam="sahtit.oifaa"; Groups=@("IDE","Médical")}
    @{Sam="salvador.gladys"; Groups=@("IDE","Médical")}
    @{Sam="schneider.emile"; Groups=@("Technique","Cadres")}
    @{Sam="vignolo.veronique"; Groups=@("Animation","Cadres","Hébergement","Administratif")}
)

# 4. Création utilisateurs + groupes
$Results = @()
foreach ($User in $Users) {
    $Sam = $User.Sam
    $UPN = "$Sam@$Domain"
    
    try {
        # Skip si existe
        if (Get-ADUser -Filter "SamAccountName -eq '$Sam'" -ErrorAction SilentlyContinue) {
            $Status = "EXISTS"
        } else {
            # Créer utilisateur
            New-ADUser -Name $Sam -SamAccountName $Sam -UserPrincipalName $UPN -EmailAddress $UPN -AccountPassword $Password -Enabled $true -ChangePasswordAtLogon $true -Path $OUPath -ErrorAction Stop
            
            # Créer/ajouter groupes
            foreach ($Group in $User.Groups) {
                if (!(Get-ADGroup -Filter "Name -eq '$Group'" -ErrorAction SilentlyContinue)) {
                    New-ADGroup -Name $Group -GroupScope Global -GroupCategory Security -Path $OUPath
                }
                Add-ADGroupMember -Identity $Group -Members $Sam -ErrorAction SilentlyContinue
            }
            $Status = "CREATED"
        }
        
        $Results += [PSCustomObject]@{User=$Sam; Status=$Status; Groups=($User.Groups -join ",")}
        Write-Host "✓ $Status`: $Sam" -ForegroundColor Green
    }
    catch {
        $Results += [PSCustomObject]@{User=$Sam; Status="ERROR"; Groups=($User.Groups -join ","); Error=$_.Exception.Message}
        Write-Host "✗ $Sam`: $($_.Exception.Message)" -ForegroundColor Red
    }
}

# 5. Rapport
$Results | Format-Table -AutoSize
$Results | Export-Csv "AD_Creation_Report_$(Get-Date -f 'yyyyMMdd-HHmmss').csv" -NoTypeInformation -Encoding UTF8

Write-Host "`n🎉 TERMINÉ! Rapport exporté. Mot de passe: Azerty_2025! (changer au 1er login)" -ForegroundColor Cyan

```