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
