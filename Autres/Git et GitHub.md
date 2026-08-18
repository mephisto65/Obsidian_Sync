# Git / GitHub

Système de versioning distribué. Indispensable pour gérer ses notes, scripts, projets et collaborer.

## Configuration initiale

```bash
git config --global user.name "ton-pseudo"
git config --global user.email "ton-email@example.com"

# Vérifier la config
git config --list
```

## Créer un repo

### Nouveau projet local → GitHub

```bash
cd /chemin/vers/projet
git init
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/user/repo.git
git push -u origin main
```

### Cloner un repo existant

```bash
# HTTPS
git clone https://github.com/user/repo.git

# Avec token (pour les repos privés)
git clone https://user:TOKEN@github.com/user/repo.git

# SSH
git clone git@github.com:user/repo.git

# Dossier spécifique
git clone https://github.com/user/repo.git mon-dossier
```

## Workflow quotidien

```bash
# Récupérer les dernières modifications
git pull

# Voir ce qui a changé
git status

# Ajouter les modifications
git add .                     # Tout
git add fichier.md            # Un fichier spécifique
git add dossier/              # Un dossier

# Committer
git commit -m "description du changement"

# Pousser vers GitHub
git push

# Combo rapide
git add . && git commit -m "notes du jour" && git push
```

## Branches

```bash
# Lister les branches
git branch                    # Locales
git branch -a                 # Toutes (locales + remote)

# Créer une branche
git branch nouvelle-branche
git checkout -b nouvelle-branche   # Créer + se placer dessus

# Changer de branche
git checkout main
git switch main               # Syntaxe moderne

# Fusionner une branche dans main
git checkout main
git merge nouvelle-branche

# Supprimer une branche
git branch -d nouvelle-branche          # Locale
git push origin --delete nouvelle-branche  # Remote
```

## Historique

```bash
# Voir les commits
git log
git log --oneline             # Compact
git log --oneline --graph     # Avec arbre des branches
git log -5                    # Les 5 derniers
git log --author="pseudo"     # Par auteur

# Voir les changements d'un commit
git show <commit_hash>

# Différences
git diff                      # Modifications non staged
git diff --staged             # Modifications staged
git diff main..branche        # Différence entre deux branches
```

## Annuler des modifications

```bash
# Annuler les modifications d'un fichier (avant add)
git checkout -- fichier.md
git restore fichier.md        # Syntaxe moderne

# Unstage un fichier (après add, avant commit)
git reset HEAD fichier.md
git restore --staged fichier.md

# Annuler le dernier commit (garder les modifications)
git reset --soft HEAD~1

# Annuler le dernier commit (supprimer les modifications)
git reset --hard HEAD~1

# Revenir à un commit spécifique
git reset --hard <commit_hash>

# Forcer le push après un reset
git push --force
```

## Remote

```bash
# Voir les remotes
git remote -v

# Ajouter un remote
git remote add origin https://github.com/user/repo.git

# Changer l'URL du remote
git remote set-url origin https://github.com/user/nouveau-repo.git

# Supprimer un remote
git remote remove origin
```

## Stash (mettre de côté)

```bash
# Sauvegarder les modifications en cours sans committer
git stash

# Lister les stash
git stash list

# Récupérer le dernier stash
git stash pop

# Récupérer sans supprimer du stash
git stash apply

# Supprimer un stash
git stash drop
```

## .gitignore

Fichier à la racine du repo pour exclure des fichiers du versioning :

```bash
# Obsidian
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.trash/

# Système
.DS_Store
Thumbs.db
*.swp
*~

# Secrets
*.env
credentials.txt
tokens.json

# Compilés
*.pyc
__pycache__/
node_modules/
```

## Authentification

### Token (HTTPS)

```bash
# Générer sur GitHub : Settings → Developer settings → Personal access tokens
# Utiliser dans l'URL
git clone https://user:TOKEN@github.com/user/repo.git

# Ou sauvegarder le credential
git config --global credential.helper store
# Le prochain git push demandera le token et le sauvegarde
```

### Clé SSH

```bash
# Générer une clé
ssh-keygen -t ed25519 -C "ton-email@example.com"

# Copier la clé publique
cat ~/.ssh/id_ed25519.pub

# Coller sur GitHub : Settings → SSH and GPG keys → New SSH key

# Tester la connexion
ssh -T git@github.com

# Cloner en SSH
git clone git@github.com:user/repo.git
```

## Résolution de conflits

Quand un `git pull` échoue à cause de modifications concurrentes :

```bash
# Git marque les conflits dans les fichiers :
<<<<<<< HEAD
ta version locale
=======
la version du remote
>>>>>>> origin/main

# 1. Éditer le fichier, choisir la bonne version
# 2. Supprimer les marqueurs <<<<< ===== >>>>>
# 3. Add + commit
git add fichier.md
git commit -m "résolution conflit"
git push
```

## Commandes utiles

```bash
# Qui a modifié quelle ligne
git blame fichier.md

# Chercher dans l'historique
git log --all --grep="mot-clé"

# Taille du repo
git count-objects -vH

# Nettoyer les fichiers non suivis
git clean -fd

# Voir la config d'un repo
cat .git/config
```

## GitHub spécifique

```bash
# Créer un repo depuis la CLI (avec GitHub CLI)
gh repo create nom-repo --private

# Forker un repo
gh repo fork user/repo

# Créer une pull request
gh pr create --title "titre" --body "description"

# Voir les issues
gh issue list
```
