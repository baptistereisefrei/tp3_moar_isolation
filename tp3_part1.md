# Part I : A bit of exploration

Commençons par un peu d'exploration manuelle des pseudo-filesystems que sont `/proc` et `/sys`.

⚠️⚠️⚠️ **Vous n'utiliserez que les commandes `cat`, `ls` et `cd` (ou commandes similaires comme du `grep` bien sûr) pour réaliser cette partie.**

![cat /proc](./img/cat_proc.png)

## Sommaire

- [Part I : A bit of exploration](#part-i--a-bit-of-exploration)
  - [Sommaire](#sommaire)
  - [1. /proc](#1-proc)
  - [2. /sys](#2-sys)

## 1. /proc

🌞 **Afficher...** :
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/meminfo
MemTotal:         467904 kB
MemFree:          219144 kB
MemAvailable:     318640 kB
Buffers:            2708 kB
Cached:            86260 kB
SwapCached:            0 kB
Active:           139112 kB
Inactive:          15024 kB
...
```
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/cpuinfo | grep -c "^processor"
1
```
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/loadavg | awk '{print $4}' | cut -d'/' -f2
120
```
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/cmdline
BOOT_IMAGE=(hd0,msdos1)/vmlinuz-5.14.0-503.22.1.el9_5.x86_64 root=/dev/mapper/rl-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rl-swap rd.lvm.lv=rl/root rd.lvm.lv=rl/swap
```
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/net/tcp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode                        
   0: 00000000:0050 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 20855 1 0000000000000000 100 0 0 10 0
   1: 00000000:0016 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 20226 1 0000000000000000 100 0 0 10 0
   2: 6638A8C0:0016 0138A8C0:F9A0 01 00000000:00000000 02:000A9DCD 00000000     0        0 22199 3 0000000000000000 22 4 23 10 -1
```
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/sys/vm/swappiness
60
```

## 2. /sys

> N'oubliez jamais les pages du `man`, c'est un très bonne doc souvent. [Là encore pour **sysfs** (`/sys`)](https://man7.org/linux/man-pages/man5/sysfs.5.html).

🌞 **Afficher...** :
```
[Draskov@efrei-xmg4agau1 ~]$ ls /sys/block
dm-0  dm-1  sda
```
```
[Draskov@efrei-xmg4agau1 ~]$ cat /proc/modules
nft_fib_inet 12288 1 - Live 0x0000000000000000
nft_fib_ipv4 12288 1 nft_fib_inet, Live 0x0000000000000000
nft_fib_ipv6 12288 1 nft_fib_inet, Live 0x0000000000000000
nft_fib 12288 3 nft_fib_inet,nft_fib_ipv4,nft_fib_ipv6, Live 0x0000000000000000
nft_reject_inet 12288 6 - Live 0x0000000000000000
...
```
```
[Draskov@efrei-xmg4agau1 ~]$ ls /sys/class/net
enp0s3  enp0s8  lo
```