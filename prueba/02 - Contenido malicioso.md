# Contenido malicioso (EICAR)

> **Fase:** Pruebas **Objetivo:** descargar el archivo de prueba EICAR desde Kali y comprobar si el AntiVirus del FortiGate lo bloquea al pasar de Red A (Kali) a Red B (Debian). **Dirección del tráfico:** Kali sirve el archivo con `python3 -m http.server`; Debian lo descarga. Es tráfico port3 → port2, bajo la política `defensiveTOoffensive (1)`.

## 1. Primer intento (fallido)

Se escribió a mano el contenido del EICAR con `echo` y se sirvió desde Kali. Debian lo descargó con `wget` sin problema y el FortiGate no generó ningún evento de AntiVirus.

**Diagnóstico.** El archivo tecleado a mano no coincidía con la firma exacta del EICAR real: pesaba 70 bytes en vez de 68, y al comparar el texto aparecían diferencias (un carácter y un espacio de más). Los antivirus detectan el EICAR por coincidencia exacta de bytes, así que un archivo retipeado a mano, aunque se vea igual, no se detecta. Se descartó este archivo y se repitió la prueba con el EICAR real.

## 2. Acceso temporal de Kali a internet

Kali no tiene salida a internet por diseño (ver `01-topologia.md` y `02-politicas-base.md`). Para descargar el EICAR real desde `eicar.org` se editó temporalmente la política `OffensivetoWAN` (antes `DefensivetoWAN`), cambiando su origen de `port3 address` a `port2 address`.

## 3. Descarga del EICAR real desde Kali

Con la salida habilitada, se navegó a la página oficial del proyecto EICAR.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/4.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/5.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/6.png" width="600"> </p>

Se eligió el archivo `eicar.com.txt`, de 68 bytes, descargado por HTTPS desde `secure.eicar.org`.

El contenido del archivo real, sin modificar:

```
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
```

## 4. Reglas activas para esta prueba

Antes de repetir la descarga, se revisó el estado de las políticas. `defensiveTOoffensive (1)` (el sentido que usa esta prueba) ya tenía perfil de AntiVirus y de IPS aplicados:

|Política|AntiVirus|IPS|SSL inspection|
|---|---|---|---|
|`defensiveTOoffensive (1)`|`default`|`all_default_pass`|no-inspection|
|`offensiveTOdefensive (2)`|`AV_GENERAL`|`all_default_pass`|custom-deep-inspection|

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/1.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/1.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/2.png" width="600"> </p>


El perfil de IPS `all_default_pass` seguía en modo **Monitor** para todas las firmas, como se dejó en la prueba de reconocimiento.

## 5. Kali sirve el archivo

```bash
python3 -m http.server 80
```

El log del servidor muestra la petición de Debian (a través del firewall, con NAT, por eso aparece como `192.168.10.1`, la dirección de port2) y confirma que el archivo pesa **68 bytes**, igual que el original.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/8.png" width="600"> </p>

## 6. Descarga desde Debian — reacción del firewall

Desde el navegador de Debian se accede al listado de archivos de Kali.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/9.png" width="600"> </p>

Al hacer clic en `eicar.txt`, el FortiGate intercepta la descarga y muestra su página de bloqueo:

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/02/10.png" width="600"> </p>

El mensaje confirma: `You are not permitted to download the file "eicar.txt" because it is infected with the virus "EICAR_TEST_FILE"`, sobre la URL `http://192.168.10.10/eicar.txt`, con cuarentena deshabilitada (`Quarantined File Name: [disabled]`).

| Primer intento (EICAR mal copiado) | Segundo intento (EICAR real)                              |     |
| ---------------------------------- | --------------------------------------------------------- | --- |
| Modo de inspección                 | Proxy-based                                               |     |
| AntiVirus                          | `AV_GENERAL`                                              |     |
| IPS                                | `all_default_pass` (todas las firmas, acción **Monitor**) |     |
| SSL inspection                     | no-inspection                                             |     |

El AntiVirus del FortiGate respeta la regla configurada en `defensiveTOoffensive`: cuando el archivo es el EICAR real, lo bloquea antes de que llegue a Debian. El primer intento no demuestra una falla del firewall, sino un archivo de prueba inválido.
