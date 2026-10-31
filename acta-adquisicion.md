# Acta de adquisición de evidencias digitales — BORRADOR

> Borrador en Markdown que sigue la plantilla `acta-adquisicion.docx`, sección por sección.
> Cuando esté revisado, se pasa a la plantilla y se exporta a `acta-adquisicion.pdf`.
> Los datos salen de `afi-ud1-adquisicion/cuaderno-campo.md` y de las capturas.

---

## 1. Datos del caso

| Campo | Valor |
|---|---|
| Número de caso | AFI-UD1-ALMADRABA |
| Fecha | 2026-10-31 |
| Hora de inicio y de fin (indica la zona horaria) | Inicio 10:49 · fin 14:48. Hora de España peninsular (CET, UTC+1), tomada del equipo del analista. Parte 1, en vivo: 10:49–11:52. Parte 2, post-mortem: 12:08–14:48. |
| Lugar (dirección y estancia) | Equipo del analista (Windows 11 con VMware Workstation), en un laboratorio virtual aislado. El equipo investigado es la máquina virtual FORENSIC_10, conectada solo a una red interna (host-only). |
| Solicitante del análisis | Bahía Sistemas, la consultora que mantiene los equipos de Almadraba Diseño. |
| Base legal y referencia | Encargo de Bahía Sistemas en nombre de Almadraba Diseño, titular del equipo, que facilita la máquina y la cuenta de administrador. Referencia: actividad AFI UD1 «Adquirir un equipo comprometido». |
| Objetivo del análisis | Adquirir las evidencias del ordenador del cuarto de informática (FORENSE-06), sospechoso de estar comprometido por conexiones extrañas. Se adquieren en vivo (memoria, datos volátiles, red y triaje) y en frío (imagen del disco), preservando su integridad y la cadena de custodia, para analizarlas en la UD2. |

## 2. Personas presentes

| Nombre y apellidos | Documento de identidad | Función | Firma |
|---|---|---|---|
| Jesús Alfonso Pedreño Domínguez | *(a mano al firmar; ver nota sobre el DNI)* | Analista | *(a mano)* |
| No hubo testigo | no aplica | Testigo | no aplica |
| No hubo fedatario ni representante de la parte | no aplica | Fedatario / representante | no aplica |

> **Nota sobre el DNI:** el repositorio de la entrega es **público**. Antes de poner el DNI en el PDF,
> pregunta al profesor si hace falta o si basta con la firma.

## 3. Estado de la escena

| Campo | Valor |
|---|---|
| Descripción de la escena y de los equipos | Un único equipo: la máquina virtual FORENSIC_10, recibida como `FORENSIC_10.ova` (SHA-256 `DAF0EF5255D98276A6912A53611DB5C0CBF2CCCBB49A180DD7FCC0F95E14930C`). Windows 7 Professional SP1 x64, hostname FORENSE-06, 1 GB de RAM, 1 CPU y un disco virtual de 32 GiB. Se importa en VMware Workstation (`C:\Users\er_pi\Documents\Virtual Machines\FORENSIC_10\FORENSIC_10.vmx`). Antes de encenderla (2026-10-30, hacia las 22:30), la red pasa de *bridged* a *host-only* y el USB a modo 2.0. Configuración comprobada en la captura 00. |
| Estado de cada equipo | Se recibe apagada y se enciende a las **10:52:11** (hora tomada de `vmware.log`) para la adquisición en vivo. Al arrancar aparece *Windows Error Recovery*, por un apagado no limpio anterior, y la opción por defecto (*Start Windows Normally*) se elige sola. Se inicia sesión como Administrador con la contraseña facilitada. Al entrar, Windows pide reiniciar para aplicar drivers: **no se reinicia**. Red: *host-only*, IP fija `10.28.0.25`, sin conexiones hacia fuera. Sin volúmenes cifrados. Se apaga con un corte de corriente a las **11:46:32** (`vmware.log`: *hard powerOff*). |
| Hora del sistema frente a hora real (desfase) | Sistema **11:12:29** frente a real **11:12:51**: el equipo va **unos 22 s atrasado**. Zona horaria del equipo: UTC+01:00 (Madrid), la misma que la del analista. Captura 06. En la estación CAINE, la hora mostrada es la de España aunque la etiquete como UTC/GMT, porque VMware le pasa la hora local del anfitrión; las horas de Guymager son, por tanto, hora de España. Captura 24. |
| Fotografías tomadas (numeración) | Capturas **00 a 43**, en la carpeta `capturas/`, numeradas en el orden en que se hicieron. La hora de cada una es la de creación del fichero en el equipo del analista. |

## 4. Evidencias adquiridas

| ID | Descripción | Método y herramienta (con versión) | Formato y fichero | SHA-256 original / copia | N.º precinto |
|---|---|---|---|---|---|
| EV-01 | Memoria RAM de FORENSE-06 (1 GiB) | Volcado en vivo con WinPmem mini (`winpmem_mini_x64_rc2.exe`), lanzado desde el USB. 11:19:00–11:22:00 | RAW · `memoria/FORENSIC_10-ram.raw` (1 073 741 824 B) | `1f440ce6547b7b88640852b588a50cb55212080674591d65199149e2d05e2b2a` / copia: pendiente | No aplica (soporte USB-RESP) |
| EV-02 | Datos volátiles: procesos, conexiones, servicios, tareas, inicio automático, usuarios, USB… | Script `respuesta-en-vivo-ps2.ps1` (adaptación del script del módulo a PowerShell 2.0), lanzado desde el USB. 11:24:40–11:24:51 | TXT (19 secciones + `hashes-sha256.csv`) · `vivo/AFI-UD1-ALMADRABA_FORENSIC_10_20261031-102418/` | `00-registro.txt`: `f6650bc89fa010df3498f9b248f61aeb0a09dbc04aecf3de45fadc7d2484ee07`; resto en `hashes/sha256-usb-evidencias.txt` | No aplica (soporte USB-RESP) |
| EV-03 | Tráfico de red de la interfaz del equipo (switch VMnet1) | Captura desde fuera del equipo, como un TAP, con `vnetsniffer` (VMware Workstation), sin instalar nada en él. 11:31:20–~11:49 | PCAP (libpcap) · `red/FORENSIC_10-red.pcap` (5 104 B, 34 paquetes) | `0ab1fda310ab5e8049880106f32dec32c5b485e62f87c0aca32131f06127676c` / copia: pendiente | No aplica (soporte USB-RESP) |
| EV-04 | Triaje: `$MFT`, colmenas SAM/SYSTEM/SOFTWARE/SECURITY/DEFAULT, `NTUSER.DAT` y registros `.evtx` | Velociraptor 0.77.3 *legacy*, artefacto `Windows.Search.FileFinder` con accessor `ntfs` (lectura en bruto), lanzado desde el USB. 11:34:27–11:34:52 | ZIP (100 ficheros) · `triaje/FORENSIC_10-triaje-velociraptor.zip` (17 931 246 B) | `4e3515ef066ea32f7ebd22c91eb665a65ac5206382ecfe1ae77e6be4e085eaf8` / copia: pendiente | No aplica (soporte USB-RESP) |
| EV-05 | Comprobación de volúmenes cifrados | EDD 3.1 (`EDDv310.exe /batch`), solo lectura, lanzado desde el USB. 11:38:41 | TXT · `cifrado/2026-10-31_113819/edd-salida.txt` | `7fadb304b124b7306517d785fc050ca74f7967d945f1fe5c575dd51a99e39aec` (coincide con el de `certutil` en el equipo) | No aplica (soporte USB-RESP) |
| EV-06 | Disco del equipo: 32 GiB (34,4 GB), tabla DOS, NTFS *System Reserved* + NTFS del sistema | Imagen post-mortem con Guymager 0.8.13 en CAINE 14 (live). El disco se conecta en modo independiente no persistente, el equivalente de VMware a «Inmutable», y se verifican el origen y la imagen. 13:21:15–13:41:41 | E01 (Guymager), en trozos de 2047 MiB · `imagen/FORENSIC_10.E01`, `.E02` y `.E03` + `FORENSIC_10.info` | Contenido del disco (adquisición = origen = imagen): `28f828d2aa46a171cd46c44424c9ce1f7bd37284f5f05d2ca8788187cb19d20a`. Ficheros `.E0x` en `hashes/sha256-usb-evidencias.txt`. Copia de trabajo: pendiente | No aplica (soporte USB-RESP) |
| ORIG | Fichero del disco virtual original `FORENSIC_10-disk1.vmdk` (13 280 280 576 B). No se copia: se conserva en el equipo del analista | `Get-FileHash` (PowerShell) antes (12:08:53) y después (13:55:18) de la adquisición post-mortem | VMDK · `Documents\Virtual Machines\FORENSIC_10\` | Antes = después: `173a4388c7ad6c9bd6f5f4861c771eee3138463abfbcd5d5e962fdddf85f26df` | No aplica |

## 5. Incidencias y observaciones

### Incidencias

**Herramientas cambiadas respecto al enunciado** (detalle y pruebas en `validacion-herramientas/README.md`):

- VirtualBox → VMware Workstation. Para el modo «Inmutable» se usa el disco independiente no persistente.
- DumpIt → WinPmem mini. El driver de DumpIt 3.6 no carga en este Windows 7 sin parches (error `0xC0000428`).
- `respuesta-en-vivo.ps1` → `respuesta-en-vivo-ps2.ps1`, porque el equipo tiene PowerShell 2.0.
- KAPE → Velociraptor *legacy*, porque KAPE necesita .NET 4.5.2.
- `nictrace` → `vnetsniffer`.

**Acciones sobre el equipo original y su efecto:**

- Arranque con *Windows Error Recovery*: la opción por defecto se elige sola por la cuenta atrás.
- Inicio de sesión de Administrador: queda un evento de inicio de sesión.
- Windows instala drivers del hardware de VMware y pide reiniciar. Se cierra el aviso con la X y no se reinicia; queda pendiente.
- Marca: Bloc de notas sin guardar y fichero `marca-jesuspedreno.txt` en el escritorio. La marca se copia con Ctrl+C, así que también está en el portapapeles, y Windows crea `marca-jesuspedreno.lnk` en *Recent*.
- Se abre el diálogo de fecha y hora para leerla, sin cambiar nada, y se cierra con *Cancel*.
- Al conectar el USB de respuesta (`D:`), Windows instala su driver: queda huella en `setupapi.dev.log`, `USBSTOR` y `MountedDevices`. *AutoPlay* se cierra sin elegir ninguna opción.
- Se navega con el Explorador solo dentro de `D:`, y el USB se extrae con *Eject* antes del apagado.
- WinPmem carga y borra un driver temporal (en `%TEMP%\pme*.tmp`).

**Otras incidencias:**

- `vnetsniffer` no arrancó al primer intento porque no existía la carpeta de salida. Arrancó a las 11:31:20, después de los datos volátiles, en el orden del enunciado.
- `vivo/19-portapapeles.txt` sale vacío aunque la marca se había copiado. No se ha investigado la causa.
- Guymager avisa de que el destino tiene menos espacio que el origen (27,1 GB frente a 34,4 GB). Se continúa porque el E01 va comprimido; ocupó unos 5,3 GB.
- En CAINE, el USB se vuelve a detectar con otro nombre (`sdc` → `sdd` → `sde`). Se identifica siempre por la etiqueta, y la verificación de la imagen sale bien.
- Errores de escritura en comandos de CAINE (rutas mal tecleadas), sin efecto sobre las evidencias. Se ven en las capturas 35, 37, 39 y 42.
- Hubo un primer intento el 2026-10-30, descartado cuando falló DumpIt. Se reimportó la OVA, verificada por hash, y se empezó de cero.

### Observaciones

**La marca `AFI-JesusPedreno-20261031` aparece en las tres adquisiciones:**

- RAM: 5 apariciones en UTF-16 (`strings -el | grep`). Captura 34.
- Triaje: en el `$MFT` aparecen `marca-jesuspedreno.txt` y `marca-jesuspedreno.lnk` en UTF-16. Captura 36.
- Imagen montada en solo lectura: `Users/Administrador/Desktop/marca-jesuspedreno.txt`, de 25 bytes, con el contenido de la marca (`od -c`). Capturas 41 y 43.

**Integridad:** el hash del `.vmdk` es idéntico antes y después de la adquisición post-mortem. Guymager termina con *Source verification OK* e *Image verification OK*, sin sectores defectuosos. Al terminar, los hashes de las evidencias del USB coinciden con los de la parte 1.

**Apagado:** se elige el corte de corriente para que Windows no escriba en el disco al apagarse: no se aplica el reinicio pendiente, no se generan eventos de apagado y no se ejecutan posibles rutinas antiforenses del malware. La RAM ya estaba adquirida. Antes se extrae el USB con seguridad para no corromper su exFAT, que no tiene diario.

**A revisar en el análisis (UD2):** `crea_user.py` y `crea_user.txt` en el escritorio del Administrador (2017-08-09); IP fija `10.28.0.25`; nombres consultados por LLMNR en el pcap.

**Hora de referencia:** España peninsular (CET, UTC+1). Algunas herramientas registran en UTC (el script de datos volátiles, con sufijo `Z`) o con la hora del equipo investigado (22 s atrasado); se han convertido.

## 6. Conformidad

> Las personas firmantes declaran que el proceso descrito se ha realizado en su presencia y que los datos anteriores son correctos.

| Analista | Testigo | Representante de la parte / fedatario |
|---|---|---|
| Jesús Alfonso Pedreño Domínguez — firma: *(a mano)* | No hubo testigo · no aplica | No hubo fedatario ni representante · no aplica |

---

## Anexo · Cronología de actuaciones

> El enunciado pide «cada paso en el orden en que lo hiciste, empezando por tu marca». La plantilla no
> tiene una tabla para eso, así que va como anexo. Hora de España (CET, UTC+1). Las horas con segundos
> salen de los metadatos de las capturas o de los registros de las herramientas; el resto, del cuaderno.

| Hora | Paso | Actuación y resultado | Captura |
|---|---|---|---|
| 2026-10-30 ~22:30 | Preparación (antes de encender) | Red *bridged* → *host-only*; USB en modo 2.0 | — |
| 10:51:35 | Preparación | Captura de la configuración de la VM, apagada | 00 |
| 10:52:11 | Encendido | *Power on* en VMware (`vmware.log`) | — |
| 10:52:59 | Arranque | *Windows Error Recovery*: arranque normal por defecto | 01 |
| 10:55:29 | Arranque | Pantalla de inicio de sesión | 02 |
| ~11:01 | Inicio de sesión | Administrador; aviso de reinicio por drivers, que no se aplica | 03 |
| 11:06:12 | 0 · Marca | Bloc de notas con `AFI-JesusPedreno-20261031`, sin guardar | 04 |
| 11:10:27 | 0 · Marca | Fichero `marca-jesuspedreno.txt` en el escritorio | 05 |
| 11:12:51 | 1 · Estado inicial | Pantalla; hora del sistema 11:12:29 frente a real 11:12:51 (−22 s) | 06 |
| 11:15:26 | Conexión del USB | USB-RESP montado como `D:` | 07 |
| 11:19:00–11:22:00 | 2 · RAM | WinPmem mini → `FORENSIC_10-ram.raw` | 08 |
| 11:24:40–11:24:51 | 3 · Datos volátiles | `respuesta-en-vivo-ps2` → `vivo/` | 09 |
| 11:31:20 | 4 · Red (inicio) | `vnetsniffer` en el anfitrión, VMnet1 | 10 |
| 11:34:27–11:34:52 | 5 · Triaje | Velociraptor *legacy* → `FORENSIC_10-triaje-velociraptor.zip` | 11 |
| 11:38:41 | 6 · Cifrado | EDD 3.1: sin volúmenes cifrados | 12 |
| 11:44:29 | 7a · Extracción del USB | *Eject*: «Safe To Remove Hardware» | 13 |
| 11:46:32 | 7b · Apagado | *Power Off*, corte de corriente (`vmware.log`: *hard powerOff*) | 14 |
| ~11:49 | 4 · Red (fin) | Ctrl+C en `vnetsniffer`: 34 paquetes | 15 |
| 11:52 | Hashes de la parte 1 | SHA-256 de las evidencias en el equipo del analista | — |
| 12:08:53 | Post-mortem 1 | SHA-256 del `.vmdk`, antes | 16 |
| 12:13–12:18 | Post-mortem 2 | Disco conectado a CAINE como independiente no persistente | 17 |
| 12:30–12:57 | Post-mortem 3 | Arranque de CAINE, identificación de discos (evidencia = `sdb`, solo lectura) y destino en USB-RESP | 18–23 |
| 13:00:32 | Post-mortem 3 | Reloj de CAINE documentado | 24 |
| 13:21:15–13:41:41 | Post-mortem 4 | Guymager: E01 verificado (origen e imagen OK) | 25–31 |
| 13:55:18 | Post-mortem 5 | SHA-256 del `.vmdk`, después: idéntico | 32 |
| 13:58:24 | Retirada del original | Disco de la evidencia desconectado de CAINE | 33 |
| 14:11:47 | Marca en la RAM | 5 apariciones en UTF-16 | 34 |
| 14:21:45 | Marca en el triaje | `marca-jesuspedreno.txt` y `.lnk` en el `$MFT` | 35–36 |
| 14:29–14:48 | Marca en la imagen | E01 montado en solo lectura; fichero y contenido verificados | 37–43 |
| 15:04:48 | Soporte | Copia del pcap a USB-RESP (`red/`), mismo SHA-256; `sha256-usb-evidencias.txt` en el USB | — |
