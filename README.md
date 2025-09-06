# Arch Linux Post-Install Guide  

📺 Video Reference: [YouTube](https://www.youtube.com/watch?v=YG2oQgGhdIQ)  

This guide will help you configure your Arch Linux system after installation:  
- System optimization  
- Package mirrors & updates  
- Essential services (power, Bluetooth, firewall, etc.)  
- Development & virtualization tools  
- Snapshots with Btrfs  
- Gaming stack (Steam, Lutris, Heroic)  
- Shell customization (Bash, Zsh, Powerlevel10k)  
- Visual tweaks (Plymouth, Fastfetch, icons)  

---

## Table of Contents
1. [System File Configuration](#1-system-file-configuration)  
2. [Services & Applications](#2-services--applications)  
3. [Software Sources & Runtimes](#3-software-sources--runtimes)  
4. [Virtualization & Containers](#4-virtualization--containers)  
5. [Boot Splash (Plymouth)](#5-boot-splash-plymouth)  
6. [AppArmor](#6-apparmor)  
7. [Snapshots (Btrfs + Snapper)](#7-snapshots-btrfs-with-snapper)  
8. [Gaming Stack](#8-gaming-stack-lutris--heroic--steam)  
9. [Shells](#9-shells)  
10. [Icons](#10-icons)  
11. [Fastfetch](#11-fastfetch-enhanced)  
12. [Extra Tips](#12-extra-quality-of-life)  

---

## 1. System File Configuration  

### Hosts File  
```bash
sudo nano /etc/hosts
```
Replace `{hostname}` with your machine name:  
```txt
127.0.0.1  localhost {hostname}
::1        localhost {hostname}
```

---

### Pacman Configuration  
```bash
sudo nano /etc/pacman.conf
```
```txt
# Misc options
Color
ParallelDownloads = 10
ILoveCandy
```

---

### Faster Mirrorlist  
Install dependencies:  
```bash
sudo pacman -S pacman-contrib
```

(Optional) Replace current mirrors with all available:  
```bash
curl https://archlinux.org/mirrorlist/all/ | sudo tee /etc/pacman.d/mirrorlist
```

Backup:  
```bash
sudo cp /etc/pacman.d/mirrorlist{,.old}
sudo cp /etc/pacman.d/mirrorlist{,.backup}
```

Uncomment mirrors:  
```bash
sudo sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
```

Rank the **6 fastest mirrors**:  
```bash
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup | sudo tee /etc/pacman.d/mirrorlist
```

---

### Firmware Updates  
```bash
sudo pacman -S fwupd fwupd-efi fwupd-docs
fwupdmgr refresh
fwupdmgr get-updates
sudo fwupdmgr update
```

---

## 2. Services & Applications  

### Sensors (temperature & fans)  
```bash
sudo pacman -S lm_sensors i2c-tools rrdtool
sudo modprobe i2c_dev
sudo systemctl enable --now sensord.service
```
- Intel: `sudo modprobe i2c-i801`  
- AMD: `sudo modprobe i2c-piix4`  

Make permanent:  
```bash
echo "i2c-dev" | sudo tee -a /etc/modules-load.d/i2c.conf
```

---

### Liquid Cooling (AIO)  
```bash
sudo pacman -S liquidctl
```

---

### Power Management (KDE)  
```bash
sudo pacman -S powerdevil power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon.service
```

---

### Bluetooth  
```bash
sudo systemctl enable --now bluetooth
```

---

### AUR Helpers  
**Yay:**  
```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
```

**Pamac (GUI):**  
```bash
yay -S libpamac-full pamac-all
```

**Octopi:**  
```bash
yay -S octopi
```

---

### KDE Applications  
```bash
sudo pacman -S kde-system spectacle gwenview ark filelight isoimagewriter kate kcalc kdialog kfind kwalletmanager sweeper yakuake dolphin-plugins inotify-tools okular kgpg
```

**KDE Connect:**  
```bash
sudo pacman -S kdeconnect
```

---

### Development Tools  
```bash
sudo pacman -S linux-headers base-devel bash-completion
```

### Fuse (filesystem mounts)  
```bash
sudo pacman -S fuse fuse2fs fuseiso lvm2 dosfstools
```

---

### Browsers  
**Firefox (Spanish locale):**  
```bash
sudo pacman -S firefox firefox-i18n-es-es
```

**Vivaldi:**  
```bash
sudo pacman -S vivaldi vivaldi-ffmpeg-codecs
```

---

### Networking (mDNS)  
```bash
sudo pacman -S avahi nss-mdns
sudo nano /etc/nsswitch.conf
```
Edit line:  
```txt
hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns
```
Enable:  
```bash
sudo systemctl enable --now avahi-daemon
```

---

### Firewall  
```bash
sudo pacman -S firewalld python-pyqt6
sudo systemctl enable --now firewalld
```
Add services:  
```bash
sudo firewall-cmd --set-default-zone=home
sudo firewall-cmd --permanent --add-service=mdns
sudo firewall-cmd --permanent --add-service=kdeconnect
```

---

## 3. Software Sources & Runtimes  

### Flatpak  
```bash
sudo pacman -S flatpak flatpak-kcm xdg-desktop-portal-gtk xdg-desktop-portal-kde
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

### Snap  
```bash
yay -S snapd
sudo systemctl enable --now snapd.socket
sudo systemctl enable --now snapd.apparmor.service   # if using AppArmor
```

---

## 4. Virtualization & Containers  

### KVM/QEMU + libvirt  
```bash
sudo pacman -S qemu-full virt-manager virt-viewer dnsmasq bridge-utils libguestfs ebtables vde2 openbsd-netcat
sudo usermod -aG libvirt $USER
sudo systemctl enable --now libvirtd
sudo virsh net-start default
sudo virsh net-autostart default
```

### Distrobox (Podman containers)  
```bash
sudo pacman -S podman distrobox
systemctl --user enable --now podman.socket
loginctl enable-linger $USER
```

---

## 5. Boot Splash (Plymouth)  

```bash
sudo pacman -S plymouth
sudo nano /etc/mkinitcpio.conf
```
Add:  
```txt
HOOKS=(base udev plymouth autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```

**GRUB:**  
```bash
sudo nano /etc/default/grub
```
```txt
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash"
```
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**systemd-boot:**  
```bash
sudo nano /boot/loader/entries/*.conf
```
Add to `options`:  
```txt
quiet splash
```

```bash
sudo mkinitcpio -P
```

---

## 6. AppArmor  

Check support:  
```bash
zgrep CONFIG_LSM= /proc/config.gz
```

**GRUB:**  
```bash
sudo nano /etc/default/grub
```
```txt
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash lsm=landlock,lockdown,yama,integrity,apparmor,bpf audit=1"
```
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**systemd-boot:**  
```bash
sudo nano /boot/loader/entries/*.conf
```
Append to `options`:  
```txt
lsm=landlock,lockdown,yama,integrity,apparmor,bpf audit=1
```

Install tools:  
```bash
sudo pacman -S tk apparmor python-audit python-notify2
sudo systemctl enable --now apparmor.service
sudo systemctl enable --now auditd.service
```

Reboot:  
```bash
reboot
```

Create group & enable notifications:  
```bash
sudo groupadd -r audit
sudo gpasswd -a "$(whoami)" audit
```

Edit `/etc/audit/auditd.conf`:  
```txt
log_group = audit
```

Autostart notifications:  
```bash
mkdir -p ~/.config/autostart
nano ~/.config/autostart/apparmor-notify.desktop
```
```ini
[Desktop Entry]
Type=Application
Name=AppArmor Notify
Exec=aa-notify -p -s 1 -w 60 -f /var/log/audit/audit.log
NoDisplay=true
```

---

## 7. Snapshots (Btrfs with Snapper)  

```bash
sudo pacman -S snapper snap-pac grub-btrfs
yay -S btrfs-assistant
```

Check subvolumes:  
```bash
sudo btrfs subvolume list /
```

Remove old:  
```bash
cd /
sudo umount /.snapshots
sudo rm -rf /.snapshots
```

Create config:  
```bash
sudo snapper -c root create-config /
sudo btrfs subvolume delete /.snapshots
```

Recreate dir:  
```bash
sudo mkdir /.snapshots
sudo mount -a
sudo chmod 750 /.snapshots
sudo chown :wheel /.snapshots
```

Snapshot:  
```bash
sudo snapper -c root create -d "Base System"
```

Enable timers:  
```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

Exclude from locate:  
```bash
sudo nano /etc/updatedb.conf
```
Add `.snapshots`:  
```txt
PRUNENAMES = ".git .hg .svn .snapshots"
```
```bash
sudo updatedb
```

Enable grub integration:  
```bash
sudo systemctl enable --now grub-btrfsd.service
```

Add overlayfs hook:  
```bash
sudo nano /etc/mkinitcpio.conf
```
```txt
HOOKS=(base ... filesystems fsck grub-btrfs-overlayfs)
```
```bash
sudo mkinitcpio -P
```

---

## 8. Gaming Stack (Lutris / Heroic / Steam)  

**Heroic Games Launcher:**  
```bash
yay -S heroic-games-launcher
```

**Lutris + dependencies:**  
```bash
sudo pacman -S wine-staging giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama lib32-libgcrypt libgcrypt lib32-libxinerama ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt libva lib32-libva gtk3 lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader cups samba
sudo pacman -S lutris
```

**Steam:**  
```bash
sudo pacman -S steam
```

**Proton Tools:**  
```bash
yay -S protonup-qt protontricks
```

**Xbox Controller Drivers:**  
```bash
yay -S xpadneo-dkms-git xone-dkms-git xone-dongle-firmware
```

**GameMode:**  
```bash
sudo pacman -S gamemode
```
Use with Steam launch options:  
```txt
gamemoderun %command%
```

---

## 9. Shells  

### Bash + Oh-My-Bash  
```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
```
Edit `~/.bashrc`:  
```txt
OSH_THEME="standard"
```

---

### Zsh + Oh-My-Zsh + Powerlevel10k  
```bash
sudo pacman -S zsh wget lsd bat git
chsh -s /bin/zsh
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

Install fonts + theme:  
```bash
sudo pacman -S ttf-hack-nerd ttf-meslo-nerd
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git   ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
Edit `~/.zshrc`:  
```txt
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Aliases:  
```bash
echo "alias ls='lsd'" >> ~/.zshrc
echo "alias cat='bat'" >> ~/.zshrc
```

Plugins:  
```bash
sudo pacman -S zsh-autocomplete zsh-autosuggestions zsh-history-substring-search zsh-syntax-highlighting
```
Append to `~/.zshrc`:  
```txt
source /usr/share/zsh/plugins/zsh-autocomplete/zsh-autocomplete.plugin.zsh
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.plugin.zsh
source /usr/share/zsh/plugins/zsh-history-substring-search/zsh-history-substring-search.zsh
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.plugin.zsh
```

---

## 10. Icons  
Download Arch icons from [GitHub repo](https://github.com/KernelsAndDragons/ArchPostInstall/tree/main/icons).  

---

## 11. Fastfetch (Enhanced)  
```bash
sudo pacman -S fastfetch
mkdir -p ~/.config/fastfetch
cd ~/.config/fastfetch
fastfetch --gen-config
rm config.jsonc
wget https://raw.githubusercontent.com/KernelsAndDragons/ArchPostInstall/refs/heads/main/config.jsonc
```
Restart terminal and run:  
```bash
fastfetch
```

---

## 12. Extra Quality-of-Life  

### Pacman Full Sync  
```bash
sudo pacman -Syyu
```

### Sensors Detection  
```bash
sudo sensors-detect
sudo systemctl restart sensord.service
```

---

✅ You now have a **fully optimized Arch Linux setup** with essential services, development tools, gaming, security, and customization.  
