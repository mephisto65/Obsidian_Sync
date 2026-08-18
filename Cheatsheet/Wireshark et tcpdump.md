# Wireshark / tcpdump

Outils de capture et d'analyse de trafic réseau.

## tcpdump

### Capture de base

```bash
tcpdump -i eth0                                  # Tout le trafic sur eth0
tcpdump -i eth0 -A                               # Afficher le contenu ASCII
tcpdump -i eth0 -s 0                             # Capturer le paquet entier
tcpdump -i eth0 -w capture.pcap                  # Sauvegarder dans un fichier
tcpdump -r capture.pcap                          # Lire un fichier
```

### Filtres courants

```bash
tcpdump -i eth0 host 192.168.1.1                 # Trafic vers/depuis un hôte
tcpdump -i eth0 src 192.168.1.1                  # Depuis une source
tcpdump -i eth0 dst 192.168.1.1                  # Vers une destination
tcpdump -i eth0 port 80                          # Un port spécifique
tcpdump -i eth0 tcp                              # TCP uniquement
tcpdump -i eth0 udp                              # UDP uniquement
tcpdump -i eth0 not arp                          # Exclure ARP
tcpdump -i eth0 'port 3306 and not arp'          # Combiner des filtres
```

### Combos utiles

```bash
# Capture MITM entre deux hôtes
tcpdump -i eth0 -A -s 0 'host 10.0.0.2 and host 10.0.0.4 and not arp'

# Capturer les credentials en clair
tcpdump -i eth0 -A -s 0 'port 21 or port 23 or port 80 or port 110 or port 143'

# Chercher des mots-clés dans le trafic
tcpdump -i eth0 -A -s 0 | grep -i "pass\|user\|login"

# Capture MySQL
tcpdump -i eth0 -A -s 0 'port 3306 and not arp'
```

## Wireshark - Filtres d'affichage

### Filtres par protocole

```
http                                              # Trafic HTTP
dns                                               # Requêtes DNS
tcp                                               # Trafic TCP
udp                                               # Trafic UDP
arp                                               # Trafic ARP
ospf                                              # Trafic OSPF
smb2                                              # Trafic SMB
kerberos                                          # Trafic Kerberos
ldap                                              # Trafic LDAP
mysql                                             # Trafic MySQL
```

### Filtres par adresse

```
ip.addr == 192.168.1.1                            # Trafic vers/depuis
ip.src == 192.168.1.1                             # Source
ip.dst == 192.168.1.1                             # Destination
eth.addr == aa:bb:cc:dd:ee:ff                     # Adresse MAC
```

### Filtres par port

```
tcp.port == 80                                    # Port TCP
udp.port == 53                                    # Port UDP
tcp.dstport == 443                                # Port destination
```

### Filtres combinés

```
ip.addr == 192.168.1.1 && tcp.port == 80
http.request.method == "POST"
tcp.flags.syn == 1 && tcp.flags.ack == 0          # SYN sans ACK (scan)
frame contains "password"                         # Chercher un mot
```

### Analyse d'authentification

```
# OSPF auth
ospf                                              # Voir Auth Type et Auth Data

# HTTP credentials
http.authbasic                                    # Auth HTTP Basic
http.request.method == "POST"                     # Formulaires de login

# MySQL
mysql.command == 3                                # Requêtes SQL
```

## Wireshark - Raccourcis utiles

- `Ctrl+F` : chercher dans les paquets
- `Ctrl+Shift+E` : suivre un flux TCP (Follow TCP Stream)
- `Statistics > Protocol Hierarchy` : vue d'ensemble des protocoles
- `Statistics > Conversations` : voir les échanges entre hôtes
- `Statistics > Endpoints` : lister toutes les machines
