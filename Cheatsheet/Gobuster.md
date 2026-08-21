# Gobuster

**Catégorie** : Web
**Phase** : Reconnaissance
**Tags** : #web #enumeration #bruteforce #directories

---

## Principe

Gobuster est un outil de brute-force écrit en Go qui permet d'énumérer des répertoires, fichiers, sous-domaines, vhosts et buckets S3 sur une cible. Il envoie des requêtes pour chaque mot d'une wordlist et analyse les codes de réponse HTTP pour identifier les ressources existantes.

## Modes

| Mode | Usage |
|---|---|
| `dir` | Répertoires et fichiers |
| `dns` | Sous-domaines |
| `vhost` | Virtual hosts |
| `fuzz` | Fuzzing générique (comme ffuf) |
| `s3` | Buckets Amazon S3 |

## Énumération de répertoires (mode dir)

### Basique

```bash
gobuster dir -u http://10.129.31.146 -w /usr/share/wordlists/dirb/common.txt
```

### Avec extensions

```bash
# Chercher des fichiers spécifiques
gobuster dir -u http://cible.com -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt,bak,old
```

### Contrôler les codes de réponse

```bash
# Exclure les 404 et 403
gobuster dir -u http://cible.com -w /usr/share/wordlists/dirb/common.txt \
  -b 404,403

# N'afficher que les 200
gobuster dir -u http://cible.com -w /usr/share/wordlists/dirb/common.txt \
  -s 200
```

### Récursif et avec threads

```bash
gobuster dir -u http://cible.com -w /usr/share/wordlists/dirb/common.txt \
  -t 50 \          # 50 threads
  -r \             # suivre les redirections
  --no-error       # masquer les erreurs de connexion
```

### Avec authentification

```bash
# Basic auth
gobuster dir -u http://cible.com -w wordlist.txt \
  -U admin -P password

# Cookie de session
gobuster dir -u http://cible.com -w wordlist.txt \
  -c "PHPSESSID=abc123"
```

## Énumération de sous-domaines (mode dns)

```bash
gobuster dns -d cible.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Afficher les IPs résolues
gobuster dns -d cible.com -w subdomains.txt -i
```

## Énumération de vhosts

```bash
gobuster vhost -u http://cible.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain
```

## Wordlists courantes

```
/usr/share/wordlists/dirb/common.txt            # ~4600 mots, rapide
/usr/share/wordlists/dirb/big.txt               # ~20000 mots
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  # ~220000 mots
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-words.txt
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## Options utiles

| Option | Description |
|---|---|
| `-t` | Nombre de threads (défaut 10) |
| `-x` | Extensions à tester |
| `-b` | Codes HTTP à exclure |
| `-s` | Codes HTTP à inclure |
| `-r` | Suivre les redirections |
| `-c` | Cookie à envoyer |
| `-H` | Header personnalisé |
| `-o` | Sauvegarder dans un fichier |
| `-k` | Ignorer les erreurs TLS |
| `-a` | User-Agent personnalisé |

## Liens

- [[Ffuf]]
