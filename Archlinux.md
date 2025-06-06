# 分区与格式化

```shell
# 找到需要分区的硬盘
fdisk -l
# 开始分区
fdisk /dev/sda
# GPT格式分区
g
# 添加第一块分区sda1
n
1(default)
2048(default)
+1G
# 添加第二块分区sda3
n
3
2099200(default)
+1G
# 剩下的都给sda2
n
2(default)
2099200(default)
33552383(default)
# 保存退出
w
# 查看分区信息
lsblk
# 格式化分区
# 启动分区sda1
mkfs.fat -F32 /dev/sda1
# 交换分区/dev/sda3

mkswap /dev/sda3
swapon /dev/sda3
# 根目录
mkfs.ext4 /dev/sda2
```

# 制作文件系统

```shell
# 挂载分区
mount /dev/sda2 /mnt
mkdir -p /mnt/boot
mount /dev/sda1 /mnt

# 配置网络和源
echo 'Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch' >> /etc/pacman.d/mirrorlist
cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist
pacman -Sy archlinux-keyring

# 安装基础包
pacstrap /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt
# 设置密码
passwd
# 安装grub引导加载程序，UEFI引导管理程序，intel处理器微码，操作系统探测工具
pacman -S vim
pacman -Sy archlinux-keyring
pacman -S networkmanager #netctl
systemctl start NetworkManager
systemctl enable NetworkManager
pacman -S dhcp
pacman -S grub efibootmgr intel-ucode os-prober
mkdir /boot/grub
grub-mkconfig > /boot/grub/grub.cfg
# 系统架构
uname -m
# 虚拟机开启EFI
grub-install --target=x86_64-efi --efi-directory=/boot
# 退出关机
exit
umount /mnt/boot
umount /mnt
poweroff
```

# 一些基本配置

```shell
# 重新进入系统
# 设置终端字体
# setfont /usr/share/kbd/consolefonts/LatGrkCry-12x22.psfu.gz

# 修改键位
# echo 'keycode 1 = Caps_Lock' >> /etc/keys.conf
# echo 'keycode 58 = Escape' >> /etc/keys.conf
# loadkeys /etc/keys.conf

# 检查网络
ip link
ip link set enp0s3 up
ip addr

# 连接WIFI
iwlist enp0s3 scan
wpa_passphrase wifi_name password > wifi_name.conf
wpa_supplicant -c wifi_name.conf -i enp0s3 &
dhcpd &
ping baidu.com

# 时区
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
timedatectl set-ntp true

# 本地化
locale-gen
vim /etc/locale.conf
LANG=en_US.UTF-8 
```

# 配置桌面环境

```shell
# 新建组，用户
pacman -S tcsh
groupadd incam -g 501
useradd incam -u 501 -g 501 -m -d /home/incam -s /bin/csh
passwd incam

# gnome
pacman -S xorg
pacman -S gdm
systemctl enable gdm
pacman -S gnome gnome-extra
```
