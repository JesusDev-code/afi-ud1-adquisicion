# Notas de trabajo — AFI UD1 · Adquirir un equipo comprometido

**Hora de referencia:** la del equipo del analista (anfitrión), en CET (UTC+1). Fechas siempre en formato `AAAA-MM-DD`.
La hora de cada captura es la de **creación del fichero** en el anfitrión (metadatos), al segundo.

---

## Datos del caso

| Campo | Valor |
|---|---|
| Número de caso | `AFI-UD1-ALMADRABA` |
| Etiqueta del equipo (en todas las herramientas) | `FORENSIC_10` |
| Hostname real del equipo | `FORENSE-06` |
| Analista | Jesús Alfonso Pedreño Domínguez |
| Testigo | No hubo |
| Solicitante | Bahía Sistemas (para Almadraba Diseño) |
| Fichero de configuración de la VM | `C:\Users\er_pi\Documents\Virtual Machines\FORENSIC_10\FORENSIC_10.vmx` (hardware Workstation 11.x; 1 GB RAM, 1 CPU, disco SCSI 32 GB) |
| USB de respuesta | Pendrive PHILIPS 32 GB, exFAT, etiqueta `USB-RESP` |
| Fecha y hora de inicio | 2026-10-31 10:49 (CET, anfitrión) |
| Fin de la parte 1 | 2026-10-31 11:52 (hashes calculados en el anfitrión; USB conectado al anfitrión como `E:`) |
| Fin de la parte 2, disco | 2026-10-31 14:48 |
| Pendrive, copias y fichero de hashes | 2026-11-03, de 09:00 a 16:34 |

## Recepción de la máquina

| Campo | Valor |
|---|---|
| Fichero recibido | `FORENSIC_10.ova` |
| SHA-256 | `DAF0EF5255D98276A6912A53611DB5C0CBF2CCCBB49A180DD7FCC0F95E14930C` |
| Verificado antes de reimportar | 2026-10-30 ~20:40, sin cambios |
| Importada en (ruta) | `C:\Users\er_pi\Documents\Virtual Machines\FORENSIC_10\` |

---

## Preparación del laboratorio (antes de encender)

| Hora | Cambio | Motivo |
|---|---|---|
| 2026-10-30 ~22:30 | Red: de *bridged* (venía así en la OVA) a *host-only* | Aislamiento: el equipo está comprometido |
| 2026-10-30 ~22:30 | Controlador USB en compatibilidad USB 2.0 | Windows 7 no trae drivers de USB 3.0 |
| 2026-10-31 10:51:35 | Captura de la configuración de la VM, sin encender (`capturas/00-config-vm-hostonly-usb2.png`) | Evidencia del aislamiento antes de tocar el equipo |

*00-config-vm-hostonly-usb2.png*

![Captura 00-config-vm-hostonly-usb2.png](capturas/00-config-vm-hostonly-usb2.png)

---

## Herramientas de la parte 1 (validadas en este mismo Windows 7, ver el anexo)

| Paso | Dónde se ejecuta | Herramienta |
|---|---|---|
| 2 · RAM | VM | `D:\windows\win7\memoria\volcar-memoria.bat` (WinPmem mini) |
| 3 · Volátiles | VM | `D:\windows\win7\respuesta-en-vivo.bat` (adaptación a PowerShell 2.0) |
| 4 · Red | **Anfitrión**, PowerShell como administrador | `cd "C:\Program Files\VMware\VMware Workstation"` y `.\vnetsniffer.exe /w <ruta>\red.pcap VMnet1`; Ctrl+C para parar |
| 5 · Triaje | VM | `D:\windows\win7\triaje\triaje-velociraptor.bat` (Velociraptor legacy) |
| 6 · Cifrado | VM | `D:\windows\win7\cifrado\detectar-cifrado.bat` (EDD) |

Todos los `.bat`: **clic derecho → Run as administrator**, e **Intro, Intro** para aceptar caso y equipo.

Por qué cada herramienta (detalle y pruebas en el anexo):

| Adquisición | Herramienta | Motivo |
|---|---|---|
| RAM | WinPmem mini | El driver de DumpIt 3.6 no carga en este Windows 7 (`0xC0000428`); el de WinPmem sí |
| Datos volátiles | `respuesta-en-vivo-ps2.ps1` | El script original no arranca en PowerShell 2.0; la adaptación mantiene los 19 pasos |
| Red | `vnetsniffer` | Es el equivalente en VMware de `nictrace`: captura desde fuera, sin instalar nada en el equipo |
| Triaje | Velociraptor 0.77.3 *legacy* | KAPE necesita .NET 4.5.2 y el equipo tiene .NET 4.0 |
| Cifrado | EDD 3.1 | Funciona en el equipo, es de solo lectura y deja salida con hash |

---

## Parte 1 — Adquisición en vivo

| Hora | Paso | Acción (herramienta y versión) | Resultado / fichero generado | SHA-256 | Captura n.º |
|---|---|---|---|---|---|
| 11:06:12 (captura) | 0 · Marca (Bloc de notas, sin guardar) | Notepad de Windows 7, texto `AFI-JesusPedreno-20261031`, ventana *Untitled* abierta y sin guardar | Marca solo en RAM. Reloj de la VM en la captura: 11:05 AM | — | `04-marca-bloc-de-notas.png` |
| 11:10:27 (captura) | 0 · Marca (`marca-jesuspedreno.txt` en el escritorio) | Se copia el texto del primer Notepad con Ctrl+C (la marca queda también en el portapapeles, en RAM) y se pega en un segundo Notepad; *Save As* → Desktop → `marca-jesuspedreno`, y se cierra ese segundo Notepad. El primero sigue abierto y *Untitled* | Fichero `marca-jesuspedreno` visible en el escritorio (extensiones ocultas en Win7: se confirmará `.txt` en el triaje). Reloj de la VM: 11:10 AM | se calcula en el triaje | `05-marca-fichero.png` |
| 11:12:51 (captura) | 1 · Estado inicial | Reloj de la barra de tareas → *Change date and time settings…* (solo lectura, se cierra con *Cancel*). En pantalla: el escritorio, el Notepad de la marca abierto, `crea_user` ×2 y `marca-jesuspedreno` | Hora del sistema 11:12:29, zona UTC+01:00 (Madrid), sábado 2026-10-31. Desfase: ver abajo | — | `06-estado-inicial.png` |
| 11:19:00 → 11:22:00 (real; en el log del equipo 11:18:38.43 → 11:21:38.53) | 2 · RAM | `D:\windows\win7\memoria\volcar-memoria.bat` (Run as administrator, Intro ×2) → WinPmem mini `winpmem_mini_x64_rc2.exe`. Se navega hasta el `.bat` con el Explorador, solo dentro de `D:` | `D:\casos\AFI-UD1-ALMADRABA\FORENSIC_10\memoria\FORENSIC_10-ram.raw`, **1 073 741 824 bytes** (1 GiB, igual a la RAM de la VM), código de salida **1 = correcto**, ~3 min. Log: `adquisicion-winpmem.log` y `winpmem-salida.txt` en la misma carpeta. **La marca aparece en el volcado** en ASCII y en UTF-16 (comprobado en el anfitrión con `rg`) | `1F440CE6547B7B88640852B588A50CB55212080674591D65199149E2D05E2B2A` (anfitrión, `Get-FileHash`, 11:5x) | `08-ram-final.png` (11:22:30) |
| 11:24:40 → 11:24:51 (real; en el script 10:24:18Z → 10:24:29Z, en UTC) | 3 · Datos volátiles | `D:\windows\win7\respuesta-en-vivo.bat` (Run as administrator, Intro ×2) → `respuesta-en-vivo-ps2.ps1` (adaptación a PowerShell 2.0 del script del módulo) | `D:\casos\AFI-UD1-ALMADRABA\FORENSIC_10\vivo\AFI-UD1-ALMADRABA_FORENSIC_10_20261031-102418\`, 19 secciones (procesos, servicios, tareas programadas, inicio automático, usuarios, administradores, discos, USB conectados, portapapeles…), terminado en 11 s. **`19-portapapeles.txt` sale vacío** (solo la marca BOM), aunque la marca se había copiado con Ctrl+C: causa no investigada todavía | El script guarda el SHA-256 de cada fichero en `hashes-sha256.csv`. `00-registro.txt`: `F6650BC89FA010DF3498F9B248F61AEB0A09DBC04AECF3DE45FADC7D2484EE07` (anfitrión) | `09-vivo-final.png` (11:25:25) |
| 11:31:20 (inicio del proceso, PID 7868) | 4 · Red (inicio de captura) | En el **anfitrión**, PowerShell como administrador: `.\vnetsniffer.exe /w <…>\afi-ud1-adquisicion\red\FORENSIC_10-red.pcap VMnet1` (VMware Workstation). Escucha el switch *host-only* desde fuera, sin instalar nada en el equipo, como un TAP. Se deja corriendo durante el triaje y el cifrado | `red\FORENSIC_10-red.pcap` creado a las 11:31:20 (0 bytes al principio: escribe en bloques). Primer intento a las ~11:28: `vnetsniffer` volvió al prompt sin mostrar ningún mensaje y sin capturar, porque no existía la carpeta `red\`. Se crea con `mkdir` (11:30) y se relanza | — | `10-red-inicio.png` (11:32:36): se ven el intento fallido, el `mkdir` y el comando que sí captura |
| ~11:49 (Ctrl+C, después del apagado) | 4 · Red (fin de captura) | Ctrl+C en la ventana de `vnetsniffer` del anfitrión. Se para **después** del *Power Off* para cubrir hasta el final | `red\FORENSIC_10-red.pcap`, **5 104 bytes**, libpcap (cabecera `d4c3b2a1`), **34 paquetes** entre 11:35:17 y 11:44:06. Del equipo (`10.28.0.25`, IP fija, fuera de la subred de VMnet1): difusiones NetBIOS (UDP 137/138 → `10.28.0.255`), LLMNR (UDP 5355 → `224.0.0.252` / `ff02::1:3`) y DHCPv6 (546 → 547). El resto (`192.168.16.1` ↔ `.254`, DHCP y ARP) es del anfitrión y del DHCP de VMware. Ninguna conexión hacia fuera | `0AB1FDA310AB5E8049880106F32DEC32C5B485E62F87C0ACA32131F06127676C` | `15-red-fin.png` (11:49:03) |
| 11:34:27 → 11:34:52 (real; en el log del equipo 11:34:05.43 → 11:34:30.09) | 5 · Triaje | `D:\windows\win7\triaje\triaje-velociraptor.bat` (Run as administrator, Intro ×2) → Velociraptor `v0.77.3-windows-amd64-legacy`, artefacto `Windows.Search.FileFinder`, accessor `ntfs` (lectura en bruto), sin instalar nada. Recoge `$MFT`, SAM, SYSTEM, SOFTWARE, SECURITY, DEFAULT, `Users\*\NTUSER.DAT` y `winevt\Logs\*.evtx` | `D:\casos\AFI-UD1-ALMADRABA\FORENSIC_10\triaje\FORENSIC_10-triaje-velociraptor.zip`, **17 931 246 bytes** (~17 MB), código de salida **0 = correcto**, 25 s. Log: `adquisicion-velociraptor.log` y `velociraptor-salida.txt`. Comprobado en el anfitrión: 100 ficheros (`$MFT` de 69 MB, SOFTWARE, SYSTEM, `.evtx`…). **La marca aparece en el `$MFT`**: el fichero es tan pequeño que su contenido vive dentro de su propia entrada (*resident*) | `4E3515EF066EA32F7EBD22C91EB665A65AC5206382ECFE1AE77E6BE4E085EAF8` (anfitrión) | `11-triaje-final.png` (11:36:16) |
| ~11:38:41 (real; sello del equipo `2026-10-31_113819`) | 6 · Cifrado | `D:\windows\win7\cifrado\detectar-cifrado.bat` (Run as administrator, Intro ×2) → EDD 3.1 (`EDDv310.exe /batch`), solo lectura | `D:\casos\AFI-UD1-ALMADRABA\FORENSIC_10\cifrado\2026-10-31_113819\` (`edd-salida.txt`, su `.sha256` y `adquisicion.log`). **No se detectan volúmenes cifrados** | `7FADB304B124B7306517D785FC050CA74F7967D945F1FE5C575DD51A99E39AEC`: el de `certutil` en el equipo (`edd-salida.txt.sha256`) **coincide** con el del anfitrión | `12-detectar-cifrado.png` (11:38:47) |
| 11:44:29 (captura) | 7a · Extracción segura del USB | En el equipo: se cierran las ventanas de `cmd` y el Explorador abiertos en `D:`; *Computer* → `USB-RESP (D:)` → *Eject*. El icono de *Safely Remove Hardware* no aparecía en el área de notificaciones | *"Safe To Remove Hardware: the 'USB-RESP (D:)' device can now be safely removed"*. Se vacían las escrituras pendientes del exFAT, que no tiene diario. Deja huella mínima en el equipo (expulsión del dispositivo) | — | `13-usb-extraido.png` |
| **11:46:32** (`vmware.log`: `2026-10-31T10:46:32.042Z … Trying hard powerOff`) | 7b · Apagado | VMware → VM → Energía → **Apagar** (*Power Off*, corte de corriente). No se usa *Shut Down Guest* | VM apagada (*Condición: Apagada*). El Notepad de la marca se pierde con la RAM, que ya está en el volcado | — | `14-vm-apagada.png` (11:46:57) |

*04-marca-bloc-de-notas.png*

![Captura 04-marca-bloc-de-notas.png](capturas/04-marca-bloc-de-notas.png)

*05-marca-fichero.png*

![Captura 05-marca-fichero.png](capturas/05-marca-fichero.png)

*06-estado-inicial.png*

![Captura 06-estado-inicial.png](capturas/06-estado-inicial.png)

*08-ram-final.png*

![Captura 08-ram-final.png](capturas/08-ram-final.png)

*09-vivo-final.png*

![Captura 09-vivo-final.png](capturas/09-vivo-final.png)

*10-red-inicio.png*

![Captura 10-red-inicio.png](capturas/10-red-inicio.png)

*15-red-fin.png*

![Captura 15-red-fin.png](capturas/15-red-fin.png)

*11-triaje-final.png*

![Captura 11-triaje-final.png](capturas/11-triaje-final.png)

*12-detectar-cifrado.png*

![Captura 12-detectar-cifrado.png](capturas/12-detectar-cifrado.png)

*13-usb-extraido.png*

![Captura 13-usb-extraido.png](capturas/13-usb-extraido.png)

*14-vm-apagada.png*

![Captura 14-vm-apagada.png](capturas/14-vm-apagada.png)

### Paso 1 · Horas

| Hora del sistema investigado | Hora real (anfitrión) | Desfase |
|---|---|---|
| 11:12:29 (diálogo *Date and Time*, captura `06`) | 11:12:51 (creación de `06-estado-inicial.png`, 11:12:51.160) | **El equipo va unos 22 s atrasado** (margen de ~1 s por el retardo de la captura). Misma zona horaria que el anfitrión (UTC+1). Coherente con las capturas `04` y `05` |

Para la línea temporal: **hora real ≈ hora del equipo + 22 s**.

### Paso 6 · Cifrado

- Resultado de la comprobación: EDD 3.1 no encuentra volúmenes TrueCrypt, PGP, BitLocker (ni en la comprobación secundaria), VeraCrypt, SafeBoot, BestCrypt, Checkpoint, Sophos ni Symantec, ni procesos de cifrado en ejecución. Discos: `C:` (PhysicalDrive0, fijo, NTFS, 34 GB, 23 GB libres) y `D:` (USB-RESP, PhysicalDrive1, extraíble, exFAT, 29 GB, 27 GB libres).
- Decisión tomada: no hace falta adquirir el volumen lógico en vivo antes de apagar. La imagen en frío de la parte 2 será legible sin claves.

### Paso 7 · Apagado

- Método elegido: corte de corriente (*Power Off* en VMware, `hard powerOff` en `vmware.log`), después de extraer el USB con seguridad.
- Por qué: un apagado normal escribiría en el disco. Aplicaría el reinicio pendiente de la instalación de drivers, generaría eventos de apagado y ejecutaría lo que hubiera programado para el cierre, incluida una posible rutina antiforense del malware. El corte congela el disco tal como estaba.
- Qué se pierde con este método: la RAM y el estado volátil, ya adquiridos en los pasos 2 y 3. El NTFS queda sin cierre limpio, lo que es asumible porque su diario lo resuelve.
- Qué se conserva: el disco sin las escrituras del apagado, listo para la imagen en frío de la parte 2. El USB se extrajo antes para no corromper el exFAT (sin diario) con escrituras pendientes.

---

## Parte 2 — Adquisición post-mortem

| Hora | Paso | Acción (herramienta y versión) | Resultado / fichero generado | SHA-256 | Captura n.º |
|---|---|---|---|---|---|
| 12:08:53 (captura) | 1 · Hash del disco original, VM apagada | PowerShell en el anfitrión: `Get-FileHash …\FORENSIC_10\FORENSIC_10-disk1.vmdk -Algorithm SHA256` | `FORENSIC_10-disk1.vmdk`, 13 280 280 576 bytes, última modificación 11:46:32 (el *Power Off*) | `173A4388C7AD6C9BD6F5F4861C771EEE3138463ABFBCD5D5E962FDDDF85F26DF` | `16-hash-vmdk-antes.png` |
| 12:13:51 → 12:18:02 | 2 · Protección contra escritura | VMware → Caine → Editar configuración → Agregar → Disco duro → SCSI → *Usar un disco virtual existente* → `FORENSIC_10-disk1.vmdk` (no aparece el aviso de conversión de formato) → Avanzado → **Independiente – No persistente** → Aceptar. CAINE arranca desde `caine14.0.iso` | Disco 2 de CAINE: 32 GB (No persistente). Comprobado en `Caine.vmx`: `scsi0:1.mode = "independent-nonpersistent"`. El `.vmdk` sigue con fecha de modificación 11:46:32 | — | `17-disco-no-persistente.png` (12:16:55; renumerada desde `16-…`) |
| 12:30:11 (captura) | 3 · Arranque de CAINE | Se enciende la VM Caine (live desde `caine14.0.iso`, base Ubuntu 24.04) con el disco de FORENSIC_10 como disco 2. CAINE monta los dispositivos en **solo lectura** por defecto | Escritorio de CAINE | — | `18-caine-escritorio.png` |
| 12:38:16 (captura) | 3b · Identificación de discos | Se conecta el USB a CAINE (VMware → Dispositivos extraíbles → PHILIPS → Conectar). En CAINE: `lsblk -o NAME,SIZE,FSTYPE,LABEL,RO,MOUNTPOINT` | `sda` 20G (`sda1` ext4 `DESTINO`, RO=1): disco propio de la VM CAINE. **`sdb` 32G (`sdb1` NTFS *System Reserved* 100M + `sdb2` NTFS 31,9G), RO=1: disco de FORENSIC_10 (evidencia)**. `sdc` 29,3G (`sdc1` exFAT `USB-RESP` 27,3G + `sdc2` vfat `EVIDENCIA` 2G), RO=0: USB de respuesta. `sr0`: ISO de CAINE | — | `19-lsblk.png` |
| 12:43:17 (captura) | 3c · UnBlock (no se aplica) | Se abre UnBlock: `sda` y `sdb` en *Read-Only*, `sdc` (USB) ya en *Writable*, porque se conectó después del arranque. UnBlock alterna el estado, así que se cancela sin aplicar nada | La evidencia (`sdb`) sigue en solo lectura | — | `20-unblock.png` |
| 12:49:34 → 12:52:34 | 3d · Montaje del USB | Teclado de CAINE a español (`setxkbmap es`). `findmnt` sin salida: el USB aún no estaba montado. Se monta con un clic en `USB-RESP` en el gestor de archivos. Linux lo vuelve a detectar como **`sdd`** (antes `sdc`): se identifica por la etiqueta, no por el nombre | `sdd1` (USB-RESP) montado en `/media/caine/USB-RESP`, RO=0, 27,1 GB libres. CAINE crea además montajes de solo lectura propios (`loop1` → `/media/sdc1`, `loop2` → `/media/sdd1`). `sdb` sigue en RO=1 | — | `21-usb-montado.png`, `22-usb-montado-lsbk.png` |
| 12:57:47 (captura) | 3e · Carpeta de destino | `mkdir -p /media/caine/USB-RESP/casos/AFI-UD1-ALMADRABA/FORENSIC_10/imagen` y `ls -la` | Se crea `imagen/` junto a `cifrado`, `memoria`, `triaje` y `vivo`: el USB admite escritura (comprobado, no supuesto). Las carpetas de la parte 1 aparecen con 1 h menos (10:18, 10:24…) y la nueva con 12:57: indicio de que el reloj de CAINE no está en la misma zona horaria | — | `23-carpeta-imagen.png` |
| 13:00:32 (captura) | 4a · Reloj de CAINE | `date; date -u; timedatectl` | CAINE marca **12:59:50** y dice que es UTC (zona `Europe/London`, GMT +0000, sin sincronizar). Pero a esa hora eran las ~12:00 UTC, y las 13:00 en España. **VMware le pasa al reloj de la VM la hora local del anfitrión y CAINE la interpreta como UTC.** Regla para el acta: **las horas de CAINE y de Guymager son hora local de España (CET), aunque pongan UTC/GMT** (diferencia con el anfitrión inferior a 1 min; no se puede medir mejor con esta captura) | — | `24-reloj-caine.png` |
| 13:15:15 → 13:44:37 (capturas) | 4b · Imagen E01 | Guymager 0.8.13 (como superusuario) → `/dev/sdb` (VMware_Virtual_S, 34,4 GB decimales = 32 GiB) → *Acquire image*: Expert Witness Format (sub-formato Guymager), troceado a 2047 MiB. Case `AFI-UD1-ALMADRABA`, Evidence `FORENSIC_10`, Examiner `Jesus Alfonso Pedreno Dominguez`, destino `/media/caine/USB-RESP/casos/AFI-UD1-ALMADRABA/FORENSIC_10/imagen/FORENSIC_10`. MD5 + SHA-256, *Re-read source* y *Verify image*. Aviso de espacio (34,4 GB de origen frente a 27,1 GB libres): se continúa porque el E01 va comprimido y el disco solo tiene ~11 GB ocupados | **Finished – Verified & ok**, 0 sectores defectuosos, 53,46 MB/s de media. Durante la adquisición el USB pasa a llamarse `/dev/sde` (se vuelve a detectar); el `.info` confirma la verificación | ver `.info` | `26-guymager-dialogo-vacio.png`, `27-guymager-dialogo.png`, `28-guymager-aviso-espacio.png`, `29-guymager-fin.png` |
| 13:51:21 → 13:51:51 (capturas) | 4c · Revisión de la imagen y del `.info` | `ls -la` y `tail -40 FORENSIC_10.info` en la carpeta `imagen/` | `FORENSIC_10.E01` (2 146 429 770 B), `.E02` (2 146 428 283 B), `.E03` (987 364 035 B): **~5,3 GB en total** para un disco de 34,4 GB. `FORENSIC_10.info` (7 339 B). *No bad sectors* en la adquisición ni en la verificación, *Finished successfully*. **Source verification OK** y **Image verification OK**. Horas según CAINE (≈ hora de España, ver 4a): inicio 13:21:15, verificación 13:34:24, fin 13:41:41 (20 min 26 s; adquisición a 41,58 MB/s y verificación a 74,98 MB/s). Guymager no calcula el hash de cada fichero `.E0x` (`CalcImageFileMD5` desactivado) | Del **contenido del disco** (iguales en adquisición, origen releído e imagen): MD5 `5645de47fb61ca321eeed2c67af75af3`, SHA-256 `28f828d2aa46a171cd46c44424c9ce1f7bd37284f5f05d2ca8788187cb19d20a` | `30-imagen-info-1.png`, `31-imagen-info-2.png` |
| 13:55:18 (captura) | 5 · Hash del disco original, después | CAINE apagada con *Shutdown* normal (es la estación de trabajo, no la evidencia). PowerShell en el anfitrión: `Get-FileHash …\FORENSIC_10-disk1.vmdk -Algorithm SHA256` | **Idéntico al de antes**: el original no se modificó. Fecha de modificación del `.vmdk`: sigue siendo 31/10 11:46:32 | `173A4388C7AD6C9BD6F5F4861C771EEE3138463ABFBCD5D5E962FDDDF85F26DF` ✅ | `32-hash-vmdk-despues.png` (renombrada desde `32-hash.png`) |
| 13:58:24 (captura) | 6a · Retirada del disco de la evidencia | VMware → Caine → Editar configuración → Disco duro 2 (SCSI, 32 GB, No persistente) → Quitar → Aceptar. Con la imagen hecha, el original deja de estar conectado a ninguna VM | `Caine.vmx` (13:58:28) ya no tiene `scsi0:1.present`. El `.vmdk` no se borra (mismo tamaño, modificación 11:46:32) | — | `33-disco-evidencia-quitado.png` |
| 14:11:47 (captura) | 6b · Marca en la RAM | CAINE sin el disco de la evidencia; USB montado en `/media/caine/USB-RESP`. `strings -el FORENSIC_10-ram.raw \| grep "AFI-Jesus" \| sort \| uniq -c` (con `grep "AFI-"`, la del enunciado, salen cientos de líneas del número de caso `AFI-UD1-ALMADRABA`) | **`5 AFI-JesusPedreno-20261031`**: la marca aparece 5 veces en UTF-16 en el volcado | — | `34-marca-ram.png` (renombrada desde `34- marca-ram.png`) |
| 14:21:45 (captura) | 6c · Marca en el triaje | Se extrae solo el `$MFT` del zip a `/tmp/triaje` (RAM de CAINE, nada se escribe en el USB): `unzip -o ../triaje/FORENSIC_10-triaje-velociraptor.zip '*MFT' -d /tmp/triaje`, y `find /tmp/triaje -name '$MFT' -exec strings -el {} \; \| grep "marca-" \| sort \| uniq -c`. Primer intento con `.../triaje` (tres puntos): *cannot find or open* | En el `$MFT`, en UTF-16: **`marca-jesuspedreno.txt`** (confirma la extensión `.txt`, que el escritorio de Win7 ocultaba) y **`marca-jesuspedreno.lnk`**: el acceso directo que Windows crea en *Recent* al guardar el fichero, otra huella de la actividad del analista | — | `35-error-unzip.png`, `36-marca-triaje.png` |
| 14:27:07 → 14:29:46 | 6d · Imagen expuesta y particiones | `sudo ewfmount ../imagen/FORENSIC_10.E01 /mnt/ewf` (solo lectura; une los tres segmentos en `/mnt/ewf/ewf1`) y `sudo mmls /mnt/ewf/ewf1`. Primer intento fallido por una errata en la ruta (`AFI-AD1`): *libewf_handle_open: invalid filenames* | Tabla DOS: partición 002 NTFS, sectores 2048–206847 (100 MB, *System Reserved*); **partición 003 NTFS, desde el sector 206848** (sistema, ~31,9 GB) | — | `37-error-ewfmount.png`, `38-mmls.png` |
| 14:39:45 → 14:48:28 | 6e · Marca en la imagen | `sudo mount -o ro,loop,offset=105906176 /mnt/ewf/ewf1 /mnt/img` (206848 × 512; solo lectura). El aviso *"(hint) your fstab has been modified…"* es informativo. Antes, un intento fallido por crear `/mnr/img` en vez de `/mnt/img`. `ls -la /mnt/img/Users/*/Desktop/`. El `cat` no muestra nada (causa no aclarada); `xxd` no está instalado en CAINE, así que se usa `od -c` | `/mnt/img/Users/Administrador/Desktop/marca-jesuspedreno.txt`, **25 bytes**, 2026-10-31 10:09 UTC (11:09 en España, coherente con la captura `05`). `od -c`: **`AFI-JesusPedreno-20261031`** (25 bytes, sin salto de línea final). En el mismo escritorio: `crea_user.py` (11 003 B) y `crea_user.txt` (0 B), del 9 de agosto de 2017 | — | `39-error-mount.png`, `40-mount.png`, `41-marca-imagen-ls.png`, `42-error-xdd.png`, `43-marca-imagen-od.png` |
| — | (nota) | Disco `DESTINO` (`sda1`, ext4, 20 GB, dentro de la VM CAINE): no se recuerda con certeza cómo se preparó, así que **no se usa** como destino | Destino elegido: el USB de respuesta (`USB-RESP`), que admite la entrega al compañero y lo lee el anfitrión | — | — |

*16-hash-vmdk-antes.png*

![Captura 16-hash-vmdk-antes.png](capturas/16-hash-vmdk-antes.png)

*17-disco-no-persistente.png*

![Captura 17-disco-no-persistente.png](capturas/17-disco-no-persistente.png)

*18-caine-escritorio.png*

![Captura 18-caine-escritorio.png](capturas/18-caine-escritorio.png)

*19-lsblk.png*

![Captura 19-lsblk.png](capturas/19-lsblk.png)

*20-unblock.png*

![Captura 20-unblock.png](capturas/20-unblock.png)

*21-usb-montado.png*

![Captura 21-usb-montado.png](capturas/21-usb-montado.png)

*22-usb-montado-lsbk.png*

![Captura 22-usb-montado-lsbk.png](capturas/22-usb-montado-lsbk.png)

*23-carpeta-imagen.png*

![Captura 23-carpeta-imagen.png](capturas/23-carpeta-imagen.png)

*24-reloj-caine.png*

![Captura 24-reloj-caine.png](capturas/24-reloj-caine.png)

*26-guymager-dialogo-vacio.png*

![Captura 26-guymager-dialogo-vacio.png](capturas/26-guymager-dialogo-vacio.png)

*27-guymager-dialogo.png*

![Captura 27-guymager-dialogo.png](capturas/27-guymager-dialogo.png)

*28-guymager-aviso-espacio.png*

![Captura 28-guymager-aviso-espacio.png](capturas/28-guymager-aviso-espacio.png)

*29-guymager-fin.png*

![Captura 29-guymager-fin.png](capturas/29-guymager-fin.png)

*30-imagen-info-1.png*

![Captura 30-imagen-info-1.png](capturas/30-imagen-info-1.png)

*31-imagen-info-2.png*

![Captura 31-imagen-info-2.png](capturas/31-imagen-info-2.png)

*32-hash-vmdk-despues.png*

![Captura 32-hash-vmdk-despues.png](capturas/32-hash-vmdk-despues.png)

*33-disco-evidencia-quitado.png*

![Captura 33-disco-evidencia-quitado.png](capturas/33-disco-evidencia-quitado.png)

*34-marca-ram.png*

![Captura 34-marca-ram.png](capturas/34-marca-ram.png)

*35-error-unzip.png*

![Captura 35-error-unzip.png](capturas/35-error-unzip.png)

*36-marca-triaje.png*

![Captura 36-marca-triaje.png](capturas/36-marca-triaje.png)

*37-error-ewfmount.png*

![Captura 37-error-ewfmount.png](capturas/37-error-ewfmount.png)

*38-mmls.png*

![Captura 38-mmls.png](capturas/38-mmls.png)

*39-error-mount.png*

![Captura 39-error-mount.png](capturas/39-error-mount.png)

*40-mount.png*

![Captura 40-mount.png](capturas/40-mount.png)

*41-marca-imagen-ls.png*

![Captura 41-marca-imagen-ls.png](capturas/41-marca-imagen-ls.png)

*42-error-xdd.png*

![Captura 42-error-xdd.png](capturas/42-error-xdd.png)

*43-marca-imagen-od.png*

![Captura 43-marca-imagen-od.png](capturas/43-marca-imagen-od.png)

**Desviación:** el enunciado usa el tipo *Inmutable* de VirtualBox. En VMware el equivalente es el modo de disco **Independiente – No persistente**: las escrituras van a un fichero temporal que se descarta al apagar. La prueba de que funcionó es el hash del `.vmdk` antes y después.

---

## Incidencias del 2026-10-31

| Hora | Qué pasó | Actuación | Afecta a (evidencia) |
|---|---|---|---|
| **10:52:11** | Encendido de la VM (*Power on* en VMware). Hora exacta según VMware: *"Último encendido solicitado: 31/10/2026 10:52:11"* (captura `14`) y `vmware.log` (`2026-10-31T09:52:11.409Z … CPU reset: hard`). El analista había anotado 10:53 de memoria | — | Inicio de la manipulación del equipo |
| 10:52:59 | Al arrancar aparece *Windows Error Recovery* ("Windows did not shut down successfully") | Captura `01-error-recovery-.png`. No se pulsa nada: la opción por defecto *Start Windows Normally* se selecciona sola al acabar la cuenta atrás (25 s en la captura) | Ninguna; se anota porque es una decisión sobre el arranque |
| 10:55:29 | Arranque completo: pantalla de inicio de sesión del usuario `Administrador` (Windows 7 Professional) | Captura `02-inicio.png` | — |
| ~11:01 (recordada; antes de 11:01:57) | Inicio de sesión como `Administrador` | Contraseña facilitada en el enunciado | Genera un evento de inicio de sesión en el registro de seguridad |
| 11:01:57 | Escritorio. Aparece el aviso de Windows *"You must restart your computer to apply these changes"* (Restart Now / Restart Later). Origen probable: Windows detecta el hardware virtual de VMware, distinto del de la OVA (VirtualBox), e instala drivers. No verificado. En el escritorio: `crea_user` (texto), `crea_user` (script de Python) y la Papelera. Reloj del sistema: 11:01 AM, 10/31/2026 | Captura `03-escritorio.png`. **No se reinicia**: se perdería todo lo volátil. No se abre ningún fichero del escritorio | La instalación de drivers modifica el registro y `setupapi` del equipo: consecuencia de virtualizarlo en otro hipervisor |
| 11:15:26 (captura) | Se conecta el USB de respuesta a la VM (VMware → Dispositivos extraíbles → PHILIPS → Conectar). Windows instala el driver (*"Device driver software installed successfully"*) y lo monta como **`USB-RESP (D:)`**. Sale *AutoPlay*: no se elige ninguna opción y se cierra con la X | Captura `07-usb-conectado.png` (renumerada desde `06-usb-conectado.png`, la hora de creación no cambia) | El USB deja rastro en el equipo: `setupapi.dev.log`, `USBSTOR` y `MountedDevices` en el registro. Inevitable en adquisición en vivo |
| entre 11:01:57 y 11:06:12 | Se cierra con la X el aviso *"You must restart your computer"* (equivale a *Restart Later*) | No se reinicia | El reinicio queda pendiente en Windows; el corte de corriente del paso 7 evita aplicarlo |

*01-error-recovery-.png*

![Captura 01-error-recovery-.png](capturas/01-error-recovery-.png)

*02-inicio.png*

![Captura 02-inicio.png](capturas/02-inicio.png)

*03-escritorio.png*

![Captura 03-escritorio.png](capturas/03-escritorio.png)

*07-usb-conectado.png*

![Captura 07-usb-conectado.png](capturas/07-usb-conectado.png)

---

---

## Parte 2 — Documentación de la máquina y sus ficheros

Carpeta de la máquina: `C:\Users\er_pi\Documents\Virtual Machines\FORENSIC_10\`.

| Comprobación | Resultado |
|---|---|
| Discos | Uno solo: `FORENSIC_10-disk1.vmdk` (`scsi0:0`, 32 GiB, 13 280 280 576 B) |
| Instantáneas | Ninguna: `FORENSIC_10.vmsd` ocupa 0 bytes y no hay ficheros `.vmsn` ni discos `-0000*.vmdk` |
| Equivalencias con VirtualBox | `.vmx` en lugar de `.vbox`; `vmware.log` en lugar de `Logs/` |

Comandos, en Git Bash, el 2026-11-03 (de 12:55:01 a 12:56:30):

```bash
cd "/c/Users/er_pi/Documents/Virtual Machines/FORENSIC_10"
sha256sum FORENSIC_10.vmx FORENSIC_10.vmxf FORENSIC_10.vmsd FORENSIC_10-file1.nvram FORENSIC_10.scoreboard vmware.log FORENSIC_10-disk1.vmdk > /tmp/vm.sha
cp -pv FORENSIC_10.vmx FORENSIC_10.vmxf FORENSIC_10.vmsd FORENSIC_10-file1.nvram FORENSIC_10.scoreboard vmware.log "/e/casos/AFI-UD1-ALMADRABA/FORENSIC_10/imagen/vm-config/"
cd "/e/casos/AFI-UD1-ALMADRABA/FORENSIC_10/imagen/vm-config" && grep -v 'vmdk' /tmp/vm.sha | sha256sum -c
```

Resultado: 6 de 6 `OK`. El hash del `.vmdk` sigue siendo `173a4388…26df`. Los valores están en `hashes/sha256-vm-config.txt`.

**Desviación:** estos ficheros no se copiaron ni se les calculó el hash antes de la imagen del disco, porque se hizo solo con el `.vmdk`. Se completó el 2026-11-03. El `.vmx`, el `.vmsd`, el `.vmxf`, el `.nvram`, el `.scoreboard` y `vmware.log` conservan fechas de modificación anteriores a la parte 2 (2026-10-30 21:02 o 2026-10-31 11:46), y la máquina no se ha vuelto a encender.

### Por qué Guymager para el disco

El enunciado admite otra herramienta si se justifica. CAINE monta los discos en solo lectura por defecto, Guymager genera un E01 comprimido, relee el origen y verifica la imagen, y el disco se conectó en modo no persistente. FTK Imager 8.3 no abre en el Windows 7 investigado. El resultado se comprueba con el hash del `.vmdk` antes y después y con las verificaciones del `.info`.

---

## Parte 2 — El pendrive encontrado (EV-07)

**Soporte:** el responsable de la actividad autorizó el 2026-11-03 usar USB-RESP como pendrive encontrado, en lugar de un pendrive distinto. Se valoró un segundo pendrive de 8 GB, que no se modificó ni se usó.

**Datos:** PHILIPS, modelo FM32FD70B, 32 GB en la etiqueta y 29,3 GB reales (61 440 000 sectores de 512 B). No lleva número de serie impreso; el de Windows y FTK Imager es `3798771057290437452`.

**Por qué FTK Imager con bloqueo por software:** es una de las dos vías del enunciado. En la parte 2 quedó comprobado que CAINE deja escribible un USB conectado después del arranque (captura `20`), así que el bloqueo por script se puede activar y comprobar antes de conectar. No se puede grabar la imagen en el propio pendrive, así que se crea en `C:` y después se copia al USB como maestra.

| Hora (2026-11-03) | Paso | Acción | Resultado | Captura |
|---|---|---|---|---|
| ~10:20 | Fotografías | Fotografía general y de los datos del pendrive. Las fotos no conservan EXIF; la hora es la de los ficheros | Sin datos de ubicación | `45-pendrive-general.jpg`, `46-pendrive-datos.jpg` |
| ~10:35 | Marca en el pendrive | FTK Imager exporta `marca-jesuspedreno.txt` de la copia de trabajo de EV-06 a `E:\`. No se arranca FORENSIC_10 porque cambiaría el hash del `.vmdk` | 25 bytes, SHA-256 `ccaa0e42248f5cf60e7ed788d951467c3cc1290a49cbfc7e3fdd2e443b2cd8d8` | — |
| 10:40:38 | Bloqueo | USB desconectado. Terminal como administrador: `powershell -NoProfile -ExecutionPolicy Bypass -File "...\windows\moderno\imagen\bloqueo-usb.ps1"`. En la ventana, se activa la protección global | `bloqueo-usb.log`: `WriteProtect=1` | — |
| 10:41:28 | Comprobación | Se conecta el USB y se pulsa Comprobar | `E:` y `F:`: `no deja escribir -> PROTEGIDO` | `47-bloqueo-protegido.png` (10:41:52) |
| 11:36:16 | Opciones | FTK Imager 8.3.0.27 → *File → Create Disk Image → Physical Drive* → `\\.\PHYSICALDRIVE1` → E01, destino `C:\Users\er_pi\Documents\AFI-UD2\maestra-EV-07\USB-RESP-EV07`, *Verify images after they are created* | Caso `AFI-UD1-ALMADRABA`, evidencia `EV-07` | `48-imagen-usb-opciones.png` |
| 11:36:53 → 12:07:40 | Adquisición | Windows sigue indicando `IsReadOnly True` a las 10:42:26, 11:37:11 y 12:11:18 | 8 segmentos `USB-RESP-EV07.E01` a `.E08` | — |
| 12:07:40 → 12:09:34 | Verificación | Automática | MD5 `0349af97692063613497b598cb1732dd`, SHA-1 `b37668d99971fab23bab87379458df194d22c730`, `verified` | `49-imagen-usb-verificacion.png` (12:09:48) |
| 12:15:49 | Fin del bloqueo | Se desactiva desde la ventana del script | `bloqueo-usb.log`: `WriteProtect=0` | — |
| 12:17:40 → 12:55:01 | Maestra en el USB | Comandos de abajo | 9 de 9 `OK` | — |
| 12:56:40 | Copia de trabajo | La carpeta de `C:` pasa a llamarse `copia-trabajo-EV-07`. El `.E01.txt` conserva la ruta anterior `maestra-EV-07` | Hashes idénticos a los de la maestra | — |

*45-pendrive-general.jpg*

![Captura 45-pendrive-general.jpg](capturas/45-pendrive-general.jpg)

*46-pendrive-datos.jpg*

![Captura 46-pendrive-datos.jpg](capturas/46-pendrive-datos.jpg)

*47-bloqueo-protegido.png*

![Captura 47-bloqueo-protegido.png](capturas/47-bloqueo-protegido.png)

*48-imagen-usb-opciones.png*

![Captura 48-imagen-usb-opciones.png](capturas/48-imagen-usb-opciones.png)

*49-imagen-usb-verificacion.png*

![Captura 49-imagen-usb-verificacion.png](capturas/49-imagen-usb-verificacion.png)

```bash
cd ~/Documents/AFI-UD2/maestra-EV-07
sha256sum USB-RESP-EV07.E0? USB-RESP-EV07.E01.txt > /tmp/ev07.sha
cp -v USB-RESP-EV07.E0? USB-RESP-EV07.E01.txt "/e/casos/AFI-UD1-ALMADRABA/FORENSIC_10/pendrive/"
cd "/e/casos/AFI-UD1-ALMADRABA/FORENSIC_10/pendrive" && sha256sum -c /tmp/ev07.sha
```

**Alcance de la imagen:** EV-07 refleja el pendrive hasta las 12:07:40. Después se añadieron `pendrive/`, `imagen/vm-config/` y `hashes.sha256`, de modo que el soporte físico ya no coincide con la imagen. La evidencia es el E01.

---

## Parte 3 — Búsqueda de la marca

La marca está en la RAM (captura `34`), en el `$MFT` del triaje (captura `36`) y en la imagen del disco (capturas `41` y `43`). En el pendrive se buscó el 2026-11-03 desde CAINE:

```bash
sudo mkdir -p /mnt/ev07 /mnt/ev07p1
sudo mount -o remount,ro /media/caine/USB-RESP
sudo ewfmount /media/caine/USB-RESP/casos/AFI-UD1-ALMADRABA/FORENSIC_10/pendrive/USB-RESP-EV07.E01 /mnt/ev07
sudo ls -l /mnt/ev07/ewf1
sudo mmls /mnt/ev07/ewf1
sudo mount -o ro,loop,offset=1048576 /mnt/ev07/ewf1 /mnt/ev07p1
mount | grep -E "USB-RESP|ev07p1"
ls -l /mnt/ev07p1/marca-jesuspedreno.txt
od -c /mnt/ev07p1/marca-jesuspedreno.txt
sha256sum /mnt/ev07p1/marca-jesuspedreno.txt
```

| Hora | Comprobación | Resultado | Captura |
|---|---|---|---|
| 15:28:13 | Tamaño y particiones | `ewf1`: 31 457 280 000 B (61 440 000 × 512). Partición 1 exFAT desde el sector 2048; partición 2 FAT32 desde el sector 57 212 928. Desplazamiento: 2048 × 512 = 1 048 576 | `50-ev07-particiones.png` |
| 15:52:56 | Marca en la imagen | USB e imagen en `ro`. `marca-jesuspedreno.txt`, 25 bytes, `AFI-JesusPedreno-20261031`, SHA-256 `ccaa0e42…2cd8d8`, igual al del fichero copiado | `51-ev07-marca.png` |

*50-ev07-particiones.png*

![Captura 50-ev07-particiones.png](capturas/50-ev07-particiones.png)

*51-ev07-marca.png*

![Captura 51-ev07-marca.png](capturas/51-ev07-marca.png)

---

## Parte 4 — Copias de trabajo y fichero de hashes

**EV-06.** Copia en `C:\Users\er_pi\Documents\AFI-UD2\copia-trabajo\`, de 09:00:32 a 09:04:59 del 2026-11-03. Primera verificación a las 09:05:41 y segunda a las 09:08:17: 4 de 4 `OK` (`hashes/verificacion-copia-trabajo.txt`, captura `44`).

*44-copia-trabajo-verificada.png*

![Captura 44-copia-trabajo-verificada.png](capturas/44-copia-trabajo-verificada.png)

```bash
CASO=/e/casos/AFI-UD1-ALMADRABA/FORENSIC_10
mkdir -p ~/Documents/AFI-UD2/copia-trabajo
cp -v "$CASO"/imagen/FORENSIC_10.{E01,E02,E03,info} ~/Documents/AFI-UD2/copia-trabajo/
cd ~/Documents/AFI-UD2/copia-trabajo
grep "imagen/" "$CASO/sha256-usb-evidencias.txt" | tr -d '\r' | sed 's#imagen/##' | sha256sum -c
```

**EV-07.** Copia en `C:\Users\er_pi\Documents\AFI-UD2\copia-trabajo-EV-07\`. La maestra se escribió desde esa carpeta y se verificó contra sus hashes (`hashes/sha256-ev07-maestra.txt`).

**Comprobación del USB tras la incidencia de CAINE** (16:04:42, captura `52`). Sobre USB-RESP en solo lectura:

*52-reverificacion-usb.png*

![Captura 52-reverificacion-usb.png](capturas/52-reverificacion-usb.png)

```bash
cd /media/caine/USB-RESP/casos/AFI-UD1-ALMADRABA/FORENSIC_10
sha256sum -c sha256-usb-evidencias.txt | grep -v ': OK$'
echo "Fallos: ${PIPESTATUS[0]}"
```

Resultado: ninguna línea distinta de `OK` y `Fallos: 0` en las 37 entradas.

**Fichero de hashes** (terminado hacia las 16:27, captura `53` a las 16:29:41). El USB se pone en escritura solo para este paso:

*53-hashes-sha256.png*

![Captura 53-hashes-sha256.png](capturas/53-hashes-sha256.png)

```bash
cd /media/caine/USB-RESP/casos/AFI-UD1-ALMADRABA/FORENSIC_10
sudo mount -o remount,rw /media/caine/USB-RESP
find . -type f ! -name hashes.sha256 -exec sha256sum {} + > hashes.sha256
sudo mount -o remount,ro /media/caine/USB-RESP
```

Resultado: 53 líneas. Comparadas después en el equipo del analista con las listas anteriores: 37 de 37 de `sha256-usb-evidencias.txt`, 9 de 9 de la maestra de EV-07 y 6 de 6 de la configuración de la VM, sin diferencias. La línea restante es la de `sha256-usb-evidencias.txt`. El fichero se copió a `hashes/hashes.sha256` a las 16:34.

A las 16:56:26, desde Git Bash en el equipo del analista, se copió `bloqueo-usb.log` a `pendrive/`, porque el enunciado pide guardarlo junto a las evidencias, y se añadió su línea a `hashes.sha256`. El fichero queda con 54 líneas; la captura `53` muestra las 53 anteriores.

### Entrega a otra persona y devolución

**Custodia hasta la entrega.** Desde las 16:56 del 2026-11-03 hasta la entrega, USB-RESP estuvo siempre bajo el control de Jesús Alfonso Pedreño Domínguez.

**Entrega.** 2026-11-05, ~12:30 (hora aproximada, anotada después de la entrega). Jesús Alfonso Pedreño Domínguez entrega USB-RESP a Sergio Estudillo Marabot. Motivo: comprobación por un tercero.

**Comprobación de quien recibe.** En su portátil, con CAINE 14 en una VM: USB identificado a las 12:45 (`sdd1`, `USB-RESP`), montado en solo lectura a las 12:50 con `mount -o ro`, `sha256sum -c hashes.sha256` de 12:50 a 13:09 y desmontado a las 13:11. Resultado: 54 de 54 sin `FAILED`. `wc -l` da 54 líneas.

*01-lsblk-usb-identificado-12-45.png*

![Captura 01-lsblk-usb-identificado-12-45.png](capturas/verificacion-companero/01-lsblk-usb-identificado-12-45.png)

*02-montaje-solo-lectura-12-50.png*

![Captura 02-montaje-solo-lectura-12-50.png](capturas/verificacion-companero/02-montaje-solo-lectura-12-50.png)

*03-sha256sum-c-inicio-12-50.png*

![Captura 03-sha256sum-c-inicio-12-50.png](capturas/verificacion-companero/03-sha256sum-c-inicio-12-50.png)

*04-sha256sum-c-imagen-13-04.png*

![Captura 04-sha256sum-c-imagen-13-04.png](capturas/verificacion-companero/04-sha256sum-c-imagen-13-04.png)

*05-sha256sum-c-fin-sin-failed-13-09.png*

![Captura 05-sha256sum-c-fin-sin-failed-13-09.png](capturas/verificacion-companero/05-sha256sum-c-fin-sin-failed-13-09.png)

*06-wc-54-y-desmontaje-13-11.png*

![Captura 06-wc-54-y-desmontaje-13-11.png](capturas/verificacion-companero/06-wc-54-y-desmontaje-13-11.png)

Incidencia: la salida no se guardó en un fichero porque se tecleó `$/` en lugar de `~/` en el `tee`. Las capturas no muestran todas las líneas: se ven 41 de 54, con todas las de `memoria/`, `vivo/`, `triaje/` y `cifrado/`, `FORENSIC_10.info`, `FORENSIC_10.E01` y el final de `pendrive/`. Las otras 13 no se ven. El resultado se apoya en que la captura final no muestra el aviso `WARNING` que `sha256sum -c` escribe al terminar si algún fichero falla. Capturas y resumen en `capturas/verificacion-companero/`.

*01-lsblk-usb-identificado-12-45.png*

![Captura 01-lsblk-usb-identificado-12-45.png](capturas/verificacion-companero/01-lsblk-usb-identificado-12-45.png)

*02-montaje-solo-lectura-12-50.png*

![Captura 02-montaje-solo-lectura-12-50.png](capturas/verificacion-companero/02-montaje-solo-lectura-12-50.png)

*03-sha256sum-c-inicio-12-50.png*

![Captura 03-sha256sum-c-inicio-12-50.png](capturas/verificacion-companero/03-sha256sum-c-inicio-12-50.png)

*04-sha256sum-c-imagen-13-04.png*

![Captura 04-sha256sum-c-imagen-13-04.png](capturas/verificacion-companero/04-sha256sum-c-imagen-13-04.png)

*05-sha256sum-c-fin-sin-failed-13-09.png*

![Captura 05-sha256sum-c-fin-sin-failed-13-09.png](capturas/verificacion-companero/05-sha256sum-c-fin-sin-failed-13-09.png)

*06-wc-54-y-desmontaje-13-11.png*

![Captura 06-wc-54-y-desmontaje-13-11.png](capturas/verificacion-companero/06-wc-54-y-desmontaje-13-11.png)

**Devolución.** 2026-11-05, 13:13. Sergio Estudillo Marabot devuelve USB-RESP a Jesús Alfonso Pedreño Domínguez. El resumen de Sergio indica 13:12 para la desconexión y la devolución; el registro mantiene 13:13, la hora anotada al recibirlo. Número de serie comprobado al recibirlo (`3798771057290437452`). `hashes.sha256` idéntico a `hashes/hashes.sha256`, con 54 líneas. `sha256sum -c hashes.sha256` en Git Bash, de 13:14:58 a 13:30:38: 54 de 54 `OK`, sin `FAILED` ni `WARNING`. Salida completa en `hashes/verificacion-devolucion.txt`, captura `54`.

*54-verificacion-devolucion.png*

![Captura 54-verificacion-devolucion.png](capturas/54-verificacion-devolucion.png)

**Firmas.** 2026-11-05, 13:45. Firmas escritas, no manuscritas: nombre y fecha en cada casilla del acta y del registro, con la conformidad de Jesús Alfonso Pedreño Domínguez y de Sergio Estudillo Marabot, presentes los dos. Las dos filas de custodia del 2026-11-03 al 2026-11-05 se añadieron después, al revisar el registro, y las firmó Jesús Alfonso Pedreño Domínguez el 2026-11-05 14:14. Las filas de Sergio no cambiaron. El profesor acepta las firmas escritas.

---

## Incidencias del 2026-11-03

| Hora | Qué pasó | Actuación | Afecta a |
|---|---|---|---|
| Antes de 10:40 | El enunciado se amplió con el pendrive encontrado después de la adquisición del 2026-10-31. La marca no pudo copiarse a un pendrive en su momento | Se exporta el fichero desde la copia de trabajo de EV-06 | Orden de la marca, documentado |
| Antes de 10:40 | `bloqueo-usb.ps1` no abre su ventana al lanzarlo de forma normal | Se lanza desde una terminal como administrador con el comando anotado. Causa no confirmada. Script sin modificar, SHA-256 `150adbc3bee0ee8f8662bee8758e4791fb090c20a48807ac24ae9bea531eef5a` | Ninguna |
| ~10:41 → ~11:36 | El analista se ausenta. El USB queda conectado al portátil, con el bloqueo activo, en el aula y sin vigilancia directa | Se registra. `IsReadOnly True` a las 10:42:26 y a las 11:37:11 | Hueco de vigilancia de EV-07 antes de adquirirla. Protege contra escritura lógica, no contra un acceso físico |
| 11:36 | La nota del E01 dice `PROTEGIDO 11:34`, pero el registro indica 10:41:28 | No se repite la imagen. Vale la hora del registro y de la captura `47` | Dato de texto dentro de `USB-RESP-EV07.E01.txt` |
| 15:24:30 | El primer `ewfmount` se hizo con el segmento `.E07` en lugar de `.E01` | Se desmonta y se repite con `.E01`, comprobando el tamaño completo | Ninguna (`50a-error-montaje-e07.png`) |
| 15:35:44 → 15:41:28 | `mount`, `fsstat` y `fls` fallan (`can't read superblock`, `Cannot determine file system type`). `dmesg`: `exFAT-fs (sdb1) … err:-5`. El USB había pasado de `sdb` a `sdc` y `ewfmount` seguía apuntando al dispositivo antiguo. No se ejecuta `fsck` ni `ntfsfix` | Se desmonta y se vuelve a montar desde la ruta actual | Ninguna: 37 de 37 `OK` después (`50b`, `50c`, `50d`) |
| 15:39 | La raíz de USB-RESP figura modificada a esa hora, con el USB montado en escritura por CAINE. Coincide con la nueva detección del USB. Causa no confirmada | Se pasa a `ro`. Se comprueban los hashes | Solo el directorio raíz del soporte |
| 15:44:39, 15:49 | Erratas al teclear: `sudo -o`, `EV07P1` y `jesuspredeno` | Se repiten los comandos | Ninguna (`50e`, `50f`) |
| 16:27 | `hashes.sha256` figura en Windows con hora 17:27. CAINE trata la hora local como UTC (captura `24`) y exFAT guarda esa referencia, lo que explicaría la hora de más | Vale la hora de la captura | Solo la fecha del fichero |
| Revisión | `FOUND.000` en la raíz, con `FILE0000.CHK` (28 278 784 B) y `FILE0001.CHK` (471 433 216 B), del 2026-10-31 13:20. La hora cae en la sesión de CAINE del 2026-10-31, justo antes del inicio de Guymager (13:21:15). El responsable de la custodia recuerda desconexiones incorrectas en las VM y una reparación de Windows, sin hora exacta. Causa no confirmada | Se conservan sin abrir. Los hashes posteriores coinciden | Soporte; ninguna evidencia con hash |

*50a-error-montaje-e07.png*

![Captura 50a-error-montaje-e07.png](capturas/50a-error-montaje-e07.png)

*50b-error-mount-superblock.png*

![Captura 50b-error-mount-superblock.png](capturas/50b-error-mount-superblock.png)

*50c-error-sdb1-exfat.png*

![Captura 50c-error-sdb1-exfat.png](capturas/50c-error-sdb1-exfat.png)

*50d-usb-reenumerado.png*

![Captura 50d-usb-reenumerado.png](capturas/50d-usb-reenumerado.png)

*50e-error-sudo.png*

![Captura 50e-error-sudo.png](capturas/50e-error-sudo.png)

*50f-erratas-ls-grep.png*

![Captura 50f-erratas-ls-grep.png](capturas/50f-erratas-ls-grep.png)

---

## Anexo — Validación de las herramientas (2026-10-30)

Este documento explica por qué la adquisición se hizo con algunas herramientas distintas de las
que nombra el enunciado, y con qué pruebas se tomó cada decisión. Todas las pruebas se hicieron
sobre una **copia de la máquina investigada**, que después se descartó. La adquisición evaluable
se hizo sobre una importación nueva y limpia de la OVA original.

---

### A.1. Resumen

| Paso del enunciado | Herramienta pedida | Herramienta usada | Motivo |
|---|---|---|---|
| Entorno | VirtualBox | VMware Workstation | Hipervisor disponible en el equipo del analista |
| 2 · RAM | DumpIt | **WinPmem mini** | El driver de DumpIt no carga en el equipo: error `0xC0000428` |
| 3 · Datos volátiles | `respuesta-en-vivo.ps1` | **Adaptación a PowerShell 2.0** del mismo script | El original no arranca: `ParserError` por el operador `-notin` |
| 4 · Red | `VBoxManage … nictrace` | **`vnetsniffer.exe`** de VMware | Equivalente en VMware: captura desde fuera de la máquina, como un TAP |
| 5 · Triaje | KAPE o Velociraptor offline | **Velociraptor 0.77.3 *legacy*** | KAPE necesita .NET 4.5.2; el equipo solo tiene .NET 4.0 |
| 6 · Cifrado | Según los apuntes | **EDD 3.1** (Magnet) y `manage-bde -status` | Sin cambios respecto al kit |

La causa común es el sistema del equipo investigado: **Windows 7 Professional SP1 sin
actualizar** (build 7601.17514), con **PowerShell 2.0**, **.NET 4.0** y **sin soporte de firmas
SHA-2**. Las versiones actuales de DumpIt, KAPE y el script del módulo están hechas para Windows
10/11.

---

### A.2. Entorno de las pruebas

| Elemento | Detalle |
|---|---|
| Equipo del analista | Portátil con Windows 11, VMware Workstation (`vmrun` 1.17.0) |
| Máquina investigada | `FORENSIC_10.ova`, SHA-256 `DAF0EF5255D98276A6912A53611DB5C0CBF2CCCBB49A180DD7FCC0F95E14930C` |
| Datos del sistema | Hostname `FORENSE-06` · Windows 7 Professional SP1 6.1.7601 · 1 GB RAM · disco de 32 GB · teclado inglés (EE. UU.) |
| Copia de pruebas | Primera importación de la OVA (descartada al terminar) |
| Máquina de referencia | VM Windows 11 propia, para validar las herramientas en un sistema moderno |
| USB de respuesta | Pendrive PHILIPS de 32 GB, exFAT, etiqueta `USB-RESP` |
| Red de la VM | Cambiada de *bridged* (como venía en la OVA) a *host-only* antes de arrancarla. Captura de validación `10` |

*10-vm-importada-red-bridged.png*

![Captura 10-vm-importada-red-bridged.png](capturas/validacion/10-vm-importada-red-bridged.png)

---

### A.3. Cronología

| Hora | Qué se hizo | Resultado |
|---|---|---|
| 15:18 | SHA-256 de la OVA recibida | `DAF0EF52…14930C` |
| 15:28 | Configuración de la VM: red *host-only* y USB 2.0 | — |
| 15:53 | Prueba de `vnetsniffer` en la VM Windows 11 | Válido: 68 paquetes, tráfico en los dos sentidos |
| 16:00–16:41 | DumpIt y KAPE en la VM Windows 11 | Válidos en Windows 11 |
| 17:03 | Inicio del primer intento sobre FORENSIC_10 | Descartado después (ver apartado 5) |
| 17:58 | `$PSVersionTable` en FORENSIC_10 | PowerShell 2.0. Captura de validación `08` |
| 18:05 | DumpIt 3.6 en FORENSIC_10 | **Falla:** `0xC0000428`. Captura de validación `09` |
| 18:10 | Decisión: el primer intento se descarta y la máquina se usa como copia de pruebas | — |
| 18:32 | WinPmem mini en FORENSIC_10 | **Válido** |
| 18:46 | `respuesta-en-vivo.ps1` original | **Falla:** `ParserError`. Captura de validación `11` |
| 19:00 | Adaptación a PowerShell 2.0 | **Válida** |
| 19:01 | KAPE 1.3.0.2 | **Falla:** `NotSupportedException`. Captura de validación `12` |
| 19:42 | Velociraptor *legacy* sin UCRT | **Falla:** falta `api-ms-win-crt-stdio-l1-1-0.dll`. Captura de validación `13` |
| 20:10 | Velociraptor *legacy* con UCRT local | **Válido** |
| 20:16 | EDD 3.1 | **Válido** |
| 20:26 | 21 herramientas de consola de Sysinternals | 19 válidas, 2 fallan. Capturas de validación `14`–`16` |
| 20:4x | SHA-256 de la OVA antes de reimportarla | `DAF0EF52…14930C`, sin cambios |

*08-powershell-2.0.png*

![Captura 08-powershell-2.0.png](capturas/validacion/08-powershell-2.0.png)

*09-dumpit-0xC0000428.png*

![Captura 09-dumpit-0xC0000428.png](capturas/validacion/09-dumpit-0xC0000428.png)

*11-respuesta-en-vivo-original-parsererror.png*

![Captura 11-respuesta-en-vivo-original-parsererror.png](capturas/validacion/11-respuesta-en-vivo-original-parsererror.png)

*12-kape-notsupportedexception.png*

![Captura 12-kape-notsupportedexception.png](capturas/validacion/12-kape-notsupportedexception.png)

*13-velociraptor-legacy-sin-ucrt.png*

![Captura 13-velociraptor-legacy-sin-ucrt.png](capturas/validacion/13-velociraptor-legacy-sin-ucrt.png)

*14-autorunsc-entry-point.png*

![Captura 14-autorunsc-entry-point.png](capturas/validacion/14-autorunsc-entry-point.png)

*15-procdump-falta-dll.png*

![Captura 15-procdump-falta-dll.png](capturas/validacion/15-procdump-falta-dll.png)

*16-pslist-licencia.png*

![Captura 16-pslist-licencia.png](capturas/validacion/16-pslist-licencia.png)

---

### A.4. Detalle por herramienta

#### A.4.1 Memoria RAM: DumpIt → WinPmem mini

- **Pedido:** DumpIt.
- **Probado:** Magnet DumpIt 3.6.20230117 (x64), con los mismos parámetros validados en Windows
  11 (`/QUIET /NOLYTICS /NOCOMPRESS /TYPE DMP`).
- **Resultado en Windows 7:** `Error: Load driver failed with 0xC0000428 error code`. El código
  0xC0000428 significa que Windows no puede verificar la firma digital del fichero. El driver de
  DumpIt 3.6 está firmado con SHA-2 y un Windows 7 sin la actualización de soporte SHA-2 no sabe
  verificarlo. DumpIt intentó registrar el servicio `HKLM\System\CurrentControlSet\Services\DumpIt`
  (con `ImagePath = C:\Windows\system32\Drivers\DumpIt.sys`) y lo borró al fallar. Captura de validación `09`.
- **Alternativas descartadas:**
  - Una versión antigua de DumpIt: no hay ninguna fuente oficial que la distribuya. Se descarta
    usar copias de terceros sin hash oficial con el que verificarlas.
  - Desactivar la verificación de firmas al arrancar (F8): modifica el laboratorio.
- **Solución:** **WinPmem mini** (`winpmem_mini_x64_rc2.exe`, del kit desde el 2026-10-13). Su
  driver está firmado con **sha1RSA** (Binalyze LLC, 2020), que Windows 7 sí verifica. Está en la
  lista de herramientas de los apuntes.
- **Prueba en Windows 7:** volcado de 1.073.741.824 bytes en 3 minutos, código de salida 1
  (correcto en WinPmem mini). La marca `AFI-JesusPedreno-20261030` aparece 6 veces en UTF-16.
- **A tener en cuenta:** WinPmem extrae su driver a
  `C:\Users\<usuario>\AppData\Local\Temp\pme*.tmp`, lo carga y lo borra. Es una escritura en el
  disco del equipo investigado, inherente a cualquier herramienta de RAM, y se anota en el acta.

#### A.4.2 Datos volátiles: script del módulo → adaptación a PowerShell 2.0

- **Pedido:** `respuesta-en-vivo.ps1`.
- **Resultado en Windows 7:** `ParserError` en la línea 88 (`You must provide a value expression
  on the right-hand side of the '-' operator`): el operador `-notin` no existe en PowerShell 2.0.
  Al ser un error de sintaxis, **el script no llega a ejecutarse** y no recoge nada. Captura de validación `11`.
- **Otros comandos incompatibles:** `Get-CimInstance`, `Get-NetTCPConnection`,
  `Get-NetUDPEndpoint`, `Get-SmbShare`, `Get-LocalUser`, `Get-Disk`, `Get-PnpDevice`,
  `Get-Clipboard`, `Get-TimeZone`, `Get-FileHash` y `$PSScriptRoot`.
- **Solución:** `respuesta-en-vivo-ps2.ps1`, una adaptación con **los mismos 19 pasos, en el mismo
  orden y con los mismos nombres de fichero**, que cambia cada comando por su equivalente en
  Windows 7 (`Get-WmiObject`, `netstat -ano`, `net user`, `net share`…). Mantiene la autoría y la
  licencia CC BY-SA 4.0 del original. El script original no se ha modificado.
- **Prueba en Windows 7:** 19 ficheros sin errores en 12 segundos. Los 21 hashes de su
  `hashes-sha256.csv` coinciden al recalcularlos en el equipo del analista.

#### A.4.3 Red: `nictrace` de VirtualBox → `vnetsniffer` de VMware

- **Pedido:** `VBoxManage controlvm "FORENSIC_10" nictracefile1 …` (VirtualBox).
- **Solución:** `vnetsniffer.exe /w <fichero>.pcap VMnet1`, de la instalación de VMware
  Workstation, ejecutado como administrador **en el equipo del analista**. Escucha el switch
  virtual *host-only* desde fuera de la máquina, sin instalar nada en ella, igual que el
  `nictrace` del enunciado.
- **Prueba:** sobre la VM Windows 11, formato libpcap clásico (cabecera `d4c3b2a1`, Ethernet),
  68 paquetes con peticiones y respuestas en los dos sentidos.

#### A.4.4 Triaje: KAPE → Velociraptor *legacy*

- **Pedido:** KAPE o un recolector offline de Velociraptor.
- **KAPE 1.3.0.2 en Windows 7:** `Unhandled Exception: System.NotSupportedException: The requested
  security protocol is not supported` en `ServicePointManager.set_SecurityProtocol`. KAPE está
  compilado para **.NET Framework 4.5.2** y el equipo solo tiene .NET 4.0. Captura de validación `12`.
- **Velociraptor 0.77.3 normal:** compilado con Go 1.26/1.27. Go no funciona en Windows 7 desde
  la versión 1.21.
- **Solución:** la compilación oficial **`velociraptor-v0.77.3-windows-amd64-legacy.exe`**,
  publicada por el proyecto como *"For Windows 7 Only"* (compilada con Go 1.20). Firma Authenticode
  válida de **Rapid7 LLC**. SHA-256 del ejecutable:
  `7b699a6670e0caaf8ce951225dff479c141580a9fe8d9d0a1a7c5e83d32e1698`.
- **Segundo problema:** sin el *Universal C Runtime* no arranca (`api-ms-win-crt-stdio-l1-1-0.dll
  is missing`). Captura de validación `13`. Se resolvió con el **despliegue local del UCRT** que permite
  Microsoft: las 43 DLL oficiales van en la misma carpeta que el ejecutable, en el USB, y **no se
  instala nada en el equipo**. Las DLL salen del paquete KB2999226 x64 de Microsoft (versión
  10.0.10240, firma de Microsoft válida). El catálogo de Microsoft Update ya solo ofrece ese paquete
  para Vista y Server 2008; su SHA-1 coincide con el que Microsoft pone en el nombre del fichero
  (`0befbb0b78588f7c9f17ead1da3abeda2b6f4c7f`).
- **Recogida:** artefacto `Windows.Search.FileFinder` con acceso NTFS en bruto, subida de ficheros
  y hash de cada uno. Recoge `$MFT`, las colmenas `SAM`, `SYSTEM`, `SOFTWARE`, `SECURITY` y
  `DEFAULT`, los `NTUSER.DAT` y los registros de eventos `.evtx`. Desde la versión 0.75,
  Velociraptor ya no incluye el artefacto equivalente a KAPE (`Windows.KapeFiles.Targets`).
- **Prueba en Windows 7:** código 0 en 26 segundos; zip de 17,9 MB con 101 ficheros (`$MFT` de
  69 MB, 5 colmenas, 2 `NTUSER.DAT`, 69 `.evtx`). El nombre `marca-jesuspedreno` aparece en el
  `$MFT` en UTF-16, que es lo que pide el enunciado.

#### A.4.5 Cifrado: EDD

- **Magnet Encrypted Disk Detector 3.1** necesita .NET 4.0, que el equipo tiene. **Válido** en
  Windows 7: revisa los 2 discos y los 2 volúmenes y no encuentra cifrado. `certutil -hashfile …
  SHA256` también funciona en Windows 7.
- La adaptación de la respuesta en vivo incluye además `manage-bde -status` (BitLocker).

#### A.4.6 Otras herramientas del kit

- **Sysinternals:** de 21 herramientas de consola, 19 funcionan en Windows 7. Fallan
  `autorunsc64` (usa `CryptCATAdminAcquireContext2` de `WINTRUST.dll`, de Windows 8+) y
  `procdump64` (le falta `api-ms-win-core-path-l1-1-0.dll`). `pslist64` ignora `-accepteula` y
  muestra la ventana de licencia. Capturas de validación `14`–`16` y `prueba-compatibilidad-win7-FORENSE-06.txt`.
- **FTK Imager 8.3:** no abre en Windows 7. No afecta a la actividad: se usa en el equipo del
  analista, y la imagen de disco de la parte 2 se hace con CAINE y Guymager.

---

### A.5. El primer intento y la reimportación

El primer intento de adquisición empezó a las 17:03. A las 18:05 DumpIt falló y dejó claro que
las herramientas no estaban validadas para este sistema. Como la memoria no se puede adquirir dos
veces en el mismo estado, y cada intento fallido deja huellas en el equipo, se decidió:

1. **Descartar el primer intento** y usar esa importación como copia de pruebas para validar
   cada herramienta en el mismo Windows 7.
2. **Reimportar la OVA original** para la adquisición evaluable, comprobando antes que su SHA-256
   seguía siendo `DAF0EF52…14930C`.

En un caso real no se puede repetir: el equipo es único. Aquí se puede porque la evidencia
original es la OVA, que no se ha modificado.

El registro completo del primer intento se conserva aparte, fuera de esta entrega.

---

### A.6. Interacciones con el equipo que se anotan en el acta

Además de la marca, estas acciones tocan el equipo investigado y se registran con su hora:

- Inicio de sesión como `Administrador` (cuenta con SID …-1000; la cuenta integrada
  `Administrator` está deshabilitada), necesario para trabajar. Deja eventos de inicio de sesión.
- Conexión del USB de respuesta: Windows instala el driver y registra el dispositivo.
- WinPmem: driver temporal en `C:\Users\<usuario>\AppData\Local\Temp\pme*.tmp`, cargado y borrado.
- Cada herramienta ejecutada crea un proceso y ocupa memoria.

---

### A.7. Ficheros de esta validación

Capturas `08` a `16` en `capturas/validacion/`; `COMPATIBILIDAD.txt` y `prueba-compatibilidad-win7-FORENSE-06.txt` en `hashes/registros/validacion/`. Son una numeración propia, anterior a la adquisición.

---

### A.8. Fuentes

- Velociraptor, descargas y compilaciones *legacy*: <https://docs.velociraptor.app/downloads/>
- Velociraptor, definición de `Windows.Search.FileFinder` (v0.77.3):
  <https://github.com/Velocidex/velociraptor/blob/v0.77.3/artifacts/definitions/Windows/Search/FileFinder.yaml>
- Velociraptor, proyecto de triaje (`Windows.Triage.Targets`):
  <https://github.com/Velocidex/velociraptor-triage-collector>
- Microsoft Update Catalog, KB2999226: <https://www.catalog.update.microsoft.com/Search.aspx?q=KB2999226>
- Magnet RAM Capture / DumpIt: <https://magnetforensics.com/free-tool-magnet-ram-capture>
