# Enumération SMB

**Catégorie** : Reconnaissance
**Phase** : Reconnaissance active
**Tags** : #recon #enumeration #smb #windows

---

## Principe

SMB (Server Message Block) est le protocole de partage de fichiers et d'imprimantes de Windows, actif sur les ports 139 et 445. C'est une mine d'or en pentest interne : il expose des partages, des utilisateurs, des sessions, et son signing (ou son absence) détermine si le relay NTLM est possible.

## Découverte

```bash
# Scanner les machines avec SMB ouvert
nmap -p 445 --open 10.1.1.0/24

# Version SMB + OS + signing en un coup
crackmapexec smb 10.1.1.0/24

# Machines sans SMB signing (relayables)
crackmapexec smb 10.1.1.0/24 --gen-relay-list relayable.txt
```

## Enumération anonyme (null session)

```bash
# Tester si la null session est possible
crackmapexec smb <cible> -u '' -p '' --shares

# Avec smbclient
smbclient -N -L //<cible>

# Avec rpcclient
rpcclient -U '' -N <cible>
rpcclient> enumdomusers         # Lister les utilisateurs
rpcclient> enumdomgroups        # Lister les groupes
rpcclient> querydominfo         # Info sur le domaine
rpcclient> querydispinfo        # Infos détaillées des utilisateurs

# Avec enum4linux
enum4linux -a <cible>

# RID cycling (énumérer les utilisateurs même si null session bloquée partiellement)
crackmapexec smb <cible> -u '' -p '' --rid-brute
```

## Enumération authentifiée

```bash
# Partages
crackmapexec smb <cible> -u user -p pass --shares
smbclient -U 'DOMAINE\user%pass' -L //<cible>

# Utilisateurs du domaine
crackmapexec smb <cible> -u user -p pass --users

# Groupes
crackmapexec smb <cible> -u user -p pass --groups

# Sessions actives (qui est connecté où)
crackmapexec smb <cible> -u user -p pass --sessions

# Politique de mots de passe (pour calibrer le spray)
crackmapexec smb <cible> -u user -p pass --pass-pol

# Spider les shares (lister récursivement le contenu)
crackmapexec smb <cible> -u user -p pass -M spider_plus
```

## Accéder aux partages

```bash
# Lister le contenu d'un share
smbclient -U 'DOMAINE\user%pass' //<cible>/sharename
smb: \> ls
smb: \> cd dossier
smb: \> get fichier.txt
smb: \> put fichier_local.txt

# Monter un share
mount -t cifs -o username=user,password=pass //<cible>/share /mnt/share
```

## Ce qu'on cherche

- **Partages accessibles en lecture** : scripts, configs, backups, mots de passe en clair
- **SYSVOL/NETLOGON** : scripts GPO, parfois des credentials en clair (GPP passwords)
- **Partages d'installation** : images, ISO, configs avec des mots de passe par défaut
- **SMB Signing désactivé** : machines relayables → [[NTLM Relay]]
- **Politique de mots de passe** : seuil de lockout, complexité → calibrer le password spray

## Fichiers intéressants à chercher

```bash
# Dans les shares, chercher :
*.xml          # GPP passwords (cpassword), configs
*.ini          # Configurations
*.txt          # Notes, TODO, passwords.txt
*.bat / *.ps1  # Scripts avec credentials hardcodés
*.config       # Web.config, app.config
unattend.xml   # Mots de passe d'installation Windows
```

## Contre-mesures

- Désactiver les null sessions : `restrict anonymous = 2`
- Activer SMB Signing requis sur toutes les machines
- Restreindre les partages réseau au minimum nécessaire
- Auditer régulièrement le contenu des partages accessibles
- Ne jamais stocker de credentials dans des scripts ou fichiers partagés
- Supprimer les GPP passwords (KB2962486)

## Liens

- [[NTLM Relay]]
- [[CrackMapExec]]
- [[Nmap]]
