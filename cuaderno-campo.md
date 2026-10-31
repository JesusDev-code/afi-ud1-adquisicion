# Cuaderno de campo — AFI UD1 · Adquirir un equipo comprometido (intento definitivo)

Borrador de trabajo. Se rellena **en el momento**, nunca al final y de memoria.
Al terminar, se pasa a `acta-adquisicion.docx` y a los `registro-cadena-custodia.docx`.

**Hora de referencia:** la del anfitrión, en CET (UTC+1). Fechas siempre en formato `AAAA-MM-DD`.
La hora de cada captura es la de **creación del fichero** en el anfitrión (metadatos), al segundo.

---

## Datos del caso

| Campo | Valor |
|---|---|
| Número de caso | `AFI-UD1-ALMADRABA` |
| Etiqueta del equipo (en todas las herramientas) | `FORENSIC_10` |
| Hostname real del equipo | `FORENSE-06` |
| Analista | |
| Testigo (si lo hay) | |
| Solicitante | Bahía Sistemas (para Almadraba Diseño) |
| Fichero de configuración de la VM | `C:\Users\er_pi\Documents\Virtual Machines\FORENSIC_10\FORENSIC_10.vmx` (hardware Workstation 11.x; 1 GB RAM, 1 CPU, disco SCSI 32 GB) |
| USB de respuesta | Pendrive PHILIPS 32 GB, exFAT, etiqueta `USB-RESP` |
| Fecha y hora de inicio | 2026-10-31 10:49 (CET, anfitrión) |
| Fecha y hora de fin | 2026-10-31 11:52 (hashes calculados en el anfitrión; USB conectado al anfitrión como `E:`) |

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

---

## Herramientas (todas validadas en este mismo Windows 7, ver `validacion-herramientas/`)

| Paso | Dónde se ejecuta | Herramienta |
|---|---|---|
| 2 · RAM | VM | `D:\windows\win7\memoria\volcar-memoria.bat` (WinPmem mini) |
| 3 · Volátiles | VM | `D:\windows\win7\respuesta-en-vivo.bat` (adaptación a PowerShell 2.0) |
| 4 · Red | **Anfitrión**, PowerShell como administrador | `cd "C:\Program Files\VMware\VMware Workstation"` y `.\vnetsniffer.exe /w <ruta>\red.pcap VMnet1`; Ctrl+C para parar |
| 5 · Triaje | VM | `D:\windows\win7\triaje\triaje-velociraptor.bat` (Velociraptor legacy) |
| 6 · Cifrado | VM | `D:\windows\win7\cifrado\detectar-cifrado.bat` (EDD) |

Todos los `.bat`: **clic derecho → Run as administrator**, e **Intro, Intro** para aceptar caso y equipo.

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
| 13:15:15 → 13:44:37 (capturas) | 4b · Imagen E01 | Guymager 0.8.13 (como superusuario) → `/dev/sdb` (VMware_Virtual_S, 34,4 GB decimales = 32 GiB) → *Acquire image*: Expert Witness Format (sub-formato Guymager), troceado a 2047 MiB. Case `AFI-UD1-ALMADRABA`, Evidence `FORENSIC_10`, Examiner `Jesus Alfonso Pedreno Dominguez`, destino `/media/caine/USB-RESP/casos/AFI-UD1-ALMADRABA/FORENSIC_10/imagen/FORENSIC_10`. MD5 + SHA-256, *Re-read source* y *Verify image*. Aviso de espacio (34,4 GB de origen frente a 27,1 GB libres): se continúa porque el E01 va comprimido y el disco solo tiene ~11 GB ocupados | **Finished – Verified & ok**, 0 sectores defectuosos, 53,46 MB/s de media. Durante la adquisición el USB pasa a llamarse `/dev/sde` (se vuelve a detectar): pendiente de revisar en el `.info` | ver `.info` | `26-guymager-dialogo-vacio.png`, `27-guymager-dialogo.png`, `28-guymager-aviso-espacio.png`, `29-guymager-fin.png` |
| 13:51:21 → 13:51:51 (capturas) | 4c · Revisión de la imagen y del `.info` | `ls -la` y `tail -40 FORENSIC_10.info` en la carpeta `imagen/` | `FORENSIC_10.E01` (2 146 429 770 B), `.E02` (2 146 428 283 B), `.E03` (987 364 035 B): **~5,3 GB en total** para un disco de 34,4 GB. `FORENSIC_10.info` (7 339 B). *No bad sectors* en la adquisición ni en la verificación, *Finished successfully*. **Source verification OK** y **Image verification OK**. Horas según CAINE (≈ hora de España, ver 4a): inicio 13:21:15, verificación 13:34:24, fin 13:41:41 (20 min 26 s; adquisición a 41,58 MB/s y verificación a 74,98 MB/s). Guymager no calcula el hash de cada fichero `.E0x` (`CalcImageFileMD5` desactivado) | Del **contenido del disco** (iguales en adquisición, origen releído e imagen): MD5 `5645de47fb61ca321eeed2c67af75af3`, SHA-256 `28f828d2aa46a171cd46c44424c9ce1f7bd37284f5f05d2ca8788187cb19d20a` | `30-imagen-info-1.png`, `31-imagen-info-2.png` |
| 13:55:18 (captura) | 5 · Hash del disco original, después | CAINE apagada con *Shutdown* normal (es la estación de trabajo, no la evidencia). PowerShell en el anfitrión: `Get-FileHash …\FORENSIC_10-disk1.vmdk -Algorithm SHA256` | **Idéntico al de antes**: el original no se modificó. Fecha de modificación del `.vmdk`: sigue siendo 31/10 11:46:32 | `173A4388C7AD6C9BD6F5F4861C771EEE3138463ABFBCD5D5E962FDDDF85F26DF` ✅ | `32-hash-vmdk-despues.png` (renombrada desde `32-hash.png`) |
| 13:58:24 (captura) | 6a · Retirada del disco de la evidencia | VMware → Caine → Editar configuración → Disco duro 2 (SCSI, 32 GB, No persistente) → Quitar → Aceptar. Con la imagen hecha, el original deja de estar conectado a ninguna VM | `Caine.vmx` (13:58:28) ya no tiene `scsi0:1.present`. El `.vmdk` no se borra (mismo tamaño, modificación 11:46:32) | — | `33-disco-evidencia-quitado.png` |
| 14:11:47 (captura) | 6b · Marca en la RAM | CAINE sin el disco de la evidencia; USB montado en `/media/caine/USB-RESP`. `strings -el FORENSIC_10-ram.raw \| grep "AFI-Jesus" \| sort \| uniq -c` (con `grep "AFI-"`, la del enunciado, salen cientos de líneas del número de caso `AFI-UD1-ALMADRABA`) | **`5 AFI-JesusPedreno-20261031`**: la marca aparece 5 veces en UTF-16 en el volcado | — | `34-marca-ram.png` (renombrada desde `34- marca-ram.png`) |
| 14:21:45 (captura) | 6c · Marca en el triaje | Se extrae solo el `$MFT` del zip a `/tmp/triaje` (RAM de CAINE, nada se escribe en el USB): `unzip -o ../triaje/FORENSIC_10-triaje-velociraptor.zip '*MFT' -d /tmp/triaje`, y `find /tmp/triaje -name '$MFT' -exec strings -el {} \; \| grep "marca-" \| sort \| uniq -c`. Primer intento con `.../triaje` (tres puntos): *cannot find or open* | En el `$MFT`, en UTF-16: **`marca-jesuspedreno.txt`** (confirma la extensión `.txt`, que el escritorio de Win7 ocultaba) y **`marca-jesuspedreno.lnk`**: el acceso directo que Windows crea en *Recent* al guardar el fichero, otra huella de la actividad del analista | — | `35-error-unzip.png`, `36-marca-triaje.png` |
| 14:27:07 → 14:29:46 | 6d · Imagen expuesta y particiones | `sudo ewfmount ../imagen/FORENSIC_10.E01 /mnt/ewf` (solo lectura; une los tres segmentos en `/mnt/ewf/ewf1`) y `sudo mmls /mnt/ewf/ewf1`. Primer intento fallido por una errata en la ruta (`AFI-AD1`): *libewf_handle_open: invalid filenames* | Tabla DOS: partición 002 NTFS, sectores 2048–206847 (100 MB, *System Reserved*); **partición 003 NTFS, desde el sector 206848** (sistema, ~31,9 GB) | — | `37-error-ewfmount.png`, `38-mmls.png` |
| 14:39:45 → 14:48:28 | 6e · Marca en la imagen | `sudo mount -o ro,loop,offset=105906176 /mnt/ewf/ewf1 /mnt/img` (206848 × 512; solo lectura). El aviso *"(hint) your fstab has been modified…"* es informativo. Antes, un intento fallido por crear `/mnr/img` en vez de `/mnt/img`. `ls -la /mnt/img/Users/*/Desktop/`. El `cat` no muestra nada (causa no aclarada); `xxd` no está instalado en CAINE, así que se usa `od -c` | `/mnt/img/Users/Administrador/Desktop/marca-jesuspedreno.txt`, **25 bytes**, 2026-10-31 10:09 UTC (11:09 en España, coherente con la captura `05`). `od -c`: **`AFI-JesusPedreno-20261031`** (25 bytes, sin salto de línea final). En el mismo escritorio: `crea_user.py` (11 003 B) y `crea_user.txt` (0 B), del 9 de agosto de 2017 | — | `39-error-mount.png`, `40-mount.png`, `41-marca-imagen-ls.png`, `42-error-xdd.png`, `43-marca-imagen-od.png` |
| — | (nota) | Disco `DESTINO` (`sda1`, ext4, 20 GB, dentro de la VM CAINE): no se recuerda con certeza cómo se preparó, así que **no se usa** como destino | Destino elegido: el USB de respuesta (`USB-RESP`), que admite la entrega al compañero y lo lee el anfitrión | — | — |

**Desviación:** el enunciado usa el tipo *Inmutable* de VirtualBox. En VMware el equivalente es el modo de disco **Independiente – No persistente**: las escrituras van a un fichero temporal que se descarta al apagar. La prueba de que funcionó es el hash del `.vmdk` antes y después.

---

## Incidencias

| Hora | Qué pasó | Qué hice | Afecta a (evidencia) |
|---|---|---|---|
| **10:52:11** | Encendido de la VM (*Power on* en VMware). Hora exacta según VMware: *"Último encendido solicitado: 31/10/2026 10:52:11"* (captura `14`) y `vmware.log` (`2026-10-31T09:52:11.409Z … CPU reset: hard`). El analista había anotado 10:53 de memoria | — | Inicio de la manipulación del equipo |
| 10:52:59 | Al arrancar aparece *Windows Error Recovery* ("Windows did not shut down successfully") | Captura `01-error-recovery-.png`. No se pulsa nada: la opción por defecto *Start Windows Normally* se selecciona sola al acabar la cuenta atrás (25 s en la captura) | Ninguna; se anota porque es una decisión sobre el arranque |
| 10:55:29 | Arranque completo: pantalla de inicio de sesión del usuario `Administrador` (Windows 7 Professional) | Captura `02-inicio.png` | — |
| ~11:01 (recordada; antes de 11:01:57) | Inicio de sesión como `Administrador` | Contraseña facilitada en el enunciado | Genera un evento de inicio de sesión en el registro de seguridad |
| 11:01:57 | Escritorio. Aparece el aviso de Windows *"You must restart your computer to apply these changes"* (Restart Now / Restart Later). Origen probable: Windows detecta el hardware virtual de VMware, distinto del de la OVA (VirtualBox), e instala drivers. No verificado. En el escritorio: `crea_user` (texto), `crea_user` (script de Python) y la Papelera. Reloj del sistema: 11:01 AM, 10/31/2026 | Captura `03-escritorio.png`. **No se reinicia**: se perdería todo lo volátil. No se abre ningún fichero del escritorio | La instalación de drivers modifica el registro y `setupapi` del equipo: consecuencia de virtualizarlo en otro hipervisor |
| 11:15:26 (captura) | Se conecta el USB de respuesta a la VM (VMware → Dispositivos extraíbles → PHILIPS → Conectar). Windows instala el driver (*"Device driver software installed successfully"*) y lo monta como **`USB-RESP (D:)`**. Sale *AutoPlay*: no se elige ninguna opción y se cierra con la X | Captura `07-usb-conectado.png` (renumerada desde `06-usb-conectado.png`, la hora de creación no cambia) | El USB deja rastro en el equipo: `setupapi.dev.log`, `USBSTOR` y `MountedDevices` en el registro. Inevitable en adquisición en vivo |
| entre 11:01:57 y 11:06:12 | Se cierra con la X el aviso *"You must restart your computer"* (equivale a *Restart Later*) | No se reinicia | El reinicio queda pendiente en Windows; el apagado del paso 7 lo aplicará |

---

## Recordatorios aprendidos en el primer intento

- El teclado de la VM está en inglés (EE. UU.): el guion se escribe con el `-` del **teclado numérico**.
- La marca, **revisada letra por letra** antes de seguir. Nombre del fichero: `marca-jesuspedreno`, **sin** `.txt`.
- Al arrancar sale *Windows Error Recovery*: elegir **Start Windows Normally**.
- Capturas **siempre desde el anfitrión** y **nunca sobrescribir** una captura: cada una con su nombre.
- El USB conectado a la VM no lo ve el anfitrión: para leerlo, desconectarlo antes.

## Comprobación al terminar la parte 1

- [ ] Cada evidencia tiene hora, herramienta, fichero y hash.
- [ ] Ninguna salida quedó en el disco del equipo investigado (salvo lo anotado en Incidencias).
- [ ] Los hashes de esta tabla coinciden con los de los registros de las herramientas.
- [ ] Las capturas están numeradas y guardadas en `capturas/`.
