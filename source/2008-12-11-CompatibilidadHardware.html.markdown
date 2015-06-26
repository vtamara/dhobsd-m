---
title: CompatibilidadHardware
date: 2008-12-11
tags:
---
##TARJETAS ETHERNET

La gran mayoría son compatibles, pero NO son compatibles en OpenBSD 4.4:

* AMD64 - Sundance ST201, [http://www.nabble.com/openbsd-4.3-amd64-and-d-link-dfe-550tx-(st201)-td19656344.html]
* I386 - Encore ENL832-TX-RENT



##TARJETAS INALAMBRICAS

En OpenBSD-4.5 de Abr.2008 se revisaron páginas man de los controaldores
wireless, se extrajo la sección de hardware y se reorganizaron las más de 520 tarjetas  (gracias a ```awk``` y ```vim```) comenzando con el fabricante  (una versión inicial de este listado esta disponible como [ PDF | http://dhobsd.pasosdejesus.org/spages/inalambricas-OpenBSD-dic2008.pdf]):

| Fabricante | Modelo | Bus | Modos | Controlador |
| !AirLive |  WN-5000PCI |  PCI |  | ral |
| !AirVast |  WM168b Prism-3 |  USB |  | wi |
| !AmbiCom |  WL1100C-CF Prism-3 |  CF |  | wi |
| !AmbiCom |  WL54CF 88W8385 |  PCMCIA |  b/g | malo |
| !GigaFast |  WF721-AEX (R* serial) GRF5101 |  !CardBus |  | rtw |
| !LanReady |  WP2000 |  PCI |  | atw |
| !LevelOne |  WNC-0301 v2 |  PCI |  | ral |
| !LevelOne |  WNC-0301USB v3 |  |  | rum |
| !LevelOne |  WPC-0101 SA2400 |  !CardBus |  | rtw |
| !LevelOne |  WPC-0301 v2 |  !CardBus |  | ral |
| !OvisLink |  !AirLive WL-1120PCM SA2400 |  !CardBus |  | rtw |
| !OvisLink |  EVO-W54PCI |  PCI |  | ral |
| !PheeNet |  HWL-PCIG/RA |  PCI |  | ral |
| !PheeNet |  WL-503IA Prism-3 |  USB |  | wi |
| !TekComm |  NE-9321-g |  PCI |  | ral |
| !TekComm |  NE-9428-g |  !CardBus |  | ral |
| !ViewSonic |  Airsync Prism-2/5 |  USB |  | wi |
| 3Com |  !AirConnect 3CRWE737A Spectrum24 |  PCMCIA |  | wi |
| 3Com |  !AirConnect 3CRWE777A Prism-2 |  PCI |  | wi |
| 3Com |  3CRPAG175 AR5212 |  !CardBus |  a/b/g | ath |
| 3Com |  3CRWE154G72 ISL3880 |  !CardBus |  | pgt |
| 3Com |  Aolynk WUB320g |  |  | rum |
| 3Com |  !OfficeConnect 3CRSHPW796 |  !CardBus |  | atw |
| 3Com | 3CRSHEW696 | USB | | atu |
| 3COM | 3CRUSB10075 | USB | | zyd |
| A-Link |  WL54H |  PCI |  | ral |
| A-Link |  WL54PC |  !CardBus |  | ral |
| !AboCom |  WUG2700 |  |  | rum |
| !AboCom | BWU613 | USB | | atu |
| Accton | 2664W | USB | | atu |
| Acer |  Warplink USB-400 Prism-3 |  USB |  | wi |
| Acer | Peripherals AWL300 | USB | | atu |
| Acer | Peripherals AWL400 | USB | | atu |
| Acer | WLAN-G-US1 | USB | | zyd |
| Actiontec |  HWC01170 Prism-2/5 |  PCMCIA |  | wi |
| Actiontec |  HWU01170 Prism-3 |  USB |  | wi |
| Actiontec | 802UAT1 | USB | | atu |
| Adaptec |  AWN-8030 Prism-2/5 |  PCMCIA |  | wi |
| Addtron |  AWA-100 Prism-2 |  PCI |  | wi |
| Addtron |  AWP-100 Prism-2 |  PCMCIA |  | wi |
| Addtron | AWU120 | USB | | atu |
| Agere |  ORiNOCO Hermes |  PCMCIA |  | wi |
| Aincomm | AWU2000B | USB | | atu |
| Airlink | 101 AWLL3026 | USB | | zyd |
| Airlink+ | AWLL3025 | USB | | zyd |
| Airlink101 |  AWLL5025 |  |  | rum |
| Aironet |  Communications 4500 4800 |  ISA - PCI - PCMCIA |  | an |
| Alfa |  AWPC036 |  !CardBus |  | ral |
| Allnet |  ALL0182 SA2400 |  !CardBus |  | rtw |
| Ambit |  WLAN Prism-3 |  USB |  | wi |
| Amigo |  AWI-914W |  !CardBus |  | ral |
| Amigo |  AWI-922W |  Mini PCI |  | ral |
| Amigo |  AWI-926W |  PCI |  | ral |
| AMIT |  WL531C |  !CardBus |  | ral |
| AMIT |  WL531P |  PCI |  | ral |
| AMIT | WL532U | USB | | ural |
| AOpen |  AOI-831 |  PCI |  | ral |
| AOpen | 802.11g WL54 | USB | | zyd |
| Apacer |  Wireless Steno MB112 Prism-3 |  USB |  | wi |
| Apple |  Airport Extreme BCM4306 |  PCI |  b/g | bwi |
| Apple |  Airport Extreme BCM4318 |  PCI |  b/g | bwi |
| Apple |  Airport Hermes |  macobio |  | wi |
| ARtem |  Onair Hermes |  PCMCIA |  | wi |
| Askey Computer |  Voyager 1010 | USB | | atu |
| Askey Computer | WLL013 (Intersil Radio) | USB | | atu |
| Askey Computer | WLL013 (RFMD Radio) | USB | | atu |
| ASUS |  !SpaceLink WL-100 Prism-2/5 |  PCMCIA |  | wi |
| ASUS |  !SpaceLink WL-110 Prism-2/5 |  CF |  | wi |
| ASUS |  WIFI-G-AAY |  PCI |  | ral |
| ASUS |  WL-107G |  !CardBus |  | ral |
| ASUS |  WL-130G |  PCI |  | ral |
| ASUS |  WL-130N |  PCI |  | ral |
| ASUS |  WL-138g BCM4318 |  PCI |  b/g | bwi |
| ASUS |  WL-140 Prism-3 |  USB |  | wi |
| ASUS |  WL-167g ver 2 |  |  | rum |
| Asus | A9T integrated wirless | USB | | zyd |
| Asus | WL-159g | USB | | zyd |
| ASUS | WL-167g | USB | | ural |
| Atlantis Land |  A02-PCI-W54 |  PCI |  | ral |
| Atlantis Land |  A02-PCM-W54 |  !CardBus |  | ral |
| Atlantis Land |  A02-UP1-W54 |  |  | rum |
| Atmel | AT76C503 (Intersil Radio) | USB | | atu |
| Atmel | AT76C503 (RFMD Radio) | USB | | atu |
| Atmel | AT76C505 (RFMD 2958 Radio) | USB | | atu |
| Atmel | AT76C505 (RFMD Radio) | USB | | atu |
| Atmel | AT76C505A (RFMD 2958 Radio) | USB | | atu |
| Atmel | AT76C505AS (RFMD 2958 Radio) | USB | | atu |
| Belkin |  F5D6001 (version 1 only) Prism-2 |  PCI |  | wi |
| Belkin |  F5D6001 |  PCI (version 2 only) |  | atw |
| Belkin |  F5D6020 (version 1 only) Prism-2 |  PCMCIA |  | wi |
| Belkin |  F5D6020 V3 SA2400 |  !CardBus |  | rtw |
| Belkin |  F5D6060 (version 1 only) Prism-2/5 |  CF  |  | wi |
| Belkin |  F5D7000 v3 |  PCI |  | ral |
| Belkin |  F5D7010 v2 |  !CardBus |  | ral |
| Belkin |  F5D7050 ver 3 |  |  | rum |
| Belkin |  F5D9050 ver 3 |  |  | rum |
| Belkin |  F5D9050C |  |  | rum |
| Belkin | F5D6050 | USB | | atu |
| Belkin | F5D7050 (version 1000) | USB | | upgt |
| Belkin | F5D7050 v.4000 | USB | | zyd |
| Belkin | F5D7050 v2000 | USB | | ural |
| Billion | BiPAC 3011G | USB | | zyd |
| Billionton |  MIWLGRL |  Mini PCI |  | ral |
| Blitz |  !NetWave Point |  !CardBus |  | atw |
| Bluetake | BW002 | USB | | atu |
| Buffalo |  !AirStation Prism-2 CF |   |  | wi |
| Buffalo |  !AirStation Prism-2 |  PCMCIA |  | wi |
| Buffalo |  WLI-CB-B11 SA2400 |  !CardBus |  | rtw |
| Buffalo |  WLI-CB-G54 BCM4306 |  !CardBus |  b/g | bwi |
| Buffalo |  WLI-U2-G54HP |  |  | rum |
| Buffalo |  WLI-U2-SG54HP |  |  | rum |
| Buffalo | WLI-U2-KG54 | USB | | ural |
| Buffalo | WLI-U2-KG54-AI | USB | | ural |
| Buffalo | WLI-U2-KG54-YB | USB | | ural |
| Buffalo | WLI-U2-KG54L | USB | | zyd |
| Cabletron |  !RoamAbout Hermes |  PCMCIA |  | wi |
| Canyon |  CN-WF511 |  PCI |  | ral |
| Canyon |  CN-WF513 |  !CardBus |  | ral |
| CC&C |  WL-2102 |  !CardBus |  | ral |
| CC&C | WL-2203B | USB | | zyd |
| Cisco |  340 350  |  ISA - PCI - PCMCIA |  | an |
| Cisco |  AIR-CB21AG AR5212 |  !CardBus |  a/b/g | ath |
| CNet |  CWC-854 |  !CardBus |  | ral |
| CNet |  CWD-854 ver F |  |  | rum |
| CNet |  CWP-854 |  PCI |  | ral |
| CNet | CWD-854 | USB | | ural |
| Cohiba | Proto Board | USB | | upgt |
| Compaq |  Agency NC5004 Prism-2 |  PCMCIA |  | wi |
| Compaq |  R4035 onboard BCM4306 |  PCI |  b/g | bwi |
| Compaq |  W100 Prism-3 |  USB |  | wi |
| Compaq | iPAQ h54xx/h55xx Internal WLAN | USB | | atu |
| Compex |  WL54 |  !CardBus |  | ral |
| Compex |  WLP54G |  PCI |  | ral |
| Compex | WLU108AG AR5005UX | USB | | uath |
| Compex | WLU108G AR5005UG | USB | | uath |
| Compex | WLU54G 2A1100 | USB | | ural |
| Conceptronic |  C54RC |  !CardBus |  | ral |
| Conceptronic |  C54Ri |  PCI |  | ral |
| Conceptronic |  C54RU ver 2 |  |  | rum |
| Conceptronic | C11U | USB | | atu |
| Conceptronic | C54RU | USB | | ural |
| Conceptronic | WL210 | USB | | atu |
| Contec |  FLEXLAN/FX-DS110-PCC Prism-2 |  PCMCIA |  | wi |
| Corega |  CG-WLCB11V3 SA2400 |  !CardBus |  | rtw |
| Corega |  CG-WLCB54GL |  !CardBus |  | ral |
| Corega |  CG-WLPCI54GL |  PCI |  | ral |
| Corega |  CG-WLUSB2GL |  |  | rum |
| Corega |  CG-WLUSB2GO |  |  | rum |
| Corega |  CG-WLUSB2GPX |  |  | rum |
| Corega |  CGWLPCIA11 Prism-2 |  PCI |  | wi |
| Corega |  PCC-11 Prism-2 |  PCMCIA |  | wi |
| Corega |  PCCA-11 Prism-2 |  PCMCIA |  | wi |
| Corega |  PCCB-11 Prism-2 |  PCMCIA |  | wi |
| Corega |  WLUSB-11 Key Prism-3 |  USB |  | wi |
| Corega |  WLUSB-11 Prism-3 |  USB |  | wi |
| Corega | WLAN USB Stick 11 | USB | | atu |
| D-Link |  DCF-660W Prism-2 CF |   |  | wi |
| D-Link |  DWA-110 |  |  | rum |
| D-Link |  DWA-111 |  |  | rum |
| D-Link |  DWL-120 (rev F) Prism-3 |  USB |  | wi |
| D-Link |  DWL-122 Prism-3 |  USB |  | wi |
| D-Link |  DWL-520 (rev A and B only) Prism-2/5 |  PCI |  | wi |
| D-Link |  DWL-520 Rev C1 |  PCI |  | atw |
| D-Link |  DWL-520+ ACX100 |  PCI |  b | acx |
| D-Link |  DWL-610 ? |  !CardBus |  | rtw |
| D-Link |  DWL-650 (rev A1-J3 only) Prism-2/5 |  PCMCIA |  | wi |
| D-Link |  DWL-650 Rev L1 |  !CardBus |  | atw |
| D-Link |  DWL-650+ ACX100 |  !CardBus |  b | acx |
| D-Link |  DWL-A520 AR5210 |  PCI |  a | ath |
| D-Link |  DWL-A650 AR5210 |  !CardBus |  a | ath |
| D-Link |  DWL-AB650 AR5211 |  !CardBus |  a/b | ath |
| D-Link |  DWL-G122 rev C1 |  |  | rum |
| D-Link |  DWL-G520+ ACX111 |  PCI |  b/g | acx |
| D-Link |  DWL-G630+ ACX111 |  !CardBus |  b/g | acx |
| D-Link |  DWL-g650 A1 ISL3890 |  PCI |  | pgt |
| D-Link |  DWL-G650+ ACX111 |  !CardBus |  b/g | acx |
| D-Link |  WUA-1340 |  |  | rum |
| D-Link | DWL-120 rev E | USB | | atu |
| D-Link | DWL-G120 Cohiba | USB | | upgt |
| D-Link | DWL-G122 (b1) | USB | | ural |
| D-Link | DWL-G132 AR5005UG | USB | | uath |
| Dick Smith Electronics |  CHUSB 611G | USB | | atu |
| Dick Smith Electronics | WL200U | USB | | atu |
| Dick Smith Electronics | WL240U | USB | | atu |
| Dick Smith Electronics | XH1153 | USB | | atu |
| Digiconnect |  WL591C |  !CardBus |  | ral |
| Digitus |  DN-7001G-RA |  !CardBus |  | ral |
| Digitus |  DN-7003GR |  |  | rum |
| Digitus |  DN-7006G-RA |  PCI |  | ral |
| !DrayTek | Vigor 550 | USB | | zyd |
| Dynalink |  WLG25CARDBUS |  !CardBus |  | ral |
| Dynalink |  WLG25PCI |  PCI |  | ral |
| Dynalink | WLG25USB | USB | | ural |
| E-Tech |  WGPC02 |  !CardBus |  | ral |
| E-Tech |  WGPC03 |  !CardBus |  | ral |
| E-Tech |  WGPI02 |  PCI |  | ral |
| E-Tech | WGUS02 | USB | | ural |
| Edimax |  EW-7106 SA2400 |  !CardBus |  | rtw |
| Edimax |  EW-7108PCg |  !CardBus |  | ral |
| Edimax |  EW-7128g |  PCI |  | ral |
| Edimax |  EW-7318USG |  |  | rum |
| Edimax |  EW-7628Ig |  PCI |  | ral |
| Edimax |  EW-7708PN |  !CardBus |  | ral |
| Edimax |  EW-7728In |  PCI |  | ral |
| Edimax | EW-7317LDG | USB | | zyd |
| Edimax | EW-7317UG | USB | | zyd |
| Elecom |  LD-WL54 AR5211 |  !CardBus |  a | ath |
| ELSA |  XI300 Prism-2 |  PCMCIA |  | wi |
| ELSA |  XI325 Prism-2/5 |  PCMCIA |  | wi |
| ELSA |  XI325H Prism-2/5 |  PCMCIA |  | wi |
| ELSA |  XI800 Prism-2 |  CF |  | wi |
| Eminent |  EM3036 |  !CardBus |  | ral |
| Eminent |  EM3037 |  PCI |  | ral |
| Eminent | EM3035 | USB | | ural |
| EMTAC |  A2424i Prism-2 |  PCMCIA |  | wi |
| Encore |  ENLWI-G-RLAM |  PCI |  | ral |
| Encore |  ENPWI-G-RLAM |  !CardBus |  | ral |
| Ergenic |  ERG WL-003 ACX100 |  !CardBus |  b | acx |
| Ericsson |  Wireless LAN CARD C11 Spectrum24 |  PCMCIA |  | wi |
| Eusso |  UGL2454-01R |  !CardBus |  | ral |
| Eusso |  UGL2454-VPR |  PCI |  | ral |
| Fiberline |  WL-400P |  PCI |  | ral |
| Fiberline |  WL-400X |  !CardBus |  | ral |
| Fiberline | Networks WL-43OU | USB | | zyd |
| Foxconn |  WLL-3350 |  PCI |  | ral |
| FSC | !Connect2Air E-5400 USB D1700 | USB | | upgt |
| Gemtek |  WL-311 Prism-2/5 |  PCMCIA |  | wi |
| Geowave | GW-US11S | USB | | atu |
| Gigabyte |  GN-WB01GS |  |  | rum |
| Gigabyte |  GN-WI01GS |  Mini PCI |  | ral |
| Gigabyte |  GN-WI02GM |  Mini PCI |  | ral |
| Gigabyte |  GN-WI02GM |  PCI |  | ral |
| Gigabyte |  GN-WI05GS |  |  | rum |
| Gigabyte |  GN-WIKG |  Mini PCI |  | ral |
| Gigabyte |  GN-WM01GM |  !CardBus |  | ral |
| Gigabyte |  GN-WM01GS |  !CardBus |  | ral |
| Gigabyte |  GN-WMKG |  !CardBus |  | ral |
| Gigabyte |  GN-WP01GM |  PCI |  | ral |
| Gigabyte |  GN-WP01GS |  PCI |  | ral |
| Gigabyte |  GN-WPKG |  PCI |  | ral |
| Gigabyte | GN-WBKG | USB | | ural |
| Gigabyte | GN-WLBM101 | USB | | atu |
| Gigaset | USB Adapter 54 | USB | | upgt |
| Gigaset | WLAN | USB | | atu |
| Hamlet |  HNWP254 ACX111 |  !CardBus |  b/g | acx |
| Hawking |  HWC54GR |  !CardBus |  | ral |
| Hawking |  HWP54G ACX111 |  PCI |  b/g | acx |
| Hawking |  HWP54GR |  PCI |  | ral |
| Hawking |  HWU54DM |  |  | rum |
| Hawking |  HWUG1 |  |  | rum |
| Hawking |  Technology WE110P Prism-2/5 |  PCMCIA |  | wi |
| Hercules |  HWGPCI-54 |  PCI |  | ral |
| Hercules |  HWGPCMCIA-54 |  !CardBus |  | ral |
| Hercules |  HWGUSB2-54-LB |  |  | rum |
| Hercules |  HWGUSB2-54V2-AP |  |  | rum |
| Hercules | HWGUSB2-54 | USB | | ural |
| Hewlett-Packard | HN210W | USB | | atu |
| Hewlett-Packard |  nx6125 BCM4319 |  PCI |  b/g | bwi |
| I-O |  DATA WN-B11/PCM Prism-2 |  PCMCIA |  | wi |
| I-O |  DATA WN-B11/USB Prism-3 |  USB |  | wi |
| I-O |  Data WN-G54/CB ISL3890 |  PCI |  | pgt |
| I-O |  DATA WN-G54/CF 88W8385 |  PCMCIA |  b/g | malo |
| I-O | DATA USB WN-B11 | USB | | atu |
| I4 |  Z-Com XG-600 ISL3890 |  PCI |  | pgt |
| I4 |  Z-Com XG-900 ISL3890 |  PCI |  | pgt |
| IBM |  11ABG WL LAN AR5212 |  Mini PCI |  a/b/g | ath |
| iNexQ |  CR054g-009 (R03) |  PCI |  | ral |
| iNexQ | UR055g | USB | | zyd |
| Intel |  2225BG |  PCI |  | iwi |
| Intel |  PRO/Wireless 2011 Spectrum24 |  PCMCIA |  | wi |
| Intel |  PRO/Wireless 2011B Prism-3 |  USB |  | wi |
| Intel |  Wireless !WiFi Link 4965AGN !WiFi Link 5000 Series. !WiFi Link 5100/5300 |  Mini PCI Express |  | iwn |
| Intel | AP310 !AnyPoint II | USB | | atu |
| Intel | PRO/Wireless 2100 | Mini PCI | | ipw |
| Intel | PRO/Wireless 2200BG/2915ABG |  Mini PCI |  | iwi |
| Intel | PRO/Wireless 3945ABG |  Mini PCI |  | wpi |
| Intersil |  ISL3872 Prism-3 |  PCI |  | wi |
| Intersil |  Prism 2X Prism-3 |  USB |  | wi |
| Intersil |  PRISM Duette ISL3890 |  PCI |  | pgt |
| Intersil |  Prism II Prism-2 |  PCMCIA |  | wi |
| Intersil |  PRISM Indigo ISL3877 |  PCI |  | pgt |
| Intersil |  Prism-2-5 |  Mini PCI |  | wi |
| Inventel | UR045G | USB | | upgt |
| IODATA | WN-G54/US AR5005UG | USB | | uath |
| JAHT |  WN-4054P(E) |  !CardBus |  | ral |
| JAHT |  WN-4054PCI |  PCI |  | ral |
| Jensen |  !AirLink 6011 GRF5101 |  !CardBus |  | rtw |
| JVC |  MP-XP7250 Prism-3 |  USB |  | wi |
| KCORP !LifeStyle | KLS-611 |  !CardBus |  | ral |
| KCORP !LifeStyle | KLS-660 |  PCI |  | ral |
| KCORP !LifeStyle | KLS-685 | USB | | ural |
| Lexar | 2662W-AR | USB | | atu |
| Linksys |  Instant Wireless WPC11 2/5 Prism-2/5 |  PCMCIA |  | wi |
| Linksys |  Instant Wireless WPC11 3-0 Prism-3 |  PCMCIA |  | wi |
| Linksys |  Instant Wireless WPC11 Prism-2 |  PCMCIA |  | wi |
| Linksys |  WCF12 Prism-3 |  CF |  | wi |
| Linksys |  WMP54G v4 |  PCI |  | ral |
| Linksys |  WPC11 v4 MAX2820 |  !CardBus |  | rtw |
| Linksys |  WPC51AB AR5211 |  !CardBus |  a/b | ath |
| Linksys |  WPC54G Ver 3 BCM4318 |  !CardBus |  b/g | bwi |
| Linksys |  WPC54GS Ver 2 BCM4318 |  !CardBus |  b/g | bwi |
| Linksys |  WPC54Gv2 ACX111 |  !CardBus |  b/g | acx |
| Linksys |  WUSB11 v3-0 Prism-3 |  USB |  | wi |
| Linksys |  WUSB12 Prism-3 |  USB |  | wi |
| Linksys |  WUSB54G rev C |  |  | rum |
| Linksys |  WUSB54GR |  |  | rum |
| Linksys | HU200-TS | USB | | ural |
| Linksys | WUSB11 802.11b v2.8 | USB | | atu |
| Linksys | WUSB11 802.11b | USB | | atu |
| Linksys | WUSB54G v4 | USB | | ural |
| Linksys | WUSB54GP v4 | USB | | ural |
| Linksys | WUSBF54G | USB | | zyd |
| Longshine |  8301 Prism-2 |  PCI |  | wi |
| Longshine |  LCS-8031N |  PCI |  | ral |
| Longshine | LCS-8131G3 | USB | | zyd |
| Lucent |  WaveLAN Hermes |  PCMCIA |  | wi |
| Melco |  WLI-USB-KB11 Prism-3 |  USB |  | wi |
| Melco |  WLI-USB-KS11G Prism-3 |  USB |  | wi |
| Melco |  WLI-USB-S11 Prism-3 |  USB |  | wi |
| MELCO | WLI-U2-KAMG54 AR5005UX | USB | | uath |
| Microcom |  Travelcard ACX111 |  !CardBus |  b/g | acx |
| Micronet |  SP906GK |  PCI |  | ral |
| Micronet |  SP908GK V3 |  !CardBus |  | ral |
| Microsoft |  MN510 Prism-3 |  USB |  | wi |
| Microsoft |  MN520 Prism-2/5 |  PCMCIA |  | wi |
| Minitar |  MN54GCB-R |  !CardBus |  | ral |
| Minitar |  MN54GPC-R |  PCI |  | ral |
| MSI |  CB54G2 |  !CardBus |  | ral |
| MSI |  MP54G2 |  Mini PCI |  | ral |
| MSI |  MS-6833 |  Mini PCI |  | ral |
| MSI |  MS-6834 |  PCI |  | ral |
| MSI |  MS-6835 |  !CardBus |  | ral |
| MSI |  PC54G2 |  PCI |  | ral |
| MSI | MS-6861 | USB | | ural |
| MSI | MS-6865 | USB | | ural |
| MSI | MS-6869 | USB | | ural |
| MSI | US54SE | USB | | zyd |
| MSI | WLAN | USB | | atu |
| NANOSPEED |  ROOT-RZ2000 Prism-2 |  PCMCIA |  | wi |
| NDC/Sohoware |  NCP130 Prism-2 |  PCI |  | wi |
| NEC |  CMZ-RT-WP Prism-2 |  PCMCIA |  | wi |
| Netgear |  MA111 (version 1 only) Prism-3 |  USB |  | wi |
| Netgear |  MA311 Prism-2/5 |  PCI |  | wi |
| Netgear |  MA401 Prism-2 |  PCMCIA |  | wi |
| Netgear |  MA401RA Prism-2/5 |  PCMCIA |  | wi |
| Netgear |  MA521 SA2400 |  !CardBus |  | rtw |
| Netgear |  MA701 Prism-2/5 |  CF |  | wi |
| Netgear |  WAB501 AR5211 |  !CardBus |  a/b | ath |
| Netgear |  WG311v2 ACX111 |  PCI |  b/g | acx |
| Netgear |  WG311v3 88W8335 |  PCI |  b/g | malo |
| NETGEAR |  WG511 (Taiwanese not Chinese) ISL3890 |  !CardBus |  | pgt |
| Netgear |  WG511v2 88W8310 |  !CardBus |  b/g | malo |
| Netgear | MA101 rev B | USB | | atu |
| Netgear | MA101 | USB | | atu |
| Netgear | WG111T AR5005UG | USB | | uath |
| Netgear | WG111U AR5005UX | USB | | uath |
| Netgear | WG111v2 | USB | | upgt |
| Netgear | WPN111 AR5005UG | USB | | uath |
| Nintendo | Wi-Fi USB Connector | USB | | ural |
| Nokia |  C020 Wireless LAN Prism-I |  PCMCIA |  | wi |
| Nokia |  C110/C111 Wireless LAN Prism-2 |  PCMCIA |  | wi |
| Nortel |  E-mobility 211818-A Spectrum24 |  PCI |  | wi |
| Nova | Tech NV-902W | USB | | ural |
| NTT-ME |  11Mbps Wireless LAN Prism-2 |  PCMCIA |  | wi |
| Olitec | 000544 AR5005UG | USB | | uath |
| OQO | model 01 !WiFi | USB | | atu |
| !OvisLink | !AirLive WL-1120USB | USB | | atu |
| !OvisLink | !AirLive WL-1130USB | USB | | atu |
| !OvisLink | Evo-W54USB | USB | | ural |
| Philips | SNU5600 | USB | | zyd |
| Philips | SNU6500 AR5005UG | USB | | uath |
| Planet |  WL-3553 SA2400 |  !CardBus |  | rtw |
| Planet |  WL-3560 AR5211 |  !CardBus |  a/b/g | ath |
| PLANET | WDL-U357 AR5005UX | USB | | uath |
| Planet | WL-U356 | USB | | zyd |
| PLANEX |  GW-DS54G ISL3890 |  PCI |  | pgt |
| Planex |  GW-NS11H Prism-3 |  PCMCIA |  | wi |
| Planex |  GW-US11H Prism-3 |  USB |  | wi |
| Planex |  GW-US54HP |  |  | rum |
| Planex |  GW-US54Mini2 |  |  | rum |
| Planex |  GW-USMM |  |  | rum |
| Planex |  PCI-GW-DS300N |  PCI |  | ral |
| Planex | Communications GW-US11S | USB | | atu |
| Planex | GW-US54GD | USB | | zyd |
| Planex | GW-US54GXS | USB | | zyd |
| Planex | GW-US54GZL | USB | | zyd |
| Planex | GW-US54Mini | USB | | zyd |
| Pretec |  Compact WLAN OC-WLBXX-A Prism-2/5 |  CF |  | wi |
| Pro-Nets |  CB80211G |  !CardBus |  | ral |
| Pro-Nets |  PC80211G |  PCI |  | ral |
| Proxim |  Harmony Prism-2 |  PCMCIA |  | wi |
| Proxim |  RangeLAN-DS Prism-2 |  PCMCIA |  | wi |
| Proxim |  Skyline 4030 AR5210 |  !CardBus |  a | ath |
| Proxim |  Skyline 4032 AR5210 |  PCI |  a | ath |
| Q-Tec |  770WC SA2400 |  !CardBus |  | rtw |
| Q-Tec |  775WC SA2400 |  !CardBus |  | rtw |
| Ralink |  IEEE 802.11a/b/g/Draft-N |  USB |  | run |
| Raytheon |  Raylink Aviator 2/4/PRO |   |  | ray |
| Repotec |  RP-WB7108 |  !CardBus |  | ral |
| Repotec |  RP-WP0854 |  PCI |  | ral |
| Roper |  !FreeLan 802-11b SA2400 |  !CardBus |  | rtw |
| SAFECOM |  SWLCR-1100 SA2400 |  !CardBus |  | rtw |
| Safecom | SWMULZ-5400 | USB | | zyd |
| Sagem | XG 760A | USB | | zyd |
| Sagem | XG 76NA | USB | | zyd |
| Sagem | XG703A | USB | | upgt |
| Samsung |  MagicLAN SWL-2000N Prism-2 |  PCMCIA |  | wi |
| Samsung |  MagicLAN SWL-2210P Prism-2 |  PCI |  | wi |
| Samsung | SWL2100W | USB | | atu |
| Sandberg | Wireless G54 USB | USB | | zyd |
| SATech |  SN-54C |  !CardBus |  | ral |
| SATech |  SN-54P |  PCI |  | ral |
| Sceptre |  SC254W+ ACX111 |  !CardBus |  b/g | acx |
| Senao |  NL-2511CF Prism-3 |  CF |  | wi |
| Senao |  NL-2511MP Prism-2/5 |  PCI |  | wi |
| Senao |  NL-5354MP AR5212 |  Mini PCI |  a/b/g | ath |
| Senao |  NUB-3701 |  |  | rum |
| !SerComm | UB801R | USB | | ural |
| Siemens |  !SpeedStream SS1021 Prism-2 |  PCMCIA |  | wi |
| Siemens |  !SpeedStream SS1022 Prism-3 |  USB |  | wi |
| Siemens | Gigaset 108 AR5005UG | USB | | uath |
| Siemens | WLL013 | USB | | atu |
| Signamax |  065-1798 |  PCI |  | ral |
| Sitecom |  WL-022 Prism-3 |  USB |  | wi |
| Sitecom |  WL-112 |  !CardBus |  | ral |
| Sitecom |  WL-113 ver 2 |  |  | rum |
| Sitecom |  WL-115 |  PCI |  | ral |
| Sitecom |  WL-172 |  |  | rum |
| Sitecom | WL-113 | USB | | zyd |
| SMC |  2632 EZ Connect Prism-2 |  PCMCIA |  | wi |
| SMC |  2635W |  !CardBus (version 1 only) |  | atw |
| SMC |  2802Wv2 ISL3890 |  PCI |  | pgt |
| SMC |  EZ Connect g 2-4GHz SMC2802W ISL3890 |  PCI |  | pgt |
| SMC |  EZ Connect g 2-4GHz SMC2835W-v2 ISL3890 |  !CardBus |  | pgt |
| SMC |  SMC2735W AR5210 |  !CardBus |  a | ath |
| SMC | EZ Connect 11Mbps (SMC2662w) | USB | | atu |
| SMC | EZ Connect 11Mbps v2 (SMC2662wV2) | USB | | atu |
| SMC | EZ ConnectG SMC2862W-G | USB | | upgt |
| SMC | SMCWUSB-G | USB | | zyd |
| SMC | SMCWUSBT-G AR5005UG | USB | | uath |
| SMC | SMCWUSBT-G2 AR5005UG | USB | | uath |
| Sony |  PCWA-C500 AR5210 |  !CardBus |  a | ath |
| Soyo |  Aerielink ISL3890 |  !CardBus |  | pgt |
| SparkLAN |  WCFM-100 88W8385 |  PCMCIA |  b/g | malo |
| SparkLAN |  WL-611R |  !CardBus |  | ral |
| SparkLAN |  WL-660R |  PCI |  | ral |
| SparkLAN |  WMIR-215GN |  Mini PCI |  | ral |
| SparkLAN | WL-685R | USB | | ural |
| SparkLAN | WL-785A AR5005UX | USB | | uath |
| Sphairon | UB801R | USB | | ural |
| Spinnaker | DUT | USB | | upgt |
| Spinnaker | Proto Board | USB | | upgt |
| Surecom |  EP-9321-g |  PCI |  | ral |
| Surecom |  EP-9321-g1 |  PCI |  | ral |
| Surecom |  EP-9428-g |  !CardBus |  | ral |
| Surecom | EP-9001-g rev 3A | USB | | ural |
| Sweex |  LC500050 |  !CardBus |  | ral |
| Sweex |  LC700030 |  PCI |  | ral |
| Sweex |  LW053 |  |  | rum |
| Sweex | LC100060 | USB | | ural |
| Sweex | wireless USB 54 Mbps | USB | | zyd |
| Symbol |  LA4123 Spectrum24 |  PCI |  | wi |
| Symbol |  Spectrum24 Spectrum24 |  PCMCIA |  | wi |
| Syntax |  USB-400 Prism-3 |  USB |  | wi |
| TDK |  LAK-CD011WL Prism-2 |  PCMCIA |  | wi |
| Tekram/Siemens | USB adapter | USB | | zyd |
| Telegent | TG54USB | USB | | zyd |
| Tenda |  TWL541C 88W8310 |  !CardBus |  b/g | malo |
| Tenda |  TWL542P 88W8335 |  PCI |  b/g | malo |
| Tonze |  PC-6200C |  PCI |  | ral |
| Tonze |  PC-620C |  Mini PCI |  | ral |
| Tonze |  PW-6200C |  !CardBus |  | ral |
| Tonze | UW-6200C | USB | | ural |
| Tornado/ADT |  211g ACX111 |  PCI |  b/g | acx |
| TP-LINK |  TL-WN321G |  |  | rum |
| TP-Link | TL-WN620G AR5005UG | USB | | uath |
| TRENDnet |  TEW-221PC |  !CardBus |  | atw |
| TRENDnet |  TEW-226PC ? |  !CardBus |  | rtw |
| TRENDnet |  TEW-401PCplus BCM4306 |  !CardBus |  b/g | bwi |
| Trendnet | TEW-424UB | USB | | zyd |
| Trendnet | TEW-429UB | USB | | zyd |
| TRENDware | International TEW-444UB AR5005UG | USB | | uath |
| TRENDware | International TEW-504UB AR5005UX | USB | | uath |
| TwinMOS | G240 | USB | | zyd |
| Unex |  CR054g-R02 |  PCI |  | ral |
| Unex |  MR054g-R02 |  !CardBus |  | ral |
| Unex | Technology UR054ag AR5005UX | USB | | uath |
| US Robotics | 1120 Prism-3 |  USB |  | wi |
| US Robotics | 2410 Prism-2 |  PCMCIA |  | wi |
| US Robotics | 2445 Prism-2 |  PCMCIA |  | wi |
| US Robotics | 5411 BCM4318 |  !CardBus |  b/g | bwi |
| US Robotics | 5423 | USB | | zyd |
| USR |  USR5410 ACX111 |  !CardBus |  b/g | acx |
| USR |  USR5416 ACX111 |  PCI |  b/g | acx |
| VCTnet |  PC-11B1 SA2400 |  !CardBus |  | rtw |
| Winstron |  CB-200B SA2400 |  !CardBus |  | rtw |
| Wistron |  CM9 AR5212 |  Mini PCI |  a/b/g | ath |
| X-Micro | XWL-11GUZX | USB | | zyd |
| Xircom |  !CreditCard Netwave Netwave Airsurfer |  PCMCIA |  | cnw |
| Xterasys |  XN2511B |  PCI |  | atw |
| Yakumo | QuickWLAN USB | USB | | zyd |
| Z-Com |  XI-725/726 Prism-2/5 |  USB |  | wi |
| Z-Com |  XI-735 Prism-3 |  USB |  | wi |
| Zaapa | ZNWUSB-54 | USB | | ural |
| Zinwell |  ZWX-G160 |  !CardBus |  | ral |
| Zinwell |  ZWX-G360 |  Mini PCI |  | ral |
| Zinwell |  ZWX-G361 |  PCI |  | ral |
| Zinwell | ZPlus-G250 | USB | | ural |
| Zinwell | ZWX-G261 | USB | | ural |
| Zonet |  ZEW1000 GRF5101 |  !CardBus |  | rtw |
| Zonet |  ZEW1500 |  !CardBus |  | ral |
| Zonet |  ZEW1600 |  PCI |  | ral |
| Zonet | ZEW2500P | USB | | ural |
| Zonet | ZEW2501 | USB | | zyd |
| ZyXEL |  G-160 ACX111 |  !CardBus |  b/g | acx |
| ZyXEL |  G-360 EE ACX111 |  PCI |  b/g | acx |
| ZyXEL |  ZyAIR B-200 Prism-3 |  USB |  | wi |
| ZyXEL |  ZyAIR G-100 ISL3890 |  !CardBus |  | pgt |
| ZyXEL | G-202 | USB | | zyd |
| ZyXEL | XtremeMIMO M-202 AR5005UX | USB | | uath |
| ZyXEL | ZyAIR G-220 | USB | | zyd |

Al autor le ha sido posible conseguir y probar en Bogotá:

* PCI:
** D-Link DWL-G520. Que es de 108Mb
* USB:
** D-Link DWA-110


----
Esta información se dedica a Jesucristo por su Sacrificio y se libera al dominio público. vtamara@pasosdeJesus.org 2008.


