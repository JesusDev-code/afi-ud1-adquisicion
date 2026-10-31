# Validación de las herramientas y desviaciones respecto al enunciado

**Actividad:** AFI · UD1 · Adquirir un equipo comprometido
**Fecha de la validación:** 2026-10-30 (horas en CET, UTC+1)
**Analista:** Jesús Pedreño

Este documento explica por qué la adquisición se hizo con algunas herramientas distintas de las
que nombra el enunciado, y con qué pruebas se tomó cada decisión. Todas las pruebas se hicieron
sobre una **copia de la máquina investigada**, que después se descartó. La adquisición evaluable
se hizo sobre una importación nueva y limpia de la OVA original.

---

## 1. Resumen

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

## 2. Entorno de las pruebas

| Elemento | Detalle |
|---|---|
| Equipo del analista | Portátil con Windows 11, VMware Workstation (`vmrun` 1.17.0) |
| Máquina investigada | `FORENSIC_10.ova`, SHA-256 `DAF0EF5255D98276A6912A53611DB5C0CBF2CCCBB49A180DD7FCC0F95E14930C` |
| Datos del sistema | Hostname `FORENSE-06` · Windows 7 Professional SP1 6.1.7601 · 1 GB RAM · disco de 32 GB · teclado inglés (EE. UU.) |
| Copia de pruebas | Primera importación de la OVA (descartada al terminar) |
| Máquina de referencia | VM Windows 11 propia, para validar las herramientas en un sistema moderno |
| USB de respuesta | Pendrive PHILIPS de 32 GB, exFAT, etiqueta `USB-RESP` |
| Red de la VM | Cambiada de *bridged* (como venía en la OVA) a *host-only* antes de arrancarla. Captura `10` |

---

## 3. Cronología

| Hora | Qué se hizo | Resultado |
|---|---|---|
| 15:18 | SHA-256 de la OVA recibida | `DAF0EF52…14930C` |
| 15:28 | Configuración de la VM: red *host-only* y USB 2.0 | — |
| 15:53 | Prueba de `vnetsniffer` en la VM Windows 11 | Válido: 68 paquetes, tráfico en los dos sentidos |
| 16:00–16:41 | DumpIt y KAPE en la VM Windows 11 | Válidos en Windows 11 |
| 17:03 | Inicio del primer intento sobre FORENSIC_10 | Descartado después (ver apartado 5) |
| 17:58 | `$PSVersionTable` en FORENSIC_10 | PowerShell 2.0. Captura `08` |
| 18:05 | DumpIt 3.6 en FORENSIC_10 | **Falla:** `0xC0000428`. Captura `09` |
| 18:10 | Decisión: el primer intento se descarta y la máquina se usa como copia de pruebas | — |
| 18:32 | WinPmem mini en FORENSIC_10 | **Válido** |
| 18:46 | `respuesta-en-vivo.ps1` original | **Falla:** `ParserError`. Captura `11` |
| 19:00 | Adaptación a PowerShell 2.0 | **Válida** |
| 19:01 | KAPE 1.3.0.2 | **Falla:** `NotSupportedException`. Captura `12` |
| 19:42 | Velociraptor *legacy* sin UCRT | **Falla:** falta `api-ms-win-crt-stdio-l1-1-0.dll`. Captura `13` |
| 20:10 | Velociraptor *legacy* con UCRT local | **Válido** |
| 20:16 | EDD 3.1 | **Válido** |
| 20:26 | 21 herramientas de consola de Sysinternals | 19 válidas, 2 fallan. Capturas `14`–`16` |
| 20:4x | SHA-256 de la OVA antes de reimportarla | `DAF0EF52…14930C`, sin cambios |

---

## 4. Detalle por herramienta

### 4.1 Memoria RAM: DumpIt → WinPmem mini

- **Pedido:** DumpIt.
- **Probado:** Magnet DumpIt 3.6.20230117 (x64), con los mismos parámetros validados en Windows
  11 (`/QUIET /NOLYTICS /NOCOMPRESS /TYPE DMP`).
- **Resultado en Windows 7:** `Error: Load driver failed with 0xC0000428 error code`. El código
  0xC0000428 significa que Windows no puede verificar la firma digital del fichero. El driver de
  DumpIt 3.6 está firmado con SHA-2 y un Windows 7 sin la actualización de soporte SHA-2 no sabe
  verificarlo. DumpIt intentó registrar el servicio `HKLM\System\CurrentControlSet\Services\DumpIt`
  (con `ImagePath = C:\Windows\system32\Drivers\DumpIt.sys`) y lo borró al fallar. Captura `09`.
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

### 4.2 Datos volátiles: script del módulo → adaptación a PowerShell 2.0

- **Pedido:** `respuesta-en-vivo.ps1`.
- **Resultado en Windows 7:** `ParserError` en la línea 88 (`You must provide a value expression
  on the right-hand side of the '-' operator`): el operador `-notin` no existe en PowerShell 2.0.
  Al ser un error de sintaxis, **el script no llega a ejecutarse** y no recoge nada. Captura `11`.
- **Otros comandos incompatibles:** `Get-CimInstance`, `Get-NetTCPConnection`,
  `Get-NetUDPEndpoint`, `Get-SmbShare`, `Get-LocalUser`, `Get-Disk`, `Get-PnpDevice`,
  `Get-Clipboard`, `Get-TimeZone`, `Get-FileHash` y `$PSScriptRoot`.
- **Solución:** `respuesta-en-vivo-ps2.ps1`, una adaptación con **los mismos 19 pasos, en el mismo
  orden y con los mismos nombres de fichero**, que cambia cada comando por su equivalente en
  Windows 7 (`Get-WmiObject`, `netstat -ano`, `net user`, `net share`…). Mantiene la autoría y la
  licencia CC BY-SA 4.0 del original. El script original no se ha modificado.
- **Prueba en Windows 7:** 19 ficheros sin errores en 12 segundos. Los 21 hashes de su
  `hashes-sha256.csv` coinciden al recalcularlos en el equipo del analista.

### 4.3 Red: `nictrace` de VirtualBox → `vnetsniffer` de VMware

- **Pedido:** `VBoxManage controlvm "FORENSIC_10" nictracefile1 …` (VirtualBox).
- **Solución:** `vnetsniffer.exe /w <fichero>.pcap VMnet1`, de la instalación de VMware
  Workstation, ejecutado como administrador **en el equipo del analista**. Escucha el switch
  virtual *host-only* desde fuera de la máquina, sin instalar nada en ella, igual que el
  `nictrace` del enunciado.
- **Prueba:** sobre la VM Windows 11, formato libpcap clásico (cabecera `d4c3b2a1`, Ethernet),
  68 paquetes con peticiones y respuestas en los dos sentidos.

### 4.4 Triaje: KAPE → Velociraptor *legacy*

- **Pedido:** KAPE o un recolector offline de Velociraptor.
- **KAPE 1.3.0.2 en Windows 7:** `Unhandled Exception: System.NotSupportedException: The requested
  security protocol is not supported` en `ServicePointManager.set_SecurityProtocol`. KAPE está
  compilado para **.NET Framework 4.5.2** y el equipo solo tiene .NET 4.0. Captura `12`.
- **Velociraptor 0.77.3 normal:** compilado con Go 1.26/1.27. Go no funciona en Windows 7 desde
  la versión 1.21.
- **Solución:** la compilación oficial **`velociraptor-v0.77.3-windows-amd64-legacy.exe`**,
  publicada por el proyecto como *"For Windows 7 Only"* (compilada con Go 1.20). Firma Authenticode
  válida de **Rapid7 LLC**. SHA-256 del ejecutable:
  `7b699a6670e0caaf8ce951225dff479c141580a9fe8d9d0a1a7c5e83d32e1698`.
- **Segundo problema:** sin el *Universal C Runtime* no arranca (`api-ms-win-crt-stdio-l1-1-0.dll
  is missing`). Captura `13`. Se resolvió con el **despliegue local del UCRT** que permite
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

### 4.5 Cifrado: EDD

- **Magnet Encrypted Disk Detector 3.1** necesita .NET 4.0, que el equipo tiene. **Válido** en
  Windows 7: revisa los 2 discos y los 2 volúmenes y no encuentra cifrado. `certutil -hashfile …
  SHA256` también funciona en Windows 7.
- La adaptación de la respuesta en vivo incluye además `manage-bde -status` (BitLocker).

### 4.6 Otras herramientas del kit

- **Sysinternals:** de 21 herramientas de consola, 19 funcionan en Windows 7. Fallan
  `autorunsc64` (usa `CryptCATAdminAcquireContext2` de `WINTRUST.dll`, de Windows 8+) y
  `procdump64` (le falta `api-ms-win-core-path-l1-1-0.dll`). `pslist64` ignora `-accepteula` y
  muestra la ventana de licencia. Capturas `14`–`16` y `prueba-compatibilidad-win7-FORENSE-06.txt`.
- **FTK Imager 8.3:** no abre en Windows 7. No afecta a la actividad: se usa en el equipo del
  analista, y la imagen de disco de la parte 2 se hace con CAINE y Guymager.

---

## 5. El primer intento y la reimportación

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

## 6. Interacciones con el equipo que se anotan en el acta

Además de la marca, estas acciones tocan el equipo investigado y se registran con su hora:

- Inicio de sesión como `Administrador` (cuenta con SID …-1000; la cuenta integrada
  `Administrator` está deshabilitada), necesario para trabajar. Deja eventos de inicio de sesión.
- Conexión del USB de respuesta: Windows instala el driver y registra el dispositivo.
- WinPmem: driver temporal en `C:\Users\<usuario>\AppData\Local\Temp\pme*.tmp`, cargado y borrado.
- Cada herramienta ejecutada crea un proceso y ocupa memoria.

---

## 7. Lecciones

- **Una herramienta solo está validada para el sistema en el que se ha probado.** Validar en
  Windows 11 no dice nada de Windows 7. Antes de tocar el equipo hay que saber qué sistema tiene.
- **La memoria no admite un segundo intento.** Por eso las herramientas se validan antes, en una
  copia del mismo sistema.
- El kit del USB se reorganizó en `windows\moderno\` (Windows 10/11) y `windows\win7\`
  (Windows 7/8), con una tabla `COMPATIBILIDAD.txt` que dice qué herramienta se ha validado en qué
  sistema, cuándo y con qué resultado.

---

## 8. Ficheros de este directorio

| Fichero | Qué es |
|---|---|
| `COMPATIBILIDAD.txt` | Tabla herramienta × sistema con el resultado de cada prueba |
| `prueba-compatibilidad-win7-FORENSE-06.txt` | Resultado de la prueba de Sysinternals en Windows 7 |
| `capturas/08`–`16` | Capturas de cada error y de la configuración original de la VM |

Los hashes de todas las herramientas del USB están en `USB-RESPUESTA\hashes-herramientas.txt`.

---

## 9. Fuentes

- Velociraptor, descargas y compilaciones *legacy*: <https://docs.velociraptor.app/downloads/>
- Velociraptor, definición de `Windows.Search.FileFinder` (v0.77.3):
  <https://github.com/Velocidex/velociraptor/blob/v0.77.3/artifacts/definitions/Windows/Search/FileFinder.yaml>
- Velociraptor, proyecto de triaje (`Windows.Triage.Targets`):
  <https://github.com/Velocidex/velociraptor-triage-collector>
- Microsoft Update Catalog, KB2999226: <https://www.catalog.update.microsoft.com/Search.aspx?q=KB2999226>
- Magnet RAM Capture / DumpIt: <https://magnetforensics.com/free-tool-magnet-ram-capture>
