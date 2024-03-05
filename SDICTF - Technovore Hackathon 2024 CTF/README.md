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




