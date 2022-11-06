# TP1 - Partie 1

## Goal 

The goal of this labwork is to play with VMs and grasp the difficulties around them.
We will create a VM running Ubuntu with the Xen (with its hypervisor).
In that VM, we will try to perform SQLi and Denial of Service.


## Xen install 

There are basically two ways of installing Xen on a Linux kernel operating system. 
The first is to manually build it from sources and the second is to install pre-built binaries. 
We will try both. Useful resources to look can be find here : https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide

Before launching, get the list of supported operating systems and kernels in the grub : `ls /boot`
### Manual install 

1. Download Xen 4.15 from the official website. You can just run `wget https://downloads.xenproject.org/release/xen/4.15.0/xen-4.15.0.tar.gz`.
2. Unzip the archive. You can run `tar xvzf xen-4.15.0.tar.gz`
3. Install the dependencies required by Xen 
`sudo apt update
sudo apt install build-essential bcc bin86 gawk bridge-utils
iproute libcurl3 libcurl4-openssl-dev bzip2 module-init-tools
transfig tgif texinfo texlive-latex-base texlive-latex-recommended
texlive-fonts-extra texlive-fonts-recommended pciutils-dev mercurial
make gcc libc6-dev zlib1g-dev python python-dev python-twisted
libncurses5-dev patch libvncserver-dev libsdl-dev libjpeg62-dev
iasl libbz2-dev e2fslibs-dev git-core uuid-dev ocaml ocaml-findlib
libx11-dev bison flex xz-utils libyajl-dev gettext libpixman-1-dev
libaio-dev markdown pandoc libc6-dev-i386`

**Some libraries may not be supported anymore. Adjust by removing the faulty libraries from the list.**

4. Then, install remaining dependencies. 

`sudo apt build-dep xen`


5. Now enter the Xen folder and run `./configure`. This will configure headers and installation scripts based on your system.

6. Build Xen by running : 

`sudo make -j$(nproc)` *What is the meaning of $(nproc)*
`sudo make install` 
`sudo update-grub` *What is the goal of this command*

7. Check the `/boot` folder. 

8. Reboot the machine and boot with the `Xen hypervisor` 


## Too hard ? Check the prebuilt version 

1. Install Xen from the prebuilt store `sudo apt-get install xen-hypervisor-amd64`
2. Open the file `/etc/default/grub.d` et corriger `XEN_OVERRIDE_GRUB_DEFAULT` avec la valeur `1`
3. Mettez à jour la liste des systèmes et redemarrer `sudo update-grub && reboot`.

## Démarrer le service Xen 

1. Lancer le service Xen avec la commande `sudo /etc/init.d/xencommons start` 
2. Vérifier que le service est correctement lancé : `sudo xl list`. Que fait cette commande ? 
3. Quelle quantité de ressources est alloué au `dom0` ? 

## Démarrer une VM paravirtualisé

1. Téléchargeons l'image d'un système d'exploitation (Ubuntu) : `wget http://cloud-images.ubuntu.com/releases/focal/release/ubuntu-20.04-server-cloudimg-amd64.img -P /home/ubuntu/images/ -O vm.qcow2`
2. Créons le fichier de configuration pour notre futur VM. Par la suite, je suppose que le fichier s'appelle `vm1.cfg` : 
```
 bootloader = 'pygrub'
 vcpus = 2
 memory = 1024
 root = '/dev/xvda1 ro'
 disk = [
  '/home/ubuntu/images/vm.qcow2,qcow2,hda,rw'
 ]
 name = 'myvm'
 vif = [ 'bridge=br0' ] 
 ```

Changer le chemin de l'image pour correspondre à vos répertoires. 

3. Modifions le mot de passe de l'image pour pouvoir y accéder : 

 ```
 modprobe nbd max_part=8
 qemu-nbd --connect=/dev/nbd0 /home/vms/images/vm.qcow2 
 fdisk /dev/nbd0 -l
 mount /dev/nbd0p1 /mnt/
 chroot /mnt/
 passwd
 ```
 
 
Modifier le mot de passe pour mettre celui de votre choix et enregistrer cela 
```
 umount /mnt/
 qemu-nbd --disconnect /dev/nbd0
 rmmod nbd
 ```
 

4. Que pouvez vous dire du fichier de configuration ? Que nous-manque t'il ? 

5. Démarrons la VM : `sudo xl create -c /chemin/vers/vm1.cfg` 

6. Que vous donne la commande : `sudo xl list`

7. Pouvez-vous rajouter des coeurs à la VM en cours d'exécution ? Et la mémoire ? 

## Création du bridge 

1. Ouvrir le fichier `/etc/network/interfaces`
2. Rajoutons ceci 

```
auto br0
iface br0 inet dhcp 
  bridge_ports eth0 
  bridge_stp off       # disable Spanning Tree Protocol
  bridge_waitport 0    # no delay before a port becomes available
  bridge_fd 0   
``` 

3. Lancer `/etc/init.d/networking restart` et vérifier si le bridge est crée


# TP1: Partie 2
## Execution d'une attaque SQLi et déni de service

## La plateforme 

1. Développez un formulaire web de connexion simple avec les champs (**Email** et **Mot de Passe**) et un bouton **Connexion**. Vous utiliserez le langage/framework de votre choix.
2. Écrivez le code serveur qui sera appelé pour authentifier un(e) utilisateur(ice). La fonction devra regarder dans une base de données SQL (Mysql, Postgres, ou SQLite), pour authentifier un utilisateur. Vous utiliserez la méthode naïve, c.-à-d. en concaténant les entrées de l'utilisateur dans une chaine de caractère qui fera la requête d'extraction.
3. Déployez ce service dans votre VM et assuré vous que votre service est accessible à partir de l'hôte.

## SQLi Testing

1. Exploiter la faille de votre protocole d'authentification afin d'effectuer une attaque qui remplacera le mot de passe administrateur de votre base de données à **SangaNdoleOkok2038**. 
Validez votre attaque en vous connectant manuellement à votre base de données.

2. Réalisez une deuxième attaque qui aura pour but de, vous authentifiez quelque soit les informations que vous allez entrer dans le système. 

3. Que préconisez-vous comme solution pour empêcher ce genre d'attaques ?

## Correction et coût sur la performance

1. Modifier le code serveur pour utiliser des *prepared request/statements* pour gérer l'authentification. 
2. Exécuter vos deux attaques précédentes et vérifier si elles aboutissent toujours.

Maintenant, on va évaluer le cout de cette correction sur la performance de notre service d'authentification. 

3. Pour cela, écrivez un script qui effectuera $n$ requêtes d'authentification par seconde, c.-à-d. envoi du couple (*email+password*) et mesure (i) la latence d'authentification moyenne, (ii) le 95 et 99ᵉ percentile. 
4. Exploiter votre script pour obtenir les trois mesures pour n=1, 50, et 100, sur une durée de 5 minutes (300s).
Vous collectez les données pour la méthode d'authentification à risque et celle utilisant les *prepared request/statements*. 
5. Tracer les résultats obtenus sur une courbe (je vous préconise des diagrammes de bars comme la figure ci-dessous) et commenter les résultats obtenus.

![Graphique pour les courbes](./GraphiqueExe.png)

6. Aurions-nous la même tendance (d'après vos observations) si le service était déployé directement sans couche de virtualization ? Vérifier votre réponse en déployant et en évaluation le même service sur la machine physique.


## Déni de service

À présent, nous allons essayer d'effectuer un déni de service sur votre application d'authentification simple. 

1. Écrire un script qui démarre plusieurs clients en parallèle avec chaque client qui effectue 100 requêtes d'authentification par seconde. Pour chaque requête, une valeur aléatoire de l'email et mot de passe doivent être généré. 
2. Exploitez le script que vous avez écrit plus haut pour obtenir les métriques (latence moyenne, 95 et 99ᵉ percentiles) pour évaluer à partir de quel nombre de clients, la latence et les percentiles chutent de plus de 40% (comparé à quand il y a un seul client).
3. Modifier le code serveur pour utiliser un pool de connexion.
4. Refaites la question 2, pour voir si le nombre de clients (nécessaire pour faire chuter la latence et percentiles de plus de 40%) a augmenté ou baisser.
5. Quel impact l'utilisation du pool a t'il eu sur la latence de votre service d'authentification ?
6. Est-ce que ce déni de service empêche votre VM d'être réactif sur le réseau ? Tester votre hypothèse avec un **ping** et commenter vos résultats.



