# Enumération DNS

**Catégorie** : Reconnaissance
**Phase** : Reconnaissance active
**Tags** : #recon #enumeration #dns

---

## Principe

Le DNS révèle la structure d'une organisation : sous-domaines, serveurs mail, noms internes, IP des services. C'est souvent la première étape après l'OSINT pour cartographier la surface d'attaque.

## Requêtes de base

```bash
# Résoudre un domaine
dig cible.com
nslookup cible.com

# Types d'enregistrements
dig cible.com A          # Adresses IPv4
dig cible.com AAAA       # Adresses IPv6
dig cible.com MX         # Serveurs mail
dig cible.com NS         # Serveurs DNS
dig cible.com TXT        # SPF, DKIM, vérifications
dig cible.com SOA        # Serveur autoritaire
dig cible.com CNAME      # Alias
dig cible.com ANY        # Tous les enregistrements (souvent bloqué)

# Reverse DNS
dig -x 10.1.1.21         # IP → nom
nslookup 10.1.1.21
```

## Transfert de zone (AXFR)

Si le serveur DNS est mal configuré, il peut autoriser un transfert de zone complet — toute la base DNS d'un coup.

```bash
# Tester le transfert de zone
dig axfr cible.com @ns1.cible.com

# Avec host
host -t axfr cible.com ns1.cible.com

# Avec nmap
nmap --script dns-zone-transfer -p 53 ns1.cible.com
```

Rarement possible sur Internet (bien protégé), mais plus fréquent en interne.

## Brute force de sous-domaines

```bash
# Avec amass
amass enum -brute -d cible.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Avec gobuster
gobuster dns -d cible.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Avec ffuf
ffuf -u http://FUZZ.cible.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -mc 200

# Avec dnsenum
dnsenum cible.com

# Avec dnsrecon
dnsrecon -d cible.com -t brt -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## Enumération DNS interne (Active Directory)

Le DNS AD contient les enregistrements SRV qui révèlent les contrôleurs de domaine, les services Kerberos, LDAP, etc.

```bash
# Trouver les DCs via SRV
dig _ldap._tcp.dc._msdcs.domaine.local SRV @<DC_IP>
dig _kerberos._tcp.domaine.local SRV @<DC_IP>
dig _gc._tcp.domaine.local SRV @<DC_IP>

# Lister les enregistrements du domaine AD
dig domaine.local ANY @<DC_IP>

# Avec nslookup
nslookup -type=SRV _ldap._tcp.dc._msdcs.domaine.local <DC_IP>

# Reverse DNS sur le réseau interne
dnsrecon -r 10.1.1.0/24 -n <DC_IP>
```

## Wildcard et takeover

```bash
# Tester si un wildcard DNS existe (tout sous-domaine répond)
dig nimportequoi.cible.com

# Subdomain takeover : vérifier si des CNAME pointent vers des services expirés
# Outils : subjack, nuclei
subjack -w subdomains.txt -t 100 -timeout 30
nuclei -l subdomains.txt -t takeovers/
```

## Wordlists utiles

```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt    # Rapide
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt   # Complet
/usr/share/seclists/Discovery/DNS/namelist.txt                       # Alternatif
/usr/share/amass/wordlists/all.txt                                   # Très complet
```

## Contre-mesures

- Désactiver les transferts de zone vers les IPs non autorisées
- Limiter les enregistrements DNS exposés publiquement
- Surveiller les sous-domaines abandonnés (subdomain takeover)
- Split DNS : DNS interne séparé du DNS externe
- Rate limiting sur les requêtes DNS pour limiter le brute force

## Liens

- [[OSINT]]
- [[Fingerprinting]]
- [[Nmap]]
