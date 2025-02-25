# Part II. Gotta get chrooty

➜ **`chroot` c'est le vieux de la vieille de l'isolation de processus.** 

Il est là depuis si longtemps le frérot, ça marche toujours aussi bien, mais c'est vrai qu'il paraît limité pour les standards d'isolation d'aujourd'hui.

> *Ca reste un grand classique vous allez pas y couper !*

Le principe de `chroot` c'est de changer l'emplacement de la racine du disque pour un processus donné.

Genre on fait croire à un programme que son dossier `/` c'est genre `/toto/super_chroot/`. Comme ça quand il fait `ls /`, il l'ignore, mais en réalité, il liste le contenu du dossier `/toto/super_chroot/`.

> Pour que ça fonctionne normalement et correctement il n'est pas rare de remettre les dossiers qu'on trouve habituellement à la racine du disque dans `/toto/super_chroot/`

## Sommaire

- [Part II. Gotta get chrooty](#part-ii-gotta-get-chrooty)
  - [Sommaire](#sommaire)
  - [1. Play manually](#1-play-manually)
  - [2. SSH old friend](#2-ssh-old-friend)

## 1. Play manually

🌞 **Créez le dossier `/srv/get_chrooted/`**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo mkdir -p /srv/get_chrooted
```

🌞 **Essayez de `chroot` à l'intérieur en lançant un shell**

Création des dossiers bin et lib64 dans le dossier get_chrooted :

```
sudo mkdir -p /srv/get_chrooted/bin
sudo mkdir -p /srv/get_chrooted/lib64
```

Copie de l'executable bash dans le bin de get_chrooted et des libs de bash (après avoir fait ldd) dans lib64 de get_chrooted :
```
sudo cp /bin/bash /srv/get_chrooted/bin/
sudo cp /lib64/libtinfo.so.6 /srv/get_chrooted/lib64/
sudo cp /lib64/libc.so.6 /srv/get_chrooted/lib64/
sudo cp /lib64/ld-linux-x86-64.so.2 /srv/get_chrooted/lib64/
```

Test :
```
[Draskov@efrei-xmg4agau1 ~]$ sudo chroot /srv/get_chrooted /bin/bash
bash-5.1# echo "lol"
lol
```

- possible que ça fonctionne pas immédiatement car y'a pas de shells dans votre `chroot` :d
- déplacez le nécessaire dans `/srv/get_chrooted/` pour pouvoir lancé un shell `chroot`é à l'intérieur

![chroot](./img/chroot.gif)

## 2. SSH old friend

Keskivien foutr là tu vas me dire. OpenSSH, ce bro, comme d'hab, va nous faire le café.

On peut indiquer dans la conf OpenSSH qu'un nouvel utilisateur doit être automatiquement `chroot`é dans un dossier donné quand il se connecte.

🌞 **Créez un user `imsad`**

🌞 **Modifier la configuration du serveur SSH**

```
PS C:\Users\Bapti> ssh imsad@192.168.56.102
imsad@192.168.56.102's password:
Last login: Tue Feb 25 14:47:51 2025 from 192.168.56.1
-bash-5.1$ ls
-bash: ls: command not found
```

