---

title: Instalar adJ 6.8b1 en un portatil Lenovo E490 junto con Ubuntu y Windows
date: 2021-01-10 16:18 UTC
tags: 

---

# Preparar el Windows pre-configurado y deshabilitar Bitlocker

En su primer arranque este portátil instalará un Windows.
Es posible que el disco este cifrado con Bitlocker, y en tal caso debe
desactivar Bitlocker para poder instalar otro sistema operativo.

Para desactivarlo ingresar como un usuario administrador local (o crear uno
si hace falta), y desde el buscador digite `Administrar Bitlocker` o 
desde el `Panel de control` → `Sistema y Seguridad` → 
`Cifrado de Unidad Bitlocker`.

Elegir la opción `Desactivar Bitlocker` y aceptar los pasos que siguen 
incluyendo el reinicio.

# Instalar Ubuntu 

Poner el instalador de Ubuntu en una memoria USB y arrancar el computador 
con esta insertada.  Presionar Enter para obtener un menú de opciones y 
elegir F12 para tener otras opciones de arranque. Como dispositivo
de arranque elegir USB.

El instalador de Ubuntu 20.04 es aceptado por el BIOS UEFI de este portátil
y arrancará permitiendo redimensionar la partición de Windows.  Se sugiere
no eliminar la partición EFI pues se requerirá para arrancar OpenBSD/adJ.
Deje espacio tanto para el Ubuntu como para el adJ.

Realice el proceso típico de instalación de Ubuntu, una vez concluido en
cada arranque debe poder ver GRUB que le permite elegir entre Ubuntu y 
Windows.


# Instalar adJ

Descargue y aliste el instalador de adJ 6.8b1 en una USB (ver
<https://aprendiendo.pasosdeJesus.org>)

Con la configuración por omisión de este portátil, no podrá arrancar
con esa USB, antes tendrá que ingresar al BIOS UEFI y desde `Security`
deshabilitar `Secure Boot`.  Tras hacerlo ya podrá arrancar
con la USB de adJ --análogo al arranque de la USB de Ubuntu-- y de hecho
tendrá que mantener deshabilitado `Secure Boot` para poder arrancar
posteriormente el sistema que instale.

El proceso de instalación es típico (ver
<https://pasosdejesus.github.io/usuario_adJ/index.html#idm1671239424>), 
utilice el espacio que reservó con
el Ubuntu como espacio para OpenBSD y haga las subparticiones que
requiera (sugerimos / de al menos 20G, /var de al menos 20G,
/build si planea compilar el sistema de al menos 10G y /home).

Una vez concluido el proceso seguirá sin poder arrancar el adJ hasta
que lo configure como una entrada más del GRUB, según se documenta en
<https://dhobsd.defensor.info/grub2.html>, desde el Ubuntu edite el
archivo `/etc/grub.d/40_custom` y agregue unas lineas de la forma:

<pre>
menuentry "adJ" {
  set root=(hd0,gpt1)
  chainloader /efi/Boot/bootx64.efi
}
</pre>

edite `/etc/default/grub` y cambia las líneas:

<pre>
GRUB_DEFAULT=4   # Cambie 4 por el númer de la entrada de OpenBSD en el menu
#!GRUB_HIDDEN_TIMEOUT=0
!GRUB_HIDDEN_TIMEOUT_QUIET=true
</pre>

y finalmente ejecute
<pre>
sudo update-grub2
</pre>


