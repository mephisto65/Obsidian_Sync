# Burp Suite

Proxy d'interception pour les tests d'applications web. Permet de capturer, modifier et rejouer les requêtes HTTP/HTTPS entre le navigateur et le serveur.

## Configuration

### Proxy navigateur

Configurer le navigateur pour utiliser le proxy Burp :
- Adresse : `127.0.0.1`
- Port : `8080`

Installer le certificat CA de Burp pour intercepter HTTPS :
- Aller sur `http://burpsuite` dans le navigateur proxifié
- Télécharger et installer le certificat CA

### Scope

Limiter l'interception aux domaines cibles pour éviter le bruit :
- Target > Scope settings > Add : `*.cible.com`
- Proxy > Options > Intercept Client Requests > cocher "URL Is in target scope"

## Onglets principaux

### Proxy (interception)

```
Intercept is on  → chaque requête est bloquée pour inspection
Intercept is off → les requêtes passent, mais sont loguées dans HTTP History
```

Raccourcis utiles :
- **Forward** : envoyer la requête
- **Drop** : supprimer la requête
- **Send to Repeater** (`Ctrl+R`) : envoyer vers Repeater pour tester
- **Send to Intruder** (`Ctrl+I`) : envoyer vers Intruder pour brute force

### Repeater (test manuel)

Modifier et renvoyer des requêtes individuelles. L'outil le plus utilisé pour tester des injections manuellement.

```
1. Modifier un paramètre (ex: id=1 → id=1' OR 1=1--)
2. Cliquer "Send"
3. Analyser la réponse
4. Itérer
```

Raccourcis :
- `Ctrl+R` : envoyer depuis Proxy vers Repeater
- `Ctrl+Shift+R` : renvoyer la dernière requête

### Intruder (attaques automatisées)

Automatiser des tests sur un ou plusieurs paramètres.

**Types d'attaque** :

| Type | Usage |
|------|-------|
| Sniper | Un seul paramètre, une wordlist | 
| Battering Ram | Même payload dans tous les paramètres |
| Pitchfork | Un payload différent par paramètre (en parallèle) |
| Cluster Bomb | Toutes les combinaisons de payloads |

**Exemple — brute force login** :
1. Capturer la requête POST de login
2. Send to Intruder
3. Marquer les positions : `username=§user§&password=§pass§`
4. Charger les wordlists
5. Lancer l'attaque
6. Trier par taille de réponse ou code HTTP pour repérer le succès

### Decoder

Encoder/décoder des données :
- Base64, URL encoding, HTML entities, Hex, ASCII
- Hashage : MD5, SHA1, SHA256

### Comparer

Comparer deux réponses côte à côte pour repérer les différences (utile pour les injections blind).

## Techniques de test courantes

### SQL Injection

```
# Dans Repeater, modifier le paramètre
id=1' OR '1'='1
id=1' UNION SELECT null,null,null--
id=1' AND SLEEP(5)--
```

### XSS

```
# Tester dans les paramètres et les headers
<script>alert(1)</script>
"><img src=x onerror=alert(1)>
javascript:alert(1)
```

### LFI / Path Traversal

```
file=../../../etc/passwd
file=....//....//....//etc/passwd
file=/etc/passwd%00.jpg
```

### IDOR (Insecure Direct Object Reference)

```
# Changer les IDs dans les requêtes
GET /api/users/123 → GET /api/users/124
GET /invoice?id=1001 → GET /invoice?id=1002
```

### Authentication bypass

```
# Modifier les cookies ou tokens
Cookie: role=user → Cookie: role=admin
Cookie: isAdmin=false → Cookie: isAdmin=true

# Forcer l'accès direct
GET /admin/dashboard (sans auth)
```

## Extensions utiles

| Extension | Usage |
|-----------|-------|
| **Logger++** | Logging avancé avec filtres |
| **Autorize** | Tester les contrôles d'accès automatiquement |
| **Param Miner** | Découvrir des paramètres cachés |
| **Turbo Intruder** | Intruder rapide pour les race conditions |
| **JWT Editor** | Manipuler les tokens JWT |
| **Hackvertor** | Encodage/décodage avancé inline |
| **Active Scan++** | Améliorer le scanner actif |

## Raccourcis clavier

| Raccourci | Action |
|-----------|--------|
| `Ctrl+R` | Send to Repeater |
| `Ctrl+I` | Send to Intruder |
| `Ctrl+Shift+R` | Renvoyer la requête (Repeater) |
| `Ctrl+F` | Chercher dans la réponse |
| `Ctrl+U` | URL-encoder la sélection |
| `Ctrl+Shift+U` | URL-décoder la sélection |
| `Ctrl+B` | Base64-encoder la sélection |
| `Ctrl+Shift+B` | Base64-décoder la sélection |

## Astuces

- Toujours définir le scope en premier pour ne pas se noyer dans le bruit
- Utiliser les filtres dans HTTP History (par host, par statut, par MIME type)
- Match and Replace (Proxy > Options) pour modifier automatiquement des headers (ex: supprimer CSP pour tester les XSS)
- La version Community (gratuite) est limitée en vitesse sur Intruder — Turbo Intruder contourne cette limite
