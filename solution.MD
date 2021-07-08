## 2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

Нет, не могут, потому как, по-сути, файл один! а "обращений"(ссылок) может быть сколько угодно.

---

## 4. Используя fdisk, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

```
$ lsblk

NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
sdc                    8:32   0  2.5G  0 disk

$ fdisk /dev/sdb

# далее в интерактивном режиме:
n - p - Enter - +2G
Created a new partition 1 of type 'Linux' and of size 2 GiB.

n - p - Enter - Enter
Created a new partition 2 of type 'Linux' and of size 511 MiB.

w # выход
```

---

## 5. Используя sfdisk, перенесите данную таблицу разделов на второй диск.

```
sfdisk -d /dev/sdb | sfdisk /dev/sdc
```

---

## 6. Соберите mdadm RAID1 на паре разделов 2 Гб.

```
mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

---

## 7. Соберите mdadm RAID0 на второй паре маленьких разделов.

```
mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
```

---

## 8. Создайте 2 независимых PV на получившихся md-устройствах.

```
pvcreate /dev/md0 /dev/md1
```

---

## 9. Создайте общую volume-group на этих двух PV.

```
vgcreate vg1 /dev/md0 /dev/md1
```

---

## 10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```
lvcreate -n lv1 -L 100M /dev/vg1 /dev/md0
```

---

## 11. Создайте mkfs.ext4 ФС на получившемся LV.

```
mkfs.ext4 /dev/vg1/lv1
```

---

## 12. Смонтируйте этот раздел в любую директорию, например, /tmp/new

```
mkdir /tmp/new
mount /dev/vg1/lv1 /tmp/new
```

---

## 13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

```
wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz /tmp/new
```

---

## 14. Прикрепите вывод `lsblk`.

```
NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part  /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
sdb                    8:16   0  2.5G  0 disk
├─sdb1                 8:17   0    2G  0 part
│ └─md1                9:1    0    2G  0 raid1
└─sdb2                 8:18   0  511M  0 part
  └─md0                9:0    0 1018M  0 raid0
    └─vg1-lv1        253:2    0  100M  0 lvm   /tmp/new
sdc                    8:32   0  2.5G  0 disk
├─sdc1                 8:33   0    2G  0 part
│ └─md1                9:1    0    2G  0 raid1
└─sdc2                 8:34   0  511M  0 part
  └─md0                9:0    0 1018M  0 raid0
    └─vg1-lv1        253:2    0  100M  0 lvm   /tmp/new
```

---

## 15. Протестируйте целостность файла:

```
$ gzip -t /tmp/new/test.gz
$ echo $?

0
```

---

## 16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

```
pvmove /dev/md0 /dev/md1
```

---

## 17. Сделайте `--fail` на устройство в вашем RAID1 md.

```
mdadm /dev/md1 --fail /dev/sdb1
```

---

## 18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

```
$ dmesg -H

> .....
  [Nov 4 16:21] md/raid1:md1: Disk failure on sdb1, disabling device.
                md/raid1:md1: Operation continuing on 1 devices.
```

---

## 19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

```
$ gzip -t /tmp/new/test.gz
$ echo $?

0
```

---

P.S. Все команды выполнял из под root потому без sudo
