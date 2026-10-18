# FortiGate-VM (NGFW)

> **Fase:** 0 (Entorno)
> **Máquina:** Firewall FortiGate-VM
> **Estado de red en este documento:** provisional (interfaces por defecto del OVF, todas en modo Bridge).
> La segmentación definitiva se documenta en `01-topologia.md`.

## 1. Propósito de la máquina

El FortiGate-VM es el elemento central del laboratorio. Se ubica entre
la Red A (Kali Linux, atacante) y la Red B (Debian, víctima), enruta el
tráfico entre ambas y aplica las políticas de seguridad que se
configuran en las fases siguientes. Las pruebas de `pruebas/` miden si
este firewall respeta esas políticas.

## 2. Características de la máquina virtual

| Parámetro                  | Valor                                               |
| -------------------------- | --------------------------------------------------- |
| Hipervisor                 | VMware Workstation Pro 26H1                         |
| Producto                   | FortiGate-VM64                                      |
| Versión de FortiOS         | FortiGate-VM64 v8.0.1, build0245, 260909 (GA.F)     |
| Hostname                   | FGT-VM64-Y4ERR6 (asignado por defecto)              |
| RAM                        | 2 GB (valor por defecto del OVF)                    |
| Procesadores               | 1 (valor por defecto del OVF)                       |
| Disco 1 (SCSI)             | 2 GB                                                |
| Disco 2 (SCSI)             | 30 GB                                               |
| Interfaces (provisional)   | 10 adaptadores, todos en Bridge (valor por defecto) |
| Compatibilidad de hardware | Workstation 17.x                                    |

## 3. Descarga de la imagen

La imagen se descargó desde el portal de soporte de Fortinet
(FortiCare), sección **Downloads → VM Images**, con el producto
**FortiGate** y la plataforma **VMware ESXi**. 

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/fortinet_platform.png" width="600"> </p>

El archivo descargado es `FGT_VM64-v8.0.1.F-build0245-FORTINET.out.ovf.zip`
(despliegue nuevo, no actualización de una versión anterior).

> En la misma página aparece también FortiFirewall (`FFW_VM64`). Se
> descargó la imagen de FortiGate (`FGT_VM64`).

## 4. Extracción del paquete

El ZIP se extrae en una carpeta local de descargas.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/extra_forti.png" width="600"> </p>

El contenido incluye varias plantillas (`.hw15`, `.hw17`, `.nsxt`,
`.vapp`, `-ZTNA.vapp`). Se utilizó `FortiGate-VM64`.

## 5. Importación en VMware Workstation

Desde VMware Workstation: **File → Open**, y se selecciona la plantilla
`FortiGate-VM64`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/1.png" width="600"> </p>

El asistente de importación pide aceptar el contrato de licencia
(EULA) de Fortinet.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/2.png" width="600"> </p>

## 6. Configuración resultante de la VM

Tras la importación, la VM queda apagada con la configuración por
defecto de la plantilla.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/3.png" width="600"> </p>

Observacion:

- Los 10 adaptadores quedan en modo Bridge. Esto se reemplaza por la
  segmentación definida en `01-topologia.md`.

## 7. Primer arranque

Al encender la VM, el sistema carga la imagen y prepara las particiones
en el primer inicio.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/4.png" width="600"> </p>

Al terminar el arranque, la consola muestra el número de serie y el
prompt de login. El hostname por defecto es `FGT-VM64-Y4ERR6`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/5.png" width="600"> </p>

## 8. Primer acceso y verificación

###  Login y cambio obligatorio de contraseña

El primer acceso se hace por la consola de VMware con el usuario
`admin`. 

El sistema obliga a cambiar la contraseña en el primer login y aplica
una política de contraseñas con estos requisitos:

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/fortigate/6.png" width="600"> </p>
