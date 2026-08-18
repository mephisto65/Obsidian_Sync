# Fingerprinting

**Catégorie** : Réseau / Web
**Phase** : Reconnaissance
**Tags** : #recon #enumeration #osint

---

## Principe

Le fingerprinting consiste à identifier les technologies, versions et configurations d'une cible sans avoir besoin de s'authentifier. Ces informations permettent de chercher des CVE spécifiques, de cibler les exploits et de cartographier la surface d'attaque.

On distingue le fingerprinting **passif** (observation sans interaction) et **actif** (envoi de requêtes vers la cible).

## Fingerprinting réseau

### Détection d'OS

```bash
# Nmap OS detection (actif)
nmap -O <cible>

# Nmap agressif (OS + version + scripts)
nmap -A <cible>

# p0f (passif, écoute le trafic)
p0f -i eth0
```

Nmap déduit l'OS à partir du comportement de la pile TCP/IP : TTL par défaut, taille de fenêtre, options TCP, réponses aux paquets forgés.

| OS | TTL par défaut | Indice |
|-----|---------------|--------|
| Linux | 64 | |
| Windows | 128 | |
| Cisco/réseau | 255 | |

### Détection de services

```bash
# Version des services
nmap -sV <cible>

# Bannières en direct
nc -v <cible> <port>
echo "" | nc -v <cible> 21    # Bannière FTP
echo "" | nc -v <cible> 22    # Bannière SSH
```

### Protocoles réseau (passif)

```bash
# LLDP : modèle switch, firmware, port
tcpdump -i eth0 -nn ether proto 0x88cc

# VRRP : gateway virtuelle, priorité
tcpdump -i eth0 -nn proto 112

# SNMP : si community string connue
snmpwalk -v2c -c public <cible>
```

## Fingerprinting web

### Headers HTTP

```bash
# Version serveur web
curl -sI https://cible.com | grep -i "server\|x-powered-by\|x-aspnet"

# Headers de sécurité (CSP, HSTS, X-Frame-Options)
curl -sI https://cible.com | grep -iE "content-security|strict-transport|x-frame|x-xss"
```

### Technologies web

```bash
# whatweb : détection automatique (CMS, frameworks, plugins)
whatweb https://cible.com

# wappalyzer (extension navigateur) : même chose en graphique

# WordPress
wpscan --url https://cible.com --enumerate u,p,t

# Nuclei : fingerprinting + vulnérabilités
nuclei -u https://cible.com -t technologies/
```

### Assets versionés

Les fichiers statiques (JS, CSS) révèlent souvent la version exacte dans leur nom ou contenu :

```bash
# Chercher des versions dans le code source
curl -s https://cible.com | grep -oP '[\w-]+\.\d+\.\d+[\.\d]*\.(js|css)'

# Exemples trouvés
# /js/portalframe.10.2.2.1-90sv.js → SonicWall SMA version exacte
# /elementor/assets/js/swiper.8.4.5.min.js → Swiper.js vulnérable
```

### Endpoints d'information

```bash
# WordPress
curl https://cible.com/wp-json/wp/v2/users          # Utilisateurs
curl https://cible.com/readme.html                    # Version WP

# FileMaker
curl https://cible.com/fmi/data/v1/databases          # Liste des BDD

# API génériques
curl https://cible.com/api/version
curl https://cible.com/server-info
curl https://cible.com/server-status
```

## Fingerprinting Active Directory

```bash
# Enumération LDAP anonyme
ldapsearch -x -H ldap://<DC_IP> -s base namingContexts

# Version du DC
crackmapexec smb <DC_IP>

# Enumération d'utilisateurs (Kerbrute)
kerbrute userenum -d domaine.local --dc <DC_IP> users.txt

# BloodHound : cartographie complète
bloodhound-python -d domaine.local -u user -p pass -c All -ns <DC_IP>
```

## Fingerprinting SSL/TLS

```bash
# Certificats (révèle noms internes, sous-domaines, CA)
openssl s_client -connect cible.com:443 </dev/null 2>/dev/null | openssl x509 -text

# Scan TLS complet
sslscan cible.com
testssl.sh cible.com

# Nmap
nmap --script ssl-enum-ciphers -p 443 cible.com
```

## Pourquoi c'est critique

Chaque version identifiée est une porte vers des CVE. Le fingerprinting transforme une cible opaque en une liste de composants avec des vulnérabilités connues. C'est la base de toute la phase de reconnaissance.

## Contre-mesures

- Masquer les versions dans les headers : `server_tokens off` (nginx), `ServerTokens Prod` (Apache)
- Retirer `X-Powered-By` et autres headers informatifs
- Ne pas versionner les fichiers statiques avec la version du produit (utiliser un hash de contenu)
- Restreindre les endpoints d'information (REST API, server-status)
- Désactiver LLDP sur les ports access
- Filtrer les bannières des services (SSH, FTP, SMTP)

## Liens

- [[Nmap]]
- [[LDAP]]
- [[Kerberos]]
