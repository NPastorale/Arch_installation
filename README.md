# Instalación Arch Linux
* Revisar que no esté bloqueada la placa de WiFi
    ```
    rfkill
    ```
* Conectarse al WiFi
    ```
    wifi-menu
    ```
* Verificar tener una IP asignada y corroborarlo
  ```
  ip a
  ping google.com
  ```
* Activar NTP
  ```
  timedatectl set-ntp true
  ```
* Comprobar la fecha y hora
  ```
  timedatectl status
  ```
* Listar los discos para decidir sobre cuál instalar
  ```
  fdisk -l
  ```
* Definir las particiones
  ```
  fdisk /dev/nvme0n1
  ```
    | Mount Point | Partition        |  Partition Type  | Suggested Size |
    | ----------- | ---------------- | :--------------: | -------------: |
    | `/mnt/efi`  | `/dev/nvme0n1p1` |    EFI System    |        512 MiB |
    | `/mnt`      | `/dev/nvme0n1p2` | Linux filesystem |         50 GiB |
    | `/mnt/home` | `/dev/nvme0n1p3` | Linux filesystem |      Remainder |
* Formatear las particiones
  * ```
    mkfs.fat -F32 /dev/nvme0n1p1
    ```
  * ```
    mkfs.ext4 /dev/nvme0n1p2
    ```
  * ```
    mkfs.ext4 /dev/nvme0n1p3
    ```
* Montar la partición *root*
  ```
  mount /dev/nvme0n1p2 /mnt
  ```
* Crear el directorio *home* y montar la partición
  ```
  mkdir /mnt/home
  mount /dev/nvme0n1p3 /mnt/home
  ```
* Instalar el sistema base y paquetes extra que van a ser necesarios
  ```
  pacstrap -i /mnt base base-devel grub efibootmgr network-manager-applet networkmanager wireless_tools wpa_supplicant linux linux-firmware xf86-video-intel intel-ucode
  ```
* Cambiar root a */mnt*
  ```
  arch-chroot /mnt
  ```
* Generar el fstab para el automontado de las particiones
  ```
  genfstab -U /mnt >> /mnt/etc/fstab
  ```
* Editar `/etc/locale.gen` y descomentar `en_US.UTF-8 UTF-8`
* Crear el archivo `/etc/locale.conf`, editarlo y agregar una línea con `LANG=en_US.UTF-8`
* Generar los locale
  ```
  locale-gen
  ```
* Configurar la zona horaria
  ```
  ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
  ```
* Asumiendo que el hardware clock está en UTC, ejecutar
  ```
  hwclock --systohc
  ```
* Crear y editar el archivo `/etc/hostname`, agregar *`myhostname`*
* Editar el archivo `/etc/hosts` de la siguiente forma
  ```
  127.0.0.1  localhost
  ::1        localhost
  127.0.1.1  myhostname.localdomain  myhostname
  ```
* Crear el directorio para montar la partición EFI
  ```
  mkdir /boot/EFI
  ```
* Montar la partición *EFI*
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
* Definir la contraseña de **root**
  ```
  passwd
  ```
* Salir del chroot, desmontar las particiones y reiniciar
  ```
  exit
  umount -R /mnt
  reboot
  ```