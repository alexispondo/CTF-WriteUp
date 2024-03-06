# Technovore Hackathon 2024 CTF
![download](https://github.com/alexispondo/CTF-WriteUp/assets/47490330/ed6c5c5c-0720-4a5e-a07e-e08ec9938f5f)

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

La question étant de trouver le flag web, l'une des première chose à éffectuer avant de faire une recherche manuelle consiste à aspirer le site et à rechercher depuis notre terminale le patterne `SDICTF{` qui constitue le debut du flag.

Pour cela plusieurs options s'offrent à nous:
- La prémière option est d'utiliser l'outil wget
```
wget -r -l5 -k -E "http://10.10.239.204:1337/"
```
-r : récursif sur le site
-l5 : cinq niveaux de récursion au maximum
-k : convertir les destinations des liens pour une lecture locale
-E : convertir les types de fichier au format HTML (pour éviter que la lecture de sites en PHP ne foire en lecture sous Firefox).

Cela va asspirer le site dans le dossier `10.10.239.204:1337/`

Nous allons ensuite faire la recherche du patterne `SDICTF{` de façon recurssive dans le dossier téléchargé
```
cd 10.10.239.204:1337/
rgrep -i "SDICTF{" .
```
et on obtient ainsi le flag web
```
./chat/index.html:              <div class="message"><span class="user">Flag Web:</span> SDICTF{*****************************************}</div>
```


