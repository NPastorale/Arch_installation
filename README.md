# Introducción

Esto es una guía creada a modo de documentación, y recovery en caso de ser necesario, que responde a mis necesidades específicas para la instalación y configuración básica de Arch Linux en una Lenovo Legion Y540. Puede ser utilizada para cualquier otro sistema realizando pequeñas modificaciones en caso de que fuesen necesarias. La gran mayoría del proceso está basado en la [ArchWiki](https://wiki.archlinux.org/), en diferentes respuestas que fuí encontrando a los problemas que se me presentaron durante el uso de Arch y lo que aprendí en el proceso.

- [Instalación Arch Linux](#instalación-arch-linux)
  - [Instalación base](#instalación-base)
    - [Conexión a Internet y configuración de fecha y hora](#conexión-a-internet-y-configuración-de-fecha-y-hora)
    - [Definición y formateo de particiones](#definición-y-formateo-de-particiones)
    - [Montaje y creación de directorios necesarios](#montaje-y-creación-de-directorios-necesarios)
    - [Instalación y configuración del sistema base](#instalación-y-configuración-del-sistema-base)
    - [Configuración de GRUB](#configuración-de-grub)
    - [Pasos finales](#pasos-finales)
  - [Configuración](#configuración)
    - [Básica](#básica)
    - [Ajustes necesarios](#ajustes-necesarios)
    - [Paquetes para el funcionamiento básico](#paquetes-para-el-funcionamiento-básico)

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
- Instalar y ejecutar **reflector** para actualizar la lista de mirrors a la más rápida
  ```
  pacman -S reflector
  reflector -n 12 -p https --sort rate --score 10 --save /etc/pacman.d/mirrorlist
  ```

### Instalación y configuración del sistema base

- Instalar el sistema base y paquetes extra que van a ser necesarios
  ```
  pacstrap -i /mnt base base-devel bash-completion grub efibootmgr networkmanager linux linux-firmware intel-ucode vim
  ```
  - _base_ - Paquetes básicos iniciales de la instalación
  - _base-devel_ - Paquetes necesarios para utilizar AUR y administración del sistema
  - _bash-completion_ - Extensión del autocompletado de bash
  - _grub_ - Bootloader
  - _efibootmgr_ - Utilidad para booteo por UEFI
  - _networkmanager_ - Administrador de conexiones
  - _linux_ - Kernel
  - _linux-firmware_ - Firmware necesario para Intel Wireless-AC 9560
  - _intel-ucode_ - Microcode para Intel
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

- Montar la partición _EFI_
  ```
  mount /dev/nvme0n1p1 /boot/EFI
  ```
- Instalar GRUB
  ```
  grub-install --target=x86_64-efi --bootloader-id=grub-efi --efi-directory=/boot/EFI
  ```
- Configurar los locale de GRUB
  ```
  mkdir /boot/grub/locale
  cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
  ```
- Generar el archivo de configuración de GRUB
  ```
  grub-mkconfig -o /boot/grub/grub.cfg
  ```

### Pasos finales

- Definir la contraseña de **root**
  ```
  passwd
  ```
- Activar NetworkManager
  ```
  systemctl enable NetworkManager
  ```
- Salir del chroot, desmontar las particiones y reiniciar
  ```
  exit
  umount -R /mnt
  reboot
  ```

## Configuración

### Básica

- Crear un link de **vim** a **vi**
  ```
  ln -sf /usr/bin/vim usrb/bin/vi
  ```
- Editar los permisos de `sudo` y descomentar la línea `%wheel ALL=(ALL) ALL`
  ```
  visudo
  ```
- Agregar el usuario principal y asignarle una contraseña
  ```
  useradd -m -g users -G wheel,audio,video <USUARIO>
  passwd <USUARIO>
  ```
- Agregar el nombre del usuario creado
  ```
  chfn -f <NOMBRE_COMPLETO> <USUARIO>
  ```
- Desloguearse e iniciar sesión con el nuevo usuario

### Ajustes necesarios

- Conectarse al WiFi
  ```
  nmcli radio wifi
  nmcli dev wifi list
  sudo nmcli --ask dev wifi connect <NOMBRE_DE_LA_RED>
  ```
- Instalar paquetes necesarios para la Y540
  ```
  sudo pacman -S network-manager-applet xf86-video-intel nvidia xorg-server man-db git
  ```
  - _network-manager-applet_ - Applet de NetworkManager
  - _xf86-video-intel_ - Drivers de Xorg para Intel **(Investigar si conviene utilizar modeset)**
  - _nvidia_ - Drivers de NVIDIA
  - _xorg-server_ - Servidor de X
  - _man-db_ - Implementación de man pages
  - _git_ - Sistema de control de versiones
- Instalar el helper de AUR **yay**
  ```
  git clone https://aur.archlinux.org/yay.git
  cd yay
  makepkg -si
  ```

### Paquetes para el funcionamiento básico

- Inslatar un DM, WM y utilidad de wallpaper TODO: definir los paquetes
- Instalar desde AUR **Google Chrome** y **Visual Studio Code**
  ```
  yay google-chrome
  yay visual-studio-code-bin
  ```
  <!--
  TODO: AGREGAR ACA LIGHTDM COMO DISPLAY MANAGER Y LIGHTDM-MINI-GREETER
    instalar yay lightdm-mini-greeter, cambiar la configuracion en /etc/lightdm/lightdm.conf y /etc/lightdm/lightdm-mini-greeter.conf
  TODO: INVESTIGAR DIFERENTES TEMAS PARA LIGHTDM
  TODO: HACER UN SYSTEMCTL ENABLE Y EL DISPLAY MANGER QUE HAYA INSTALADO (LIGHTDM)
  TODO: INSTALAR CHROME
  TODO: INSTALAR DELUGE
  TODO: INSTALAR DELUGE
  TODO: PROBAR NEWSBOAT
  TODO: PROBAR REMMINA
  TODO: PROBAR URXVT Y TERMITE
  TODO: COMPARAR SCREEN CON TMUX
  TODO: PROBAR RANGER
  TODO: PROBAR THUNAR
  TODO: INVESTIGAR KEYBOARD LAYOUT SWITCHERS
  TODO: WATERFALL PARECE UNA BUENA ALTERNATIVA PARA VER FUENTES
  TODO: PROBAR CLIGHT PARA EL BRILLO DE LA PANTALLA Y QUE LO AJUSTE AUTOMATICAMENTE
  TODO: PROBAR QUE XBACKLIGHT FUNCIONE CON MODESETTING EN INTEL
  TODO: INVESTIGAR BLUETOOTH, BLUEMAN, BLUEZ Y BLUEZ-UTILS
  TODO: INVESTIGAR SXHKD COMO REEMPLAZO DE i3 EN LA ASIGNACION DE HOTKEYS
  TODO: INVESTIGAR DIFERENTES SOLUCIONES DE BACKUP
  TODO: AGREGAR FEH
  TODO: AGREGAR FOFI
  TODO: INVESTIGAR LOGOUT UI
  TODO: AGREGAR INSTRUCCIONES PARA EL SCROLLING NATURAL DEL TRACKPAD
  TODO: AGREGAR INSTRUCCIONES PARA EL TAP TO CLICK
  TODO: TEMA PERSONALIZADO DE GRUB
  TODO: GRUB HOLD SHIFT PROBAR FUNCIONAMIENTO
  TODO: PROBAR PANTALLA DE BLOQUEO DE i3
  TODO: INVESTIGAR BLUR EN COMPTON (PICOM) U OTRO COMPOSITOR
  TODO: INVESTIGAR UNDERVOLT
  TODO: INVESTIGAR OPTIMUS
  TODO: CONFIGURACION DE i3
  TODO: INSTRUCCIONES PARA QUE FUNCIONE EL CONTROL DEL BRILLO (XBACKLIGHT)
  TODO: INVESTIGAR PLYMOUTH Y OTROS BOOT SPLASH SCREEN
  TODO: INVESTIGAR CPU FREQUENCY SCALING
  TODO: LEER IMPROVING PERFORMANCE EN LA WIKI
  TODO: LEER SOLID STATE DRIVE EN LA WIKI
  TODO: CUSTOMIZAR EL PROMPT PS1
  TODO: LEER SYSTEM MAINTENANCE EN LA WIKI
  TODO: LEER SECURITY EN LA WIKI
  TODO: Agregar sección con VISUDO
  -->
