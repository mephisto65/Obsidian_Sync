# MITM (Man-in-the-Middle)

**Catégorie** : Concept
**Tags** : #concept #reseau #interception

---

## C'est quoi

Une attaque Man-in-the-Middle consiste à se placer entre deux parties qui communiquent, de manière à intercepter, lire et potentiellement modifier les échanges sans que les victimes ne s'en rendent compte. Chaque partie croit communiquer directement avec l'autre, alors que tout transite par l'attaquant.

## Positionnement

Pour se placer "au milieu", l'attaquant doit détourner le flux réseau vers sa machine. Plusieurs techniques existent selon la couche réseau ciblée :

### Couche 2 (Liaison)

| Technique | Principe | Note |
|-----------|----------|------|
| [[ARP Spoofing]] | Fausses réponses ARP pour associer sa MAC à l'IP d'une autre machine | Le plus courant sur un LAN |
| [[VLAN Hopping]] | Trames 802.1Q double-taguées pour atteindre un autre VLAN | Exploite une mauvaise config des ports switch |
| MAC Flooding | Saturer la table MAC du switch pour le forcer en mode hub (broadcast tout) | Bruyant, facilement détectable |

### Couche 3 (Réseau)

| Technique | Principe | Note |
|-----------|----------|------|
| [[DHCPv6 Spoofing]] | Faux serveur DHCPv6 → devenir le DNS de la victime | Très efficace car Windows préfère IPv6 |
| [[DHCP Spoofing]] | Faux serveur DHCP → fausse gateway et faux DNS | Bloqué par DHCP Snooping |
| ICMP Redirect | Paquet ICMP forgé disant à la victime de passer par l'attaquant | Rarement efficace en pratique |
| BGP Hijacking | Annonce de routes frauduleuses pour détourner du trafic Internet | Attaque à l'échelle d'un ISP |

### Couche 7 (Application)

| Technique | Principe | Note |
|-----------|----------|------|
| [[LLMNR-NBT-NS Poisoning]] | Réponse frauduleuse aux requêtes de résolution de noms broadcast | Capture de hashes NTLMv2 |
| [[WPAD Poisoning]] | Faux serveur proxy pour intercepter le trafic HTTP | Souvent combiné avec LLMNR |
| DNS Spoofing | Fausses réponses DNS pour rediriger vers un serveur contrôlé | Phishing, interception HTTPS (si pas de HSTS) |
| SSL Stripping | Downgrade HTTPS → HTTP pour intercepter en clair | Bloqué par HSTS |
| Rogue AP | Faux point d'accès Wi-Fi imitant un réseau légitime | Attaque sur les réseaux sans fil |

## Ce qu'on peut faire une fois positionné

**Interception passive** : lire le trafic sans le modifier. Tout protocole non chiffré est lisible : HTTP, FTP, Telnet, DNS, SMTP, requêtes SQL, credentials en clair, tokens, cookies.

**Interception active** : modifier le trafic en transit. Injection de code dans les pages HTTP, modification de réponses DNS, altération de fichiers téléchargés, injection de commandes dans des sessions Telnet/FTP.

**Relay** : ne pas lire le contenu mais réutiliser l'authentification. C'est le principe du [[NTLM Relay]] — l'attaquant relaie l'auth de la victime vers une autre machine pour s'y authentifier à sa place.

## Ce qui résiste au MITM

| Protection | Ce qu'elle protège |
|------------|-------------------|
| TLS/HTTPS | Chiffrement du contenu + vérification du certificat serveur |
| HSTS | Empêche le downgrade HTTPS → HTTP |
| SSH (avec vérification du fingerprint) | Chiffrement + authentification du serveur |
| Kerberos | Authentification mutuelle, pas de secret partagé sur le réseau |
| IPsec | Chiffrement + intégrité au niveau réseau |
| Certificate Pinning | Empêche l'acceptation de faux certificats |

Attention : même avec TLS, le MITM permet de voir les métadonnées (qui communique avec qui, quand, combien de données) — seul le contenu est protégé.

## Détection

- **arpwatch** : alerte sur les changements d'association IP-MAC
- **IDS/IPS** : détection de patterns ARP anormaux
- **802.1X** : empêche les machines non autorisées de se connecter
- **Dynamic ARP Inspection (DAI)** : valide les réponses ARP contre la table DHCP Snooping
- Surveillance des logs DNS pour les résolutions inhabituelles

## Liens

- [[ARP Spoofing]]
- [[NTLM Relay]]
- [[DHCPv6 Spoofing]]
- [[LLMNR-NBT-NS Poisoning]]
- [[VLAN Hopping]]
- [[Wireshark et tcpdump]]
