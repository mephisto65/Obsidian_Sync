# NTLM

**Catégorie** : Concept
**Tags** : #concept #protocole #AD #authentification

---

## C'est quoi

NTLM (NT LAN Manager) est le protocole d'authentification legacy de Windows. Bien que [[Kerberos]] l'ait remplacé comme protocole par défaut dans Active Directory, NTLM reste omniprésent comme mécanisme de fallback. C'est le protocole le plus exploité en pentest interne car il est relayable, interceptable et craquable.

## Versions

| Version | Hash | Sécurité |
|---------|------|----------|
| LM (LAN Manager) | DES sur le mot de passe en majuscules, tronqué à 14 chars | Catastrophique, trivial à cracker |
| NTLMv1 | MD4(UTF-16LE(password)) + challenge DES | Faible, facilement craquable |
| NTLMv2 | HMAC-MD5 avec challenge client + serveur + timestamp | Mieux, mais toujours relayable |

En pratique sur un réseau moderne, on rencontre quasi exclusivement du **NTLMv2**.

## Le hash NTLM

Le hash NTLM est simplement `MD4(mot_de_passe_en_UTF-16LE)`. Il est stocké dans la SAM (comptes locaux) ou dans NTDS.dit (comptes AD). Ce hash sert directement à l'authentification — le mot de passe en clair n'est jamais nécessaire, ce qui rend le [[Pass-the-Hash]] possible.

```
Mot de passe : "Password123"
UTF-16LE :     P.a.s.s.w.o.r.d.1.2.3
Hash NTLM :    MD4(UTF-16LE) = a]4b...
```

## Flux d'authentification (challenge-response)

```
1. NEGOTIATE : Client → Serveur
   "Je veux m'authentifier via NTLM"

2. CHALLENGE : Serveur → Client
   "Voici un challenge aléatoire de 8 octets"

3. AUTHENTICATE : Client → Serveur
   "Voici ma réponse" (calculée avec mon hash NTLM + le challenge)
   + username, domain, hostname
```

Le serveur recalcule la réponse de son côté (il connaît le hash de l'utilisateur via la SAM ou en interrogeant le DC) et compare. Si ça matche, l'authentification réussit.

### Calcul de la réponse NTLMv2

```
NTLMv2 Hash = HMAC-MD5(NTLM_Hash, UPPER(username) + domain)
NTProofStr  = HMAC-MD5(NTLMv2_Hash, server_challenge + blob)
Response    = NTProofStr + blob
```

Le "blob" contient un timestamp, un challenge client aléatoire et des informations sur la cible.

## Format du hash NTLMv2 (capturé par Responder)

```
username::DOMAINE:server_challenge:NTProofStr:NTLMv2_response_sans_NTProofStr
```

Craquable avec :
```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

## Où NTLM est utilisé

- **SMB** : partages de fichiers, accès admin distant
- **HTTP** : authentification Windows Integrated (IWA), WPAD, Exchange/OWA
- **LDAP** : bind NTLM vers les DCs
- **WinRM** : administration à distance PowerShell
- **MSSQL** : connexion aux bases SQL Server
- **RDP** : dans certaines configurations (NLA désactivé)

## Pourquoi NTLM est encore présent

Kerberos nécessite que le client connaisse le FQDN du service et qu'un KDC soit joignable. NTLM prend le relais quand :

- Le client utilise une **IP** au lieu d'un hostname (`\\10.1.1.10` au lieu de `\\serveur`)
- Le serveur n'est **pas joint au domaine** (NAS QNAP, Samba, Linux)
- Le client est **hors réseau** (VPN non connecté, réseau externe)
- Le **DNS échoue** et les protocoles de fallback prennent le relais
- L'application ne supporte pas Kerberos

## Pourquoi c'est dangereux

### 1. Pass-the-Hash

Le hash NTLM suffit pour s'authentifier. Pas besoin du mot de passe en clair.

```bash
impacket-psexec -hashes :<NTLM_HASH> 'DOMAINE/admin'@<cible>
```

→ Voir [[Pass-the-Hash]]

### 2. Relay

L'authentification NTLM peut être relayée en temps réel vers une autre machine. L'attaquant n'a jamais besoin de connaître le hash — il transmet les messages entre la victime et la cible.

```bash
impacket-ntlmrelayx -tf targets.txt -smb2support
```

→ Voir [[NTLM Relay]]

### 3. Interception et crack

Les hashes NTLMv2 capturés sur le réseau (via [[Responder]], [[ARP Spoofing]]) sont craquables hors ligne.

```bash
hashcat -m 5600 hash.txt rockyou.txt
```

### 4. Pas d'authentification mutuelle

Contrairement à Kerberos, NTLM ne vérifie pas l'identité du serveur. Le client envoie son hash à n'importe qui prétendant être le serveur — c'est ce qui rend le poisoning et le relay possibles.

## Différence NTLM vs Kerberos

| | NTLM | [[Kerberos]] |
|---|---|---|
| Auth mutuelle | Non | Oui |
| Relayable | Oui | Non |
| Nécessite un DC | Non (peut être local) | Oui (KDC) |
| Fonctionne avec IP | Oui | Non (FQDN requis) |
| Tickets | Non | Oui (TGT, TGS) |
| Offline cracking | Hash NTLMv2 craquable | TGS/AS-REP craquable |
| Pass-the-Hash | Oui | Non (mais Pass-the-Ticket) |
| Sécurité | Faible | Fort (avec AES) |

## Où sont stockés les hashes

| Emplacement | Contient | Extraction |
|-------------|----------|------------|
| SAM | Hashes des comptes locaux | `secretsdump -sam SAM -system SYSTEM LOCAL` |
| LSASS | Hashes des sessions actives | `mimikatz sekurlsa::logonpasswords` |
| NTDS.dit | Tous les hashes du domaine AD | `secretsdump 'DA:pass'@DC` ([[DCSync]]) |
| Cache (mscash2) | Hashes des dernières connexions | `secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL` |

## Contre-mesures

- **Désactiver NTLM** autant que possible via GPO : `Network Security: Restrict NTLM: Incoming/Outgoing NTLM traffic`
- **Protected Users** : force Kerberos, bloque NTLM pour les membres du groupe
- **SMB Signing** requis : empêche le relay via SMB
- **LDAP Signing + Channel Binding** : empêche le relay via LDAP
- **Credential Guard** : isole LSASS, empêche l'extraction des hashes en mémoire
- **LAPS** : mots de passe admin locaux uniques → un hash ne fonctionne que sur une machine
- Désactiver LLMNR et NBT-NS pour empêcher la capture de hashes
- Surveiller Event ID 4624 avec `Authentication Package: NTLM` (devrait être rare si Kerberos est privilégié)

## Liens

- [[Kerberos]]
- [[Pass-the-Hash]]
- [[NTLM Relay]]
- [[LLMNR-NBT-NS Poisoning]]
- [[Responder]]
- [[DCSync]]
- [[ARP Spoofing]]
- [[Hashcat]]
