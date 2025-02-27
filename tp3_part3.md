# Part III : CGroup

➜ **Les *CGroup* c'est un mécanisme du noyau Linux pour faire des groupes de processus et restreindre leur accès aux ressources de la machine.**

On parle ici notamment de restreindre l'accès à :

- la mémoire (par exemple : on définit une quantité de mémoire maximum que le groupe de processus aura le droit d'utiliser)
- le CPU (par exemple : on définit un poids pour qu'un groupe de processus soit prioritaire)
- écriture/lecture (par exemple : limitation des écritures sur le disque)
- d'autres trucs, c'pour vous donner une idée

➜ **Les *CGroup* permettent aussi de monitorer en temps réel l'accès aux ressources d'un groupe de processus.**

Vous pouvez constater ça avec les commandes `systemd-cgls` et `systemd-cgtop` notamment.

![Cgroups](./img/cgroup.jpg)

> C'est dessiné par **[Julia Evans](https://jvns.ca/)**, elle a fait plein de ptites illustrations tech de ce genre, je recommande :D

## Sommaire

- [Part III : CGroup](#part-iii--cgroup)
  - [Sommaire](#sommaire)
  - [1. Explore](#1-explore)
  - [2. Do it](#2-do-it)
  - [3. systemd](#3-systemd)
    - [A. One-shot](#a-one-shot)
    - [B. Real service](#b-real-service)

## 1. Explore

Pour rappel : la configuration actuelle des *CGroups* est dispo dans `/sys/fs/cgroup`.

🌞 **Afficher...** :
```
[root@efrei-xmg4agau1 ~]# ls /sys/fs/cgroup
cgroup.controllers      cpuset.cpus.isolated   io.cost.qos                    sys-kernel-config.mount
cgroup.max.depth        cpuset.mems.effective  io.stat                        sys-kernel-debug.mount
cgroup.max.descendants  cpu.stat               memory.numa_stat               sys-kernel-tracing.mount
cgroup.procs            cpu.stat.local         memory.reclaim                 system.slice
cgroup.stat             dev-hugepages.mount    memory.stat                    user.slice
cgroup.subtree_control  dev-mqueue.mount       misc.capacity
cgroup.threads          init.scope             misc.current
cpuset.cpus.effective   io.cost.model          sys-fs-fuse-connections.mount
```
```
[root@efrei-xmg4agau1 cgroup]# ls | grep memory
memory.numa_stat
memory.reclaim
memory.stat

Pas de memory.max
```
```
[root@efrei-xmg4agau1 ~]# find /sys/fs/cgroup -type d
/sys/fs/cgroup
/sys/fs/cgroup/sys-fs-fuse-connections.mount
/sys/fs/cgroup/sys-kernel-config.mount
/sys/fs/cgroup/sys-kernel-debug.mount
/sys/fs/cgroup/dev-mqueue.mount
/sys/fs/cgroup/user.slice
...
```

## 2. Do it

🌞 **Créer un nouveau *CGroup*** :
```
sudo mkdir /sys/fs/cgroup/meow
```

```
[root@efrei-xmg4agau1 meow]# ls
cgroup.controllers      cpu.max                          cpu.weight.nice      memory.stat
cgroup.events           cpu.max.burst                    memory.current       memory.swap.current
cgroup.freeze           cpuset.cpus                      memory.events        memory.swap.events
cgroup.kill             cpuset.cpus.effective            memory.events.local  memory.swap.high
cgroup.max.depth        cpuset.cpus.exclusive            memory.high          memory.swap.max
cgroup.max.descendants  cpuset.cpus.exclusive.effective  memory.low           memory.swap.peak
cgroup.procs            cpuset.cpus.partition            memory.max           memory.zswap.current
cgroup.stat             cpuset.mems                      memory.min           memory.zswap.max
cgroup.subtree_control  cpuset.mems.effective            memory.numa_stat     pids.current
cgroup.threads          cpu.stat                         memory.oom.group     pids.events
cgroup.type             cpu.stat.local                   memory.peak          pids.max
cpu.idle                cpu.weight                       memory.reclaim       pids.peak
[root@efrei-xmg4agau1 meow]# cat cgroup.controllers
cpuset cpu memory pids
```
```
[root@efrei-xmg4agau1 meow]# cat cgroup.subtree_control
cpuset cpu memory
```
🌞 **Créer un nouveau sous-CGroup** :
```
[root@efrei-xmg4agau1 meow]# sudo mkdir /sys/fs/cgroup/meow/task1
[root@efrei-xmg4agau1 meow]# cat /sys/fs/cgroup/meow/task1/cgroup.controllers
cpuset cpu memory
```

🌞 **Mettez en place une limitation RAM**
```
[root@efrei-xmg4agau1 meow]# echo $((150 * 1024 * 1024)) | sudo tee /sys/fs/cgroup/meow/task1/memory.max
157286400
[root@efrei-xmg4agau1 meow]# cat /sys/fs/cgroup/meow/task1/memory.max
157286400
```

🌞 **Prouvez que la limite est effective**
```
[Draskov@efrei-xmg4agau1 ~]$ stress-ng --vm 1 --vm-bytes 90%
```
```
[Draskov@efrei-xmg4agau1 ~]$ free -m
               total        used        free      shared  buff/cache   available
Mem:             456         426           8          12          46          30
Swap:            819          81         738
```
```
[Draskov@efrei-xmg4agau1 ~]$ echo $$ | sudo tee /sys/fs/cgroup/meow/task1/cgroup.procs
802
```
```
[Draskov@efrei-xmg4agau1 ~]$ stress-ng --vm 1 --vm-bytes 200M
stress-ng: info:  [6253] defaulting to a 1 day, 0 secs run per stressor
stress-ng: info:  [6253] dispatching hogs: 1 vm
```
```
Every 0.1s: ps -eo cmd,rss,pid | grep stress        efrei-xmg4agau1.etudiants.campus.villejuif: Thu Feb 27 11:01:24 2025
stress-ng --vm 1 --vm-bytes  3072    6253
stress-ng --vm 1 --vm-bytes  1724    6254
stress-ng --vm 1 --vm-bytes 151324   6255
grep stress                  2048    6768
```
🌞 **Créer un nouveau sous-*CGroup*** :
```
[Draskov@efrei-xmg4agau1 ~]$ sudo mkdir /sys/fs/cgroup/meow/task2
```

🌞 **Appliquer des restrictions CPU** :
```
[Draskov@efrei-xmg4agau1 ~]$ echo 50 | sudo tee /sys/fs/cgroup/meow/task1/cpu.weight
50
[Draskov@efrei-xmg4agau1 ~]$ echo 200 | sudo tee /sys/fs/cgroup/meow/task2/cpu.weight
200
```
```
shell 1:
echo $$ | sudo tee /sys/fs/cgroup/meow/task1/cgroup.procs
stress-ng --cpu 1 --cpu-method=div16
```
```
shell 2:
echo $$ | sudo tee /sys/fs/cgroup/meow/task2/cgroup.procs
stress-ng --cpu 1 --cpu-method=div16
```
```
top - 11:59:30 up  1:28,  3 users,  load average: 0.70, 0.26, 0.10
Tasks: 112 total,   3 running, 109 sleeping,   0 stopped,   0 zombie
%Cpu(s): 99.3 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.7 hi,  0.0 si,  0.0 st
MiB Mem :    456.9 total,    208.6 free,    116.0 used,    169.4 buff/cache
MiB Swap:    820.0 total,    771.7 free,     48.3 used.    341.0 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   8187 Draskov   20   0   33380   1980   1536 R  79.4   0.4   0:15.35 stress-ng-cpu
   8185 Draskov   20   0   33380   1980   1536 R  19.9   0.4   0:21.30 stress-ng-cpu
```
- utilisez la mécanique de `cpu.weight` pour définir des priorités différentes à `task1` et `task2`
- utilisez `stress-ng` ou un bon vieux `cat /dev/random` pour lancer un processus CPU-intensive, et pour prouver que la restriction est en place
- pour tester, vous devez :
  - lancer deux shells en même temps
  - ajouter le premier au *CGroup* `task1`
  - ajouter le deuxième au *CGroup* `task2`
  - dans les deux shells, lancer un processus CPU-intensive
  - constatez avec un `htop` par exemple que les deux processus ne se répartissent pas équitablement la puissance du CPU

## 3. systemd

Par défaut, sous Linux, y'a systemd (ouais encore lui). Il utilise pas mal les *CGroup* nativement. Il fait naturellement deux trucs :

- **il met les `service` dans des `slice`s**
  - créer un `slice` systemd, c'est juste créer un cgroup (dont le nom se terminera par `.slice`)
  - tous les `service`s sont forcément dans un `slice`
- **il met des programmes lancés à l'extérieur (genre des trucs qui sont pas des `services`) dans des `scope`**
  - c'est pareil : un `scope` c'est juste un CGroup créé automatiquement par systemd, dont le nom se termine par `.scope`
  - ça permet à systemd de gérer certains processus même si c'est pas lui qui les lance
  - par exemple quand t'ouvres une session SSH n_n

Bref, l'un des rôles principaux de systemd c'est lancer et gérer des processus (services). Il est donc tout naturel qu'il utilise les *CGroups* Linux pour mener à bien ce job.

### A. One-shot

Y'a une commande rigolote et parfois pratique qui permet de jouer avec tout ça. Une commande qui permet de créer un service système temporaire à la volée en une seule ligne de commande : `systemd-run`.

> Pour plusieurs tests dans le TP, on se servira du serveur Web embarqué par Python. Il se lance en une seule commande, fait des trucs sur le système (serveur web, donc il lit des fichiers, fait des connexions réseau), c'est parfait pour faire des tests ! Pour le lancer : `python -m http.server 8888`.

🌞 **Lancer un serveur Web Python sous forme de service temporaire**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo systemd-run --unit=python_web -d python3 -m http.server 8888
Running as unit: python_web.service
[Draskov@efrei-xmg4agau1 ~]$ systemctl status python_web
● python_web.service - /bin/python3 -m http.server 8888
     Loaded: loaded (/run/systemd/transient/python_web.service; transient)
  Transient: yes
     Active: active (running) since Thu 2025-02-27 12:16:16 CET; 10s ago
   Main PID: 8270 (python3)
      Tasks: 1 (limit: 2664)
     Memory: 11.9M
        CPU: 47ms
     CGroup: /system.slice/python_web.service
             └─8270 /bin/python3 -m http.server 8888

Feb 27 12:16:16 efrei-xmg4agau1.etudiants.campus.villejuif systemd[1]: Started /bin/python3 -m http.server 8888.
```

```bash
# lancer un sleep sous la forme d'un service nommé meow_test.service
sudo systemd-run -u meow_test sleep 9999
```

🌞 **Appliquer à la volée des restrictions**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo systemd-run --unit=limited_service -p MemoryMax=234M sleep 9999
Running as unit: limited_service.service
[Draskov@efrei-xmg4agau1 ~]$ systemctl status limited_service
● limited_service.service - /bin/sleep 9999
     Loaded: loaded (/run/systemd/transient/limited_service.service; transient)
  Transient: yes
     Active: active (running) since Thu 2025-02-27 12:21:12 CET; 10s ago
   Main PID: 8331 (sleep)
      Tasks: 1 (limit: 2664)
     Memory: 204.0K (max: 234.0M available: 233.8M)
        CPU: 1ms
     CGroup: /system.slice/limited_service.service
             └─8331 /bin/sleep 9999

Feb 27 12:21:12 efrei-xmg4agau1.etudiants.campus.villejuif systemd[1]: Started /bin/sleep 9999
```
🌞 **Restrictions *CGroup* ?**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo grep -nri $(( 234 * 1024 * 1024 )) /sys/fs/cgroup/ 2>/dev/null
/sys/fs/cgroup/system.slice/limited_service.service/memory.max:1:245366784
```

### B. Real service

🌞 **Créez un service `web.service`**
```
[Draskov@efrei-xmg4agau1 ~]$ sudo nano /etc/systemd/system/web.service
```

🌞 **Restriction *CGroup***
```
[Unit]
Description=Python Web Server
After=network.target

[Service]
ExecStart=/usr/bin/python3 -m http.server 9999
Restart=always
User=root
Group=root
WorkingDirectory=/home

# Restrictions CGroup
MemoryMax=321M
CPUQuota=50%
IOWriteBandwidthMax=/dev/sda 1M
IOReadBandwidthMax=/dev/sda 1M

[Install]
WantedBy=multi-user.target
```

🌞 **Prouvez que ces restrictions ont été mises en place avec les *CGroups***
```
[Draskov@efrei-xmg4agau1 ~]$ sudo systemctl daemon-reload
[Draskov@efrei-xmg4agau1 ~]$ sudo systemctl restart web.service
[Draskov@efrei-xmg4agau1 ~]$ sudo systemctl enable web.service
[Draskov@efrei-xmg4agau1 ~]$ systemctl status web.service
● web.service - Python Web Server
     Loaded: loaded (/etc/systemd/system/web.service; enabled; preset: disabled)
     Active: active (running) since Thu 2025-02-27 12:38:08 CET; 9s ago
   Main PID: 8610 (python3)
      Tasks: 1 (limit: 2664)
     Memory: 9.1M (max: 321.0M available: 311.8M)
        CPU: 38ms
     CGroup: /system.slice/web.service
             └─8610 /usr/bin/python3 -m http.server 9999
```
```
[Draskov@efrei-xmg4agau1 ~]$ sudo grep -nri $(( 321 * 1024 * 1024 )) /sys/fs/cgroup/ 2>/dev/null
/sys/fs/cgroup/system.slice/web.service/memory.max:1:336592896
```
