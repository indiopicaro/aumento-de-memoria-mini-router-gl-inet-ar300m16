# Expandiendo la Memoria Interna del Mini Router GL.iNet AR300M16 con una Memoria USB

<p align="center">
  <img src="https://github.com/user-attachments/assets/7c1b8f42-2f69-4cae-bc31-5a09c7aa8a8c" width="300" style="display: inline-block; margin-right: 10px;">
  <img src="https://github.com/user-attachments/assets/0c824499-30d9-4872-bbdc-18c02a2781e8" width="300" style="display: inline-block;">
</p>

## Descripción

Este repositorio proporciona una guía paso a paso para extender la memoria interna del mini router GL.iNet AR300M16 utilizando una memoria USB. Esto es útil para instalar paquetes adicionales o almacenar archivos sin preocuparse por el espacio limitado de la memoria interna del dispositivo.

### Requisitos

- Mini router GL.iNet AR300M16

- Memoria USB (preferiblemente de al menos 8GB, formateada en ext4)

- Acceso a la interfaz de SSH del router

- Conexión a Internet para la instalación de paquetes

# Pasos de Configuración

## 1. Identificar la Memoria USB

Una vez dentro, lista los dispositivos de almacenamiento disponibles:

    lsblk
    
## 3. Formatear la Memoria USB en ext4 (si es necesario)
Si la memoria no está formateada en ext4, puedes hacerlo con:

     mkfs.ext4 /dev/sda1

  Monta la memoria USB:

    mount /dev/sda1 /mnt/usb

  Verifica que está montada correctamente:

    df -h

## 4. Instala los paquetes requeridos

    opkg update
    opkg install block-mount kmod-fs-ext4 e2fsprogs parted kmod-usb-storage

  (NOTA) si existe dificultad en la instalacion de algun paquete. solo queda desinstalar alguno desde el apartado de complementos en la interfaz web.

## 5. Particion de memoria USB

    DISK="/dev/sda"
    parted -s ${DISK} -- mklabel gpt mkpart extroot 2048s -2048s
    DEVICE="${DISK}1"

## 6. Configuracion de extroot

    eval $(block info ${DEVICE} | grep -o -e 'UUID="\S*"')
    eval $(block info | grep -o -e 'MOUNT="\S*/overlay"')
    uci -q delete fstab.extroot
    uci set fstab.extroot="mount"
    uci set fstab.extroot.uuid="${UUID}"
    uci set fstab.extroot.target="${MOUNT}"
    uci commit fstab

## 7. Configuracion rootfs_data / ubifs

    ORIG="$(block info | sed -n -e '/MOUNT="\S*\/overlay"/s/:\s.*$//p')"
    uci -q delete fstab.rwm
    uci set fstab.rwm="mount"
    uci set fstab.rwm.device="${ORIG}"
    uci set fstab.rwm.target="/rwm"
    uci commit fstab

## 8. Transferencia de datos

    mount ${DEVICE} /mnt
    tar -C ${MOUNT} -cvf - . | tar -C /mnt -xf -

## 9. aplicacion de cambios

    reboot
