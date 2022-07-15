1-) cfdisk /dev/sda

sda1| 512M > boot

sda2| XXG > LUKS #Diskin geri kalan alani luks ile sifrelenecek

2-) mkfs.ext4 /dev/sda1 #Boot bolumu ext4 olarak bicimlendirilecek

3-) modprobe dm-crypt

4-) cryptsetup luksFormat /dev/sda2 #Sifre belirlenecek

5-) cryptsetup luksOpen /dev/sda2 lvm

6-) /etc/init.d/lvmetad --nodeps restart # "WARNING: Failed to connect to lvmetad" uyarisini kaldirmak icin gerekli

7-) pvscan

8-) lvm pvcreate /dev/mapper/lvm

9-) vgcreate vg0 /dev/mapper/lvm

10-) lvcreate -L 25G -n root vg0 # Diskte ne kadar alan varsa ona gore boyutunu ayarlayabilirsiniz, bu sadece ornek komut

11-) lvcreate -L 40G -n var vg0

12-) lvcreate -l 100%FREE -n home vg0

13-) mkfs.ext4 /dev/mapper/vg0-root

14-) mkfs.ext4 /dev/mapper/vg0-var

15-) mkfs.ext4 /dev/mapper/vg0-home

16-) mkdir /mnt/gentoo

17-) mkdir /mnt/gentoo/var

18-) mkdir /mnt/gentoo/home

19-) mount /dev/mapper/vg0-root /mnt/gentoo

20-) mount /dev/mapper/vg0-var /mnt/gentoo/var

21-) mount /dev/mapper/vg0-home /mnt/gentoo/home/

22-) cd /mnt/gentoo

23-) wget https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20220710T170538Z/stage3-amd64-openrc-20220710T170538Z.tar.xz

24-) tar -xvf stage3-amd64-openrc-20220710T170538Z.tar.xz --xattrs --numeric-owner

25-) mkdir /mnt/gentoo/etc/portage/repos.conf

26-) cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

27-) cp /etc/resolv.conf /mnt/gentoo/etc/resolv.conf

28-) mount --types proc /proc /mnt/gentoo/proc

29-) mount --rbind /sys /mnt/gentoo/sys

30-) mount --make-rslave /mnt/gentoo/sys

31-) mount --rbind /dev /mnt/gentoo/dev

32-) mount --make-rslave /mnt/gentoo/dev

33-) mount --bind /run /mnt/gentoo/run

34-) mount --make-slave /mnt/gentoo/run

35-) test -L /dev/shm && rm /dev/shm && mkdir /dev/shm

36-) mount -t tmpfs -o nosuid,nodev,noexec shm /dev/shm

37-) chmod 1777 /dev/shm

38-) chroot /mnt/gentoo /bin/bash

39-) source /etc/profile

40-) mount /dev/sda1 /boot

41-) emerge-webrsync

42-) echo Europe/Istanbul > /etc/timezone

43-) emerge --config sys-libs/timezone-data

44-) nano -w /etc/locale.gen

45-) locale-gen

46-) env-update && source /etc/profile

47-) blkid # Bolumlerin UUID numaralarini kaydet. /etc/fstab dosyasi su sekilde olmalidir;

                                              <mountpoint>    <type>          <opts>          <dump/pass>

UUID=DB1D-89C5                                /boot           ext4            noauto,noatime  1 2

UUID=6bedbbd8-cea9-4734-9c49-8e985c61c120     /               ext4            defaults        0 1

UUID=61e4cc83-a1ee-4190-914b-4b62b49ac77f     /var            ext4            defaults        0 1

UUID=5d6ff087-50ce-400f-91c4-e3378be23c00     /home           ext4            defaults        0 1

#tmps

tmpfs                                         /tmp            tmpfs           size=4G         0 0

48-) emerge sys-kernel/gentoo-sources

49-) echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" | tee -a /etc/portage/package.license

50-) emerge sys-kernel/genkernel

51-) eselect kernel set linux*

52-) emerge sys-fs/cryptsetup

53-) genkernel --luks --lvm --no-zfs all

54-) genkernel --luks --lvm initramfs

55-) echo "sys-boot/grub:2 device-mapper" >> /etc/portage/package.use/sys-boot

56-) emerge -av grub

57-) nano /etc/default/grub

GRUB_CMDLINE_LINUX="dolvm crypt_root=UUID=sda2_bolumunun_UUID_numarasi"

58-) mount /boot

59-) grub-install --target=i386-pc /dev/sda

60-) grub-mkconfig -o /boot/grub/grub.cfg

61-) emerge app-misc/livecd-tools

62-) net-setup

63-) nano /etc/hostname > name #hostname dosyasina bir isim yaz

64-) nano /etc/hosts # localhost'un altina ekle, /etc/hostname dosyasina yazdigin isimle ayni olmali

127.0.1.1       name

65-) nano /etc/resolv.conf # iserseniz farkli bir DNS adresi girebilirsiniz, onerim;

nameserver 9.9.9.11

nameserver 149.112.112.112

66-) useradd -m -G users,wheel,audio -s /bin/bash username

67-) passwd username

68-) passwd root

69-) emerge doas

70-) nano /etc/doas.conf

permit username as root

71-) cd /

72-) rm -rf stage3-amd64-openrc-20220710T170538Z.tar.xz

73-) umount -a

74-) reboot -f
