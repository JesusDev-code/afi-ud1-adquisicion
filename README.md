# Adquirir un equipo comprometido — AFI UD1

**Alumno:** Jesús Alfonso Pedreño Domínguez
**Módulo:** Análisis forense informático · Curso de Especialización en Ciberseguridad · IES Rafael Alberti
**Caso:** AFI-UD1-ALMADRABA · equipo FORENSIC_10 (hostname FORENSE-06), de Almadraba Diseño
**Fecha de la adquisición:** 2026-10-31

## Contenido

| Documento | Qué contiene |
|---|---|
| [`acta-adquisicion.pdf`](acta-adquisicion.pdf) | Acta firmada: cada paso en el orden en que se hizo, empezando por la marca, con herramienta, fichero y hash |
| [`registro-custodia.pdf`](registro-custodia.pdf) | Un registro de custodia por evidencia (EV-01 a EV-06), con todas las transferencias, incluidas la entrega al compañero y la devolución, firmadas |
| [`hashes/`](hashes/) | SHA-256 de todas las evidencias y registros de las herramientas (el `.info` de Guymager y los de WinPmem, Velociraptor, EDD y el script de datos volátiles) |
| [`capturas/`](capturas/) | Capturas 00 a 43, numeradas en orden |
| [`cuaderno-campo.md`](cuaderno-campo.md) | Notas tomadas durante la adquisición, con todas las horas y decisiones |
| [`validacion-herramientas/`](validacion-herramientas/README.md) | Por qué se cambiaron algunas herramientas del enunciado y cómo se probó cada una en Windows 7 |

## Evidencias

Todas están en el soporte USB-RESP, en `casos/AFI-UD1-ALMADRABA/FORENSIC_10/`. **No se suben al
repositorio** porque ocupan gigas: se suben sus hashes.

| ID | Evidencia | Herramienta | SHA-256 |
|---|---|---|---|
| EV-01 | Memoria RAM (1 GiB) | WinPmem mini | `1f440ce6547b7b88640852b588a50cb55212080674591d65199149e2d05e2b2a` |
| EV-02 | Datos volátiles (19 secciones) | `respuesta-en-vivo-ps2.ps1` | ver [`hashes/sha256-usb-evidencias.txt`](hashes/sha256-usb-evidencias.txt) |
| EV-03 | Tráfico de red (34 paquetes) | `vnetsniffer` (VMware) | `0ab1fda310ab5e8049880106f32dec32c5b485e62f87c0aca32131f06127676c` |
| EV-04 | Triaje (`$MFT`, registro, `.evtx`) | Velociraptor 0.77.3 *legacy* | `4e3515ef066ea32f7ebd22c91eb665a65ac5206382ecfe1ae77e6be4e085eaf8` |
| EV-05 | Comprobación de cifrado | EDD 3.1 | `7fadb304b124b7306517d785fc050ca74f7967d945f1fe5c575dd51a99e39aec` |
| EV-06 | Imagen E01 del disco (32 GiB) | Guymager 0.8.13 en CAINE 14 | contenido: `28f828d2aa46a171cd46c44424c9ce1f7bd37284f5f05d2ca8788187cb19d20a` |

El fichero del disco original (`FORENSIC_10-disk1.vmdk`) da el mismo SHA-256 antes y después de la
adquisición post-mortem: `173a4388c7ad6c9bd6f5f4861c771eee3138463abfbcd5d5e962fdddf85f26df`.

## Capturas que pide el enunciado

| Qué | Captura |
|---|---|
| Pantalla inicial | [`06-estado-inicial.png`](capturas/06-estado-inicial.png) |
| Marca encontrada en la RAM | [`34-marca-ram.png`](capturas/34-marca-ram.png) |
| Marca encontrada en el triaje | [`36-marca-triaje.png`](capturas/36-marca-triaje.png) |
| Marca encontrada en la imagen | [`41-marca-imagen-ls.png`](capturas/41-marca-imagen-ls.png) y [`43-marca-imagen-od.png`](capturas/43-marca-imagen-od.png) |
| Verificación final de la imagen | [`29-guymager-fin.png`](capturas/29-guymager-fin.png), [`30-imagen-info-1.png`](capturas/30-imagen-info-1.png) y [`31-imagen-info-2.png`](capturas/31-imagen-info-2.png) |

## Cambios respecto al enunciado

El trabajo se hizo en **VMware Workstation**, no en VirtualBox. Además, varias herramientas del
enunciado no funcionan en este Windows 7 sin parches. Se sustituyeron por otras equivalentes,
probadas antes en el mismo sistema:

| Enunciado | Usado | Motivo |
|---|---|---|
| DumpIt | WinPmem mini | El driver de DumpIt 3.6 no carga (`0xC0000428`) |
| `respuesta-en-vivo.ps1` | `respuesta-en-vivo-ps2.ps1` | El equipo tiene PowerShell 2.0 |
| KAPE | Velociraptor *legacy* | KAPE necesita .NET 4.5.2 |
| `nictrace` (VirtualBox) | `vnetsniffer` (VMware) | Captura igual desde fuera del equipo |
| Disco «Inmutable» | Disco independiente no persistente | Equivalente en VMware; el hash del `.vmdk` lo confirma |

El detalle está en [`validacion-herramientas/README.md`](validacion-herramientas/README.md).
