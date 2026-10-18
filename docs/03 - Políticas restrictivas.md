# Políticas restrictivas

> **Fase:** 3 (Políticas restrictivas) 
> **Objetivo:** preparar los controles de seguridad del FortiGate (DoS Policy, perfil de AntiVirus y perfiles de IPS) que se usarán durante las pruebas, para comparar el comportamiento del firewall sin ellos y con ellos. 
> **Estado al cierre de este documento:** la DoS Policy está creada y activa. El perfil de AntiVirus está creado pero **no aplicado** a ninguna política. Los perfiles de IPS son los predefinidos y tampoco están aplicados. El modo de inspección de la política pasó a Proxy-based. 
> **Documento anterior:** `02-politicas-base.md`.

## 1. Estrategia

Con el límite de **3 políticas de firewall** de la licencia de evaluación no se pueden crear políticas nuevas. Por eso los controles se preparan como objetos aparte (DoS Policy y perfiles) y se activarán durante las pruebas **editando las políticas existentes**.

Cada prueba se hace con y sin el control, para tener una comparación directa. Por eso los perfiles se crean ahora, pero no se aplican hasta el momento de la prueba.

| Control                           | Estado ahora                            | Se usa en                      |
| --------------------------------- | --------------------------------------- | ------------------------------ |
| DoS_LAB (DoS Policy)              | Creada y **activa** desde su creación   | Pruebas 1 y 3                  |
| AV_GENERAL (perfil AntiVirus)     | Creado, sin aplicar (Ref. 0)            | Prueba 4 (EICAR)               |
| Perfil de IPS                     | Solo predefinidos, ninguno aplicado     | Escaneos                       |
| Modo de inspección de la política | **Proxy-based** (cambiado en esta fase) | Todas las pruebas con perfiles |
| Servicio `ALL` acotado            | `ALL`                                   | Todas                          |

## 2. DoS Policy

En **Policy & Objects → DoS Policy**. La lista estaba vacía.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/1.png" width="600"> </p>

### 2.1 Parámetros generales

|Campo|Valor|
|---|---|
|Nombre|`DoS_LAB`|
|Interfaz de entrada|Offensive_LAN (port2)|
|Dirección de origen|port2 address|
|Dirección de destino|port3 address|
|Servicio|ALL|

La política cubre el sentido Kali → Debian, que es el de las pruebas de reconocimiento y fuzzing.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/2.png" width="600"> </p>

### 2.2 Anomalías

Todas las anomalías tienen acción **Block** y registro (**Logging**) activado. Los umbrales son los de la pantalla de creación.

|Capa|Anomalía|Acción|Umbral|
|---|---|---|---|
|L3|`ip_src_session`|Block|3000|
|L3|`ip_dst_session`|Block|3000|
|L4 TCP|`tcp_syn_flood`|Block|1000|
|L4 TCP|`tcp_port_scan`|Block|500|
|L4 TCP|`tcp_src_session`|Block|3000|
|L4 TCP|`tcp_dst_session`|Block|3000|
|L4 UDP|`udp_flood`|Block|1000|
|L4 UDP|`udp_scan`|Block|1000|
|L4 UDP|`udp_src_session`|Block|3000|
|L4 UDP|`udp_dst_session`|Block|3000|
|L4 ICMP|`icmp_flood`|Block|250|
|L4 ICMP|`icmp_sweep`|Block|100|
|L4 ICMP|`icmp_src_session`|Block|300|
|L4 ICMP|`icmp_dst_session`|Block|1000|
|L4 SCTP|`sctp_flood`|Block|1000|
|L4 SCTP|`sctp_scan`|Block|1000|
|L4 SCTP|`sctp_src_session`|Block|3000|
|L4 SCTP|`sctp_dst_session`|Block|3000|

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/3.png" width="600"> </p>
<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/4.png" width="600"> </p>

### 2.3 Resultado

La política queda creada con el ID 1 y **activa desde que se creó**. Se evalúa durante las pruebas.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/4.png" width="600"> </p>
<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/5.png" width="600"> </p>


**Observación sobre la licencia.** La DoS Policy se creó sin error con las 3 políticas de firewall ya existentes. En esta versión y con esta licencia no cuenta contra el límite de 3 políticas de firewall.

## 3. Perfil de AntiVirus

En **Security Profiles → AntiVirus**. Había dos perfiles predefinidos: `default` y `wifi-default`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/6.png" width="600"> </p>

Se creó un perfil nuevo en lugar de editar `default`, para no alterar el predefinido.

|Campo|Valor|
|---|---|
|Nombre|`AV_GENERAL`|
|Comentario|Antivirus para la Defensive_LAN|
|AntiVirus scan|Activado, acción **Block**|
|Feature set|**Proxy-based**|

Protocolos inspeccionados:

|Protocolo|Estado|
|---|---|
|HTTP|Activado|
|POP3|Activado|
|IMAP|Activado|
|SSH|Activado|
|SMTP, FTP, CIFS, MAPI, WebSocket|Desactivados|

Las opciones de "APT Protection Options" visibles en la captura (Content Disarm and Reconstruction, tratar ejecutables de Windows en correo como virus, envío a FortiSandbox y a FortiNDR) están desactivadas.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/7.png" width="600"> </p>

El perfil queda con **Ref. 0**: existe, pero ninguna política lo usa todavía.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/8.png" width="600"> </p>

**Dónde se aplicará.** El nombre y el comentario indican que es para Defensive_LAN. En la prueba 4 la descarga de EICAR la inicia Debian (Red B) hacia Kali, así que el perfil se aplicará a `defensiveTOoffensive` (port3 → port2).

**Limitaciones:**

- La inspección SSL está en `no-inspection`, así que el tráfico HTTPS no se escanea. La descarga de EICAR por HTTP (`python3 -m http.server`) sí es inspeccionable.
- No se ha comprobado si FortiGuard (AntiVirus e IPS) funciona con la licencia de evaluación. Si algo no se puede probar, se documenta como limitación en `04-conclusiones.md`.

## 4. Perfiles de IPS

En **Security Profiles → Intrusion Prevention**. No se creó ningún perfil nuevo. La lista trae 8 perfiles predefinidos, todos con Ref. 0 salvo `wifi-default`.

|Perfil|Descripción (según la GUI)|
|---|---|
|`all_default`|Todas las firmas predefinidas con la acción por defecto|
|`all_default_pass`|Todas las firmas predefinidas con acción PASS|
|`default`|Prevenir ataques críticos|
|`high_security`|Bloquea todas las vulnerabilidades Critical, High y Medium, y algunas Low|
|`protect_client`|Protege contra vulnerabilidades del lado cliente|
|`protect_email_server`|Protege contra vulnerabilidades de servidores de correo|
|`protect_http_server`|Protege contra vulnerabilidades de servidores HTTP|
|`wifi-default`|Configuración por defecto para tráfico WiFi|

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/9.png" width="600"> </p>

## 5. Estado de la política de firewall

Se revisó la política ID 2 (`offensiveTOdefensive`). Todos los perfiles de seguridad están desactivados (AntiVirus, Web filter, Video filter, DNS filter, Application control, IPS y File filter), la inspección SSL está en `no-inspection` y NAT sigue activo.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/03-politicas-restrictivas/10.png" width="600"> </p>

**Cambio de modo de inspección.** En `02-politicas-base.md` la línea base era **Flow-based**. En esta fase el modo se cambió a **Proxy-based** para que el laboratorio funcione. 
## 6. Plan de activación por prueba

| Prueba                         | Controles                                                                                                                                    |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Reconocimiento (Nmap)       | Sin política, con política base y con política restrictiva. DoS_LAB está activa.                                                             |
| 2. Acceso web                  | IPS/DoS/AV                                                                                                                                   |
| 3. Fuzzing                     | Pasada lenta y pasada agresiva con DoS_LAB. Los umbrales de `tcp_syn_flood`, `tcp_src_session` y `tcp_dst_session` son los más relacionados. |
| 4. Contenido malicioso (EICAR) | Sin perfil y luego con `AV_GENERAL` en `defensiveTOoffensive`.                                                                               |

## 7. Decisiones y límites

- **Límite de 3 políticas de firewall.** Los controles se preparan como objetos aparte y se activan editando las políticas existentes.
- **DoS Policy y perfiles no cuentan contra el límite.** Se crearon sin error con las 3 políticas ya existentes.
- **Memoria.** La RAM del FortiGate trabaja al 97 % de 2 GiB. La licencia de evaluación limita la RAM a 2 GiB, así que **no se puede ampliar**. Lo recomendable es asignar más RAM (con una licencia que lo permita), sobre todo con el modo Proxy-based, que consume más recursos.
- **Inspección SSL.** Sigue en `no-inspection`: el HTTPS no se inspecciona.
