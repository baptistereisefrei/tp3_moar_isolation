# Part IV : Linux Namespaces

> Le terme *namespace* est utilisé dans plein plein de contextes différents en informatique. Ici, on parle des *namespaces* du noyau Linux (qui n'a rien à votre avec les *namespaces* Java ou Kubernetes ou autres.)

➜ **Les Linux namespaces permettent d'isoler les processus les uns des autres, en leur proposant une vue limitée et restreinte de l'OS.**

Ici on parle pas de l'accès aux ressources (matérielles) de la machine comme avec les CGroups, mais plutôt de l'accès aux features de l'OS.

➜ **Les namespaces isolent notamment les processus en terme de** (liste non-exhaustive) :

- **PID** : les processus voient qu'une partie des autres processus qui s'exécutent
- **network** : les processus ne voient pas toutes les cartes réseau du système
- **user** : les processus n'ont pas les même users (pas les même fichiers `/etc/passwd` et `/etc/shadow` par exemple)
- **mount** : les processus ne voient pas les même partitions et points de montage que les autres

> On peut choisir d'isoler un processus uniquement en terme de network ou uniquement en terme de PID, ou tout à la fois, y'a pas de contraintes à ce niveau là.

![container](./img/container.png)

## Sommaire

- [Part IV : Linux Namespaces](#part-iv--linux-namespaces)
  - [Sommaire](#sommaire)
  - [1. Explore](#1-explore)
  - [2. Create](#2-create)
    - [A. net](#a-net)
    - [B. pid](#b-pid)
  - [3. AND MY CONTAINERS](#3-and-my-containers)
    - [A. Quick install](#a-quick-install)
    - [B. A simple container](#b-a-simple-container)
    - [C. CGroup](#c-cgroup)

## 1. Explore

🌞 **Utiliser /proc**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo ls -l /proc/$$/ns/
total 0
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:03 uts -> 'uts:[4026531838]'

[Draskov@efrei-xmg4agau1 ~]$ sudo ls -l /proc/1/ns/
total 0
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 net -> 'net:[4026531840]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0 Feb 27 13:03 uts -> 'uts:[4026531838]'
```

🌞 **Lister tous les *namespaces* en cours d'utilisation**
```
[Draskov@efrei-xmg4agau1 ~]$ [Draskov@efrei-xmg4agau1 ~]$ sudo lsns
        NS TYPE   NPROCS   PID USER   COMMAND
4026531834 time      109     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531835 cgroup    109     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531836 pid       109     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531837 user      109     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531838 uts       106     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531839 ipc       109     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531840 net       109     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531841 mnt        98     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026531862 mnt         1    23 root   kdevtmpfs
4026532119 mnt         1  8678 root   /usr/lib/systemd/systemd-udevd
4026532120 uts         1  8678 root   /usr/lib/systemd/systemd-udevd
4026532155 mnt         1   621 root   /sbin/auditd
4026532156 mnt         2   647 dbus   /usr/bin/dbus-broker-launch --scope system --audit
4026532157 mnt         1   659 chrony /usr/sbin/chronyd -F 2
4026532158 mnt         1  8686 root   /usr/lib/systemd/systemd-logind
4026532159 uts         1  8686 root   /usr/lib/systemd/systemd-logind
4026532160 uts         1   659 chrony /usr/sbin/chronyd -F 2
4026532161 mnt         1   662 root   /usr/sbin/NetworkManager --no-daemon
4026532238 mnt         2   795 root   nginx: master process /usr/sbin/nginx
4026532240 mnt         1   761 root   /usr/sbin/rsyslogd -n
```

## 2. Create

### A. net

🌞 **Créer un nouveau *namespace* `network`**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo unshare --net bash
```

🌞 **Prouvez que votre nouveau *namespace* est bien là**
```
[root@efrei-xmg4agau1 Draskov]# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
```
[root@efrei-xmg4agau1 Draskov]# echo $$
9103
[Draskov@efrei-xmg4agau1 ~]$ sudo lsns | grep net
4026531840 net       115     1 root   /usr/lib/systemd/systemd --switched-root --system --deserialize 31
4026532242 net         1  9103 root   -bash
```
```
[root@efrei-xmg4agau1 Draskov]# sudo ls -al /proc/$$/ns/net
lrwxrwxrwx. 1 root root 0 Feb 27 13:39 /proc/9103/ns/net -> 'net:[4026532242]'

[Draskov@efrei-xmg4agau1 ~]$ sudo ls -al /proc/9103/ns/net
lrwxrwxrwx. 1 root root 0 Feb 27 13:39 /proc/9103/ns/net -> 'net:[4026532242]'
```
### B. pid

🌞 **Créer un nouveau *namespace* `pid`**

```
[Draskov@efrei-xmg4agau1 ~]$ sudo unshare --mount-proc --pid  --fork
```

🌞 **Prouvez que votre nouveau *namespace* est bien là**
```
[root@efrei-xmg4agau1 Draskov]# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 13:57 pts/0    00:00:00 -bash
root          28       1  0 13:57 pts/0    00:00:00 ps -ef
```
```
[Draskov@efrei-xmg4agau1 ~]$ lsns
        NS TYPE   NPROCS   PID USER    COMMAND
4026531834 time        3  8926 Draskov -bash
4026531835 cgroup      3  8926 Draskov -bash
4026531836 pid         3  8926 Draskov -bash
4026531837 user        3  8926 Draskov -bash
4026531838 uts         3  8926 Draskov -bash
4026531839 ipc         3  8926 Draskov -bash
4026531840 net         3  8926 Draskov -bash
4026531841 mnt         3  8926 Draskov -bash
```
```
[Draskov@efrei-xmg4agau1 ~]$ sudo ls -al /proc/8926/ns/pid
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:37 /proc/8926/ns/pid -> 'pid:[4026531836]'
[Draskov@efrei-xmg4agau1 ~]$ sudo ls -al /proc/$$/ns/pid
lrwxrwxrwx. 1 Draskov Draskov 0 Feb 27 13:50 /proc/9208/ns/pid -> 'pid:[4026531836]'
```

## 3. AND MY CONTAINERS

Oèoè les conteneurs les conteneurs, ils arrivent ils arrivent.

Un outil comme Docker repose **complètement** sur les *namespaces* et les *CGroups* pour lancer des processus de façon isolée sur la machine.

> Maintenant vous comprenez pourquoi j'me vénère quand on me dit que c'est des ptites VMs (genre non, mais alors pas du tout). Aussi pourquoi le conteneur est "isolé" du reste du système. Aussiiiiiiii pourquoi Docker est une techno fondamentalement Linux : Linux *namespaces* + Linux *CGroups* bois&gyals.

### A. Quick install

🌞 **Installer Docker sur la machine**

- suivez les instructions de la doc officielle spécifique pour notre OS !
- une fois installé, ajoutez votre user au groupe docker : `sudo usermod -aG docker $(whoami)`
- et démarrez le service docker ! `sudo systemctl enable --now docker`

### B. A simple container

🌞 **Lancer un simple conteneur qui sleep**

- utilisez la commande suivante :

```bash
docker run -d debian sleep 9999
```

🌞 **Avez les commandes de votre choix, avec le plus de détails possible, prouvez-que :**

- ce processus `sleep` est bien lancé sur votre machine, par votre utilisateur courant
- ce processus `sleep` est isolé à l'aide de *namespaces*
- un *CGroup* a été attribué à ce conteneur
- tout nouveau processus "dans" le conteneur est lui aussi isolé (voir ci-dessous pour lancer d'autres process dans le conteneur)

```bash
# pour obtenir un shell dans un conteneur existant
docker exec -it <conteneur> bash
# par exemple, si le conteneur est nommé "toto"
docker exec -it toto bash

# depuis là, vous pouvez lancez des nouveaux programmes
apt update -y
## procps contient la commande ps
## iproute2 contient la commande ip
## iputils-ping contient la commande ping
apt install -h procps iproute2 iputils-ping
```

### C. CGroup

Et les CGroups dans tout ça ?

🌞 **Lancer un conteneur restreint**

- avec des options du `docker run`
- limiter l'accès RAM de ce conteneur à 456M

🌞 **CGroup ?**

- prouvez que cette limite a été mise en place avec une conf CGroup
- un *CGroup* est automatiquement créé à chaque fois que vous lancez un conteneur (un `scope` systemd) !

![not afraid](./img/nowask.png)

