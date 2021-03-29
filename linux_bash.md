# Networking
## Commandes
```bash
# Verifier si un port est ouvert
nc -zvw3 <host> <port>
```
# Storage
## Fonctionnement
Quand on a un nouveau disque il faut commencer par le decouper en partition. Ce sont des 'containers' dans Linux qui permettent de decouper un disque en plusieurs partitions et sur chaque partition on peut appliquer 1 FS. Il existe 2 systems pour gerer les tables de partition, chaque disque doit avoir un systeme de partionnement :
- **MBR**: le systeme historique, limite a 8 partitions et 2TiB
  - utilise fdisk
- **GPT** qui utile les GUUID, jusque 128 partitions
  - utilise gdisk  

**LVM** est arrive car c'etati devenu complique de creer des partitions a chaque fois et cela permet pas de redimenssionner les partitions. A la place on cree une seule partition LVM:
- on peut faire ce qu'on veut vu que c'est des partitions logiques. 
- cela permet aussi de faire des snapshots d'un LV
- `pvmove` permet de deplacer les datas sur un disque defaillant vers un autre pv

Design:  
1. on cree un premier niveau avec le physical volume (pv) qui correspond a un device ou une partition
2. on consolide ces disque dans un volume groupe qui permet d'ajouter de la flexibilite
3. on cree enfin un volume groupe qu'on peut etendre

## Commandes
```bash
# Lister partitions
gdisk -l
# Espace dispo
gdisk <device>
# Supprimer tout le contenu d'une partitions
wipefs -a <device>

# Gerer partition MBR
fdisk
# Gerer partition GPT
gdisk
# Step importante apres partitionnement
partprob <device>

# Formater un nouveau FS
mkfs.<fs>
# Mettre un label
e2label
tune2fs -L
xfs_admin -L <label>

# Formatter un nouveau FS SWAP
mkfswap
swapon
swapoff

# Monter un nouveau FS via fstab
mount -av

# Lister block device
lsblk -f

# Listr
pvs
vgs
lvs

# Creer
vgcreate
lvcreate -L 2G -n <lv_name> <vg_name>

# Extend
lvextend -L -<value>

# Emplacement physique
/dev/<vg>/<lv>
/dev/mapper/<vg><lv>
```


# Gestion File / Folder
## Hierarchie (visible dans man hier)
- `/usr`: Contient les programme et fichiers de l'OS (Ici qu'on vient installer les programmes de l'utilisateur), l'utilisateur n'a normalement pas besoin decrire ici.
- `/boot`: ici concerne tout ce qui est necessaire au boot de l'OS dont le KERNEL. Il est important de le mettre sur un device dediee
- `/var` : contient tous les fichiers qui changent beaucoup, utile de le mettre sur un device dediee pour pas remplir le fs root
- `root file system (/)`: c'est le niveau 0 de l'arborescence, c'est la que le kernel va demarrer
- `/proc`: permet de lire les informations du kernel (cpu, ram, version, cmdline, ...), il s'agit d'un VFS (type proc)

## Inode
inode: contient toutes les metadatas d'un fichier sauf le nom du fichier (c'est chaque fichier qui sait quel est son inode, il y autant de noms que de hardlink, un nom de fichier est un hardlink, si on supprime 1 hardlink les autres ne sont pas affectes):  
- data block avec le contenu du fichier
- creation, access, modification date
- permissions
- owners
- nombre de hardlink qui pointe vers cet inode (compteur visible dans ls -l)

## Commandes
```bash
# Copy en conservant attributs
cp -a <folder> <folder>

# Copy sans le dossier
cp -aT <folder> <folder> 
cp -a <folder>/. <folder>

# Move avec tous les fichiers
mv <folder>/{.,}* <folder>

# Creation
mkdir -p <directory/subdirectory>

# Find and exec 
find / -name '<string>' -exec cp '{}' <dir> \;

# Tar avec compression bzip2
tar -cjf <archive.tar> <files>
# Ajout d'un fichier
tar -rf <archive.tar> <file>
# Extraire contenu
tar -xf <archive.tar> <file a extraire> -C <destination>
# Lister
tar -tf <archive.tar>

# Copy network
scp <source> <user>@<server>:<destination>
rsync -a <source> <server>:<destination>
```


# Gestion du systeme
## Boot processing
![Boot processing](http://cdn.mogile.archive.st-hatena.com/v1/image/mixi_PR/297791556648504227.png)  
1. Le bios (ou UEFI) va chercher le bootloader GRUB2 ici sur le bootable device qui consiste a chercher un fichier de 512kb, taille exacte car le seul fichier pouvant etre lu: `/boot/grub2/i386-pc/boot.img`. Le seul interet de ce fichier est d'aller charger le core.img
2. core.img (<32KB) contient le core du GRUB qui permet de charger les drivers de base et de loader le kernel (vmlinuz and initrd process qui monte en ram initramfs contenant /root), ainsi on a en ram le kernel avec les modules essentiels pour booter (initramfs est cree pendant linstallation)
3. rd.break va permettre de s'arreter au moment ou initramfs est monte en RAM mais pas le /mount depuis le disque 
4. Toujours depuis la RAM, le Kernel demarre systemd qui va s'occuper de demarrer toutes les units d'abord via initrd.target (en RAM) puis quand c'est ok il va charger sur disque default.target

## Systemd
Les services sont geres par target. Une target va demarrer un ensemble de service qui sont dans le dossier wants correspondant. 
C'est le tout premier process qui demarre et qui s'occupe de gerer l'ordonancement et le demarrage de unit une unit peut etre divers (service, socket, mount, target). Cela permet d'avoir une interface universel pour gerer toutes les units.  
Les units files sont a 3 endroits:  
- `/usr/lib/systemd/system`: fichier par defaut installes par RPM + les wants pour chaque target
- `/etc/systemd/system`: fichiers unit custom et gere par systemctl edit + les wants pour chaque target + les isolate targets
- `/run/systemd/system`: contient les fichiers unit automatiquement generes.  
- Les differents status (systemctl status <unit) d'un unit:
  - `enabled`: l'unit demare au prochain demarrage
  - `active`: 1 process est en cours d'execution
  - `active(exited)`: un service oneshot qui a demarre un p rocess et sest arrete
  - `Inactive(dead)`: signifie que le service est arrete
  - `Static`: on peut pas enable cette unit mais elle sera declencher par une autre automatiquement  
L'etat final, l'etat desire c'est l'unit target. 

## Commandes
```bash
# Stop ou reboot
systemctl poweroff / reboot

# Changement target
systemctl set-default multi-user

# Creer un want (activer un service)
systemctl enable --now <service>

# Desactiver un service
systemctl mask <service>

# Mode graphique
systemctl isolate graphical-target

# Verifier un service
systemctl is-active / is-enabled
systemctl --type target
systemctl --type target --all
systemctl status <service>
sysctl -a

# Changer le GRUB au boot
/etc/default/grub (ligne grub_cmdline_linux)

# Commandes utils
history -cw
file <file>
export <variable>="<commande>"
/etc/redhat-release (version exacte)\
hostname status

# Casser mdp root
# grub
rd.break
mount -o rw,remount /sysroot
passwd
touch /.autorelabel
```

# Gestion des process 

## Thread/process
Un thread est un sous-ensemable d'un process ce qui permet de tirer profit des cpu multirecore, multithread. Les threads partagent l'adress space, memory, etc ... Alors que le process est isole des autres process.

## Les statuts
Les differents status:
- `R`: Running (OK)
- `S`: Sleep (OK)
- `D`: Bloque (I/O)

## Le load average
Il s'agit du nombre total de process en R ou D sur 1/5/15 mins, ne doit pas depasser le nombre total de core

## Commandes
```bash
# Identifier process d'un fichier
fuser -mu <file/folder>
# Le killer
fuser -k <file/folder>
# Sur un port
fuser -k -n tcp 80
# Activite
strace -p <pid>

# Lister des process
ps fax
pgrep
top | grep <process>

# Killer
killall <pname>
pkill <pname>
kill -l

# Nombre de cores
lscpu 

# Nice
renice -n -<niceness> -p <pid>
```

# Getion des paquets (YUM)
## Template d'un nouveau repo
```
[<nom repo>]
name=<nom>
baseurl=<http://|file:///|ftp://>/
mirrorlist=
gpgcheck=<1 ou 0>
gpgkey=<key>
```

## Commandes
```bash
yum install bash-completion

# Repos
yum repolist

# Paquet a l'origine du fichier
yum provides <file>
yum info <rpm>

# Groupes
yum group list hidden
yum group info <groupe>

# Historique
yum history
yum history undo <id>

# RPMs
# Chercher RPM associe a un fichier
rpm -qf <file>
# Lister les fichiers d'un RPM
rpm -ql <rpm>
rpm -qd <rpm>
rpm -q --script <paquet>
```

# Gestion des groupes / users

## Les permissions
Les differentes permissions:
- read (4): lire 1 fichier ou lister contenu d'un dossier
- write (2): ecrire dans un fichier ou creer un nouveau fichier
- execute (1): donner les droits en execute d'un fichier ou de se deplacer dans un dossier (cd dossier)  

Special permissions:
- SUID / SGID s (1 bit supplementaire, soit 4 au total, ex chmod 2755) :
- SUID, cela permet d'executer (x) avec les droits owner du fichierls sans les avoir
- SGID, cela permet de creer un fichier avec le primary group du dossier contenant le fichier, utile pour dossier partage  
Sticky bit (t):  
- sur un dossier cela permet d'eviter qu'un user qui a les droits write sur un dossier ne supprime un autre fichier dont il a pas les droits, il pourra supprimer qu'a 2 conditions:
  - si il est owner du fichier ou owner du dossier qui contient le fichier a supprimer

## Commandes
```bash
# Creer 1 utilisateur avec 1 grp primaire different de celui par defaut
useradd -g <grp> <user>
# Avec groupe secondaire
useradd -G <grp> <user>
# Sans shell
useradd -s /bin/false
# Suppression
userdel -r <user>

# Changer proprietes
chage -l <user>
chage <user>

# Creer un nouveau groupe
groupadd <grp>

# Ajouter 1 grp secondaire a 1 utilisateur
usermod -aG <grp> <user>

# Modifier le groupe primaire
usermod -g <grp> <user>
# Changer temporairement de groupe prim
newgrp <grp>

# Ajout commande Sudo
<user> ALL=/usr/bin/<commande>
<user> ALL=/usr/bin<commande> <param1>,!/usr/bin/<commande> <param2>
[a-zA-Z0-9]*

# Chmod specifique dossier
chmod a+X <dir> -R

# Sticky bit (personne peut supprimer sauf le owner)
chmod +t <file>

# Affecter SGID sur un dossier (le groupe du folder devient owner du dossier, ideal partage)
chmod g+s <dir>
```


# Log
## Journalctl
Il permet de lire la log de systemd via daemon journald. Il faut activer `storage=persistent` pour stocker durablement le contenu.
```bash
systemd-journald
journalctl -k kernel
systemctl status <unit>
```

## Rsyslog
Il va gerer des facility (template et enface un fichier de sortie)  
Attention on peut pas en creer de nouvelles (il faut utiliser les `local*`)
```bash
logger -p <facility>.<priority>
```


# CRON
## Formattage
Le format d'un nouveau cron est le suivant: `* * * * *`  
Cela correspond a: minute hour day month day of the week (mettre plusieurs valeurs: 1,15)

## Dossier
Il existe differentes manieres de creer un cron
- `/etc/cron.d` avec Root
- `/etc/cron.hourly/daily/weekly` avec un User (affecter +x)
- `crontab -e` avec un User (ca creer un fichier dans `/var/spool/profile`)

# Programmation en BASH
## Loop
### Until
Utile pour executer une commande tant qu'elle ne retourne pas 0.  
Exemple:  
```bash
# Tant qu'un port n'est pas ouvert, on affiche un message sinon LOL
until nc -zvw3 axis_1 9504 &> /dev/null; do echo "Waiting for the first node" && sleep 1; done; echo "lol"
```
## File descriptor
il existe 3 file descriptor important dans Linux:  
- `0` (equivalent a <): permet d'envoyer input vers une commande qui attend quelque chose entree
- `1` (equivalent a >): permet d'envoyer output standard (STDOUT) a l'ecran, ca ne contient pas les erreurs, uniquement ce qui est 'OK'
```bash
ls Documents > <file>
ls Documents 1> <file>
```
- 2 (equivalent a 2>): permet de rediriger la sortie erreur (STDERR)
2>&1 permet de rediriger STDERR vers STDOUT, ca change rien a l'ecran mais ca permet d'envoyer STDERR sur la sortie precedentev
- `<commande> 1><lolg> 2>&1`: permet denvoyer STDOUT + STDERR vers le meme fichier (par defaut > envoit que STDOUT)
  - 2>/dev/null: permet d'effacer de l'ecran les erreurs et n'avoir que STDOUT
Par defaut les commandes redirige 1 et 2 a l'ecran.  
Autres infos:
Le PIPE (|) permet de rediriger STDOUT (1) vers STDIN (0) de la commande qui suit (STDERR ne sera pas redirige)
    
    
## Wildcards
wildcard:
- `*` remplacer un nombre illimite de caracteres
- `?` remplacer un seul caractere
- `!` inverser un string ex `rm -r !(*.html)` va supprimer tout sauf html
- `\` retour a la ligne
- `[xyz]` filtrer sur certains caracteres
- `{1,2,...}` faire plusieurs actions


## Keywords importants
```bash
#!/bin/bash

# Recuperer arguments
echo $1
$@
for i in "$@"; do echo $i; done
# Nombre d'arguments
$#
```
# Manipulation de strings
## Cut / AWK
Le cut permet d'extraire un string d'un fichier
```bash
cut -d: -f1 <file>
awk -F: '{print $1}'
```
## Trier un fichier avec SED
```bash
# Trier
sort -t: -k2 <file> (stk)

# Supprimer
sed '0,5d' <file>

# Remplacer
sed 's/<old string>/<new string>/g'
sed -i
```

## Commandes utiles
### Grep
`grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /<path>`

### Find
```bash
find . -type f -name '*2017*' -exec mv {} todel/ \;
find . -type f -name '*2017*' -exec du -ch {} + | grep total
find . -type f -printf '%T+ %p\n' | sort | head -n 1
find . -type f -not -newer <fichier>
find /net/teamxisb/TRANSFERT \( -name "*.POSTGRES.*" -o -name "*dmp*" -o -name "*dump*" -o -name "*.json*" \) -mtime +7 -type f -exec rm {} \;
find /net/teamxisb/TRANSFERT \( -name "*.POSTGRES.*" -o -name "*dmp*" -o -name "*dump*" -o -name "*.json*" \) -mtime +10 -type f -print0 | du --files0-from=- -hc | tail -n1
find /home/postgres/osep01/instance/pg_log -type f -name 'postgresql-2018-09*.log' -exec du -ch {} + | grep total$
```
### Disk Usage
```bash
du -sh * |  sort -h | head -10
```
