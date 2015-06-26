---
title: Instalaci_n_de_OpenBSD_en_computador_Simply_RI80
date: 2008-04-14
tags:
---
Hasta donde entiendo Simply es una empresa que ensambla portátiles en Colombia. (Felicitaciones!)

No es fácil lograr soporte técnico en Internet, los manuales no tiene datos de contacto, su número de soporte parece ser 018000910250

El modelo RI 80 adquirible en almacenes Exito (al menos de Cali) en Feb.2008, tiene Windows Vista Starter Edition preinstalado. Además de las dificultades de compatibilidad de ese sistema, sólo permite ejecutar 3 aplicaciones simultaneamente. Según el soporte técnico de Simply porque es la capacidad del hardware (ver dmesg).

La instalación no ha sido todo menos directa por lo que aquí documento.

<pre>
# dmesg

OpenBSD 4.3-beta (GENERIC) #664: Sun Feb 24 14:57:44 MST 2008
    deraadt?i386:/usr/src/sys/arch/i386/compile/GENERIC
cpu0: Intel(R) Celeron(R) CPU 530 @ 1.73GHz ("GenuineIntel?" 686-class) 1.73 GHz
cpu0: FPU,V86,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,CFLUSH,DS,ACPI,MMX,FXSR,SSE,SSE2,SS,TM,SBF,SSE3,MWAIT,DS-CPL,TM2,CX16,xTPR
real mem  = 1063677952 (1014MB)
avail mem = 1020583936 (973MB)
mainbus0 at root
bios0 at mainbus0: AT/286+ BIOS, date 08/22/07, BIOS32 rev. 0 @ 0xfdc10, SMBIOS rev. 2.4 @ 0x3f6e0000? (30 entries)
bios0: vendor Phoenix Technologies LTD version "1.00" date 08/22/2007
bios0: OEM OEM
acpi0 at bios0: rev 2
acpi0: tables DSDT FACP APIC HPET MCFG TMOR APIC SLIC SSDT SSDT SSDT SSDT SSDT
acpi0: wakeup devices HDEF(S4) PXSX(S4) PXSX(S4) PXSX(S4) PXSX(S4) PXSX(S4) PXSX(S4) USB1(S3) USB2(S3) USB3(S3) USB4(S3) USB5(S3) EHC1(S3) EHC2(S3)
acpitimer0 at acpi0: 3579545 Hz, 24 bits
acpihpet0 at acpi0: 14318179 Hz
acpiprt0 at acpi0: bus 0 (PCI0)
acpiprt1 at acpi0: bus -1 (PEGP)
acpiprt2 at acpi0: bus 2 (RP01)
acpiprt3 at acpi0: bus -1 (RP02)
acpiprt4 at acpi0: bus 4 (RP03)
acpiprt5 at acpi0: bus 6 (RP04)
acpiprt6 at acpi0: bus -1 (RP05)
acpiprt7 at acpi0: bus -1 (RP06)
acpiprt8 at acpi0: bus 8 (PCIB)
acpiec0 at acpi0
acpicpu0 at acpi0: C2, C1
acpitz0 at acpi0: critical temperature 99 degC
acpibat0 at acpi0: BAT0 model "OEM" type LiON oem "OEM"
acpibtn0 at acpi0: PWRB
acpiac0 at acpi0: AC unit online
acpibtn1 at acpi0: LID_
acpibtn2 at acpi0: SLPB
bios0: ROM list: 0xc0000/0xee00! 0xe0000/0x1800!
cpu0 at mainbus0
pci0 at mainbus0 bus 0: configuration mode 1 (no bios)
pchb0 at pci0 dev 0 function 0 "Intel GM965 Host" rev 0x03
agp0 at pchb0: aperture at 0xd0000000, size 0x8000000
vga1 at pci0 dev 2 function 0 "Intel GM965 Video" rev 0x03
wsdisplay0 at vga1 mux 1: console (80x25, vt100 emulation)
wsdisplay0: screen 1-5 added (80x25, vt100 emulation)
"Intel GM965 Video" rev 0x03 at pci0 dev 2 function 1 not configured
uhci0 at pci0 dev 26 function 0 "Intel 82801H USB" rev 0x03: irq 5
uhci1 at pci0 dev 26 function 1 "Intel 82801H USB" rev 0x03: irq 11
ehci0 at pci0 dev 26 function 7 "Intel 82801H USB" rev 0x03: irq 7
usb0 at ehci0: USB revision 2.0
uhub0 at usb0 "Intel EHCI root hub" rev 2.00/1.00 addr 1
azalia0 at pci0 dev 27 function 0 "Intel 82801H HD Audio" rev 0x03: irq 11
azalia0: codecs?: Realtek ALC883
audio0 at azalia0
ppb0 at pci0 dev 28 function 0 "Intel 82801H PCIE" rev 0x03: irq 10
pci1 at ppb0 bus 2
ppb1 at pci0 dev 28 function 2 "Intel 82801H PCIE" rev 0x03: irq 7
pci2 at ppb1 bus 4
ppb2 at pci0 dev 28 function 3 "Intel 82801H PCIE" rev 0x03: irq 10
pci3 at ppb2 bus 6
re0 at pci3 dev 0 function 0 "Realtek 8101E" rev 0x01: RTL8101E (0x3400), irq 10, address 00:03:0d:7c:91:29
rlphy0 at re0 phy 7: RTL8201L 10/100 PHY, rev. 1
uhci2 at pci0 dev 29 function 0 "Intel 82801H USB" rev 0x03: irq 10
uhci3 at pci0 dev 29 function 1 "Intel 82801H USB" rev 0x03: irq 10
uhci4 at pci0 dev 29 function 2 "Intel 82801H USB" rev 0x03: irq 7
ehci1 at pci0 dev 29 function 7 "Intel 82801H USB" rev 0x03: irq 10
usb1 at ehci1: USB revision 2.0
uhub1 at usb1 "Intel EHCI root hub" rev 2.00/1.00 addr 1
ppb3 at pci0 dev 30 function 0 "Intel 82801BAM Hub-to-PCI" rev 0xf3
pci4 at ppb3 bus 8
ichpcib0 at pci0 dev 31 function 0 "Intel 82801HBM LPC" rev 0x03: PM disabled
pciide0 at pci0 dev 31 function 1 "Intel 82801HBM IDE" rev 0x03: DMA, channel 0 configured to compatibility, channel 1 configured to compatibility
atapiscsi0 at pciide0 channel 0 drive 0
scsibus0 at atapiscsi0: 2 targets
cd0 at scsibus0 targ 0 lun 0: <TSSTcorp, CDDVDW TS-L632H, CH00> SCSI0 5/cdrom removable
cd0(pciide0:0:0): using PIO mode 4, Ultra-DMA mode 2
pciide0: channel 1 disabled (no drives)
ahci0 at pci0 dev 31 function 2 "Intel 82801HBM SATA" rev 0x03: irq 10, AHCI 1.1
ahci0: PHY offline on port 1
ahci0: PHY offline on port 2
scsibus1 at ahci0: 32 targets
sd0 at scsibus1 targ 0 lun 0: <ATA, Hitachi HTS54161, SBDO> SCSI3 0/direct fixed
sd0: 114473MB, 14593 cyl, 255 head, 63 sec, 512 bytes/sec, 234441648 sec total
ichiic0 at pci0 dev 31 function 3 "Intel 82801H SMBus" rev 0x03: irq 10
iic0 at ichiic0
spdmem0 at iic0 addr 0x50: 1GB DDR2 SDRAM non-parity PC2-5300CL5 SO-DIMM
usb2 at uhci0: USB revision 1.0
uhub2 at usb2 "Intel UHCI root hub" rev 1.00/1.00 addr 1
usb3 at uhci1: USB revision 1.0
uhub3 at usb3 "Intel UHCI root hub" rev 1.00/1.00 addr 1
usb4 at uhci2: USB revision 1.0
uhub4 at usb4 "Intel UHCI root hub" rev 1.00/1.00 addr 1
usb5 at uhci3: USB revision 1.0
uhub5 at usb5 "Intel UHCI root hub" rev 1.00/1.00 addr 1
usb6 at uhci4: USB revision 1.0
uhub6 at usb6 "Intel UHCI root hub" rev 1.00/1.00 addr 1
isa0 at ichpcib0
isadma0 at isa0
pckbc0 at isa0 port 0x60/5
pckbd0 at pckbc0 (kbd slot)
pckbc0: using irq 1 for kbd slot
wskbd0 at pckbd0: console keyboard, using wsdisplay0
pms0 at pckbc0 (aux slot)
pckbc0: using irq 12 for aux slot
wsmouse0 at pms0 mux 0
pcppi0 at isa0 port 0x61
midi0 at pcppi0: <PC speaker>
spkr0 at pcppi0
npx0 at isa0 port 0xf0/16: reported by CPUID; using exception 16
biomask edfd netmask edfd ttymask ffff
mtrr: Pentium Pro MTRR support
ugen0 at uhub1 port 3 "Realtek RTL8187" rev 2.00/1.00 addr 2
softraid0 at root
root on sd0a swap on sd0b dump on sd0b
</pre>

!Instalación sistema base

Los instaladores i386 de 4.2 se congelan con un kernel panic al arranque (similar a lo descrito en http://archives.neohapsis.com/archives/openbsd/2007-05/2465.html)

El instalador de 4.3 (4.2-current en el momento de este escrito) logra avanzar pero se congelan después de mostrar como monta disco raíz.

Una persona de soporte de Simply encontró que debe deshabilitarse AHCI en el BIOS para que pueda continuar la instalación. Por cierto se pierde la garantía de software cuando se instala algo diferente a Windows Vista SE.

Aunque inicialmente intenté dejar Vista y reacomodar espacio para una nueva partición de OpenBSD, al cambiar el tamaño del sistema de archivos NTFS del Vista con gparted (de un RIP Linux) se estropeó el Windows de forma que (afortunadamente) no arrancó más.


!X-Window

```Xorg -configure``` no detecta una configuración usable, pues no detecta datos del monitor, después de hacer algunos cambios llegué al siguiente ```/etc/X11/xorg.conf```

<pre>
Section "ServerLayout"
        Identifier     "X.org Configured"
        Screen      0  "Screen0" 0 0
        InputDevice    "Mouse0" "CorePointer"
        InputDevice    "Keyboard0" "CoreKeyboard"
EndSection

Section "Files"
        RgbPath      "/usr/X11R6/share/X11/rgb"
        ModulePath   "/usr/X11R6/lib/modules"
        FontPath     "/usr/X11R6/lib/X11/fonts/misc/"
        FontPath     "/usr/X11R6/lib/X11/fonts/TTF/"
        FontPath     "/usr/X11R6/lib/X11/fonts/OTF"
        FontPath     "/usr/X11R6/lib/X11/fonts/Type1/"
        FontPath     "/usr/X11R6/lib/X11/fonts/100dpi/"
        FontPath     "/usr/X11R6/lib/X11/fonts/75dpi/"
EndSection

Section "Module"
        Load  "GLcore"
        Load  "dbe"
        Load  "extmod"
        Load  "glx"
        Load  "record"
        Load  "xtrap"
        Load  "freetype"
        Load  "type1"
EndSection

Section "InputDevice"
        Identifier  "Keyboard0"
        Driver      "kbd"
        Option "XkbLayout" "es"
EndSection

Section "InputDevice"
        Identifier  "Mouse0"
        Driver      "mouse"
        Option            "Protocol" "wsmouse"
        Option            "Device" "/dev/wsmouse"
        Option            "ZAxisMapping" "4 5 6 7"
EndSection

Section "Monitor"
        Identifier   "Monitor0"
        VendorName   "Monitor Vendor"
        ModelName    "Monitor Model"
        Option "DPMS"
#    HorizSync   31.5 - 64.3
#    VertRefresh 40-150
EndSection

Section "Device"
        ### Available Driver options are:-
        ### Values: <i>: integer, <f>: float, <bool>: "True"/"False",
        ### <string>: "String", <freq>: "<f> Hz/kHz/MHz"
        ### [arg]: arg optional
        #Option     "ShadowFB"                   # [<bool>]
        #Option     "DefaultRefresh"             # [<bool>]
        #Option     "ModeSetClearScreen"         # [<bool>]
        Identifier  "Card0"
        Driver      "intel"
        VendorName  "Intel Corporation"
        BoardName   "Mobile GM965/GL960 Integrated Graphics Controller"
        !BusID       "PCI:0:2:0"
EndSection

Section "Screen"
        Identifier "Screen0"
        Device     "Card0"
        Monitor    "Monitor0"
        DefaultDepth 16
        SubSection "Display"
                Viewport   0 0
                Depth     1
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     4
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     8
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     15
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     16
                Modes "1280x1024"
        EndSubSection
        SubSection "Display"
                Viewport   0 0
                Depth     24
        EndSubSection
EndSection
</pre>

----

Esta información se cede al dominio público y se dedica al Autor de la belleza.

