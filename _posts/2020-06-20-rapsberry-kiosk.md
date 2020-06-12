---
layout: post
title: "Paramétrer une Debian Buster en mode kiosque"
date: 2020-06-20 8:55:00 -0120
comments: false
published: false
category:
- linux
- raspberry
---

> Il peut être intéressant d'utiliser le Raspberry PI pour créer un kiosque doté d'un écran tactile. À part la problématique de la carte SD, le système est intéressant. Voici une procédure qui vous permettra de peaufiner la séquence de démarrage de votre applicatif. Dans cet exemple, nous allons démarrer une application Web en plein écran.


# Pré-requis


# PC Linux hôte (de production)

Cet ordinateur est équipé d'un Linux Ubuntu.

Sur le PC hôte (de prod) qui sert à générer le package debian :

```shell
gpg --gen-key
```

# Paramétrage du mode Kiosque

## Installer une image Raspberry sur Le compute module ou sur SD Card

Raspian Lite Buster
Procédure pour le compute module : (FIXME)

## Activer/désactiver des fonctions

```shell
sudo raspi-config
```

  * Activer SSH
  * Activer port série
  * Désactiver console série

En option:

  * Locale en français 
  * Clavier en français

## Installer les paquets nécessaires


```shell
sudo apt-get install lightdm xinit openbox chromium-browser unclutter fbi plymouth
```

```shell
sudo raspi-config
```

Mettre la résolution à 1920x1080
Boot: login automatique Desktop


Edit /etc/xdg/openbox/autostart and replace its content with the following:

```shell
# Disable any form of screen saver / screen blanking / power management
xset s off
xset s noblank
xset -dpms

# matchbox-keyboard --daemon &

# Start Chromium in kiosk mode

sed -i 's/"exited_cleanly": false/"exited_cleanly": true/' ~/.config/chromium-browser Default/Preferences
chromium-browser --noerrdialogs --kiosk --app=http://127.0.0.1:8081 --disable-translate
```
Remarque : l'option --incognito de chrome supprime les extensions, et donc le clavier virtuel. Ne pas utiliser.

/boot/cmdline.txt 

```
dwc_otg.lpm_enable=0 console=tty3 loglevel=0 root=PARTUUID=76cbabc0-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait logo.nologo consoleblank=0 quiet splash vt.global_cursor_default=0 plymouth.ignore-serial-consoles fbcon=map:2
```

changer console=tty1 en tty3 puis ajouter à la fin de la ligne :
```
loglevel=0 logo.nologo consoleblank=0 quiet splash vt.global_cursor_default=0 plymouth.ignore-serial-consoles fbcon=map:2
```

/boot/config.txt 


```
#### KIOSK ####

dtoverlay=pi3-disable-wifi

dtoverlay=pi3-disable-bt
 
disable_splash=1

# temperature icon
avoid_warnings=1

disable_overscan=1


##############
```


## Réglage de l'écran (dépend de l'écran utilisé)

Overscan and HDMI Modes

In some cases the image may not fill the screen.

The overscan setting below can help, as well as forcing certain HDMI modes as document [here](From https://www.raspberrypi.org/documentation/configuration/config-txt/README.md) and here.

Edit /boot/config.txt to change these settings, for example hdmi_mode=16
is 1080p@60Hz when hdmi_group=1 (CEA).

disable_overscan=1
hdmi_group=1
hdmi_mode=16


## Désactiver le screen saver

 Force the screen to stay on

```
sudo nano /etc/lightdm/lightdm.conf
```
Add the following lines to the [SeatDefaults] section:

```
# don't sleep the screen
xserver-command=X -s 0 dpms
```

## Sécurité USB : désactivation des périphériques type HID (souris, clavier)


Create a file called, e.g., usbhid.conf in /etc/modprobe.d/ and add the following line:

```shell
blacklist usbhid
```

Then re-generate your initramfs with:

```shell
update-initramfs -u -k $(uname -r)
```

or, if you want to rebuild the initramfs for all installed kernel versions:

```shell
update-initramfs -u -k all
```

After you reboot, usbhid.ko will be prevented from loading. This will persist for any new kernel versions you install until you either delete the /etc/modprobe.d/usbhid.conf file or comment out the blacklist line it contains (of course, you have to re-generate the initramfs again).

BTW if you need to attach a USB kbd/mouse to work on the console for any reason, you can ssh in and run (as root):

```shell
insmod /lib/modules/$(uname -r)/kernel/drivers/hid/usbhid/usbhid.ko
```

and plug the keyboard/mouse into a usb socket. Unlike modprobe, the insmod command ignores any entries (incl. blacklist and module options) in /etc/modprobe.d/.

Don't forget to rmmod usbhid when you don't need to use the kbd/mouse any more.

## RAM FS

```shell
echo "deb http://packages.azlux.fr/debian/ buster main" | sudo tee /etc/apt/sources.list.d/azlux.list
wget -qO - https://azlux.fr/repo.gpg.key | sudo apt-key add -
apt update
apt install log2ram
```

Par défaut, les logs sont écrits sur le disque tous les jours. On peut le changer facilement en changeant la périodicité de ce script Cront, par exemple en déplacement le ficher /etc/cron.daily/log2ram vers /etc/cron.weekly/log2ram.

Vérifier que log2ram fonctionne bien. Avec la commande df, vous devriez notament voir le montage spécial du dossier /var/log :
```shell
/dev/root           30391116 3760324   25659460  13% /
devtmpfs              469544       0     469544   0% /dev
tmpfs                 474152       0     474152   0% /dev/shm
tmpfs                 474152    6676     467476   2% /run
tmpfs                   5120       4       5116   1% /run/lock
tmpfs                 474152       0     474152   0% /sys/fs/cgroup
/dev/mmcblk0p1        258095   53034     205061  21% /boot
tmpfs                  94828       0      94828   0% /run/user/1000
log2ram               204800   63340     141460  31% /var/log
```
Si ce n'est pas le cas, log2ram a probablement un problème, notamment si la taille du disque virtuel en RAM est trop petit. Pour cela, il faut comparer la valeur SIZE dans le fichier cat `/etc/log2ram.conf` et la taille de votre répertoire sur votre disque avec la commande `sudo du -sh /var/log`. Tapez systemctrl status log2ram pour voir le message d'erreur.

Dans mon cas, j'ai eu un soucis effectivement de taille, voici la taille réelle du disque, alors que la taille par défaut du fichier de configuration est 40M :

```shell
pi@raspberrypi:~ $ sudo du -sh /var/log  
62M     /var/log
```

Dans mon cas, j'en ai profité pour désactiver l'envoi de mail en cas de problème (qui repose sur le package `mailutils` si vous en avez besoin) et utiliser rsync au lieu de cp.

```ini
# Configuration file for Log2Ram (https://github.com/azlux/log2ram) under MIT license.
# This configuration file is read by the log2ram service

# Size for the ram folder, it defines the size the log folder will reserve into the RAM.
# If it's not enough, log2ram will not be able to use ram. Check you /var/log size folder.
# The default is 40M and is basically enough for a lot of applications.
# You will need to increase it if you have a server and a lot of log for example.
SIZE=200M

# This variable can be set to true if you prefer "rsync" rather than "cp".
# I use the command cp -u and rsync -X, so I don't copy the all folder every time for optimization.
# You can choose which one you want. Be sure rsync is installed if you use it.
USE_RSYNC=true

# If there are some errors with available RAM space, a system mail will be send
# Change it to false and you will have only a log if there is no place on RAM anymore.
MAIL=false

# Variable for folders to put in RAM. You need to specify the real folder `/path/folder` , the `/path/hdd.folder` will be automatically created. Multiple path can be separeted by `;`. Do not add the final `/` !
# example : PATH_DISK="/var/log;/home/test/FolderInRam"
PATH_DISK="/var/log"

# **************** Zram backing conf  *************************************************

# ZL2R Zram Log 2 Ram enables a zram drive when ZL2R=true ZL2R=false is mem only tmpfs
ZL2R=false
# COMP_ALG this is any compression algorithm listed in /proc/crypto
# lz4 is fastest with lightest load but deflate (zlib) and Zstandard (zstd) give far better compression ratios
# lzo is very close to lz4 and may with some binaries have better optimisation
# COMP_ALG=lz4 for speed or Zstd for compression, lzo or zlib if optimisation or availabilty is a problem
COMP_ALG=lz4
# LOG_DISK_SIZE is the uncompressed disk size. Note zram uses about 0.1% of the size of the disk when not in use
# LOG_DISK_SIZE is expected compression ratio of alg chosen multiplied by log SIZE
# lzo/lz4=2.1:1 compression ratio zlib=2.7:1 zstandard=2.9:1
# Really a guestimate of a bit bigger than compression ratio whilst minimising 0.1% mem usage of disk size
LOG_DISK_SIZE=100M
```


Nettoyer un peu /var/log :

find /var/log -type f -name "*.gz" -delete

```shell
pi@raspberrypi:/var/log $ sudo du -sh /var/log
56M     /var/log
```
C'est un peu mieux, mais un peu effrayant. La rotation des logs est elle assurée par logrotate, nous allons donc simiter la taille de tous les logs à 10M avec un historique de rotation d'une semaine :

```
/var/log/dpkg.log {
	monthly
	rotate 1
    size 10M
	compress
	delaycompress
	missingok
	notifempty
	create 644 root root
}
```

On force maintenant la rotation et on affiche le résultat :

```shell
sudo logrotate -f /etc/logrotate.conf
pi@raspberrypi:/var/log $ sudo du -sh /var/log
22M     /var/log
```

C'est mieux ! La taille principale vient de ... mon application, qui tourne en tâche de fond

## Partitionner le disque

(FIXME) Utilité ?? Pour faire des sauvegardes ?


## Nginx (proxy web)

Pour que l'application soit dispo sur le port 80 et à distance (de l'extérieur) :

```shell
sudo nano /etc/nginx/sites-available/default 
```

```shell
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name _;

        location / {
                proxy_pass http://127.0.0.1:8081;
        }
}
 
# Configuration for Websocket
server {
        listen 8083;
        listen [::]:8083;

        location / {
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_pass "http://localhost:8084";
        }
}


```

## Sécurités anti-plantages

Automatically reloading the active page every hour. If the page you're displaying doesn't automatically update itself, this is effectively the same as hitting Ctrl + R every hour. Very crude. Very effective.

## Firewall

```
    iptables -F
    iptables -X
    iptables -t nat -F
    iptables -t nat -X
    iptables -t mangle -F
    iptables -t mangle -X
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
```

Pour vérifier que ça a bien fonctionné, entrez ensuite la commande suivante qui vous affichera les règles en cours dans IPTables (et normalement, il n’y en aura aucune)

    iptables -nvL


# Splash screen au démarrage

etc/systemd/system/splashscreen.service

```ini
[Unit]
Description=Splash screen
DefaultDependencies=no
After=local-fs.target

[Service]
ExecStart=/usr/bin/fbi -d /dev/fb0 --noverbose -a /opt/alpha-airport/splash.jpg
StandardInput=tty
StandardOutput=tty

[Install]
WantedBy=sysinit.target
```

```
systemctl enable splashscreen 
systemctl start splashscreen
```


## Comment lancer vos services ?


```ini
[Unit]
Description=FELUN server
After=network.target

[Service]
WorkingDirectory=/opt/alpha-airport
ExecStart=/opt/alpha-airport/felun
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=felun
User=pi

[Install]
WantedBy=multi-user.target
```
