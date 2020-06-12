---
layout: post
title: "Monter automatiquement une clé USB et le détecter en C"
date: 2020-06-12 8:55:00 -0120
comments: false
published: true
category:
- linux
- programming
---

> Dans le cadre d'un projet de kiosque pour un de mes clients, j'ai eu besoin de gérer le cas d'une mise à jour du logiciel sur site. Le cas d'usage est le suivant : un utilisateur branche une clé USB contenant la mise à jour, elle est détectée par l'application et une indication apparaît à l'écran invitant l'utilisateur à lancer cette mise à jour.

# Auto-montage du disque

L'architecture de cet automontage fait appel à trois outils ; l'architecture générale est la suivante :

![image]({{ site.url }}/assets/articles/automount-usbdrive/archi.png)


Dans un premier temps, nous allons installer le logiciel 'pmount' qui sera la pierre angulaire de notre fonctionnel.

```shell
sudo apt install pmount
```

Puis, nous créons un service systemd :

```shell
sudo nano /etc/systemd/system/usb-mount@.service
```

Le symbole @ dans le nom du fichier est pour les services spéciaux qui peuvent posséder plusieurs instances de lancées. Par exemple, getty@.service est un service qui fournit des instances de terminaux textuels. Lorsque vous pressez Ctrl+Alt+F2, getty@tty2.service est démarré en créant le terminal virtuel #2.

Ce qui suit le caractères @ est donc founit en argument au service, qui peut ensuite être récupérer par le script avec %i, comme montré ici :


```ini
[Unit]
Description=Mount USB Drive on %i
[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/usr/bin/pmount --umask 000 /dev/%i /media/%i
ExecStop=/usr/bin/pumount /dev/%i
```

Ce script appellera donc pmount, notre disque sera monté dans /media.

Enfin, dernière manipulation, il faut créer une règle Udev : c'est lui qui détectera l'insertion du matériel et qui appellera le service systemd.

```shell
sudo nano /etc/udev/rules.d/99-usb-mount.rules
```

```shell
ACTION=="add",KERNEL=="sd[a-z][0-9]*",SUBSYSTEMS=="usb",RUN+="/bin/systemctl start usb-mount@%k.service"
ACTION=="remove",KERNEL=="sd[a-z][0-9]*",SUBSYSTEMS=="usb",RUN+="/bin/systemctl stop usb-mount@%k.service"
```

# Capturer les événements de udev en C/C++

Dans un premier temps, nous allons ajouter la librairie udev de développement :

```shell
sudo apt-get install libudev-dev
```

Voici notre thread qui va s'occuper de monitorer le système. La librairie udev nous donne à dsposition un handler de fichier que nous allons surveiller avec la command bien connue 'select', et oui c'est la magie d'unix, tout est fichier donc standard !

Dès lors, lorsqu'un événement sur ce fichier survient, on récupère l'information dudit événement et on exécute notre code, ici on réagit face à un ajout de partition.

```cpp
#include <libudev.h>

void UpdateController::Run()
{
    struct udev *udev;
    udev = udev_new();
    if (!udev)
    {
        printf("Can't create udev\n");
        exit(1);
    }

    struct udev_device *dev;
    struct udev_monitor *mon;
    mon = udev_monitor_new_from_netlink(udev, "udev");
    assert(mon != nullptr);
    /*  int udev_monitor_filter_add_match_subsystem_devtype(struct udev_monitor *udev_monitor,const char *subsystem, const char *devtype);
        filters to select messages that get delivered to a listener.
        On Success it returns an integer greater than, or equal to, 0. On failure, a negative error code is returned.
    */
    assert(udev_monitor_filter_add_match_subsystem_devtype(mon, "block", NULL) >=0);
    assert(udev_monitor_filter_add_match_subsystem_devtype(mon, "usb","usb-device") >=0);

    udev_monitor_enable_receiving(mon);
    /* Get the file descriptor (fd) for the monitor.
       This fd will get passed to select() */

    int fd = udev_monitor_get_fd(mon);


    /* Begin polling for udev events. Events occur when devices attached to the system are added, removed, or change state.
       udev_monitor_receive_device() will return a device object representing the device which changed and what type of change occured.

       The select() system call is used to ensure that the call to udev_monitor_receive_device() will not block.

       This section will run continuously, calling usleep() at the end of each pass. This is to  use  udev_monitor in a non-blocking way. */
    while (!mStop)
    {
        /*
            int select(int nfds, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, struct timeval *timeout);

            select()  allows  a  program  to  monitor  multiple  file descriptors,  waiting  until one or more of the file descriptors
            become "ready" for some class of I/O operation.

            Set up the call to select(). In this case, select() will only operate on a single file descriptor, the one associated
            with our udev_monitor. Note that the timeval object is set to 0, which will cause select() to not block. */
        fd_set fds;
        struct timeval tv;
        int ret;

        FD_ZERO(&fds); //clear fds
        FD_SET(fd, &fds);// Add fd to fds
        /*
            The timeout argument specifies the interval that select() should block waiting for a file descriptor to become ready.
            This interval will be rounded up to the system  clock  granularity, and kernel scheduling delays mean that the
            blocking interval may overrun by a small amount.  If both fields of the timeval structure are zero, then select()
            returns immediately. (This is useful for polling.)If timeout is NULL (no timeout), select() can block indefinitely.
        */
        tv.tv_sec = 0;
        tv.tv_usec = 0;
        /*
          nfds specifies how big the list of file descriptors is because the total number can be vast.
          So, if you want to monitor file descriptors 24-31, you'd set nfds to 32.
          man - nfds is the highest-numbered file descriptor in any of the three sets, plus 1.
        */
        ret = select(fd+1, &fds, nullptr, nullptr, &tv);

        /* Check if our file descriptor has received data. */
        if (ret > 0 && FD_ISSET(fd, &fds)) {
            printf("\nselect() says there should be data\n");

            /* Make the call to receive the device.
               select() ensured that this will not block. */
            dev = udev_monitor_receive_device(mon);
            if (dev)
            {

                std::string devtype=udev_device_get_devtype(dev);
                std::string action=udev_device_get_action(dev);
                std::string devnode=udev_device_get_devnode(dev);

                // On filtre sur le type d'événement, ici on ne souhaite que 
                // réagir sur l'ajout d'un périphérique
                if (devtype.compare("partition")==0 && action.compare("add") == 0)
                {
                    printf("Got Device\n");
                    printf("   Node: %s\n", udev_device_get_devnode(dev));
                    printf("   Subsystem: %s\n", udev_device_get_subsystem(dev));
                    printf("   Devtype: %s\n", udev_device_get_devtype(dev));
                    printf("   syspath:%s\n",udev_device_get_syspath(dev));
                    printf("   sysname:%s\n",udev_device_get_sysname(dev));
                    printf("   devpath:%s\n",udev_device_get_devpath(dev));
                    printf("   subsystem:%s\n",udev_device_get_subsystem(dev));
                    printf("   Action: %s\n", udev_device_get_action(dev));


                    printf("A new partition detected at %s\n",devnode.c_str());
                    
                    // Appeler ici votre code, 
                    // ...
                }
                udev_device_unref(dev);
            }
            else {
                printf("No Device from receive_device(). An error occured.\n");
            }
        }
        usleep(250*1000);
        printf(".");
        fflush(stdout);
    }

}
```

Pour information, voici le code lancé dès qu'un périphérique USB apparaît. Ici, nous utilisons les nouveaux ajouts au standard C++ qui nous permet de facilement manipuler les répertoires et les fichiers.

```cpp
void UpdateController::ScanForUpdates()
{
    try
    {
        std::vector<std::string> dirs = get_directories("/media"); // petite fonction listant les répertoires

        for (const auto &d : dirs)
        {
            std::vector<std::string> files = get_files(d, R"(monprojet_(\d\.\d\.\d).deb)"); // cette fonction liste les fichiers ayant un motif de nom particulier

            for (const auto &f : files)
            {
                std::string fullPath = std::filesystem::path(d) / f; // slash separator in a path
                TLogInfo("Found DEB package: " + fullPath);
            }
        }
    }
    catch(const std::filesystem::filesystem_error &e)
    {
        TLogError("[UPDATE] Error: " + std::string(e.what()));
    }
}
```

À vous de jouer pour la suite, vous disposez maintenant de l'emplacement d'un fichier contenant la mise à jour. N'oubliez pas d'ajouter un peu de sécurité au niveau de votre package comme l'ajout d'une signature permettante de vérifier son origine.
