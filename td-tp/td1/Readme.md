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
3. Mettez ?? jour la liste des syst??mes et redemarrer `sudo update-grub && reboot`.

## D??marrer le service Xen 

1. Lancer le service Xen avec la commande `sudo /etc/init.d/xencommons start` 
2. V??rifier que le service est correctement lanc?? : `sudo xl list`. Que fait cette commande ? 
3. Quelle quantit?? de ressources est allou?? au `dom0` ? 

## D??marrer une VM paravirtualis??

1. T??l??chargeons l'image d'un syst??me d'exploitation (Ubuntu) : `wget http://cloud-images.ubuntu.com/releases/focal/release/ubuntu-20.04-server-cloudimg-amd64.img -P /home/ubuntu/images/ -O vm.qcow2`
2. Cr??ons le fichier de configuration pour notre futur VM. Par la suite, je suppose que le fichier s'appelle `vm1.cfg` : 
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

Changer le chemin de l'image pour correspondre ?? vos r??pertoires. 

3. Modifions le mot de passe de l'image pour pouvoir y acc??der : 

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

5. D??marrons la VM : `sudo xl create -c /chemin/vers/vm1.cfg` 

6. Que vous donne la commande : `sudo xl list`

7. Pouvez-vous rajouter des coeurs ?? la VM en cours d'ex??cution ? Et la m??moire ? 

## Cr??ation du bridge 

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

3. Lancer `/etc/init.d/networking restart` et v??rifier si le bridge est cr??e


# TP1: Partie 2
## Execution d'une attaque SQLi et d??ni de service

## La plateforme 

1. D??veloppez un formulaire web de connexion simple avec les champs (**Email** et **Mot de Passe**) et un bouton **Connexion**. Vous utiliserez le langage/framework de votre choix.
2. ??crivez le code serveur qui sera appel?? pour authentifier un(e) utilisateur(ice). La fonction devra regarder dans une base de donn??es SQL (Mysql, Postgres, ou SQLite), pour authentifier un utilisateur. Vous utiliserez la m??thode na??ve, c.-??-d. en concat??nant les entr??es de l'utilisateur dans une chaine de caract??re qui fera la requ??te d'extraction.
3. D??ployez ce service dans votre VM et assur?? vous que votre service est accessible ?? partir de l'h??te.

## SQLi Testing

1. Exploiter la faille de votre protocole d'authentification afin d'effectuer une attaque qui remplacera le mot de passe administrateur de votre base de donn??es ?? **SangaNdoleOkok2038**. 
Validez votre attaque en vous connectant manuellement ?? votre base de donn??es.

2. R??alisez une deuxi??me attaque qui aura pour but de, vous authentifiez quelque soit les informations que vous allez entrer dans le syst??me. 

3. Que pr??conisez-vous comme solution pour emp??cher ce genre d'attaques ?

## Correction et co??t sur la performance

1. Modifier le code serveur pour utiliser des *prepared request/statements* pour g??rer l'authentification. 
2. Ex??cuter vos deux attaques pr??c??dentes et v??rifier si elles aboutissent toujours.

Maintenant, on va ??valuer le cout de cette correction sur la performance de notre service d'authentification. 

3. Pour cela, ??crivez un script qui effectuera $n$ requ??tes d'authentification par seconde, c.-??-d. envoi du couple (*email+password*) et mesure (i) la latence d'authentification moyenne, (ii) le 95 et 99??? percentile. 
4. Exploiter votre script pour obtenir les trois mesures pour n=1, 50, et 100, sur une dur??e de 5 minutes (300s).
Vous collectez les donn??es pour la m??thode d'authentification ?? risque et celle utilisant les *prepared request/statements*. 
5. Tracer les r??sultats obtenus sur une courbe (je vous pr??conise des diagrammes de bars comme la figure ci-dessous) et commenter les r??sultats obtenus.

![Graphique pour les courbes](./graphs.PNG)

6. Aurions-nous la m??me tendance (d'apr??s vos observations) si le service ??tait d??ploy?? directement sans couche de virtualization ? V??rifier votre r??ponse en d??ployant et en ??valuation le m??me service sur la machine physique.


## D??ni de service

?? pr??sent, nous allons essayer d'effectuer un d??ni de service sur votre application d'authentification simple. 

1. ??crire un script qui d??marre plusieurs clients en parall??le avec chaque client qui effectue 100 requ??tes d'authentification par seconde. Pour chaque requ??te, une valeur al??atoire de l'email et mot de passe doivent ??tre g??n??r??. 
2. Exploitez le script que vous avez ??crit plus haut pour obtenir les m??triques (latence moyenne, 95 et 99??? percentiles) pour ??valuer ?? partir de quel nombre de clients, la latence et les percentiles chutent de plus de 40% (compar?? ?? quand il y a un seul client).
3. Modifier le code serveur pour utiliser un pool de connexion.
4. Refaites la question 2, pour voir si le nombre de clients (n??cessaire pour faire chuter la latence et percentiles de plus de 40%) a augment?? ou baisser.
5. Quel impact l'utilisation du pool a t'il eu sur la latence de votre service d'authentification ?
6. Est-ce que ce d??ni de service emp??che votre VM d'??tre r??actif sur le r??seau ? Tester votre hypoth??se avec un **ping** et commenter vos r??sultats.



