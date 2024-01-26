Домашнее задание: работа с mdadm

Задание:
• добавить в Vagrantfile еще дисков
• собрать R0/R5/R10 на выбор
• прописать собранный рейд в конф, чтобы рейд собирался при загрузке
• сломать/починить raid 
• создать GPT раздел и 5 партиций и смонтировать их на диск.

1) Создаём виртальную машину:
vagrant up

2) Подключаемся к ВМ:
vagrant ssh

3) Посмотрим какие блочные устройства у нас есть:

А) [vagrant@otuslinux ~]$ sudo fdisk -l
Disk /dev/sda: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0f434249

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   2099199   2097152    1G 83 Linux
/dev/sda2       2099200 268435455 266336256  127G 8e Linux LVM


Disk /dev/sde: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs_centos8s-root: 125 GiB, 134171590656 bytes, 262053888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs_centos8s-swap: 2 GiB, 2189426688 bytes, 4276224 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
[vagrant@otuslinux ~]$ [vagrant@otuslinux ~]$ sudo fdisk -l
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 250 MiB, 262144000 bytes, 512000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs_centos8s-root: 125 GiB, 134171590656 bytes, 262053888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/cs_centos8s-swap: 2 GiB, 2189426688 bytes, 4276224 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Б)[vagrant@otuslinux ~]$ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0  128G  0 disk
|-sda1                 8:1    0    1G  0 part /boot
`-sda2                 8:2    0  127G  0 part
  |-cs_centos8s-root 253:0    0  125G  0 lvm  /
  `-cs_centos8s-swap 253:1    0    2G  0 lvm  [SWAP]
sdb                    8:16   0  250M  0 disk
sdc                    8:32   0  250M  0 disk
sdd                    8:48   0  250M  0 disk
sde                    8:64   0  250M  0 disk

В)[vagrant@otuslinux ~]$ lsscsi
[2:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda
[3:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdb
[4:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdc
[5:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sdd
[6:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sde

4) Занулим суперблоки:
[vagrant@otuslinux ~]$ sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde

5) Создадим raid 5 из 4х дисков:
[vagrant@otuslinux ~]$ sudo mdadm --create --verbose /dev/md0 -l 5 -n 4 /dev/sd{b,c,d,e}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

6) Проверим, что RAID собрался нормально:
a) [vagrant@otuslinux ~]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sde[4] sdd[2] sdc[1] sdb[0]
      761856 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]

unused devices: <none>

b) [vagrant@otuslinux ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jan 26 08:21:43 2024
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Fri Jan 26 08:22:28 2024
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 01c8e516:3b1e0d8b:5a22a52b:136b01f3
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde

7) Информация о raid:
[vagrant@otuslinux ~]$ sudo mdadm --detail --scan --verbose
ARRAY /dev/md0 level=raid5 num-devices=4 metadata=1.2 name=otuslinux:0 UUID=01c8e516:3b1e0d8b:5a22a52b:136b01f3
   devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde

8) mdadm.conf во вложении

9) "Фейлим" диск
[vagrant@otuslinux raid]$ sudo mdadm /dev/md0 --fail /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md0

10) Информация о RAID:

a) [vagrant@otuslinux raid]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sde[4] sdd[2](F) sdc[1] sdb[0]
      761856 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/3] [UU_U]

unused devices: <none>

b)[vagrant@otuslinux raid]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jan 26 08:21:43 2024
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Fri Jan 26 12:41:34 2024
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otuslinux:0  (local to host otuslinux)
              UUID : 01c8e516:3b1e0d8b:5a22a52b:136b01f3
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       -       0        0        2      removed
       4       8       64        3      active sync   /dev/sde

       2       8       48        -      faulty   /dev/sdd

11) Удалим сломанный диск из массива:
[vagrant@otuslinux raid]$ sudo mdadm /dev/md0 --remove /dev/sdd
mdadm: hot removed /dev/sdd from /dev/md0

12) Добавляем новый диск:
[vagrant@otuslinux raid]$ sudo mdadm /dev/md0 --add /dev/sdd
mdadm: added /dev/sdd

13) Статус RAID:
[vagrant@otuslinux raid]$  cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdd[5] sde[4] sdc[1] sdb[0]
      761856 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]

unused devices: <none>

14) Создаем раздел GPT на RAID:
[vagrant@otuslinux raid]$ sudo parted -s /dev/md0 mklabel gpt

15) Создаем партиции:
[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 0% 30%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 30% 60%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.

16) Создадим ФС:
[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 0% 30%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 30% 60%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 60% 80%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ sudo parted /dev/md0 mkpart primary ext4 80% 100%
Information: You may need to update /etc/fstab.

[vagrant@otuslinux raid]$ for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 227328 1k blocks and 56896 inodes
Filesystem UUID: 768101d9-e6ce-4cef-92b0-d9d5a1372f61
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 228864 1k blocks and 57344 inodes
Filesystem UUID: 9d21daf0-1ebc-4b81-9657-a1616166d924
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801, 221185

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 152064 1k blocks and 38152 inodes
Filesystem UUID: 9dbf2540-46fe-4d6d-9584-deeb4fb7c295
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 150528 1k blocks and 37696 inodes
Filesystem UUID: eb16c207-4373-4b1a-b897-d37b1b4ea055
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

mke2fs 1.45.6 (20-Mar-2020)
The file /dev/md0p5 does not exist and no size was specified.

17) Монтируем по каталогам:
[vagrant@otuslinux raid]$ sudo mkdir -p /raid/part{1,2,3,4}

[vagrant@otuslinux raid]$ for i in $(seq 1 4); do sudo mount /dev/md0p$i /raid/part$i; done

18) Результат:
[vagrant@otuslinux raid]$ lsblk
NAME                 MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   128G  0 disk
|-sda1                 8:1    0     1G  0 part  /boot
`-sda2                 8:2    0   127G  0 part
  |-cs_centos8s-root 253:0    0   125G  0 lvm   /
  `-cs_centos8s-swap 253:1    0     2G  0 lvm   [SWAP]
sdb                    8:16   0   250M  0 disk
`-md0                  9:0    0   744M  0 raid5
  |-md0p1            259:0    0   222M  0 md    /raid/part1
  |-md0p2            259:1    0 223.5M  0 md    /raid/part2
  |-md0p3            259:2    0 148.5M  0 md    /raid/part3
  `-md0p4            259:3    0   147M  0 md    /raid/part4
sdc                    8:32   0   250M  0 disk
`-md0                  9:0    0   744M  0 raid5
  |-md0p1            259:0    0   222M  0 md    /raid/part1
  |-md0p2            259:1    0 223.5M  0 md    /raid/part2
  |-md0p3            259:2    0 148.5M  0 md    /raid/part3
  `-md0p4            259:3    0   147M  0 md    /raid/part4
sdd                    8:48   0   250M  0 disk
`-md0                  9:0    0   744M  0 raid5
  |-md0p1            259:0    0   222M  0 md    /raid/part1
  |-md0p2            259:1    0 223.5M  0 md    /raid/part2
  |-md0p3            259:2    0 148.5M  0 md    /raid/part3
  `-md0p4            259:3    0   147M  0 md    /raid/part4
sde                    8:64   0   250M  0 disk
`-md0                  9:0    0   744M  0 raid5
  |-md0p1            259:0    0   222M  0 md    /raid/part1
  |-md0p2            259:1    0 223.5M  0 md    /raid/part2
  |-md0p3            259:2    0 148.5M  0 md    /raid/part3
  `-md0p4            259:3    0   147M  0 md    /raid/part4
