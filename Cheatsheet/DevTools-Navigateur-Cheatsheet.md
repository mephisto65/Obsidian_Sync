# DevTools Navigateur — Cheatsheet Pentest Web

**Catégorie** : Web / Reconnaissance client
**Phase** : Reconnaissance / Exploitation
**Tags** : #web #devtools #cookies #http #outil #ctf

## Principe

Les DevTools (F12) sont l'outil de base pour tout challenge web côté client : inspecter, modifier et rejouer des échanges HTTP sans avoir besoin de Burp/ZAP pour les cas simples. Quatre onglets à connaître : **Éléments**, **Console**, **Réseau**, **Stockage/Application**.

Raccourci d'ouverture : `F12` ou `Ctrl+Shift+I` (Windows/Linux), `Cmd+Opt+I` (Mac).

## Onglet Stockage / Application — Cookies

C'est l'onglet clé pour ce type de challenge (`Application` sur Chrome, `Stockage` sur Firefox).

- Voir tous les cookies du domaine courant : nom, valeur, domaine, path, expiration, flags (`HttpOnly`, `Secure`, `SameSite`)
- **Modifier une valeur** : double-clic sur la cellule "Valeur" → éditer → `Entrée` pour valider
- **Ajouter un cookie** : clic droit dans la liste → "Ajouter" (selon navigateur)
- **Supprimer un cookie** : sélectionner la ligne → touche `Suppr` ou clic droit → "Supprimer"
- Après modification, **recharger la page (F5)** pour que le serveur relise le cookie

> ⚠️ Un cookie marqué `HttpOnly` n'est pas lisible/modifiable via JavaScript (`document.cookie`), mais reste visible et modifiable directement dans cet onglet DevTools — la protection HttpOnly ne bloque que l'accès scripté, pas l'accès humain via l'UI.

Autres données stockées visibles ici :
- `Local Storage` / `Session Storage` — souvent des tokens ou états d'app JS
- `IndexedDB` — données structurées côté client
- Cache navigateur

## Onglet Réseau (Network)

Capture toutes les requêtes HTTP émises par la page.

- **Rejouer une requête** : clic droit sur une requête → "Copier" → "Copier en tant que cURL" (permet de la rejouer et modifier en ligne de commande)
- **Voir les headers envoyés/reçus** : onglet "En-têtes" d'une requête sélectionnée — utile pour repérer des cookies envoyés, des tokens CSRF, ou des headers custom
- **Voir la réponse brute** : onglet "Réponse" — utile si le flag est caché dans un commentaire HTML ou un header de réponse
- **Filtrer par type** : XHR/Fetch pour ne voir que les appels API/AJAX
- **Throttling / mode offline** : simuler une connexion lente, utile pour intercepter des requêtes rapides

Astuce CTF : conserver les logs entre les rechargements de page via l'option "Preserve log" / "Conserver le journal", sinon Network se vide à chaque navigation.

## Onglet Éléments (Elements/Inspector)

Inspecte et modifie le DOM en direct.

- **Modifier le HTML affiché** : double-clic sur un attribut ou texte pour l'éditer (ex: retirer un attribut `disabled`, `readonly`, ou `hidden` sur un champ/bouton)
- **Chercher dans le code source** : `Ctrl+F` dans le panneau Éléments pour chercher une chaîne (utile pour trouver des commentaires HTML contenant des indices)
- **Voir le CSS appliqué** : panneau "Styles" à droite — utile si du contenu est caché via `display:none` ou `visibility:hidden`
- **Event Listeners** : onglet dédié pour voir quels événements JS sont attachés à un élément (utile pour comprendre une logique de validation côté client)

> Modifier le DOM via cet onglet est purement visuel/local — ça ne modifie rien côté serveur. Utile pour débloquer un formulaire verrouillé côté client (ex: bouton désactivé) avant de soumettre une requête.

## Onglet Console

Exécute du JavaScript directement dans le contexte de la page.

```js
// Lire tous les cookies non-HttpOnly
document.cookie

// Modifier/ajouter un cookie (uniquement si pas HttpOnly)
document.cookie = "login=1; path=/"

// Lire le localStorage
localStorage.getItem("clé")
Object.entries(localStorage)

// Modifier le localStorage
localStorage.setItem("clé", "valeur")

// Voir toutes les variables globales exposées par l'app (souvent révélateur)
window
```

Astuce : la console garde un historique des requêtes réseau échouées (erreurs CORS, 404, etc.) — souvent la première chose à checker en arrivant sur une page.

## Voir le code source brut (sans DevTools)

- `Ctrl+U` : affiche le HTML source tel que servi (avant exécution JS) — utile pour trouver des commentaires `<!-- -->` contenant des indices, des chemins cachés, ou du JS non minifié
- `view-source:https://exemple.com` : équivalent en tapant l'URL directement

## Workflow type pour un challenge "cookie"

1. `F12` → onglet Stockage/Application → repérer le cookie suspect (souvent un nom explicite : `login`, `role`, `admin`, `auth`)
2. Noter sa valeur actuelle (ex: `0`, `false`, `guest`)
3. La modifier vers une valeur plus privilégiée (`1`, `true`, `admin`) directement dans l'éditeur DevTools
4. `F5` pour recharger et déclencher la relecture côté serveur
5. Si rien ne change, vérifier l'onglet Réseau pour confirmer que le cookie modifié est bien envoyé dans la requête suivante (header `Cookie:`)
6. Si la valeur est encodée (base64, hash simple), la décoder/reconstruire avant de la réinjecter

## Limites des DevTools (quand passer à Burp/ZAP)

- Impossible d'intercepter/modifier une requête **avant** son envoi de façon simple (DevTools ne fait qu'observer, sauf via "Edit and Resend" sur Firefox récent)
- Pas de repeater pour bruteforcer des valeurs facilement
- Pas de proxy pour intercepter du trafic hors navigateur (apps mobiles, clients lourds)
- Firefox propose "Modifier et renvoyer" (clic droit sur une requête dans Réseau) qui comble en partie ce manque pour des tests rapides

## Liens

- [MDN - DevTools Storage Inspector](https://developer.mozilla.org/fr/docs/Tools/Storage_Inspector)
- [MDN - HTTP cookies](https://developer.mozilla.org/fr/docs/Web/HTTP/Cookies)
