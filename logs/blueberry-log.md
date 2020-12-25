# Base install

set keymap

```
loadkeys dvorak
```

boot from live usb & ensure efi boot

```
ls /sys/firmware/efi/efivars
```

connect to wifi

```
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <ssid>
```

turn on ntp

```
timedatectl set-ntp true
```

partition disks (vfat esp 100M, the rest ext4)

```
fdisk /dev/mmcblk0
mkfs.vfat /dev/mmcblk0p1
mkfs.ext4 /dev/mmcblk0p2
```

install base system plus text editor, wifi, bootloader, microcode updates, ...

```
mount /dev/mmcblk0p1 /mnt
pacstrap /mnt base linux linux-firmware vi nvim networkmanager grub efibootmgr intel-ucode zsh sudo git
```

mount boot partition and make fstab

```
mkdir /mnt/boot/esp
mount /dev/mmcblk0p1 /mnt/boot/esp
genfstab -U /mnt >> /mnt/etc/fstab
```

chroot and set timezone

```
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/America/Vancouver /etc/localtime
```

generate /etc/adjtime

```
hwclock --systohc
```

uncomment `en_US.UTF-8 UTF-8`

```
nvim /etc/locale.gen
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

set keymap

```
echo KEYMAP=dvorak > /etc/vconsole.conf
```

set hostname

```
echo blueberry > /etc/hostname
```

edit /etc/hosts

```
127.0.0.1	localhost
::1		localhost
127.0.1.1 blueberry.lan	blueberry
```

set root password

```
passwd
```

check firmware boot manager entries and delete them all e.g.

```
efibootmgr -Bb 0000
```

install grub to esp

```
grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB
```

set boot order

```
efibootmgr -o 0000
```

make grub config

```
grub-mkconfig -o /boot/grub/grub.cfg
```

add main user

```
useradd -m -s /usr/bin/zsh jamie
passwd jamie
```

add "jamie ALL=(ALL) ALL" to sudoers

```
visudo
```

```
shutdown now
```

remove usb drive and boot

sign into wifi

```
nmcli d wifi list
nmcli d wifi connect <ssid> password <password>
```


## Get ready for ansible provisioning

login as jamie

```
sudo pacman -S openssh
```

generate keypair with password

```
ssh-keygen -t ed25519
```

add public key to github e.g. with another computer and usb stick

git config (maybe not necessary in future)

```
git config --global user.email "jamie.alban@gmail.com"
git config --global user.name "Jamie Macdonald"
```

clone ansible repo to make changes

```
mkdir ~/my
cd ~/my
git clone git@github.com:jameh/ansible.git
```

pull changes from ansible

```
sudo pacman -S ansible
sudo ansible-pull https://github.com/jameh/ansible.git
```


TODO:
- try kernel flags to get rid of memory corruption error

```
memmap=64K$0 memory_corruption_check=0
```



