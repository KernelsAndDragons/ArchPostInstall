# Arch Post Install

Enlace al video: https://www.youtube.com/watch?v=YG2oQgGhdIQ

# Configuración de archivos del sistema.

## Hosts

`sudo nano /etc/hosts`

> Reemplaza {hostname} por el nombre de tu máquina
```
127.0.0.1  localhost {hostname}
::1        localhost {hostname}
```

## Pacman

`sudo nano /etc/pacman.conf
`

```
# Misc options  
 Color  
 ParallelDownloads = 10  
 ILoveCandy  
```
## Fastest mirror existent list

mirror list: https://archlinux.org/mirrorlist/all/

Instalar las dependencias
```
sudo pacman -S  pacman-contrib
```

Ordenar los mirrors
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.old
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```
```
awk '/^## Country Name$/{f=1; next}f==0{next}/^$/{exit}{print substr($0, 1);}' /etc/pacman.d/mirrorlist.backup
```
```
sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
```
```
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
```
```
rm /etc/pacman.d/mirrorlist.backup
```

# Instalación de Servicios y aplicaciones

## Sensors para refrigeración

```
sudo pacman -S lm_sensors i2c-tools rrdtool
```

```
sudo modprobe i2c_dev
```
```
sudo systemctl enable --now sensord.service
```
---
  intel
  ```
  sudo modprobe i2c-i801
  ```
  amd
  ```
  sudo modprobe i2c-piix4
  ```
---

### Añadir los módulos permanentemente 

```
sudo touch /etc/modules-load.d/i2c.conf
sudo sh -c 'echo "i2c-dev" >> /etc/modules-load.d/i2c.conf'
```
---
  intel
  ```
  sudo sh -c 'echo "i2c-i801" >> /etc/modules-load.d/i2c.conf'
  ```
  amd
  ```
  sudo sh -c 'echo "i2c-piix4" >> /etc/modules-load.d/i2c.conf'
  ```
---


## Control de refrigeración AIO All-In-One

```
sudo pacman -S liquidctl
```

## Gestión de energía (KDE PLASMA)

```
sudo pacman -S powerdevil power-profiles-daemon
```

```
sudo systemctl enable --now power-profiles-daemon.service
```
----
## Bluetooth

```
sudo systemctl enable --now bluetooth
```
---
## AUR 

### YAY

```
sudo pacman -S --needed git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si
```

### PAMAC

```
yay -S libpamac-full pamac-all
```

### OCTOPI

```
yay -S octopi
```
---
## Aplicaciones de Plasma

```
sudo pacman -S kde-system spectacle gwenview ark filelight isoimagewriter kate kcalc kdialog kfind kwalletmanager sweeper yakuake dolphin-plugins inotify-tools okular kgpg
```

## Bash Scripts

```
sudo pacman -S bash-completion
```

## Herramientas de compilación

```
sudo pacman -S linux-headers base-devel
```

## Fuse

```
sudo pacman -S fuse fuse2fs fuseiso
```
## Navegador Firefox

```
sudo pacman -S firefox firefox-i18n-es-es
```

## MDNS para carpetas compartidas en red

```
sudo pacman -S avahi nss-mdns
```
---
`sudo nano /etc/nsswitch.conf`

```
hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns
```
---
```
sudo systemctl enable --now avahi-daemon
```

## Firewall

```
sudo pacman -S firewalld python-pyqt6
```

```
sudo systemctl enable --now firewalld
```

```
sudo firewall-cmd --set-default-zone=home
sudo firewall-cmd --permanent --add-service=mdns
```

## Flatpak

```
sudo pacman -S flatpak xdg-desktop-portal-gtk xdg-desktop-portal-kde
```

## Snap

```
yay -S snapd
```
```
sudo systemctl enable --now snapd.service
```
> Si utilizas apparmor ejecutar el siguiente comando. 
```
sudo systemctl enable --now snapd.apparmor.service
```

## Virtualización

```
sudo pacman -S qemu-full virt-manager virt-viewer dnsmasq bridge-utils libguestfs ebtables vde2 openbsd-netcat
```

```
sudo usermod -aG libvirt $USER
```

```
sudo systemctl enable --now libvirtd
```

```
sudo virsh net-start default
sudo virsh net-autostart default
```

## Distrobox

```
sudo pacman -S podman distrobox
```

```
sudo systemctl enable --now podman
```

## Plymouth

```
sudo pacman -S plymouth
```
---
`sudo nano /etc/mkinitcpio.conf`

añadir `plymouth`
```
HOOKS=(base udev plymouth autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
```
---
### Grub
`sudo nano /etc/default/grub`

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash"
```
### Systemd-boot
`sudo nano  /boot/loader/entries/*.conf `

add `quiet` and `splash`
```
options root=PARTUUID=df4a860f-0cad-4bd0-8c75-e245a41eeab2 zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs quiet splash
```
---
```
sudo mkinitcpio -P
```

## AppArmor

```
sudo pacman -S apparmor
```

```
sudo systemctl enable apparmor.service
```
---
`sudo nano /etc/default/grub`
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash apparmor=1 security=apparmor"
```
---

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
## Locate

```
sudo pacman -S locate
```
```
sudo updatedb
```


# Snapshots
> He reemplazado Timeshift por Snapper.
## Snapper 

```
sudo pacman -S snapper snap-pac grub-btrfs
```
```
yay -S btrfs-assistant
```

> Si has utilizado archinstall y cuentas con el subvolumen **@.snapshots**, debes desmontar y eliminar el directorio `/.snapshots` **para que el Paso 2 no falle**.

Podemos ver los subvolumenes con el comando:
```
sudo btrfs subvolume list /
```
Paso 1: Eliminar el directorio `/.snapshots`
```
cd /
sudo umount /.snapshots
sudo rm -rf /.snapshots
```
Paso 2: Crear configuración de snapper
```
sudo snapper -c root create-config /
```

Paso 3: Eliminar subvolumen creado por snapper
```
sudo btrfs subvolume delete .snapshots
```

Paso 4: Crear otra vez el directorio `.snapshots` para montarlo con subvolumen `@.snapshots`
```
sudo mkdir /.snapshots
sudo mount -a
```
Paso 5: Corregir los permisos necesarios para `/.snapshots`
```
sudo chmod 750 /.snapshots
sudo chown :wheel /.snapshots
```

Paso 6: Crear snapshot
```
sudo snapper -c root create -d "**Sistema base**"
```

Paso 7: Automatizar los snapshots
```
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

Paso 8: Añadir excepción de indexado para `locate`

`sudo nano /etc/updatedb.conf`

Añadir `.snapshots`
```
PRUNENAMES = ".git .hg .svn .snapshots"
```
Actualizar la base de datos
```
sudo updatedb
```
Paso 9: Habilitar grub-btrfs
```
sudo systemctl enable --now grub-btrfsd.service 
```
Paso 10: Crear condición de arranque a un snapshot con permisos readonly
`sudo nano /etc/mkinitcpio.conf`
añadir `grub-btrfs-overlayfs`
```
HOOKS=(base ... fsck grub-btrfs-overlayfs)
```
```
sudo mkinitcpio -P
```
Paso 11: ¡Utilizar la aplicación gráfica btrfs-assistant para gestionar cómodamente la programación y los snapshots!

# Launchers Lutris, Heroic, Steam y Herramientas Gaming

## Heroic

```
yay -S heroic-games-launcher
```

## Lutris

```
sudo pacman -S --needed wine-staging giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls \
mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error \
lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo \
sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama lib32-libgcrypt libgcrypt lib32-libxinerama \
ncurses lib32-ncurses ocl-icd lib32-ocl-icd libxslt lib32-libxslt libva lib32-libva gtk3 \
lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader winetricks
```

```
sudo pacman -S lutris
```

## Steam

```
sudo pacman -S steam
```
## Proton Tools

```
yay -S protonup-qt protontricks
```

## XBOX Controller Drivers

```
yay -S xpadneo-dkms-git xone-dkms-git xone-dongle-firmware
```

## Dependencias de SteamTinkerLaunch

```
sudo pacman -S xdotool xorg-xwininfo yad 
```
## Gamemode
```
sudo pacman -S gamemode
```
# Bash
## Oh-My-Bash

via curl
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
```

via wget
```
bash -c "$(wget https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh -O -)"
```

THEMES: https://github.com/ohmybash/oh-my-bash/blob/master/themes/THEMES.md

`nano ~/.bashrc`

```
OSH_THEME="standard"
```

# ZSH

```
sudo pacman -S zsh wget lsd bat git
```

## Oh-My-ZSH

```
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

## Powerlevel10k

```
sudo pacman -S ttf-hack-nerd  ttf-meslo-nerd
```

```
fc-cache -f -v
```

```
cd ~
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
---
`nano ~/.zshrc`

```
ZSH_THEME="powerlevel10k/powerlevel10k"
```
---
## ZSH PLUGINS (SOLO para Oh-My-ZSH) 

```
cd ~

git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zdharma-continuum/fast-syntax-highlighting.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting

git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autocomplete
```
---
`nano ~/.zshrc`

```
plugins=(git zsh-autosuggestions zsh-syntax-highlighting fast-syntax-highlighting zsh-autocomplete)

fpath+=${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions/src

source "$ZSH/oh-my-zsh.sh
```

```
# Alias  
alias ls='lsd'  
alias ll='lsd -l'  
alias la='lsd -a'  
alias lla='lsd -la'  
alias cat='bat'
```
---


