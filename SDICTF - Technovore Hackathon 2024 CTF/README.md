# Technovore Hackathon 2024 CTF

![image](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/7583d144-aa93-401d-901d-85cec5bc50ca)

## Description

Notre équipe de défense a intercepté un message crypté alarmant : le redoutable groupe de hackers DarkAtt4ck prépare une attaque cyberterroriste d'envergure contre notre entreprise TechInnovate Corp.

Votre mission, si vous l'acceptez, est cruciale : infiltrer le système d'information de DarkAtt4ck et prendre le contrôle total afin de les neutraliser avant qu'ils ne causent des dommages irréparables.

Votre première tâche consite à bien analyser le site web de darkatt4ck et trouver le flag web

## #Q1 Trouvez le flag sur l'application WEB

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

Pour cela plusieurs options s'offrent à nous :
- La première option est d'utiliser l'outil wget
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

