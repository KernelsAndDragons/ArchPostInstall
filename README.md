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
## Mirrorlist más rápidos

mirror list: https://archlinux.org/mirrorlist/all/

Instalar las dependencias
```
sudo pacman -S  pacman-contrib
```
(Opcional) añadir todos los mirrors.
> Omite este paso para utilizar tus mirrors activados actualmente.
```
curl https://archlinux.org/mirrorlist/all/ > /etc/pacman.d/mirrorlist
```
Generar el backup y una copia temporal.
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.old
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```
Activar todos los repositorios de mirrorlist.backup
```
awk '/^## Country Name$/{f=1; next}f==0{next}/^$/{exit}{print substr($0, 1);}' /etc/pacman.d/mirrorlist.backup
```
```
sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
```
Generar el nuevo mirrorlist con los 6 repositorios más rápidos (puedes modificar éste número como prefieras) **¡PUEDE TARDAR UNOS MINUTOS!**
```
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
```
(Opcional) puedes eliminar la copia temporal.
```
rm /etc/pacman.d/mirrorlist.backup
```
## Firmware Updater

```
sudo pacman -S fwupd fwupd-efi fwupd-docs
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
## KDE-Connect
```
sudo pacman -S kdeconnect dbus  gcc-libs  glibc  kcmutils  kconfig  kcoreaddons  kcrash  kdbusaddons kdeclarative  kguiaddons  ki18n  kiconthemes  kio  kirigami  kirigami-addons kitemmodels  kjobwidgets  knotifications  kpeople  kservice  kstatusnotifieritem kwidgetsaddons  kwindowsystem  libfakekey  libx11  libxkbcommon  libxtst modemmanager-qt  openssl  pulseaudio-qt  qqc2-desktop-style  qt6-base qt6-connectivity  qt6-declarative  qt6-multimedia  qt6-wayland  solid  wayland
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

## Navegador Vivaldi

```
sudo pacman -S vivaldi vivaldi-ffmpeg-codecs
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
sudo firewall-cmd --permanent --add-service=kdeconnect
```

## Flatpak

```
sudo pacman -S flatpak flatpak-kcm xdg-desktop-portal-gtk xdg-desktop-portal-kde
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
### OPCIÓN 1 - Grub
---
`sudo nano /etc/default/grub`

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash"
```
---
### OPCIÓN 2 - Systemd-boot
---
`sudo nano  /boot/loader/entries/*.conf `

añadir al final `quiet` y `splash`
```
options root=PARTUUID=df4a860f-0cad-4bd0-8c75-e245a41eeab2 zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs quiet splash
```
---
```
sudo mkinitcpio -P
```

## AppArmor

Inserta la siguiente línea en la terminal
```
zgrep CONFIG_LSM= /proc/config.gz
```

### OPCIÓN 1 Para Grub
---
```
sudo nano /etc/default/grub
```

Modificar la siguiente línea de esta manera
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash lsm=landlock,lockdown,yama,integrity,apparmor,bpf audit=1"
```

Actualizar grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
---
### OPCIÓN 2 Para systemd-boot
---
```
sudo nano /boot/loader/entries/*.conf  
```

Modificar la siguiente línea de esta manera
```
options root=PARTUUID=df4a860f-0cad-4bd0-8c75-e245a41eeab2 zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs quiet splash lsm=landlock,lockdown,yama,integrity,apparmor,bpf audit=1
```
---

Instalar paquetes y dependencias
```
sudo pacman -S tk apparmor python-audit python-notify2
```

Activar los servicios
```
sudo systemctl enable --now apparmor.service
sudo systemctl enable --now auditd.service
```

Reiniciar el sistema
```
reboot
```
Crear grupo audit y añadir usuarios
```
sudo groupadd -r audit
sudo gpasswd -a $(whoami) audit
```

Añadir a `/etc/audit/auditd.conf` la siguiente línea
```
log_group = audit
```

Crear directorio autostart
```
mkdir -p ~/.config/autostart
```

Creamos el fichero `apparmor-notify.desktop`
```
nano ~/.config/autostart/apparmor-notify.desktop
```
```
[Desktop Entry]
Type=Application
Name=AppArmor Notify
Comment=Receive on screen notifications of AppArmor denials
TryExec=aa-notify
Exec=aa-notify -p -s 1 -w 60 -f /var/log/audit/audit.log
StartupNotify=false
NoDisplay=true
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
Añadimos los alias para lsd y bat

```
echo '# ALIAS' >> ~/.zshrc
echo "alias ls='lsd'" >> ~/.zshrc
echo "alias ll='lsd -l'" >> ~/.zshrc
echo "alias la='lsd -a'" >> ~/.zshrc
echo "alias lla='lsd -la'" >> ~/.zshrc
echo "alias cat='bat'" >> ~/.zshrc
```

---
## ZSH PLUGINS  

```
sudo pacman -S zsh-autocomplete zsh-autosuggestions zsh-history-substring-search zsh-syntax-highlighting 
```
Los plugins se descargan en la ruta `/usr/share/zsh/plugins/`

Añadimos los plugins al fichero `.zshrc` 
```
echo '# PLUGINS' >> ~/.zshrc
echo 'source /usr/share/zsh/plugins/zsh-autocomplete/zsh-autocomplete.plugin.zsh' >> ~/.zshrc
echo 'source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.plugin.zsh' >> ~/.zshrc
echo 'source /usr/share/zsh/plugins/zsh-history-substring-search/zsh-history-substring-search.zsh' >> ~/.zshrc
echo 'source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.plugin.zsh' >> ~/.zshrc
```
---

# Iconos 
<img src="icons/386451_arch%20linux_archlinux_icon.svg" width="50" height="50"><img src="icons/arch-linux-seeklogo.com.svg" width="50" height="50"><img src="icons/archlinux-icon.svg" width="50" height="50">

Puedes descargar iconos `.svg` de arch [aquí](https://github.com/KernelsAndDragons/ArchPostInstall/tree/main/icons)

# Fastfetch Mejorado

Instalar fastfetch
```
sudo pacman -S fastfetch
```
Navega a tu directorio `.config/fastfetch`
```
cd ~/.config/fastfetch
```
Si no existe la carpeta fastfetch , creala 
```
mkdir -p ~/.config/fastfetch
cd ~/.config/fastfetch
```
Genera la configuración por defecto 
```
fastfetch --gen-config
```
Elimina el archivo de configuración por defecto 
```
rm config.jsonc
```
Descarga mi configuración actualizada 
```
wget https://raw.githubusercontent.com/KernelsAndDragons/ArchPostInstall/refs/heads/main/config.jsonc
```
Cierra tu terminal y ábrela de nuevo.

Ahora, ejecuta 
```
fastfetch
```
![fastfetch](https://github.com/KernelsAndDragons/ArchPostInstall/blob/main/.github/fastfetch.png)

