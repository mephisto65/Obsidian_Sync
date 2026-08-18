# OSINT (Open Source Intelligence)

**Catégorie** : Reconnaissance
**Phase** : Reconnaissance passive
**Tags** : #recon #osint #passive

---

## Principe

L'OSINT consiste à collecter des informations sur une cible à partir de sources publiques, sans interagir directement avec son infrastructure. C'est la première étape d'un pentest en boîte noire — tout ce qu'on trouve ici est ce qu'un attaquant voit avant même de lancer un scan.

## Sous-domaines

```bash
# Amass — le plus complet
amass enum -d cible.com -o subdomains.txt

# Subfinder — rapide
subfinder -d cible.com -o subdomains.txt

# Combiner les deux et dédupliquer
cat amass.txt subfinder.txt | sort -u > all_subs.txt

# Vérifier lesquels sont vivants
httpx -l all_subs.txt -o alive.txt
```

### Sources en ligne

- [crt.sh](https://crt.sh/?q=%25.cible.com) — certificats TLS (Certificate Transparency)
- [VirusTotal](https://www.virustotal.com) — sous-domaines passifs
- [SecurityTrails](https://securitytrails.com) — historique DNS
- [Shodan](https://shodan.io) — scan Internet (ports, bannières, versions)
- [Censys](https://censys.io) — même chose que Shodan

```bash
# crt.sh en ligne de commande
curl -s "https://crt.sh/?q=%25.cible.com&output=json" | jq -r '.[].name_value' | sort -u
```

## Emails et employés

```bash
# theHarvester — emails, noms, sous-domaines
theHarvester -d cible.com -b all

# Sources manuelles
# LinkedIn : chercher "cible.com" → profils des employés
# Hunter.io : pattern d'emails (prenom.nom@cible.com)
# Phonebook.cz : emails et sous-domaines
```

### Déduire le pattern d'emails

Si on trouve `fforest@cible.com` et `ydebray@cible.com`, le pattern est `[initiale][nom]@cible.com`. On peut ensuite générer une liste d'emails/usernames à partir de LinkedIn.

## Google Dorks

```
# Fichiers sensibles
site:cible.com filetype:pdf
site:cible.com filetype:xlsx
site:cible.com filetype:sql
site:cible.com filetype:env
site:cible.com filetype:bak

# Pages d'administration
site:cible.com inurl:admin
site:cible.com inurl:login
site:cible.com intitle:"index of"

# Informations exposées
site:cible.com "password" | "mot de passe"
site:cible.com "internal use only"
site:cible.com "confidential"

# Erreurs et debug
site:cible.com "error" | "warning" | "stack trace"
site:cible.com ext:log
```

## GitHub / GitLab

```bash
# Chercher des credentials dans les repos
# Outils : trufflehog, gitleaks, gitrob

trufflehog github --org=cible-org
gitleaks detect --source=https://github.com/cible/repo

# Recherche manuelle GitHub
# "cible.com" password
# "cible.com" api_key
# "cible.com" secret
# "cible.com" token
```

## Fuites de données

- [Have I Been Pwned](https://haveibeenpwned.com) — vérifier si des comptes sont dans des fuites
- [DeHashed](https://dehashed.com) — recherche dans les fuites (payant)
- [LeakCheck](https://leakcheck.io) — même chose
- Bases de données de combos sur les forums spécialisés

## Réseaux sociaux

- **LinkedIn** : organigramme, technologies utilisées, offres d'emploi (révèlent la stack technique)
- **Twitter/X** : communications informelles, parfois des infos techniques
- **GitHub perso** : code des employés, parfois avec des credentials hardcodés

## Métadonnées de fichiers

```bash
# Exiftool — extraire les métadonnées des fichiers publics (PDF, images, docs)
exiftool document.pdf

# Ce qu'on trouve : noms d'utilisateurs, logiciels utilisés, chemins internes, OS
# Télécharger les fichiers publics d'un site
wget -r -l 1 -A pdf,docx,xlsx https://cible.com
exiftool *.pdf *.docx 2>/dev/null | grep -i "author\|creator\|software\|producer"
```

## Shodan

```bash
# Recherche web
# org:"Nom de l'entreprise"
# hostname:cible.com
# ssl.cert.subject.cn:cible.com

# CLI
shodan search "hostname:cible.com" --fields ip_str,port,org,product,version
shodan host 1.2.3.4
```

## Workflow OSINT typique

1. **Sous-domaines** : amass + subfinder + crt.sh → liste complète
2. **Ports/services** : Shodan/Censys sur les IPs découvertes
3. **Emails** : theHarvester + Hunter.io + LinkedIn
4. **Pattern** : déduire le format des usernames/emails
5. **Fuites** : HIBP + DeHashed pour trouver des credentials existants
6. **Fichiers** : Google Dorks + métadonnées
7. **GitHub** : chercher des secrets dans les repos de l'organisation
8. **Stack technique** : offres d'emploi + Wappalyzer + whatweb

## Liens

- [[Fingerprinting]]
- [[Enumeration DNS]]
- [[Nmap]]
