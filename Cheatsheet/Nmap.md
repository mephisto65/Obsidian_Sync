# Nmap

Scanner de ports et de services. Outil de base pour toute phase de reconnaissance.

## Scan de découverte (host alive)

```bash
nmap -sn 192.168.1.0/24              # Ping sweep, pas de scan de ports
nmap -sn -PE 192.168.1.0/24          # ICMP echo uniquement
nmap -Pn 192.168.1.1                 # Skip la découverte, scan direct
```

## Scan de ports

```bash
nmap 192.168.1.1                     # Top 1000 ports TCP
nmap -p 80,443,8080 192.168.1.1      # Ports spécifiques
nmap -p 1-65535 192.168.1.1          # Tous les ports
nmap -p- 192.168.1.1                 # Raccourci pour tous les ports
nmap --top-ports 100 192.168.1.1     # Top 100 ports les plus courants
```

## Types de scan

```bash
nmap -sT 192.168.1.1                 # TCP connect (pas besoin de root)
nmap -sS 192.168.1.1                 # SYN scan (par défaut, root requis)
nmap -sU 192.168.1.1                 # UDP scan (lent)
nmap -sV 192.168.1.1                 # Détection de version des services
nmap -O 192.168.1.1                  # Détection d'OS
nmap -A 192.168.1.1                  # Agressif : OS + version + scripts + traceroute
```

## Scripts NSE

```bash
nmap --script=default 192.168.1.1                  # Scripts par défaut
nmap --script=vuln 192.168.1.1                     # Scripts de vulnérabilités
nmap --script=smb-enum-shares 192.168.1.1          # Enumération SMB
nmap --script=ldap-search 192.168.1.1              # Enumération LDAP
nmap --script=dns-brute example.com                # Brute force DNS
nmap --script="http-*" 192.168.1.1                 # Tous les scripts HTTP
```

## Timing et performance

```bash
nmap -T0 192.168.1.1                 # Paranoïaque (très lent, discret)
nmap -T1 192.168.1.1                 # Sneaky
nmap -T2 192.168.1.1                 # Polite
nmap -T3 192.168.1.1                 # Normal (par défaut)
nmap -T4 192.168.1.1                 # Agressif (recommandé en CTF)
nmap -T5 192.168.1.1                 # Insane (peut rater des ports)
```

## Output

```bash
nmap -oN scan.txt 192.168.1.1       # Format normal
nmap -oX scan.xml 192.168.1.1       # Format XML
nmap -oG scan.grep 192.168.1.1      # Format greppable
nmap -oA scan 192.168.1.1           # Les 3 formats d'un coup
```

## Combos utiles

```bash
# Scan complet rapide
nmap -sC -sV -O -T4 -p- 192.168.1.1

# Scan discret
nmap -sS -T2 -f --data-length 50 192.168.1.1

# Scan UDP rapide (top ports)
nmap -sU --top-ports 50 -T4 192.168.1.1

# Enumération AD
nmap -p 88,135,139,389,445,636,3268 -sV 192.168.1.1
```
