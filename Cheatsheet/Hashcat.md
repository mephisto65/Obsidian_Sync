# Hashcat

Outil de cracking de hash par GPU. Le plus rapide pour les attaques par dictionnaire et brute force.

## Syntaxe de base

```bash
hashcat -m <mode> -a <attaque> <hash_file> <wordlist>
```

## Modes de hash courants (-m)

| Mode | Type |
|------|------|
| 0 | MD5 |
| 100 | SHA1 |
| 1000 | NTLM |
| 1800 | sha512crypt (Linux /etc/shadow) |
| 3000 | LM |
| 3200 | bcrypt |
| 5500 | NetNTLMv1 |
| 5600 | NetNTLMv2 |
| 7500 | Kerberos 5 TGS (etype 23) |
| 13100 | Kerberos 5 TGS-REP (Kerberoasting) |
| 18200 | Kerberos 5 AS-REP (AS-REP Roasting) |
| 22000 | WPA-PBKDF2-PMKID+EAPOL |

## Types d'attaque (-a)

```bash
hashcat -a 0 ...                     # Dictionnaire
hashcat -a 1 ...                     # Combinaison (dict1 + dict2)
hashcat -a 3 ...                     # Brute force / masque
hashcat -a 6 ...                     # Dictionnaire + masque
hashcat -a 7 ...                     # Masque + dictionnaire
```

## Attaque par dictionnaire

```bash
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

## Attaque par masque (brute force)

```bash
# ?l = minuscule, ?u = majuscule, ?d = chiffre, ?s = spécial, ?a = tout
hashcat -m 1000 -a 3 hash.txt ?u?l?l?l?l?d?d?d       # Ex: Admin123
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a             # 6 caractères, tous types
```

## Attaque avec règles

```bash
hashcat -m 1000 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 1000 hash.txt rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule
```

## Exemples concrets

```bash
# Kerberoasting
hashcat -m 13100 tgs_hash.txt /usr/share/wordlists/rockyou.txt

# AS-REP Roasting
hashcat -m 18200 asrep_hash.txt /usr/share/wordlists/rockyou.txt

# NTLM (dump SAM, secretsdump)
hashcat -m 1000 ntlm_hash.txt /usr/share/wordlists/rockyou.txt

# NetNTLMv2 (Responder)
hashcat -m 5600 netntlm_hash.txt /usr/share/wordlists/rockyou.txt

# Hash Linux shadow
hashcat -m 1800 shadow_hash.txt /usr/share/wordlists/rockyou.txt
```

## Commandes utiles

```bash
hashcat --show hash.txt              # Afficher les résultats crackés
hashcat --identify hash.txt          # Identifier le type de hash
hashcat -O hash.txt                  # Mode optimisé (plus rapide)
hashcat --force hash.txt             # Forcer sur CPU si pas de GPU
```

## Wordlists courantes

```
/usr/share/wordlists/rockyou.txt
/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
/usr/share/seclists/Passwords/xato-net-10-million-passwords.txt
```
