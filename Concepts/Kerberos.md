# Kerberos

**Catégorie** : Concept
**Tags** : #concept #protocole #AD #authentification

---

## C'est quoi

Kerberos est le protocole d'authentification par défaut dans Active Directory. Il repose sur un système de tickets délivrés par un tiers de confiance (le KDC — Key Distribution Center, hébergé sur le contrôleur de domaine) pour prouver l'identité des utilisateurs sans faire transiter de mot de passe sur le réseau.

Le nom vient de Cerbère, le chien à trois têtes de la mythologie grecque — les trois composants étant le client, le serveur et le KDC.

## Port

- **88/TCP et 88/UDP** : Kerberos
- **464/TCP** : kpasswd (changement de mot de passe)

## Les trois acteurs

| Composant | Rôle |
|-----------|------|
| **Client** | L'utilisateur ou la machine qui veut accéder à un service |
| **KDC** (Key Distribution Center) | Le contrôleur de domaine, composé de l'AS et du TGS |
| **Service** | La ressource cible (partage SMB, serveur web, base de données...) |

Le KDC contient deux sous-services :
- **AS** (Authentication Service) : vérifie l'identité du client et délivre le TGT
- **TGS** (Ticket Granting Service) : délivre les tickets de service à partir du TGT

## Les tickets

| Ticket | Délivré par | Chiffré avec | Fonction |
|--------|-------------|-------------|----------|
| **TGT** (Ticket Granting Ticket) | AS | Hash du compte **krbtgt** | Prouve l'identité du client auprès du TGS |
| **TGS** (Ticket de service) | TGS | Hash du **compte de service** (SPN) | Prouve l'identité du client auprès du service |

## Flux d'authentification

```
1. AS-REQ : Client → KDC (AS)
   "Je suis user@domaine, voici un timestamp chiffré avec mon hash"
   (c'est la pré-authentification)

2. AS-REP : KDC (AS) → Client
   "Voici ton TGT" (chiffré avec le hash krbtgt)
   + clé de session (chiffrée avec le hash du client)

3. TGS-REQ : Client → KDC (TGS)
   "Je veux accéder au service X, voici mon TGT"

4. TGS-REP : KDC (TGS) → Client
   "Voici ton ticket de service" (chiffré avec le hash du compte de service)

5. AP-REQ : Client → Service
   "Voici mon ticket de service"

6. AP-REP : Service → Client (optionnel)
   "Authentification mutuelle confirmée"
```

## Concepts clés

### Pré-authentification

Le client prouve son identité en chiffrant un timestamp avec son hash NTLM. Si cette vérification est désactivée sur un compte ("Do not require Kerberos preauthentication"), le KDC renvoie l'AS-REP sans preuve → attaque [[AS-REP Roasting]].

### SPN (Service Principal Name)

Identifiant unique associé à un compte de service. Format : `service/hostname` (ex: `HTTP/web.domaine.local`). Tout utilisateur authentifié peut demander un TGS pour n'importe quel SPN → attaque [[Kerberoasting]].

### PAC (Privilege Attribute Certificate)

Structure embarquée dans chaque ticket contenant les informations d'autorisation : SID de l'utilisateur, groupes d'appartenance, sIDHistory. Le service lit le PAC pour déterminer les droits d'accès. Le KDC ne revalide pas le PAC — il fait confiance à la signature.

### Délégation

Permet à un service de s'authentifier au nom d'un utilisateur vers un autre service :
- **Non contrainte** : le service peut impersonner l'utilisateur vers n'importe quel autre service
- **Contrainte (classique)** : limitée à des services spécifiques, configurée par un admin
- **RBCD** (Resource-Based Constrained Delegation) : configurée sur l'objet cible lui-même → attaque [[RBCD]]

### S4U (Service for User)

Extensions Kerberos pour la délégation :
- **S4U2Self** : un service obtient un ticket au nom d'un utilisateur vers lui-même
- **S4U2Proxy** : un service utilise ce ticket pour obtenir un ticket vers un autre service au nom de l'utilisateur

## Types de chiffrement (etype)

| etype | Algorithme | Sécurité |
|-------|-----------|----------|
| 23 | RC4-HMAC (basé sur le hash NTLM) | Faible, rapide à cracker |
| 17 | AES128-CTS-HMAC-SHA1 | Correct |
| 18 | AES256-CTS-HMAC-SHA1 | Fort, recommandé |

RC4 (etype 23) est le plus ciblé par les attaques car il est beaucoup plus rapide à brute-forcer que AES.

## Attaques liées

| Attaque | Ce qu'elle exploite |
|---------|-------------------|
| [[Kerberoasting]] | TGS chiffré avec le hash du compte de service → crack offline |
| [[AS-REP Roasting]] | AS-REP sans pré-auth → crack offline |
| [[Golden Ticket]] | Hash krbtgt volé → forge de TGT arbitraires |
| Silver Ticket | Hash du compte de service → forge de TGS sans passer par le KDC |
| [[RBCD]] | Délégation contrainte basée sur les ressources → impersonation |
| [[DCSync]] | Droits de réplication → dump de tous les hashes y compris krbtgt |
| Kerberos Delegation Abuse | Délégation non contrainte → vol de TGT en mémoire |
| [[Shadow Credentials]] | PKINIT + msDS-KeyCredentialLink → auth par certificat persistante |

## Logs importants

| Event ID | Description |
|----------|-------------|
| 4768 | Requête TGT (AS-REQ) |
| 4769 | Requête TGS (TGS-REQ) — surveiller les demandes massives (Kerberoasting) |
| 4771 | Échec de pré-authentification |
| 4770 | Renouvellement de TGT |

## Contre-mesures globales

- Forcer AES (désactiver RC4 au niveau du domaine)
- Mots de passe longs (25+) sur les comptes de service ou utiliser des gMSA
- Ne jamais désactiver la pré-authentification
- Rotation régulière du mot de passe krbtgt
- Protected Users pour les comptes privilégiés
- Monitorer les Event ID 4769 pour les demandes massives de TGS

## Liens

- [[Kerberoasting]]
- [[AS-REP Roasting]]
- [[Golden Ticket]]
- [[Shadow Credentials]]
- [[RBCD]]
- [[DCSync]]
- [[NTLM]]
