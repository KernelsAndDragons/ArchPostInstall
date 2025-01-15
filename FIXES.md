# NO INICIA SESIÓN EN WAYLAND PERO SI EN X11 PERO SIN PRIVILEGIOS 

Es posible que tras la instalación no se haya creado el grupo polkitd, para crearlo manualmente.

```
sudo groupadd --gid 102 polkitd
```
```
sudo systemd-sysusers 
```
```
reboot
```
