# BloodHound

Outil de cartographie Active Directory. Visualise les chemins d'attaque entre n'importe quel compte et les groupes à privilèges (Domain Admins, Enterprise Admins, etc.).

## Collecte de données

### Avec bloodhound-python (depuis Linux)

```bash
# Collecte complète
bloodhound-python -d domaine.local -u user -p 'password' -c All -ns <DC_IP>

# Collecte ciblée
bloodhound-python -d domaine.local -u user -p 'password' -c DCOnly -ns <DC_IP>

# Avec hash (PtH)
bloodhound-python -d domaine.local -u user --hashes :<NTLM> -c All -ns <DC_IP>
```

### Avec SharpHound (depuis Windows)

```powershell
# Collecte complète
.\SharpHound.exe -c All

# Collecte discrète (pas de scan de sessions)
.\SharpHound.exe -c DCOnly

# Spécifier le DC
.\SharpHound.exe -c All --DomainController <DC_IP>
```

### Résultat

Fichiers JSON compressés dans un ZIP → à importer dans l'interface BloodHound.

## Lancement de BloodHound

```bash
# Démarrer la base Neo4j
sudo neo4j console

# Lancer BloodHound (credentials par défaut : neo4j/neo4j)
bloodhound
```

Importer le ZIP via le bouton "Upload Data" en haut à droite.

## Requêtes pré-construites (onglet Analysis)

Les plus utiles pour un pentest :

- **Find all Domain Admins** : visualiser les membres DA
- **Find Shortest Paths to Domain Admins** : chemins d'attaque les plus courts
- **Find Principals with DCSync Rights** : qui peut faire un DCSync
- **Find Computers where Domain Users are Local Admin** : machines vulnérables
- **Find Kerberoastable Users** : comptes avec SPN
- **Find AS-REP Roastable Users** : comptes sans pré-auth
- **Find Computers with Unconstrained Delegation** : délégation non contrainte
- **Shortest Paths from Owned Principals** : depuis les comptes déjà compromis

## Requêtes Cypher personnalisées

Accessibles via l'icône "Raw Query" en bas de l'interface.

### Chemins d'attaque

```cypher
// Chemin le plus court de user vers Domain Admins
MATCH p=shortestPath((u:User {name:"USER@DOMAINE.LOCAL"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAINE.LOCAL"}))
RETURN p

// Tous les chemins vers DA (pas que le plus court)
MATCH p=allShortestPaths((u:User)-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAINE.LOCAL"}))
RETURN p

// Chemins depuis les "owned" (comptes compromis)
MATCH p=shortestPath((u {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAINE.LOCAL"}))
RETURN p
```

### Enumération

```cypher
// Comptes Kerberoastable avec chemin vers DA
MATCH (u:User {hasspn:true}) 
MATCH p=shortestPath((u)-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAINE.LOCAL"}))
RETURN p

// Comptes AS-REP Roastable
MATCH (u:User {dontreqpreauth:true}) RETURN u.name

// Comptes avec mot de passe qui n'expire jamais
MATCH (u:User {pwdneverexpires:true}) RETURN u.name

// Machines avec unconstrained delegation
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c.name

// Utilisateurs avec droits DCSync
MATCH p=(u)-[:GetChanges|GetChangesAll*1..]->(d:Domain) RETURN p

// Admins locaux sur les machines
MATCH p=(u:User)-[:AdminTo]->(c:Computer) RETURN u.name, c.name
```

### ACL dangereuses

```cypher
// Qui a GenericAll sur des objets sensibles
MATCH p=(u)-[:GenericAll]->(t) WHERE t:User OR t:Group OR t:Computer RETURN p

// Qui peut écrire dans msDS-KeyCredentialLink (Shadow Credentials)
MATCH p=(u)-[:AddKeyCredentialLink]->(t) RETURN p

// Qui peut modifier les ACL (WriteDACL)
MATCH p=(u)-[:WriteDacl]->(t) RETURN p

// Qui peut changer le mot de passe d'un autre compte
MATCH p=(u)-[:ForceChangePassword]->(t:User) RETURN p
```

### Nettoyage

```cypher
// Marquer un compte comme compromis
MATCH (u:User {name:"USER@DOMAINE.LOCAL"}) SET u.owned=true RETURN u

// Voir tous les comptes owned
MATCH (u {owned:true}) RETURN u.name
```

## BloodHound CE (Community Edition)

La nouvelle version utilise une API et un frontend web au lieu de l'app Electron :

```bash
# Démarrer avec Docker
docker compose up -d

# Accéder à l'interface
# https://localhost:8080 (credentials affichés au premier démarrage)
```

## Astuces

- Toujours faire une collecte `-c All` pour avoir le maximum d'informations
- Marquer les comptes compromis comme "owned" pour voir les chemins depuis eux
- Regarder les "High Value Targets" (marqués en rouge) en priorité
- Les relations `GenericAll`, `WriteDACL`, `WriteOwner` sont les plus dangereuses
- Combiner avec CrackMapExec pour valider les accès trouvés par BloodHound
