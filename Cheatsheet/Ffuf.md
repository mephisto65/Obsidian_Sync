# Ffuf (Fuzz Faster U Fool)

**Catégorie** : Web
**Phase** : Reconnaissance / Exploitation
**Tags** : #web #fuzzing #enumeration #bruteforce

---

## Principe

Ffuf est un fuzzer web ultra-rapide écrit en Go. Il remplace le mot-clé `FUZZ` dans n'importe quelle partie de la requête (URL, headers, body, cookies) par chaque entrée d'une wordlist. Sa flexibilité le rend utile autant pour l'énumération de répertoires que pour le fuzzing de paramètres, sous-domaines ou valeurs.

## Énumération de répertoires

### Basique

```bash
ffuf -u http://cible.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

### Avec extensions

```bash
ffuf -u http://cible.com/FUZZ -w /usr/share/wordlists/dirb/common.txt \
  -e .php,.html,.txt,.bak
```

### Filtrage des résultats

```bash
# Filtrer par code de réponse (exclure 404)
ffuf -u http://cible.com/FUZZ -w wordlist.txt -fc 404

# Filtrer par taille de réponse (exclure les pages vides)
ffuf -u http://cible.com/FUZZ -w wordlist.txt -fs 0

# Filtrer par nombre de mots
ffuf -u http://cible.com/FUZZ -w wordlist.txt -fw 12

# Filtrer par nombre de lignes
ffuf -u http://cible.com/FUZZ -w wordlist.txt -fl 5

# Matcher uniquement certains codes
ffuf -u http://cible.com/FUZZ -w wordlist.txt -mc 200,301
```

### Auto-calibrage

```bash
# ffuf détecte et filtre automatiquement les réponses par défaut
ffuf -u http://cible.com/FUZZ -w wordlist.txt -ac
```

## Fuzzing de sous-domaines / vhosts

```bash
ffuf -u http://cible.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -H "Host: FUZZ.cible.com" \
  -fc 404
```

## Fuzzing de paramètres

### Paramètres GET

```bash
# Découvrir des paramètres cachés
ffuf -u "http://cible.com/page?FUZZ=test" -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -fc 404
```

### Paramètres POST

```bash
ffuf -u http://cible.com/login -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=FUZZ" \
  -w /usr/share/wordlists/rockyou.txt \
  -fc 401
```

## Fuzzing d'IDs (IDOR)

```bash
# Avec une séquence de nombres
ffuf -u http://cible.com/data/FUZZ -w <(seq 0 100)

# Avec un range
ffuf -u http://cible.com/api/user/FUZZ -w <(seq 1 1000) -fc 404,403
```

## Multi-fuzzing (plusieurs mots-clés)

```bash
# Deux wordlists simultanées (mode clusterbomb)
ffuf -u http://cible.com/FUZZ1/FUZZ2 \
  -w /usr/share/wordlists/dirb/common.txt:FUZZ1 \
  -w extensions.txt:FUZZ2 \
  -mode clusterbomb
```

## Avec authentification

```bash
# Cookie de session
ffuf -u http://cible.com/admin/FUZZ -w wordlist.txt \
  -b "session=abc123"

# Header custom
ffuf -u http://cible.com/api/FUZZ -w wordlist.txt \
  -H "Authorization: Bearer eyJ..."
```

## Récursif

```bash
ffuf -u http://cible.com/FUZZ -w wordlist.txt \
  -recursion -recursion-depth 2
```

## Sauvegarder les résultats

```bash
# JSON (le plus exploitable)
ffuf -u http://cible.com/FUZZ -w wordlist.txt -o results.json -of json

# CSV
ffuf -u http://cible.com/FUZZ -w wordlist.txt -o results.csv -of csv

# HTML
ffuf -u http://cible.com/FUZZ -w wordlist.txt -o results.html -of html
```

## Options utiles

| Option | Description |
|---|---|
| `-t` | Threads (défaut 40) |
| `-p` | Délai entre requêtes (ex: `0.1`) |
| `-rate` | Limite de requêtes/seconde |
| `-timeout` | Timeout par requête |
| `-mc` | Match codes HTTP |
| `-fc` | Filtre codes HTTP |
| `-ms` | Match taille réponse |
| `-fs` | Filtre taille réponse |
| `-ac` | Auto-calibrage |
| `-ic` | Ignorer les commentaires dans la wordlist |
| `-r` | Suivre les redirections |
| `-x` | Proxy (ex: `http://127.0.0.1:8080` pour Burp) |
| `-k` | Ignorer les erreurs TLS |

## Ffuf vs Gobuster

| | Ffuf | Gobuster |
|---|---|---|
| Flexibilité | Fuzz n'importe quoi (URL, headers, body) | Surtout dir/dns/vhost |
| Multi-wordlist | Oui (clusterbomb, pitchfork) | Non |
| Filtrage | Très granulaire (taille, mots, lignes, regex) | Basique (codes HTTP) |
| Vitesse | Comparable | Comparable |
| Cas d'usage idéal | Fuzzing complexe, IDOR, paramètres | Scan rapide de répertoires |

## Liens

- [[Gobuster]]
