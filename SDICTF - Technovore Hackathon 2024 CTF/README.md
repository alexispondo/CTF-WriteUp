# Technovore Hackathon 2024 CTF

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/7583d144-aa93-401d-901d-85cec5bc50ca)

## Description

Notre équipe de défense a intercepté un message crypté alarmant : le redoutable groupe de hackers DarkAtt4ck prépare une attaque cyberterroriste d'envergure contre notre entreprise TechInnovate Corp.

Votre mission, si vous l'acceptez, est cruciale : infiltrer le système d'information de DarkAtt4ck et prendre le contrôle total afin de les neutraliser avant qu'ils ne causent des dommages irréparables.

Votre première tâche consite à bien analyser le site web de darkatt4ck et trouver le flag web

## 1- Trouvez le flag sur l'application WEB

Comme dans tous défis de test d'intrusion la première étape consiste à faire le `scan` de tous les ports sur le serveur cible.

```
nmap -p- 10.10.10.10 -A
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
wget -r -l5 -k -E "http://10.10.239.204:1337/"
```
- r : récursif sur le site
- l5 : cinq niveaux de récursion au maximum
- k : convertir les destinations des liens pour une lecture locale
- E : convertir les types de fichier au format HTML (pour éviter que la lecture de sites en PHP ne foire en lecture sous Firefox).

Cela va aspirer le site dans le dossier `10.10.239.204:1337/`

Nous allons ensuite faire la recherche du pattern `SDICTF{` de façon récursive dans le dossier téléchargé
```
cd 10.10.239.204:1337/
rgrep -i "SDICTF{" .
```
et on obtient ainsi le flag web

![Capture d’écran_2024-03-06_17-50-14](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/e8ea0553-988b-49f9-ac99-e14465a22925)

## 2- Trouvez le flag sur le serveur FTP
Lorsqu'on regarde bien dans le chat, on remarque qu'une discussion a eu lieu sur la mise en place d'un certain serveur ftp

Par contre le serveur ftp n'est pas en lui-même vulnérable.

Après quelques minutes de recherche sur des pistes on remarque un lien presque invisible sur la page d'accueille du site `DarkAtt4ck test`

Il s'agit en effet d'un lien qui ne point vers aucune page, ce qui nous donne une erreur 

Le mode debug étant toujours activé nous permet donc à travers cette erreur de voir les différentes urls disponibles, cela nous permet d'avoir accès à l'url `secret_directory/`

Une fois que nous somme sur l'url nous constatons que nous pouvons lire certains fichiers sur le serveur

Nous pouvons donc tenter de faire une attaque de type `Directory traversal attack` qui pourra nous permettre de lire certains fichiers de notre choix sur le serveur.

L'un des fichiers les plus lus dans ce cas est `/etc/passwd`

Nous tentons de lire `/secret_directory/?doc=/etc/passwd`

Nous remarquons que rien ne se passe, dans le cas ou il pourrait s'agir d'un chemin absolu, nous pouvons tenter de résoudre le problème avec des retours en arrière `../../`

On reprend donc cette fois-ci avec les retours en arrière `/secret_directory/?doc=../../etc/passwd`


Cette fois-ci nous avons la possibilité de lire les fichiers que nous souhaitons sur le serveur.

On se rappel que sur le chat ils avaient parlés de la création d'un serveur FTP et que la liste des utilisateurs autorisés à accédé à ce serveur est disponible à `/home/k1ller/userftp.txt`

On tente donc de lire `/secret_directory/?doc=../../home/k1ller/userftp.txt`

Bingo donc nous avons accès au dossier home de `k1ller`

Après plusieurs minutes de recherche, nous tentons de lire le fichier générique présents par défaut dans le répertoire home d'un utilisateur sous linux, et nous constatons que le fichier `.bash_history` de k1ller est accessible en lecture et nous donne des informations intéressantes comme le mot de passe de l'utilisateur k1ller sur le serveur ftp.

Nous nous connectons donc au server ftp et nous obtenons le flag FTP

## 3- Trouvez le flag de l'utilisateur k1ller







