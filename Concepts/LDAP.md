# LDAP

**Catégorie** : Protocole
**Tags** : #protocole #annuaire #AD

---

## C'est quoi

LDAP (Lightweight Directory Access Protocol) est un protocole d'accès à des annuaires. Un annuaire est une base de données optimisée pour la lecture, qui stocke des informations organisées de manière hiérarchique : utilisateurs, groupes, machines, certificats, etc.

Active Directory de Microsoft repose sur LDAP. OpenLDAP est l'implémentation open source la plus courante.

## Ports

| Port | Usage |
|------|-------|
| 389 | LDAP en clair |
| 636 | LDAPS (LDAP over TLS) |
| 3268 | Global Catalog |
| 3269 | Global Catalog over TLS |

## Structure

L'annuaire est organisé en arbre (DIT — Directory Information Tree). Chaque entrée est identifiée par un **DN** (Distinguished Name) :

```
CN=Jean Dupont,OU=Utilisateurs,DC=entreprise,DC=local
```

- **DC** (Domain Component) : composant du domaine
- **OU** (Organizational Unit) : unité organisationnelle (dossier)
- **CN** (Common Name) : nom de l'objet

## Requêtes LDAP

Une requête LDAP utilise des filtres entre parenthèses :

```
(uid=jean)                              # Égalité
(uid=j*)                                # Wildcard
(&(uid=jean)(mail=*@entreprise.fr))     # ET logique
(|(uid=jean)(uid=pierre))               # OU logique
(!(uid=admin))                          # Négation
```

## Opérations principales

- **Bind** : authentification auprès de l'annuaire
- **Search** : rechercher des entrées avec un filtre
- **Add** : ajouter une entrée
- **Modify** : modifier un attribut
- **Delete** : supprimer une entrée
- **Unbind** : fermer la connexion

## Attributs courants (Active Directory)

| Attribut | Description |
|----------|-------------|
| sAMAccountName | Nom de connexion |
| userPrincipalName | UPN (user@domain) |
| memberOf | Groupes d'appartenance |
| userAccountControl | Flags du compte (désactivé, pré-auth, etc.) |
| servicePrincipalName | SPN (lié au [[Kerberoasting]]) |
| adminCount | 1 si compte privilégié |

## Enumération LDAP en pentest

### Avec ldapsearch

```bash
# Énumérer tout le domaine
ldapsearch -x -H ldap://<DC_IP> -D "user@DOMAIN" -w "password" -b "DC=domain,DC=local"

# Chercher les utilisateurs
ldapsearch -x -H ldap://<DC_IP> -D "user@DOMAIN" -w "password" -b "DC=domain,DC=local" "(objectClass=user)" sAMAccountName

# Chercher les comptes avec SPN
ldapsearch -x -H ldap://<DC_IP> -D "user@DOMAIN" -w "password" -b "DC=domain,DC=local" "(servicePrincipalName=*)" sAMAccountName servicePrincipalName

# Chercher les Domain Admins
ldapsearch -x -H ldap://<DC_IP> -D "user@DOMAIN" -w "password" -b "CN=Domain Admins,CN=Users,DC=domain,DC=local" member
```

### Avec Nmap

```bash
nmap -p 389 --script ldap-search <DC_IP>
nmap -p 389 --script ldap-rootdse <DC_IP>
```

### Avec Python (ldap3)

```python
from ldap3 import Server, Connection, ALL

server = Server('<DC_IP>', get_info=ALL)
conn = Connection(server, 'user@DOMAIN', 'password', auto_bind=True)
conn.search('DC=domain,DC=local', '(objectClass=user)', attributes=['sAMAccountName', 'memberOf'])

for entry in conn.entries:
    print(entry)
```

## Lien avec Active Directory

Active Directory est essentiellement un annuaire LDAP avec des extensions Microsoft. Toute la gestion des utilisateurs, groupes, GPOs et permissions passe par LDAP. C'est pourquoi l'énumération LDAP est une étape clé dans un audit AD.

## Liens

- [[LDAP Injection]]
- [[Kerberos]]
- [[Kerberoasting]]
- [[AS-REP Roasting]]
