# John the Ripper

Outil de cracking de mots de passe. Alternative CPU à Hashcat, supporte beaucoup de formats.

## Syntaxe de base

```bash
john --wordlist=<wordlist> <hash_file>
john --format=<format> --wordlist=<wordlist> <hash_file>
```

## Identifier le format

```bash
john --list=formats                   # Lister tous les formats supportés
john --list=formats | grep -i krb     # Chercher un format spécifique
```

## Formats courants

| Format | Usage |
|--------|-------|
| raw-md5 | Hash MD5 |
| raw-sha1 | Hash SHA1 |
| nt | NTLM |
| lm | LM |
| netntlmv2 | NetNTLMv2 (Responder) |
| krb5tgs | Kerberoasting |
| krb5asrep | AS-REP Roasting |
| sha512crypt | Linux /etc/shadow ($6$) |
| bcrypt | Bcrypt ($2a$, $2b$) |
| zip | Archive ZIP protégée |
| rar | Archive RAR protégée |
| ssh | Clé SSH protégée |

## Exemples concrets

```bash
# Kerberoasting
john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# AS-REP Roasting
john --format=krb5asrep --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# NTLM
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Linux shadow
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt

# ZIP protégé
zip2john fichier.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# Clé SSH
ssh2john id_rsa > ssh_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt ssh_hash.txt
```

## Afficher les résultats

```bash
john --show hash.txt                  # Mots de passe crackés
john --show --format=krb5tgs hash.txt # Avec format spécifique
```

## Modes d'attaque

```bash
john hash.txt                         # Mode par défaut (single, wordlist, incremental)
john --wordlist=rockyou.txt hash.txt  # Dictionnaire
john --incremental hash.txt           # Brute force
john --rules --wordlist=rockyou.txt hash.txt  # Dictionnaire + règles de mutation
```

## Fichiers utiles

```bash
~/.john/john.pot                      # Fichier pot (résultats sauvegardés)
cat ~/.john/john.pot                  # Voir tous les hash crackés
```

## Extracteurs (xxx2john)

```bash
zip2john                              # ZIP
rar2john                              # RAR
ssh2john                              # Clés SSH
keepass2john                          # KeePass
office2john                           # Documents Office protégés
pdf2john                              # PDF protégés
```
