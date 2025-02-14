# TP 7 - Boot, services et processus / Tâches d’administration (2) - Eloi Desbrosses

## Exercice 1. Personnalisation de GRUB

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).**
```
serveur@serveur:/etc/default/grub.d$ sudo mv 50-curtin-settings.cfg 50-curtin-settings.cfg.back
```

**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ; passé ce délai, le premier OS du menu doit être lancé automatiquement.**

```
GRUB_TIMEOUT=10
```

**3. Lancez la commande update-grub**

```
serveur@serveur:/etc/default$ sudo update-grub
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.0.0-31-generic
Found initrd image: /boot/initrd.img-5.0.0-31-generic
Found linux image: /boot/vmlinuz-5.0.0-29-generic
Found initrd image: /boot/initrd.img-5.0.0-29-generic
Found linux image: /boot/vmlinuz-5.0.0-27-generic
Found initrd image: /boot/initrd.img-5.0.0-27-generic
done
```

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte**

Les changements ont bien été pris en compte.

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.**

```
# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
GRUB_GFXMODE=1280x800 
```

**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images (après installation, celles-ci sont disponibles dans /usr/share/images/grub).**

```
root@serveur:/etc/default# apt-get install grub2-spashimages
```

**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). Installez-en un.**

```
root@serveur:/etc/default# apt-get install grub2-themes-ubuntu-mate
```

**8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.**

**9. Configurer GRUB pour que le clavier soit en français**

J'ai cassé ma machine.. Grub boot à l'infinis. Pour la réparer il faut que je lance une image en mode rescue, mais la syntax que j'ai trouvé sur internet ne fonctionne pas. à voir.

## Exercice 2. Noyau

Dans cet exercice, on va créer et installer un module pour le noyau.

**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**

**2. Créez un fichier hello.c contenant le code suivant :**

```
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("John Doe");
MODULE_DESCRIPTION("Module hello world");
MODULE_VERSION("Version 1.00");

int init_module(void)
{
printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
return 0;
}

void cleanup_module(void)
{
printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
}
```

**3. Créez également un fichier Makefile :**

```
obj-m += hello.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

install:
cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
```

**4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make install.**

```
root@serveur:~# make
make -C /lib/modules/5.0.0-29-generic/build M=/root modules
make[1]: Entering directory '/usr/src/linux-headers-5.0.0-29-generic'
  CC [M]  /root/hello.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /root/hello.mod.o
  LD [M]  /root/hello.ko
make[1]: Leaving directory '/usr/src/linux-headers-5.0.0-29-generic'
root@serveur:~# make install
cp ./hello.ko /lib/modules/5.0.0-29-generic/kernel/drivers/misc
```

**5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.**

```
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# journalctl | grep "init_module"
oct. 21 09:24:52 serveur kernel: [Hello world] - La fonction init_module() est appelée.
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# lsmod
Module                  Size  Used by
hello                  16384  0
```

**6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez notamment voir les informations figurant dans le fichier C.**

```
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# modinfo hello.ko
filename:       /lib/modules/5.0.0-29-generic/kernel/drivers/misc/hello.ko
version:        Version 1.00
description:    Module hello world
author:         John Doe
license:        GPL
srcversion:     4398A2271F215E3A6F58078
depends:
retpoline:      Y
name:           hello
vermagic:       5.0.0-29-generic SMP mod_unload
```

**7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande lsmod.**

```
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# rmmod hello
root@serveur:/lib/modules/5.0.0-29-generic/kernel/drivers/misc# journalctl | grep "cleanup_module"
oct. 21 09:31:18 serveur kernel: [Hello world] - La fonction cleanup_module() est appelée.
```

**8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.**

Pour des raisons inconnus, je n'arrive plus à charger mon module, même après la recompilation.

## Exercice 3. Interception de signaux

**1. Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par l’utilisateur**

```
#!/bin/bash
while :
do
read keyboard_input
echo $keyboard_input >> tmp.txt
done
```

**2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script poursuive son exécution ?**

Le raccourcis permet de mettre en pause un script en cours d'exécution. Il est cependant possible de résumer l'exécution du script avec ```fg k```

**3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ?**

Le raccourcis CTRL+C permet de stopper complètement l'exécution d'un script.

**4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP (= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en arrière-plan”, et dans le second cas, il doit afficher ”OK, je fais un peu de ménage avant” avant de supprimer le fichier temporaire et terminer le script.**

```
#!/bin/bash
while :
do
read keyboard_input
trap 'echo impossible de me placer en arrière-plan !' TSTP
trap 'echo OK, je fais juste un peu de ménage avant ; exit' INT
trap 'rm tmp.txt' EXIT
echo $keyboard_input >> tmp.txt
done
```

**5. Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre part la commande kill**

La commande kill et les raccourcis trigger tout les deux les évènements programmé dans le script.

**6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre script pour que ce message n’apparaisse plus.**

```
#!/bin/bash
while :
do
read keyboard_input
if [ ! -f FILENAME ]
then
touch tmp.txt
fi

trap 'echo impossible de me placer en arrière-plan !' TSTP
trap 'echo OK, je fais juste un peu de ménage avant ; exit' INT
trap 'rm tmp.txt' EXIT
echo $keyboard_input >> tmp.txt
done
```

## Exercice 4. Surveillance de l’activité du système

**1. Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?**

La commande affiche la liste des utilisateurs connectés avec d'autres informations supplémentaires, tel qu'une IP associés et les processus démarrer par l'utilisateur.

**2. Comment afficher l’historique des dernières connexions à la machine ?**

Il est possible d'accéder à l'historique des connexion à la machine via la commande `last`

**3. Quelle commande permet d’obtenir la version du noyau ?**

Il est possible d'obtenir la version actuel du noyaux via la commande `uname -r`

**4. Comment récupérer toutes les informations sur le processeur, au format JSON ?**

Les informations du processeur sont accessible via la commande `lshw -class processor`. Ont peut définir le format json en rajoutant l'argument `-json` à la fin.

**5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ? Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?**

la commande journalctl peut afficher les logs des démarrages de la machine via la commande:
```
journalctl --list-boots
```

Il est possible de spécifié un numéro de boot, par exemple la commande suivante sélectionne l'avant-dernier boot:
```
journalctl -b -1
```

**6. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?**

Il est possible d'obtenir la liste des derniers démarrages via la commande:
```journalctl --list-boots```

**7. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message à l’écran d’une maintenance le 26 mars à minuit.**

Il est possible de définir le "message of the day" dans le fichier `/etc/motd`. J'ai donc écris le message dans ce fichier.

**8. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2, avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur l’affichage de tload.**
