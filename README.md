# Introducción

Esto es una guía creada a modo de documentación, y recovery en caso de ser necesario, que responde a mis necesidades específicas para la instalación y configuración básica de Arch Linux en una Lenovo Legion Y540. Puede ser utilizada para cualquier otro sistema realizando pequeñas modificaciones en caso de que fuesen necesarias. La gran mayoría del proceso está basado en la [ArchWiki](https://wiki.archlinux.org/), en diferentes respuestas que fuí encontrando a los problemas que se me presentaron durante el uso de Arch y lo que aprendí en el proceso.

- [Instalación Arch Linux](#instalaci%c3%b3n-arch-linux)
  - [Instalación base](#instalaci%c3%b3n-base)
    - [Conexión a Internet y configuración de fecha y hora](#conexi%c3%b3n-a-internet-y-configuraci%c3%b3n-de-fecha-y-hora)
    - [Definición y formateo de particiones](#definici%c3%b3n-y-formateo-de-particiones)
    - [Montaje y creación de directorios necesarios](#montaje-y-creaci%c3%b3n-de-directorios-necesarios)
    - [Instalación y configuración del sistema base](#instalaci%c3%b3n-y-configuraci%c3%b3n-del-sistema-base)
    - [Configuración de GRUB](#configuraci%c3%b3n-de-grub)
    - [Pasos finales](#pasos-finales)
  - [Configuración](#configuraci%c3%b3n)

# Instalación Arch Linux

## Instalación base

### Conexión a Internet y configuración de fecha y hora

- Revisar que la placa de WiFi no esté bloqueada. De ser así, habilitarla con las teclas de la notebook
  ```
  rfkill
  ```
- Conectarse al WiFi
  ```
  wifi-menu
  ```
- Verificar tener una IP asignada y corroborarlo
  ```
  ip a
  ping google.com
  ```
- Activar NTP
  ```
  timedatectl set-ntp true
  ```
- Comprobar la fecha y hora
  ```
  timedatectl status
  ```

### Definición y formateo de particiones

- Listar los discos para decidir sobre cuál instalar
  ```
  fdisk -l
  ```
- Definir las particiones
  ```
  fdisk /dev/nvme0n1
  ```
  | Mount Point | Partition        |  Partition Type  | Suggested Size |
  | ----------- | ---------------- | :--------------: | -------------: |
  | `/mnt/efi`  | `/dev/nvme0n1p1` |    EFI System    |        512 MiB |
  | `/mnt`      | `/dev/nvme0n1p2` | Linux filesystem |         50 GiB |
  | `/mnt/home` | `/dev/nvme0n1p3` | Linux filesystem |      Remainder |
- Formatear las particiones
  ```
  mkfs.fat -F32 /dev/nvme0n1p1
  mkfs.ext4 /dev/nvme0n1p2
  mkfs.ext4 /dev/nvme0n1p3
  ```

### Montaje y creación de directorios necesarios

- Montar la partición _root_
  ```
  mount /dev/nvme0n1p2 /mnt
  ```
- Crear el directorio _home_ y montar la partición
  ```
  mkdir /mnt/home
  mount /dev/nvme0n1p3 /mnt/home
  ```

### Instalación y configuración del sistema base

- Instalar el sistema base y paquetes extra que van a ser necesarios
  ```
  pacstrap -i /mnt base base-devel bash-completion grub efibootmgr networkmanager network-manager-applet linux linux-firmware xf86-video-intel intel-ucode nvidia xorg-server vim
  ```
  - _base_ - Paquetes básicos iniciales de la instalación
  - _base-devel_ - Paquetes necesarios para utilizar AUR y administración del sistema
  - _bash-completion_ - Extensión del autocompletado de bash
  - _grub_ - Bootloader
  - _efibootmgr_ - Utilidad para booteo por UEFI
  - _networkmanager_ - Administrador de conexiones
  - _network-manager-applet_ - Applet de NetworkManager
  - _linux_ - Kernel
  - _linux-firmware_ - Firmware necesario para Intel Wireless-AC 9560
  - _xf86-video-intel_ - Drivers de Xorg para Intel **(Investigar si conviene utilizar modeset)**
  - _intel-ucode_ - Microcode para Intel
  - _nvidia_ - Drivers de NVIDIA
  - _xorg-server_ - Servidor de X
  - _vim_ - El mejor editor de texto
- Generar el fstab para el automontado de las particiones
  ```
  genfstab -U /mnt >> /mnt/etc/fstab
  ```
- Cambiar root a _/mnt_
  ```
  arch-chroot /mnt
  ```
- Editar `/etc/locale.gen` y descomentar `en_US.UTF-8 UTF-8`
- Crear el archivo `/etc/locale.conf`, editarlo y agregar una línea con `LANG=en_US.UTF-8`
- Generar los locale
  ```
  locale-gen
  ```
- Configurar la zona horaria
  ```
  ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
  ```
- Asumiendo que el hardware clock está en UTC, ejecutar
  ```
  hwclock --systohc
  ```
- Crear y editar el archivo `/etc/hostname`, agregar _`myhostname`_
- Editar el archivo `/etc/hosts` de la siguiente forma
  ```
  127.0.0.1  localhost
  ::1        localhost
  127.0.1.1  myhostname.localdomain  myhostname
  ```

### Configuración de GRUB

- Crear el directorio para montar la partición _EFI_
  ```
  mkdir /boot/EFI
  ```

* Montar la partición _EFI_
  ```
  mount /dev/nvme0n1p1 /boot/EFI
  ```
* Instalar GRUB
  ```
  grub-install --target=x86_64-efi --bootloader-id=grub-efi --efi-directory=/boot/EFI
  ```
* Configurar los locale de GRUB
  ```
  mkdir /boot/grub/locale
  cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
  ```
* Generar el archivo de configuración de GRUB
  ```
  grub-mkconfig -o /boot/grub/grub.cfg
  ```

### Pasos finales

- Definir la contraseña de **root**
  ```
  passwd
  ```
- Salir del chroot, desmontar las particiones y reiniciar
  ```
  exit
  umount -R /mnt
  reboot
  ```

## Configuración

- Activar e iniciar NetworkManager
  ```
  systemctl enable NetworkManager
  ```
- Agregar el usuario principal y asignarle una contraseña
  ```
  useradd -m -g users -G wheel,audio,video <USUARIO>
  passwd <USUARIO>
  ```

* Instalar el helper de AUR **yay**
  ```
  git clone https://aur.archlinux.org/yay.git
  cd yay
  makepkg -si
  ```

<!--
TODO AGREGAR ACA EL DESKTOP MANAGER QUE ELIJA
TODO HACER UN SYSTEMCTL ENABLE Y EL DM QUE HAYA INSTALADO
TODO INSTALAR CHROME
TODO INSTALAR DELUGE
TODO INSTALAR DELUGE
TODO PROBAR NEWSBOAT
TODO PROBAR REMMINA
TODO PROBAR URXVT Y TERMITE
TODO COMPARAR SCREEN CON TMUX
TODO PROBAR RANGER
TODO PROBAR THUNAR
TODO INVESTIGAR KEYBOARD LAYOUT SWITCHERS
TODO WATERFALL PARECE UNA BUENA ALTERNATIVA PARA VER FUENTES
TODO PROBAR CLIGHT PARA EL BRILLO DE LA PANTALLA Y QUE LO AJUSTE AUTOMATICAMENTE
TODO PROBAR QUE XBACKLIGHT FUNCIONE CON MODESETTING EN INTEL
TODO INVESTIGAR BLUETOOTH, BLUEMAN, BLUEZ Y BLUEZ-UTILS
TODO INVESTIGAR SXHKD COMO REEMPLAZO DE i3 EN LA ASIGNACION DE HOTKEYS
TODO INVESTIGAR DIFERENTES SOLUCIONES DE BACKUP
TODO AGREGAR FEH
TODO AGREGAR FOFI
TODO INVESTIGAR LOGOUT UI
-->
