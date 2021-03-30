# Definition:
- HEAD: c'est le pointeur/curseur correspondant a notre dernier commit sur notre branche actuelle (mais le HEAD peut etre deplace sur un ancien commit)
- origin: Un alias qui par convention s'appelle 'origin'. Cet alias est le repository distant vers lequel on veut pusher. Mais il pourrait etre different, liste: `git remote`

# Commandes
``` bash
# Config globale
git config
git config --global user.name "Your Name"
git config --global user.email "youremail@yourdomain.com"
# Voir la conf
git config --list

# Initier la creation d'un dossier
git init

# Stagger un nouveau fichier
git add .
git add <file>

# Voir la stagging area
git status

# Commiter la stagging area 
git commit -a
git commit -am '<message>'

# Supprimer un fichier
git rm <file>

# Annuler changement dans working directory
git checkout .
git checkout <file>
git checkout <commit id>

# Retirer fichier de la stagging area
git reset .
# Reset du contenu de la stagging area sans perdre fichier locaux
git reset --hard HEAD
# Rollback toutes les modification en local tracked or not (tracked veux dire dans la stagging)
git reset --hard

# Rollback un commit
git revert

# Creer une nouvelle branche qui n'existe pas
git checkout -b <branch>

# lire la log
git log

# Cloner une branche
git clone -b <branch> <url>

# Changer de branche
git branch

# Creer une branche sans switcher
git branch <name>

# Rename une branche
git branch -m <name> <new name>

# Supprimer une branche
git branch -D <name>

# Supprimer une branche remote
git push origin --delete <name>

# ecraser en local une branche (+ pusher)
git branch -f <destination> <source>
git push -f origin HEAD
git merge <branche source> <target>

# 'Ranger' des changes sans commiter et changer de branche
git stash
git stash --include-untracked --all

# Liste les change
#permet de voir les changement non commite
git diff --staged
git stash show: lister les changes
# Stasher
git stash push: pour ranger quelque chose
# Apply
git stash apply: pour appliquer les modifications
```
