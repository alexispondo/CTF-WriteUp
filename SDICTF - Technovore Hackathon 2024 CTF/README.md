# Technovore Hackathon 2024 CTF

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/7583d144-aa93-401d-901d-85cec5bc50ca)

## Description

Notre équipe de défense a intercepté un message crypté alarmant : le redoutable groupe de hackers DarkAtt4ck prépare une attaque cyberterroriste d'envergure contre notre entreprise TechInnovate Corp.

Votre mission, si vous l'acceptez, est cruciale : infiltrer le système d'information de DarkAtt4ck et prendre le contrôle total afin de les neutraliser avant qu'ils ne causent des dommages irréparables.

Votre première tâche consite à bien analyser le site web de darkatt4ck et trouver le flag web

## Infos:
- IP attaquant: 10.10.83.51
- IP cible: 10.10.201.34

## 1- Trouvez le flag sur l'application WEB

Comme dans tous défis de test d'intrusion la première étape consiste à faire le `scan` de tous les ports sur le serveur cible.

```
nmap -p- 10.10.201.34 -A
```

Nous constatons que 4 ports sont ouverts : `22, 80, 1337, 20221`.

- Port 80: Il ne donne rien de vraiment intéressant à part la page par défaut de Apache qui ne nous aidera pas
- Port 22: Service SSH dont la version n'est pas particulièrement vulnérable
- Port 20221: Service FTP dont la version n'est pas particulièrement vulnérable
- Port 1337: Service web qui héberge le site web du groupe

![Capture d’écran_2024-03-06_17-03-43](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/089629c2-e4a0-4bf7-a83c-51874c867213)

La question étant de trouver le flag web, l'une des premières chose à effectuer avant de faire une recherche manuelle consiste à aspirer le site et à rechercher depuis notre terminale le pattern `SDICTF{` qui constitue le début du flag.

Nous pouvons utiliser l'outil wget pour cela
```
wget -r -l5 -k -E "http://10.10.201.34:1337/"
```
- r : récursif sur le site
- l5 : cinq niveaux de récursion au maximum
- k : convertir les destinations des liens pour une lecture locale
- E : convertir les types de fichier au format HTML (pour éviter que la lecture de sites en PHP ne foire en lecture sous Firefox).

Cela va aspirer le site dans le dossier `10.10.201.34:1337/`

Nous allons ensuite faire la recherche du pattern `SDICTF{` de façon récursive dans le dossier téléchargé
```
cd 10.10.201.34:1337/
rgrep -i "SDICTF{" .
```
et on obtient ainsi le flag web

![Capture d’écran_2024-03-06_17-50-14](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/e8ea0553-988b-49f9-ac99-e14465a22925)

Le flag peut être également vu sur le site web dans le chat

## 2- Trouvez le flag sur le serveur FTP
Lorsqu'on regarde bien dans le chat, on remarque qu'une discussion a eu lieu sur la mise en place d'un certain serveur ftp

Par contre le serveur ftp n'est pas en lui-même vulnérable.

Après quelques minutes de recherche sur des pistes on remarque un lien presque invisible sur la page d'accueille du site `DarkAtt4ck test`

Il s'agit en effet d'un lien qui ne point vers aucune page, ce qui nous donne une erreur 

Le mode debug étant toujours activé nous permet donc à travers cette erreur de voir les différentes urls disponibles, cela nous permet d'avoir accès à l'url `secret_directory/`

Une fois que nous somme sur l'url nous constatons que nous pouvons lire certains fichiers sur le serveur

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/bfede17e-78fa-4a66-9a3b-6c64d39b7fc0)

Nous pouvons donc tenter de faire une attaque de type `Directory traversal attack` qui pourra nous permettre de lire certains fichiers de notre choix sur le serveur.

L'un des fichiers les plus lus dans ce cas est `/etc/passwd`

Nous tentons de lire `/secret_directory/?doc=/etc/passwd`

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/98f61e6a-2558-4f48-ae1d-fdcc953d9512)


Nous remarquons que rien ne se passe, dans le cas ou il pourrait s'agir d'un chemin absolu, nous pouvons tenter de résoudre le problème avec des retours en arrière `../../`

On reprend donc cette fois-ci avec les retours en arrière `/secret_directory/?doc=../../etc/passwd`

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/23fdb133-4b51-4967-a9b3-ff29c50bcde1)


Cette fois-ci nous avons la possibilité de lire les fichiers que nous souhaitons sur le serveur.

On se rappel que sur le chat ils avaient parlés de la création d'un serveur FTP et que la liste des utilisateurs autorisés à accédé à ce serveur est disponible à `/home/k1ller/userftp.txt`

On tente donc de lire `/secret_directory/?doc=../../home/k1ller/userftp.txt`

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/f27628d5-b351-49a1-aca6-68c6e3091b14)


Bingo donc nous avons accès au dossier home de `k1ller`

Après plusieurs minutes de recherche, nous tentons de lire le fichier générique présents par défaut dans le répertoire home d'un utilisateur sous linux, et nous constatons que le fichier `.bash_history` de k1ller est accessible en lecture et nous donne des informations intéressantes comme le mot de passe de l'utilisateur k1ller sur le serveur ftp.

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/96cc60ce-7748-4fbd-8f01-63dd556f4ba0)

Nous nous connectons donc au server ftp et nous obtenons le flag FTP

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/ed46e5c8-8c91-4e56-91a3-70445383233a)

## 3- Trouvez le flag de l'utilisateur k1ller

En explorant bien le serveur FTP on constate que qu'il existe un dossier `.jobs` qui contient un script qui semble faire un backups de la liste des utilisateurs ftp

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/aaf0fecd-762d-4fb6-abe7-7470d88aace2)

Nous pouvons nous en assurer en consultant le fichier /etc/crontab depuis à travers la vulnérabilité web découverte.

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/45e1130a-5873-404a-bee2-0842018cfef1)

Nous savons maintenant que k1ller exécute le fichier de backup toute les 5 minutes

Nous avons également les permissions de modification et d'exécution sur le fichier

Nous pouvons donc effectuer une attaque de reverse shell pour obtenir le shell k1ller

On télécharge et modifie le fichier de backup en y ajoutant notre payload de reverse shell

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/b7b81d00-1769-4f6b-aa4f-b6dad5be1be4)

On prépare notre terminal à recevoir le reverse shell

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/9eddd21f-a9bf-4c1d-a5d3-2d5163993a29)

On re-upload notre fichier de backup modifié

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/7d4e5b6e-e26d-490a-8f2d-b4e34cee9c52)

Et on attend 5 min au plus pour obtenir un shell

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/3ec3d1ba-ce69-4136-96ee-6f7efe5f6286)

On stabilise le shell

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/7cd2c970-8640-4693-a0ea-00dd6134f562)

Une fois avec le shell k1ller nous pouvons lire le flag

## 4- Trouvez le flag de l'utilisateur darkgod

Une fois k1ller nous pouvons vérifier nos autorisations sudo `sudo -l`

Nous pouvons constater que k1ller a la possibilité d'exécuter le binaire view en tant que darkgod.
Or GTFBins nous dit qu'il est possible d'obtenir un shell en exploitant la commande view
![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/3a82c570-9a2d-4539-91df-2f30eb231fa8)

On exécuté ainsi `sudo -u darkgod view -c ':!/bin/sh'`

Cela nous permet donc d'obtenir un shell darkgod et de lire le flag

## 5- Trouvez le flag de l'utilisateur shadowxx

En entrant `id` en tant que darkgod on constate qu'on fait partir du groupe `seniors`

En recherchant un peu on remarque les membres du groupe seniors ont accès en lecture au fichier `/etc/shadow` qui contient le hash des mots de passe des utilisateurs

On copie le contenu de ce fichier dans un fichier sur notre machine que nous pourrions par la suite cracker avec `john`

Ce la nous permet d'obtenir le mot de passe de l'utilisateur shadowxx

Nous nous connectons en tant que shadowxx ce qui nous permet de lire le flag

## 6- Trouvez le flag de l'utilisateur alpha01

En menant nos investigations nous remarquons que le binaire `/etc/program/programDarkAtt4ck` à un bit SUID activé ce qui nous permet de l'exécuter avec les droits du propriétaire

En cherchant à comprendre ce que le fichier fait nous remarquons qu'il fait appel à un autre fichier du nom de `/var/script.sh`

Regardons le contenu de ce fichier :

Nous remarquons qu'il n'est pas accessible en écriture par shadowxx (ce nous aurait permis de faire un reverse shell rapide) par contre il fait appel à des commandes `whoami` et `hostname` avec leurs noms relatifs ce qui nous permet d'exploiter la variable d'environnement PATH.

D'abord nous modifions notre variable d'environnement PATH en y ajoutant le dossier `/tmp` au début

```
PATH=/tmp:$PATH
```

Nous créons un fichier nommé `/tmp/whoami` avec les autorisations d'exécution en y insérant le code de reverse shell

On prépare la machine à accueillir notre reverse shell

Et on re-exécute le binaire `/etc/program/programDarkAtt4ck`

On obtient ainsi un nouveau shell avec notre `uid` qui a changé en devenant celui de `alpha01` ce qui nous permet de lire le flag

## 7- Trouvez le flag de l'utilisateur root

Le chemin par lequel il fallait passer est de monitorer les processus linux, cela passe par l'outil pspy qui nous permet de le faire sans les permissions root.

Nous téléchargeons et exécutons le fichier pspy sur le serveur cible 


Après quelques minutes nous remarquons qu'il y a une tache cron qui exécute avec les autorisations root un ensemble de commande qui permet d'archiver les fichiers contenu dans `/var/documentations` dans le dossier `/var/backups`

Ce qui nous intéresse ici, c'est la manière dont le dossier est archivé, en effet il archive tous les fichiers de documentations en utilisant la commande `tar` et le wildcard `*`

Nous pouvons exploiter cela en utilisant le `Tar Wildcard Injection`

L'une des choses qui pourra nous permettre de faire cette exploitation est d'avoir accès en écriture au dossier dans lequel l'archive sera créé, qui est ici `/var/documentations`. Lorsqu'on regarde les permissions de ce dossier, nous pouvons remarquer que seules l'utilisateur `root` et le groupe `managers` ont les droits d'écriture sur le dossier.


En entre une fois de plus la commande `id` pour obtenir les groupe auxquels nous avons accès

Nous constatons que nous somme toujours shadow et que seul notre uid avait changé 

Nous vérifions l'id de alpha01

On constate qu'il fait bien partir du groupe `managers`, nous devons donc trouver le moyen de devenir réellement `alpha01`, et non seulement en termes du `uid`

Un moyen de le devenir est de créer une paire de clé SSH en tant que alpha01 et de se connecter avec la clé privée 

Voici le processus :

1- Créer la paire de clé

2- Transférer la clé privée sur notre machine et lui donner les bonnes permissions

3- Transférer la clé publique dans le fichier `authorized_key` en lui donnant la bonne permission

Ensuite on se connecte par ssh et on vérifie l'id de nouveau

Nous faisons maintenant bien partir du groupe `managers`, nous pouvons donc modifier le dossier `/var/documetations`

Le processus d'exploitation se fait comme suit :

1- Créer un script de reverse shell (ex: script.sh), lui donner les droits d'exécution et préparer notre machine à le recevoir

2- Créer un fichier vide nommé `--checkpoint=1`
```
echo "" > --checkpoint=1
```
3- Créer un fichier vide nommé `"--checkpoint-action=exec=sh script.sh"`
```
echo "" > "--checkpoint-action=exec=sh script.sh"
```

Ensuite on attend une minute que notre script soit exécuté

On obtient ainsi un shell root ce qui nous permet de lire le flag
















