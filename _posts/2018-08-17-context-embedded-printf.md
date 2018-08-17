---
layout: post
title: "Une implémentation de printf() embarqué avec contexte"
date: 2018-08-17 8:55:00 -0120
comments: false
published: true
category:
- embedded
---

> Je ne connais pas de logiciel embarqué sans une utilisation de printf(), au moins pour des besoins de débogage. La version proposée ici permet de sortir des données sur différents périphériques dans un même logiciel.

# Présentation

Soit on utilise la fonction de la librairie standard, soit une fonction "maison" si on a besoin d'optimisation, par exemple en retirant le support des flottants.

Beaucoup d'implémentations utilisent un gros buffer, forcément limité, pour traiter le traitement du formattage. C'est dommage, surtout dans l'embarqué où la RAM est souvent limitée. La version que je vous propose sort les charactères à la volée, c'est à dire directement après traitement, donc l'usage en RAM est très limité.

En outre, il est souvent pratique de sortir du printf() vers de multiples périphériques : un UART, un écran LCD, un module RF, un fichier en émoire Flash ... Ma version de printf() se base donc sur un contexte d'entrée permettant de garder un coeur de code générique.

A noter que je n'ai pas implémenté en lui-même le traitement du formattage, vous trouverez beaucoup d'implémentations différentes. Vous pouvez à loisir le changer, en essayant de choisir une implémentation sans buffer tampon.

# Le code

Le code est disponible là :

[github](https://github.com/brasserie/sys_printf)

Voici un exemple d'usage :

```c
#include <stdio.h>

#include "sys_printf.h"

static void display_print_to_screen(const char *format, ...);

int main(void)
{
	display_print_to_screen("Hello, %s!\r\n", "firmware");

	return 0;
}

static void custom_print(char c)
{
	putchar(c);  // custom putchar (uart_writechar(), lcd_write_char() ...)
}

static void display_print_to_screen(const char *format, ...)
{
	sys_print_ctx_t printCtx; // our printf context

	printCtx.pPutc = custom_print; // specific putchar for that interface
	printCtx.len = 0; // init to zero, character counter
	printCtx.maxLen = 255; / max length to push

	int *varg = (int *) (&format);

	(void)sys_printf(&printCtx, 0, varg);
}
```
