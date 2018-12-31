---
layout: post
title: "Bien débuter avec l'ESP32"
date: 2018-11-20 8:55:00 -0120
comments: false
published: false
category:
- embedded
---

> L'ESP32 a popularisé le Wifi dans les développements embarqués. Espressif, le créateur du chipset, a su allier facilité d'utilisation, miniaturisation, customisation et plein d'autes mots en -tion.

# Check-list du matériel

Pour bien débuter avec l'ESP32, le mieux est d'essayer de rassembler un petit matériel de base :

  * Un ESP32, par exemple le modèle Wrover qui intègre le Bluetooth et est vraiment complet
  * Une plaque à trou
  * Des fils mono-broche rigide pour plaque à trou


Voici une sélection de liens sur Amazon :

  * <a target="_blank" href="https://www.amazon.fr/Aihasd-Breadboard-Platine-Compatible-puissance/dp/B00W3GS36W/ref=sr_1_2?ie=UTF8&amp;qid=1542809411&amp;sr=8-2&_encoding=UTF8&tag=manolab-21&linkCode=ur2&linkId=f90302407a47b54b061332c34ff36e61&camp=1642&creative=6746">Kit plaque d'essais + fils + alimentation</a><img src="//ir-fr.amazon-adsystem.com/e/ir?t=manolab-21&l=ur2&o=8" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
  * <a target="_blank" href="https://www.amazon.fr/dalimentation-adaptateur-ordinateur-imprimante-t%C3%A9l%C3%A9copieurs/dp/B00NSEPJBK/ref=sr_1_4?ie=UTF8&amp;qid=1542809917&amp;sr=8-4&_encoding=UTF8&tag=manolab-21&linkCode=ur2&linkId=a9e4a1e52a3fb58671bccbe8218a81a4&camp=1642&creative=6746">Alimentation secteur multi-tensions</a><img src="//ir-fr.amazon-adsystem.com/e/ir?t=manolab-21&l=ur2&o=8" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
  * <a target="_blank" href="https://www.amazon.fr/AZDelivery-Adaptateur-FT232RL-Arduino-Adapter/dp/B01N9RZK6I/ref=sr_1_3?s=computers&amp;ie=UTF8&amp;qid=1542810016&amp;sr=1-3&_encoding=UTF8&tag=manolab-21&linkCode=ur2&linkId=af36232151c460c6f7747a8efc9f23b0&camp=1642&creative=6746">USB-Série FTDI</a><img src="//ir-fr.amazon-adsystem.com/e/ir?t=manolab-21&l=ur2&o=8" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

  

![image]({{ site.url }}/assets/articles/raspberry-rclone/principe.png)

Il nous faudra donc :
  * Un Raspberry PI, n'importe quelle version fera l'affaire
  * Raspbian récent
  * Un câble réseau RJ45
  * Connaître l'adresse IP du PI et du disque réseau

# Montage des disques

Tout d'abord, nous allons monter le disque réseau. Cela va dépendre comment votre disque est relié au réseau et le protocole de partage : généralement c'est NFS ou Samba (partage Windows). Vous pouvez également brancher le disque dur USB directement sur votre PI, ce n'est généralement pas trop conseillé pour des raisons de performances et de fiabilité.

On va donc modifier le fichier /etc/fstab :


```shell
/dev/sda1 /media/usbdisk ext4 defaults,auto 0 0
# 192.168.1.30/WiFiDisk1_Volume1_Share /media/nas cifs username=guest,password=bidon,iocharset=utf8,sec=ntlm,auto 0 0
10.0.0.1/nas /media/nas cifs vers=3.0,user,guest,sec=ntlmssp,auto,rw,uid=pi,gid=pi,iocharset=utf8  0  0

tmpfs /media/ramdisk tmpfs defaults,size=200M 0 0
```

À noter que la configuration du montage d'un disque Samba est toujours un peu laborieux et dépendera de votre serveur. J'ai mis deux configurations différentes. J'ai dû installer les paquets Samba et redémarrer le PI (et oui!) pour que ça fonctionne.

```shell
sudo apt-get install cifs-utils samba samba-client
```

J'ai également créé un disque virtuel en RAM de façon à ce que les logs générés par rclone n'entament pas trop vite la vie de la SD Card. Ici, il fait 200Mo.

Les répertoires de montage dans /media doivent exister (mkdir /media/nas) et les droites d'accès donnés à l'utilisateur 'pi' (sudo chown -R pi:pi /media/nas).

Tentez de monter les disques 'mount /media/nas' et sauvegardez votre fstab. Redémarrez le PI pour s'assurer que les disques soient bien montés automatiquement après un re-démarrage.

# Installation de rclone

Rclone est le rsync du Cloud : il va vous synchroniser un disque source vers une destination dans les nuages. Il gère un nombre impressionnant de protocoles et de fournisseurs, vous devriez trouver votre bonheur. Dans mon cas, j'utilise PCloud.

On télécharge, on installe et on configure l'accès au nuage

```shell
wget https://downloads.rclone.org/v1.44/rclone-v1.44-linux-arm.deb
sudo dpkg -i rclone-v1.44-linux-arm.deb 
rclone config
rclone lsd remote:
```

La dernière commande permet de liste les disques distant. Testez.

# Script de synchronisation

Maintenant, utilisons l'ordonnaceur CRON pour appeler un script de synchronisation toutes les nuits à 1h00 du matin (merci https://crontab.guru/#0_1_*_*_*). Particularité : il faut éditer le script en admin root en spécifiant qu'il faut l'exécuter pour l'utilisateur 'pi'.

```shell
sudo crontab -u pi -e
0 1 * * * flock -n /home/pi/sync.lock /home/pi/sync.sh
```

Autre particularité, on protège notre lancement de script contre les exécutions multiples avec la commande flock. En effet, une grosse sauvegarde peut durer plusieurs jours avec une connexion de campagne !

Voici donc le script 'sync.sh', à rendre exécutable (chmod +x sync.sh) :

```shell
#!/bin/bash
LOG_FILE="/media/ramdisk/rclone.log" 
RCLONE_CMD="rclone -v --log-file=$LOG_FILE sync"

# Redirect to null device and -f will avoid any problems in case of file does not exist or impossible to delete
rm -f $LOG_FILE 2> /dev/null

# echo "Launching rsync USB disk to PCloud" >> $LOG_FILE rsync -arzPh --exclude=/shared /media/nas/ /media/usbdisk $

echo "Launching rclone remote:Videos" >> $LOG_FILE 
$RCLONE_CMD /media/nas/Videos remote:Videos

echo "Launching rclone remote:Music" >> $LOG_FILE
$RCLONE_CMD /media/nas/Music remote:Music

echo "Launching rclone remote:Pictures" >> $LOG_FILE
$RCLONE_CMD /media/nas/Pictures remote:Pictures
```

Le log vous aidera à comprendre ce qu'il se passe au cas où la synchronisation plante, ou pour véifier que les fichiers ont été vraiment copiés.

La ligne en commentaire permet de synchroniser deux disques avec rsync, je ne l'utilise pas encore dans cette configuration.

Voilà, on n'a plus qu'à quitter le shell du PI et à attendre le lendemain pour vérifier la sauvegarde.
