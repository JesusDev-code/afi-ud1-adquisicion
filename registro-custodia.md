# Registros de cadena de custodia — BORRADOR

> Borrador en Markdown que sigue la plantilla `registro-cadena-custodia.docx`: **un registro por evidencia**
> (EV-01 a EV-06). En el PDF final, cada uno va en su propia hoja.
>
> **Regla de la plantilla:** «cada entrega se anota en el momento, nunca después». Las filas de la entrega
> al compañero y de la devolución **se rellenan a mano en el momento**, con las firmas de los dos.
> Las seis evidencias viajan juntas en el USB-RESP, así que **cada entrega se anota en los seis registros**.
>
> Hora de España peninsular (CET, UTC+1). Todos los hashes son SHA-256 y están también en
> `hashes/sha256-usb-evidencias.txt`.

---

## Registro EV-01 · Memoria RAM

### Evidencia

| Campo | Valor |
|---|---|
| Número de caso y ID de la evidencia | AFI-UD1-ALMADRABA · EV-01 |
| Descripción (marca, modelo, n.º de serie, capacidad) | Volcado de la memoria RAM de FORENSE-06, 1 GiB. Fichero `memoria/FORENSIC_10-ram.raw` (formato RAW, 1 073 741 824 B), con sus registros `adquisicion-winpmem.log` y `winpmem-salida.txt`. Soporte: pendrive PHILIPS USB-RESP, n.º de serie `3798771057290437452`, 29,3 GiB |
| Origen (equipo, ubicación y usuario) | FORENSIC_10 (hostname FORENSE-06), ordenador del cuarto de informática de Almadraba Diseño; máquina virtual en el equipo del analista. Sesión de Administrador |
| Adquirida por, fecha y hora | Jesús Alfonso Pedreño Domínguez, 2026-10-31, de 11:19:00 a 11:22:00, con WinPmem mini (`winpmem_mini_x64_rc2.exe`) |
| SHA-256 de la evidencia | `1f440ce6547b7b88640852b588a50cb55212080674591d65199149e2d05e2b2a` |
| N.º de precinto inicial | No aplica: sin precinto físico. La integridad se controla por hash |

### Transferencias y accesos

| Fecha y hora | Entrega (nombre y firma) | Recibe (nombre y firma) | Motivo | Estado del precinto | N.º precinto nuevo | Hash verificado (sí/no) |
|---|---|---|---|---|---|---|
| 2026-10-31 11:52 | Jesús A. Pedreño (USB extraído del equipo investigado) | Jesús A. Pedreño (equipo del analista) | Cálculo del hash | No aplica | No aplica | Sí, calculado por primera vez |
| 2026-10-31 ~12:35 | Jesús A. Pedreño (equipo del analista) | Jesús A. Pedreño (estación CAINE) | Acceso de solo lectura para buscar la marca (`strings`) | No aplica | No aplica | No (se comprueba a la vuelta) |
| 2026-10-31 ~14:55 | Jesús A. Pedreño (estación CAINE) | Jesús A. Pedreño (equipo del analista) | Fin del trabajo en CAINE; comprobación de integridad | No aplica | No aplica | Sí, coincide |
| *(a mano, en el momento)* | Jesús A. Pedreño · *firma* | *Compañero/a* · *firma* | Comprobación por un tercero | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | *Compañero/a* · *firma* | Jesús A. Pedreño · *firma* | Devolución | No aplica | No aplica | *Resultado de `sha256sum -c`* |

### Destino final

| Campo | Valor |
|---|---|
| Devolución o destrucción: cómo, cuándo, dónde y por quién | Pendiente. Queda bajo custodia del analista en USB-RESP para su análisis en la UD2 |

---

## Registro EV-02 · Datos volátiles

### Evidencia

| Campo | Valor |
|---|---|
| Número de caso y ID de la evidencia | AFI-UD1-ALMADRABA · EV-02 |
| Descripción (marca, modelo, n.º de serie, capacidad) | Salida del script de respuesta en vivo: carpeta `vivo/AFI-UD1-ALMADRABA_FORENSIC_10_20261031-102418/`, con 22 ficheros de texto (registro, fecha y hora, sesiones, conexiones, puertos, ARP, DNS, rutas, red, recursos compartidos, procesos, servicios, tareas, inicio automático, usuarios, administradores, discos, USB, portapapeles) y `hashes-sha256.csv`. Soporte: USB-RESP (ver EV-01) |
| Origen (equipo, ubicación y usuario) | FORENSIC_10 (hostname FORENSE-06); sesión de Administrador |
| Adquirida por, fecha y hora | Jesús Alfonso Pedreño Domínguez, 2026-10-31, de 11:24:40 a 11:24:51, con `respuesta-en-vivo-ps2.ps1` (adaptación a PowerShell 2.0 del script del módulo) |
| SHA-256 de la evidencia | Uno por fichero, en `hashes/sha256-usb-evidencias.txt` (líneas `vivo/…`). Registro de la ejecución, `00-registro.txt`: `f6650bc89fa010df3498f9b248f61aeb0a09dbc04aecf3de45fadc7d2484ee07` |
| N.º de precinto inicial | No aplica: sin precinto físico. La integridad se controla por hash |

### Transferencias y accesos

| Fecha y hora | Entrega (nombre y firma) | Recibe (nombre y firma) | Motivo | Estado del precinto | N.º precinto nuevo | Hash verificado (sí/no) |
|---|---|---|---|---|---|---|
| 2026-10-31 11:52 | Jesús A. Pedreño (USB extraído del equipo investigado) | Jesús A. Pedreño (equipo del analista) | Cálculo del hash | No aplica | No aplica | Sí, calculado por primera vez |
| 2026-10-31 ~12:35 | Jesús A. Pedreño (equipo del analista) | Jesús A. Pedreño (estación CAINE) | El USB se monta en CAINE como destino de EV-06; esta evidencia no se abre | No aplica | No aplica | No (se comprueba a la vuelta) |
| 2026-10-31 ~14:55 | Jesús A. Pedreño (estación CAINE) | Jesús A. Pedreño (equipo del analista) | Fin del trabajo en CAINE; comprobación de integridad | No aplica | No aplica | Sí, `00-registro.txt` coincide |
| *(a mano, en el momento)* | Jesús A. Pedreño · *firma* | *Compañero/a* · *firma* | Comprobación por un tercero | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | *Compañero/a* · *firma* | Jesús A. Pedreño · *firma* | Devolución | No aplica | No aplica | *Resultado de `sha256sum -c`* |

### Destino final

| Campo | Valor |
|---|---|
| Devolución o destrucción: cómo, cuándo, dónde y por quién | Pendiente. Queda bajo custodia del analista en USB-RESP para su análisis en la UD2 |

---

## Registro EV-03 · Tráfico de red

### Evidencia

| Campo | Valor |
|---|---|
| Número de caso y ID de la evidencia | AFI-UD1-ALMADRABA · EV-03 |
| Descripción (marca, modelo, n.º de serie, capacidad) | Captura del tráfico de la interfaz del equipo en el switch virtual VMnet1: `red/FORENSIC_10-red.pcap` (libpcap, 5 104 B, 34 paquetes). Soporte: USB-RESP (ver EV-01) |
| Origen (equipo, ubicación y usuario) | Red *host-only* de FORENSIC_10 (FORENSE-06, IP `10.28.0.25`), capturada desde fuera del equipo, en el equipo del analista |
| Adquirida por, fecha y hora | Jesús Alfonso Pedreño Domínguez, 2026-10-31, de 11:31:20 a ~11:49, con `vnetsniffer` (VMware Workstation) |
| SHA-256 de la evidencia | `0ab1fda310ab5e8049880106f32dec32c5b485e62f87c0aca32131f06127676c` |
| N.º de precinto inicial | No aplica: sin precinto físico. La integridad se controla por hash |

### Transferencias y accesos

| Fecha y hora | Entrega (nombre y firma) | Recibe (nombre y firma) | Motivo | Estado del precinto | N.º precinto nuevo | Hash verificado (sí/no) |
|---|---|---|---|---|---|---|
| 2026-10-31 ~11:49 | Jesús A. Pedreño (`vnetsniffer`) | Jesús A. Pedreño (equipo del analista, `afi-ud1-adquisicion/red/`) | Fin de la captura; la evidencia se genera directamente en el equipo del analista | No aplica | No aplica | Sí, calculado por primera vez |
| 2026-10-31 15:04 | Jesús A. Pedreño (equipo del analista) | Jesús A. Pedreño (USB-RESP, `red/`) | Se copia al soporte para que viaje con el resto de evidencias | No aplica | No aplica | Sí, la copia coincide con el original |
| *(a mano, en el momento)* | Jesús A. Pedreño · *firma* | *Compañero/a* · *firma* | Comprobación por un tercero | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | *Compañero/a* · *firma* | Jesús A. Pedreño · *firma* | Devolución | No aplica | No aplica | *Resultado de `sha256sum -c`* |

### Destino final

| Campo | Valor |
|---|---|
| Devolución o destrucción: cómo, cuándo, dónde y por quién | Pendiente. Queda bajo custodia del analista en USB-RESP para su análisis en la UD2 |

---

## Registro EV-04 · Triaje

### Evidencia

| Campo | Valor |
|---|---|
| Número de caso y ID de la evidencia | AFI-UD1-ALMADRABA · EV-04 |
| Descripción (marca, modelo, n.º de serie, capacidad) | Triaje dirigido: `triaje/FORENSIC_10-triaje-velociraptor.zip` (17 931 246 B, 100 ficheros: `$MFT`, colmenas SAM, SYSTEM, SOFTWARE, SECURITY y DEFAULT, `NTUSER.DAT` de cada usuario y registros `.evtx`), con sus registros `adquisicion-velociraptor.log` y `velociraptor-salida.txt`. Soporte: USB-RESP (ver EV-01) |
| Origen (equipo, ubicación y usuario) | Disco `C:` de FORENSIC_10 (FORENSE-06), leído en bruto (NTFS) con el equipo encendido; sesión de Administrador |
| Adquirida por, fecha y hora | Jesús Alfonso Pedreño Domínguez, 2026-10-31, de 11:34:27 a 11:34:52, con Velociraptor 0.77.3 *legacy* (artefacto `Windows.Search.FileFinder`, accessor `ntfs`) |
| SHA-256 de la evidencia | `4e3515ef066ea32f7ebd22c91eb665a65ac5206382ecfe1ae77e6be4e085eaf8` |
| N.º de precinto inicial | No aplica: sin precinto físico. La integridad se controla por hash |

### Transferencias y accesos

| Fecha y hora | Entrega (nombre y firma) | Recibe (nombre y firma) | Motivo | Estado del precinto | N.º precinto nuevo | Hash verificado (sí/no) |
|---|---|---|---|---|---|---|
| 2026-10-31 11:52 | Jesús A. Pedreño (USB extraído del equipo investigado) | Jesús A. Pedreño (equipo del analista) | Cálculo del hash | No aplica | No aplica | Sí, calculado por primera vez |
| 2026-10-31 ~12:35 | Jesús A. Pedreño (equipo del analista) | Jesús A. Pedreño (estación CAINE) | Acceso de solo lectura: se extrae el `$MFT` a `/tmp` para buscar la marca | No aplica | No aplica | No (se comprueba a la vuelta) |
| 2026-10-31 ~14:55 | Jesús A. Pedreño (estación CAINE) | Jesús A. Pedreño (equipo del analista) | Fin del trabajo en CAINE; comprobación de integridad | No aplica | No aplica | Sí, coincide |
| *(a mano, en el momento)* | Jesús A. Pedreño · *firma* | *Compañero/a* · *firma* | Comprobación por un tercero | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | *Compañero/a* · *firma* | Jesús A. Pedreño · *firma* | Devolución | No aplica | No aplica | *Resultado de `sha256sum -c`* |

### Destino final

| Campo | Valor |
|---|---|
| Devolución o destrucción: cómo, cuándo, dónde y por quién | Pendiente. Queda bajo custodia del analista en USB-RESP para su análisis en la UD2 |

---

## Registro EV-05 · Comprobación de cifrado

### Evidencia

| Campo | Valor |
|---|---|
| Número de caso y ID de la evidencia | AFI-UD1-ALMADRABA · EV-05 |
| Descripción (marca, modelo, n.º de serie, capacidad) | Salida de EDD: `cifrado/2026-10-31_113819/edd-salida.txt` (1 187 B), con `edd-salida.txt.sha256` y `adquisicion.log`. Resultado: sin volúmenes cifrados. Soporte: USB-RESP (ver EV-01) |
| Origen (equipo, ubicación y usuario) | Discos y volúmenes de FORENSIC_10 (FORENSE-06), con el equipo encendido; sesión de Administrador |
| Adquirida por, fecha y hora | Jesús Alfonso Pedreño Domínguez, 2026-10-31, 11:38:41, con EDD 3.1 (`EDDv310.exe /batch`) |
| SHA-256 de la evidencia | `7fadb304b124b7306517d785fc050ca74f7967d945f1fe5c575dd51a99e39aec` (coincide con el calculado con `certutil` en el propio equipo) |
| N.º de precinto inicial | No aplica: sin precinto físico. La integridad se controla por hash |

### Transferencias y accesos

| Fecha y hora | Entrega (nombre y firma) | Recibe (nombre y firma) | Motivo | Estado del precinto | N.º precinto nuevo | Hash verificado (sí/no) |
|---|---|---|---|---|---|---|
| 2026-10-31 11:52 | Jesús A. Pedreño (USB extraído del equipo investigado) | Jesús A. Pedreño (equipo del analista) | Cálculo del hash | No aplica | No aplica | Sí, coincide con el de `certutil` |
| 2026-10-31 ~12:35 | Jesús A. Pedreño (equipo del analista) | Jesús A. Pedreño (estación CAINE) | El USB se monta en CAINE como destino de EV-06; esta evidencia no se abre | No aplica | No aplica | No (se comprueba a la vuelta) |
| 2026-10-31 ~14:55 | Jesús A. Pedreño (estación CAINE) | Jesús A. Pedreño (equipo del analista) | Fin del trabajo en CAINE; comprobación de integridad | No aplica | No aplica | Sí, coincide |
| *(a mano, en el momento)* | Jesús A. Pedreño · *firma* | *Compañero/a* · *firma* | Comprobación por un tercero | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | *Compañero/a* · *firma* | Jesús A. Pedreño · *firma* | Devolución | No aplica | No aplica | *Resultado de `sha256sum -c`* |

### Destino final

| Campo | Valor |
|---|---|
| Devolución o destrucción: cómo, cuándo, dónde y por quién | Pendiente. Queda bajo custodia del analista en USB-RESP |

---

## Registro EV-06 · Imagen del disco (maestra)

### Evidencia

| Campo | Valor |
|---|---|
| Número de caso y ID de la evidencia | AFI-UD1-ALMADRABA · EV-06 |
| Descripción (marca, modelo, n.º de serie, capacidad) | Imagen forense del disco de FORENSIC_10 (32 GiB = 34,4 GB; tabla DOS con NTFS *System Reserved* + NTFS del sistema). Formato E01 de Guymager, en tres trozos: `imagen/FORENSIC_10.E01` (2 146 429 770 B), `.E02` (2 146 428 283 B) y `.E03` (987 364 035 B), más el registro `FORENSIC_10.info`. **Imagen maestra**: no se trabaja sobre ella. Soporte: USB-RESP (ver EV-01) |
| Origen (equipo, ubicación y usuario) | Fichero del disco virtual `FORENSIC_10-disk1.vmdk`, conectado a la estación CAINE en modo independiente no persistente (equivalente a «Inmutable») |
| Adquirida por, fecha y hora | Jesús Alfonso Pedreño Domínguez, 2026-10-31, de 13:21:15 a 13:41:41, con Guymager 0.8.13 en CAINE 14 (live). Verificación del origen y de la imagen: OK |
| SHA-256 de la evidencia | Contenido del disco (adquisición = origen releído = imagen): `28f828d2aa46a171cd46c44424c9ce1f7bd37284f5f05d2ca8788187cb19d20a`. Ficheros: `.E01` `cc504abd2282c213873c23c69904daec731eda7918b1c422cb2f20e2b5802edb` · `.E02` `66ec79161dd06246d6793ecf2f6b985a574d7c6efe12eea14600585371cee471` · `.E03` `7a9a58febeda1c9177b185d7492639ed52f1d1389d15bb64a7b971913ea2cc27` · `.info` `dfbf5262be4a0d853346f5cf5138d3cf5636f3c2aa5126455f741760b16192e1`. El `.vmdk` original da `173a4388…f26df` antes y después |
| N.º de precinto inicial | No aplica: sin precinto físico. La integridad se controla por hash |

### Transferencias y accesos

| Fecha y hora | Entrega (nombre y firma) | Recibe (nombre y firma) | Motivo | Estado del precinto | N.º precinto nuevo | Hash verificado (sí/no) |
|---|---|---|---|---|---|---|
| 2026-10-31 14:27–14:48 | Jesús A. Pedreño (estación CAINE) | Jesús A. Pedreño (estación CAINE) | Acceso de solo lectura: `ewfmount` y montaje `ro` para buscar la marca | No aplica | No aplica | Ya verificada por Guymager |
| 2026-10-31 ~14:55 | Jesús A. Pedreño (estación CAINE) | Jesús A. Pedreño (equipo del analista) | Fin del trabajo en CAINE; hash de cada fichero `.E0x` | No aplica | No aplica | Sí, calculado por primera vez |
| *(a mano, en el momento)* | Jesús A. Pedreño (maestra) | Jesús A. Pedreño (copia de trabajo, equipo del analista) | Copia de trabajo para la UD2 | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | Jesús A. Pedreño · *firma* | *Compañero/a* · *firma* | Comprobación por un tercero | No aplica | No aplica | *Resultado de `sha256sum -c`* |
| *(a mano, en el momento)* | *Compañero/a* · *firma* | Jesús A. Pedreño · *firma* | Devolución | No aplica | No aplica | *Resultado de `sha256sum -c`* |

### Destino final

| Campo | Valor |
|---|---|
| Devolución o destrucción: cómo, cuándo, dónde y por quién | Pendiente. La maestra queda bajo custodia del analista en USB-RESP; el análisis de la UD2 se hace sobre la copia de trabajo |
