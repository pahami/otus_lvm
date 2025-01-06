# Тема: Файловые системы и LVM

### Домашняя работа 6-7 LVM (реализована на Ubuntu 22.04)
---
### Домашнее задание:
   - Уменьшить том под / до 8GB.  
   - Выделить том под /home.  
   - Выделить том под /var - сделать в mirror.
   - /home - сделать том для снапшотов. 
   - Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
   - Работа со снапшотами:
     - сгенерить файлы в /home/;
     - снять снапшот;
     - удалить часть файлов;
     - восстановится со снапшота.
   - *На дисках попробовать поставить btrfs/zfs — с кешем, снапшотами и разметить там каталог /opt.
---

Для выполнения задания используется виртуальная машина, сконнфигурированная посредством Vagrantfile

### Выполнение задания:

Т.к. все операции выполнеяются под суперпользователем, то переходим в root

`sudo -i`

посмотрим на диски, выполнив команды: `df -hT` и `lsblk`
<details>
<summary>
   результат выполнения команд
</summary>
   
`df -hT`

```
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs   96M  1.1M   95M   2% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    62G  5.2G   54G   9% /
tmpfs                             tmpfs  479M     0  479M   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  234M  1.6G  13% /boot
tmpfs                             tmpfs   96M  4.0K   96M   1% /run/user/1000
```
`lsblk`
```
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  89.4M  1 loop /snap/lxd/31333
loop1                       7:1    0  53.3M  1 loop /snap/snapd/19457
loop2                       7:2    0  63.7M  1 loop /snap/core20/2434
loop3                       7:3    0  44.3M  1 loop /snap/snapd/23258
loop4                       7:4    0  63.4M  1 loop /snap/core20/1974
loop5                       7:5    0 111.9M  1 loop /snap/lxd/24322
sda                         8:0    0   128G  0 disk 
├─sda1                      8:1    0     1M  0 part 
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0   126G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:1    0    63G  0 lvm  /
sdb                         8:16   0    10G  0 disk 
└─otus-test               253:0    0    10G  0 lvm  
sdc                         8:32   0     2G  0 disk 
└─otus-test               253:0    0    10G  0 lvm  
sdd                         8:48   0     1G  0 disk 
├─vg0-mirror_rmeta_0      253:2    0     4M  0 lvm  
│ └─vg0-mirror            253:6    0   816M  0 lvm  
└─vg0-mirror_rimage_0     253:3    0   816M  0 lvm  
  └─vg0-mirror            253:6    0   816M  0 lvm  
sde                         8:64   0     1G  0 disk 
├─vg0-mirror_rmeta_1      253:4    0     4M  0 lvm  
│ └─vg0-mirror            253:6    0   816M  0 lvm  
└─vg0-mirror_rimage_1     253:5    0   816M  0 lvm  
  └─vg0-mirror            253:6    0   816M  0 lvm  
sdf                         8:80   0   250M  0 disk 
```
</details>

Подготовим стенд и удалим vg предыдущих заданий

`vgremove vg0` и `vgremove otus`

Проверяем повторно:

<details>
<summary>
   результат выполнения команд
</summary>
   
`df -hT`

```
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs   96M 1004K   95M   2% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    62G  5.2G   54G   9% /
tmpfs                             tmpfs  479M     0  479M   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  234M  1.6G  13% /boot
tmpfs                             tmpfs   96M  4.0K   96M   1% /run/user/1000
```
`lsblk`
```
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  89.4M  1 loop /snap/lxd/31333
loop1                       7:1    0  53.3M  1 loop /snap/snapd/19457
loop2                       7:2    0  63.7M  1 loop /snap/core20/2434
loop3                       7:3    0  44.3M  1 loop /snap/snapd/23258
loop4                       7:4    0  63.4M  1 loop /snap/core20/1974
loop5                       7:5    0 111.9M  1 loop /snap/lxd/24322
sda                         8:0    0   128G  0 disk 
├─sda1                      8:1    0     1M  0 part 
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0   126G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:1    0    63G  0 lvm  /
sdb                         8:16   0    10G  0 disk 
sdc                         8:32   0     2G  0 disk 
sdd                         8:48   0     1G  0 disk 
sde                         8:64   0     1G  0 disk 
sdf                         8:80   0   250M  0 disk  
```
</details>
---

### 1. Уменьшим том под / до 8G:

**1.1 Подготовка временного тома для раздела::**
   
`pvcreate /dev/sdb`
<details>
<summary> результат выполнения команды: </summary>

```
  Physical volume "/dev/sdb" successfully created.
```
</details>

Создадим Volume group **vg_root** на разделе **/dev/sdb**:

`vgcreate vg_root /dev/sdb`

<details>
<summary> результат выполнения команды: </summary>

```
  Volume group "vg_root" successfully created
```
</details>

Создадим Logical Volume **lv_root** на Volume Group **vg_root**:

`lvcreate -n lv_root -l +100%FREE /dev/vg_root`

<details>
<summary> результат выполнения команды: </summary>

```
WARNING: ext4 signature detected on /dev/vg_root/lv_root at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/vg_root/lv_root.
  Logical volume "lv_root" created.
```
</details>

Создадим файловую систему **ext4** на созданном Logical volume **lv_root**:

`mkfs.ext4 /dev/vg_root/lv_root`

<details>
<summary> результат выполнения команды: </summary>

```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2620416 4k blocks and 655360 inodes
Filesystem UUID: 709102ea-efc4-4c92-a23f-e07d2139fca5
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```
</details>

Смонтируем раздел **/dev/vg_root/lv_root** в **/mnt**:

`mount /dev/vg_root/lv_root /mnt`

**1.2 Скопируем все данные с раздела / в /mnt:**

`rsync -avxHAX --progress / /mnt/`

<details>
<summary> результат выполнения команды: </summary>

```
sent 5,355,165,626 bytes  received 1,393,666 bytes  164,817,208.98 bytes/sec
total size is 5,639,924,116  speedup is 1.05

```
</details>

Проверим, что содержимое раздела **/** скопировалось в **/mnt**:

`ls -al /mnt`

<details>
<summary> результат выполнения команды: </summary>

```
total 2097252
drwxr-xr-x  21 root root       4096 Jan  3 14:02 .
drwxr-xr-x  21 root root       4096 Jan  3 14:02 ..
lrwxrwxrwx   1 root root          7 Aug 10  2023 bin -> usr/bin
drwxr-xr-x   2 root root       4096 Jan 11  2024 boot
drwxr-xr-x   2 root root       4096 Jan  3 13:48 data
drwxr-xr-x   2 root root       4096 Jan  3 14:02 data-snap
drwxr-xr-x   2 root root       4096 Jan  6 10:02 dev
drwxr-xr-x 102 root root       4096 Jan  6 09:36 etc
drwxr-xr-x   3 root root       4096 Jan 10  2024 home
lrwxrwxrwx   1 root root          7 Aug 10  2023 lib -> usr/lib
lrwxrwxrwx   1 root root          9 Aug 10  2023 lib32 -> usr/lib32
lrwxrwxrwx   1 root root          9 Aug 10  2023 lib64 -> usr/lib64
lrwxrwxrwx   1 root root         10 Aug 10  2023 libx32 -> usr/libx32
drwx------   2 root root      16384 Jan 10  2024 lost+found
drwxr-xr-x   2 root root       4096 Aug 10  2023 media
drwxr-xr-x   2 root root       4096 Jan  6 10:04 mnt
drwxr-xr-x   2 root root       4096 Aug 10  2023 opt
dr-xr-xr-x   2 root root       4096 Jan  6 09:36 proc
drwx------   5 root root       4096 Jan  3 17:46 root
drwxr-xr-x   2 root root       4096 Jan  6 09:36 run
lrwxrwxrwx   1 root root          8 Aug 10  2023 sbin -> usr/sbin
drwxr-xr-x   6 root root       4096 Jan  3 13:28 snap
drwxr-xr-x   2 root root       4096 Aug 10  2023 srv
-rw-------   1 root root 2147483648 Jan 10  2024 swap.img
dr-xr-xr-x   2 root root       4096 Jan  6 09:36 sys
drwxrwxrwt  12 root root       4096 Jan  6 09:41 tmp
drwxr-xr-x  14 root root       4096 Aug 10  2023 usr
drwxr-xr-x  13 root root       4096 Aug 10  2023 var
```
</details>

##### 1.3 Сконфигурируем grub для того, чтобы при старте перейти в новый

Сымитируем текущий root, сделаем в него chroot и обновим grub:

```
for i in /proc/ /sys/ /dev/ /run/ /boot/; \
do mount --bind $i /mnt/$i; done
chroot /mnt/
```

<details>
<summary> результат выполнения команды </summary>
```
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.0-91-generic
Found initrd image: /boot/initrd.img-5.15.0-91-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
done
```

**Обновим образ initrd**

`update-initramfs -u`

<details>
<summary> результат выполнения команды: </summary>
   
```
update-initramfs: Generating /boot/initrd.img-5.15.0-91-generic
```
</details>

перезапустим сервер:

`exit`
`reboot`

**Посмотрим картину с дисками после перезагрузки**

`lsblk`

<details>
<summary> результат выполнения команды: </summary>
   
```
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  63.7M  1 loop /snap/core20/2434
loop1                       7:1    0  63.4M  1 loop /snap/core20/1974
loop2                       7:2    0 111.9M  1 loop /snap/lxd/24322
loop3                       7:3    0  53.3M  1 loop /snap/snapd/19457
loop4                       7:4    0  44.3M  1 loop /snap/snapd/23258
loop5                       7:5    0  89.4M  1 loop /snap/lxd/31333
sda                         8:0    0   128G  0 disk 
├─sda1                      8:1    0     1M  0 part 
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0   126G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:1    0    63G  0 lvm  
sdb                         8:16   0    10G  0 disk 
└─vg_root-lv_root         253:0    0    10G  0 lvm  /
sdc                         8:32   0     2G  0 disk 
sdd                         8:48   0     1G  0 disk 
sde                         8:64   0     1G  0 disk 
sdf                         8:80   0   250M  0 disk
```
</details>

##### 1.4 Изменим размер старой VG и вернем на него / (root)

Удаляем старый LV и создаём новый на 8G

`lvremove /dev/ubuntu-vg/ubuntu-lv`

<details>
<summary> результат выполнения команды: </summary>
   
```
Do you really want to remove and DISCARD active logical volume ubuntu-vg/ubuntu-lv? [y/n]: y
  Logical volume "ubuntu-lv" successfully removed
```
</details>

`lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg`

<details>
<summary> результат выполнения команды: </summary>
   
```
  WARNING: ext4 signature detected on /dev/ubuntu-vg/ubuntu-lv at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/ubuntu-vg/ubuntu-lv.
  Logical volume "ubuntu-lv" created.
```
</details>

Создадим файловую систему ext4 на разделе **/dev/ubuntu-vg/ubuntu-lv**

`mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv`

<details>
<summary> результат выполнения команды: </summary>
   
```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 2097152 4k blocks and 524288 inodes
Filesystem UUID: c0e6553a-1dfd-4e27-9e70-f94c1d2900f8
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```
</details>mount /dev/ubuntu-vg/ubuntu-lv /mnt

Смонтируем раздел **/dev/ubuntu-vg/ubuntu-lv** в **/mnt**:

`mount /dev/ubuntu-vg/ubuntu-lv /mnt`

**1.5 Скопируем все данные с раздела / в /mnt:**

`rsync -avxHAX --progress / /mnt/`

<details>
<summary> результат выполнения команды: </summary>

```
sent 5,363,696,169 bytes  received 1,393,387 bytes  84,489,599.31 bytes/sec
total size is 5,648,453,596  speedup is 1.05
```
</details>

Проверим, что содержимое раздела **/** скопировалось в **/mnt**:

`ls -la /mnt`

<details>
<summary> результат выполнения команды: </summary>

```
otal 2097252
drwxr-xr-x  21 root root       4096 Jan  3 14:02 .
drwxr-xr-x  21 root root       4096 Jan  3 14:02 ..
lrwxrwxrwx   1 root root          7 Aug 10  2023 bin -> usr/bin
drwxr-xr-x   2 root root       4096 Jan  6 10:22 boot
drwxr-xr-x   2 root root       4096 Jan  3 13:48 data
drwxr-xr-x   2 root root       4096 Jan  3 14:02 data-snap
drwxr-xr-x   2 root root       4096 Jan  6 10:34 dev
drwxr-xr-x 102 root root       4096 Jan  6 09:36 etc
drwxr-xr-x   3 root root       4096 Jan 10  2024 home
lrwxrwxrwx   1 root root          7 Aug 10  2023 lib -> usr/lib
lrwxrwxrwx   1 root root          9 Aug 10  2023 lib32 -> usr/lib32
lrwxrwxrwx   1 root root          9 Aug 10  2023 lib64 -> usr/lib64
lrwxrwxrwx   1 root root         10 Aug 10  2023 libx32 -> usr/libx32
drwx------   2 root root      16384 Jan 10  2024 lost+found
drwxr-xr-x   2 root root       4096 Aug 10  2023 media
drwxr-xr-x   2 root root       4096 Jan  6 10:36 mnt
drwxr-xr-x   2 root root       4096 Aug 10  2023 opt
dr-xr-xr-x   2 root root       4096 Jan  6 10:25 proc
drwx------   5 root root       4096 Jan  3 17:46 root
drwxr-xr-x   2 root root       4096 Jan  6 10:28 run
lrwxrwxrwx   1 root root          8 Aug 10  2023 sbin -> usr/sbin
drwxr-xr-x   6 root root       4096 Jan  3 13:28 snap
drwxr-xr-x   2 root root       4096 Aug 10  2023 srv
-rw-------   1 root root 2147483648 Jan 10  2024 swap.img
dr-xr-xr-x   2 root root       4096 Jan  6 10:25 sys
drwxrwxrwt  12 root root       4096 Jan  6 10:35 tmp
drwxr-xr-x  14 root root       4096 Aug 10  2023 usr
drwxr-xr-x  13 root root       4096 Aug 10  2023 var
```
</details>


##### 1.6 Сконфигурируем grub для того, чтобы при старте перейти в новый /  
Сымитируем текущий root, сделаем в него chroot и обновим grub:

```
for i in /proc/ /sys/ /dev/ /run/ /boot/; \
do mount --bind $i /mnt/$i; done
chroot /mnt/
```

**Обновим образ initrd**

`update-initramfs -u`

<details>
<summary> результат выполнения команды: </summary>
   
```
update-initramfs: Generating /boot/initrd.img-5.15.0-91-generic
W: Couldn't identify type of root file system for fsck hook
```
</details>

### 2. Выделить том под /var в зеркало

Пока не перезагружаемся и не выходим из под chroot - мы можем заодно перенести /var.

##### 2.1 На свободных дисках создаем зеркало

`pvcreate /dev/sdc /dev/sdd`


<details>
<summary> результат выполнения команды: </summary>
   
```
Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
```
</details>

`vgcreate vg_var /dev/sdc /dev/sdd`

<details>
<summary> результат выполнения команды: </summary>
   
```
Volume group "vg_var" successfully created
```
</details>

`lvcreate -L 950M -m1 -n lv_var vg_var`

<details>
<summary> результат выполнения команды: </summary>
   
```
 Rounding up size to full physical extent 952.00 MiB
  Logical volume "lv_var" created.
```
</details>

##### 2.2 Создаем fs и перемещаем туда /var

`mkfs.ext4 /dev/vg_var/lv_var`

<details>
<summary> результат выполнения команды: </summary>
   
```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 243712 4k blocks and 60928 inodes
Filesystem UUID: 5e0a492b-1798-4e40-98bd-0392ec9962bb
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
</details>

`mount /dev/vg_var/lv_var /mnt`
`cp -aR /var/* /mnt/`

**На всякий случай сохраняем содержимое старого var**

`mkdir /tmp/oldvar && mv /var/* /tmp/oldvar`

##### 2.3 монтируем новый var в каталог /var

`umount /mnt`
`mount /dev/vg_var/lv_var /var`

**Правим fstab для автоматического монтирования /var**

```
echo "`blkid | grep var: | awk '{print $2}'` \
 /var ext4 defaults 0 0" >> /etc/fstab
```
**Можно успешно перезагружаться в новый (уменьшенный root) и удалять временную Volume Group**

`exit`
`reboot`

`lvremove /dev/vg_root/lv_root`
<details>
<summary> результат выполнения команды: </summary>
   
```
Do you really want to remove and DISCARD active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed
```
</details>

`vgremove /dev/vg_root`
<details>
<summary> результат выполнения команды: </summary>
   
```
Volume group "vg_root" successfully removed
```
</details>

`pvremove /dev/sdb`
<details>
<summary> результат выполнения команды: </summary>
   
```
Labels on physical volume "/dev/sdb" successfully wiped.
```
</details>

### 3. Выделить том под /home

##### 3.1 Выделяем том под /home по тому же принципу что делали для /var

lvcreate -n LogVol_Home -L 2G /dev/ubuntu-vg

<details>
<summary> результат выполнения команды: </summary>
   
```
Logical volume "LogVol_Home" created.
```
</details>

 mkfs.ext4 /dev/ubuntu-vg/LogVol_Home

<details>
<summary> результат выполнения команды: </summary>
   
```
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: eee72fcd-7ab3-4ad6-8b49-88e19a19c23a
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```
</details>

`mount /dev/ubuntu-vg/LogVol_Home /mnt/`

`cp -aR /home/* /mnt/`  

`rm -rf /home/*`  

`umount /mnt`  

`mount /dev/ubuntu-vg/LogVol_Home /home/`
 

##### 3.2 Правим **fstab** для автоматического монтирования **/home**:

```
echo "`blkid | grep Home | awk '{print $2}'` \
 /home xfs defaults 0 0" >> /etc/fstab
```
---
### 4. Работа со снапшотами

##### 4.1 Генерируем файлы в **/home**:

`touch /home/test_file{1..20}`  

##### 4.2 Подготовим снапшот c раздела **/home**:  

```
lvcreate -L 100MB -s -n home_snap \
 /dev/ubuntu-vg/LogVol_Home
```

<details>
<summary> результат выполнения команды: </summary>
   
```
  Logical volume "home_snap" created.
```
</details>

Удалим часть файлов из **/home**:

`rm -f /home/test_file{11..20}`  

Восстановим из снапшота:  

`umount /home`  

`lvconvert --merge /dev/ubuntu-vg/home_snap`

<details>
<summary> результат выполнения команды: </summary>
   
```
Merging of volume ubuntu-vg/home_snap started.
  ubuntu-vg/LogVol_Home: Merged: 100.00%
```
</details>  

`mount /dev/mapper/ubuntu--vg-LogVol_Home /home`  

`ls -la /home`

<details>
<summary> результат выполнения команды: </summary>
   
```
total 28
drwxr-xr-x  4 root    root     4096 Jan  6 12:11 .
drwxr-xr-x 19 root    root     4096 Jan 11  2024 ..
-rw-r--r--  1 root    root        0 Jan  6 12:11 file1
-rw-r--r--  1 root    root        0 Jan  6 12:11 file10
-rw-r--r--  1 root    root        0 Jan  6 12:11 file11
-rw-r--r--  1 root    root        0 Jan  6 12:11 file12
-rw-r--r--  1 root    root        0 Jan  6 12:11 file13
-rw-r--r--  1 root    root        0 Jan  6 12:11 file14
-rw-r--r--  1 root    root        0 Jan  6 12:11 file15
-rw-r--r--  1 root    root        0 Jan  6 12:11 file16
-rw-r--r--  1 root    root        0 Jan  6 12:11 file17
-rw-r--r--  1 root    root        0 Jan  6 12:11 file18
-rw-r--r--  1 root    root        0 Jan  6 12:11 file19
-rw-r--r--  1 root    root        0 Jan  6 12:11 file2
-rw-r--r--  1 root    root        0 Jan  6 12:11 file20
-rw-r--r--  1 root    root        0 Jan  6 12:11 file3
-rw-r--r--  1 root    root        0 Jan  6 12:11 file4
-rw-r--r--  1 root    root        0 Jan  6 12:11 file5
-rw-r--r--  1 root    root        0 Jan  6 12:11 file6
-rw-r--r--  1 root    root        0 Jan  6 12:11 file7
-rw-r--r--  1 root    root        0 Jan  6 12:11 file8
-rw-r--r--  1 root    root        0 Jan  6 12:11 file9
drwx------  2 root    root    16384 Jan  6 12:01 lost+found
drwxr-x---  4 vagrant vagrant  4096 Jan 11  2024 vagrant
```
</details>  

Файлы успешно восстановлены с помощью снапшота.