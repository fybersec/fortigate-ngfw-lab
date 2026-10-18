# Reconocimiento (Nmap)

> **Fase:** Pruebas 
> **Objetivo:** lanzar un reconocimiento con Nmap desde Kali contra Debian y ver si el FortiGate lo detecta y actúa. Cada comando va seguido de la reacción del firewall. Primero con el perfil de IPS en modo Monitor (deja pasar y registra) y después en modo Block. 
> **Snapshot:** antes de esta prueba se tomó un snapshot del FortiGate; al terminar se revierte, así que esto parte de cero en cada repetición.

## 1. Reglas activas antes de empezar

Sobre la política `offensiveTOdefensive (2)` (port2 → port3) se activaron:

| Control            | Valor                                                     |
| ------------------ | --------------------------------------------------------- |
| Modo de inspección | Proxy-based                                               |
| AntiVirus          | `AV_GENERAL`                                              |
| IPS                | `all_default_pass` (todas las firmas, acción **Monitor**) |
| SSL inspection     | no-inspection                                             |

La DoS Policy `DoS_LAB` (ver `docs/03-politicas-restrictivas.md`) sigue activa.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/1.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/2.png" width="600"> </p>

Antes de ejecutar nada, **Log & Report → Security Events** no mostraba eventos.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/3.png" width="600"> </p>

## 2. Primera ronda: IPS en modo Monitor

### Comando 1 — Ping sweep de la Red B

```bash
sudo nmap -sn -PE -PP -PM -T5 --min-rate 5000 -v 192.168.20.0/24
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/4.png" width="600"> </p>

**Reacción del firewall.** En Security Events aparece **1 evento**: anomalía `icmp_sweep`, acción Clear Session.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/5.png" width="600"> </p>

### Comando 2 — Todos los puertos TCP a máxima velocidad

```bash
sudo nmap -sS -p- -T5 --min-rate 5000 -v 192.168.20.10
```

Encontró 5 puertos abiertos: 22, 80, 110, 143 y 5060.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/6.png" width="600"> </p>

**Reacción del firewall.** Ahora **3 eventos**: anomalías `tcp_port_scan`, `tcp_syn_flood` e `icmp_sweep` (esta última acumulada del comando 1), todas Clear Session.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/7.png" width="600"> </p>

### Comando 3 — Escaneo exhaustivo con detección de vulnerabilidades

```bash
sudo nmap -sS -p- -A -T5 --min-rate 5000 --script vuln -v 192.168.20.10
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/8.png" width="600"> </p>

**Reacción del firewall.** Suben a **29 eventos**. Entra en juego el módulo de **Intrusion Prevention**, con 27 eventos repartidos en varias firmas (`Adobe.XML.Entity.Injection` ×12, `VMware.Server.Directory.Traversal` ×4, `Easy.Hosting.Control.Panel.FTP.Account.Security.Bypass` ×2, `MS.IIS.WebDAV.Authentication.Bypass` ×2, `Oxide.WebServer.Directory.Traversal` ×2, entre otras), todas con acción **Detected**. Las anomalías `tcp_port_scan` y `tcp_syn_flood` del comando 2 se mantienen.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/9.png" width="600"> </p>

El log detallado de Intrusion Prevention confirma lo mismo, firma por firma, con acción `detected`. Severidades vistas: Critical (`Bash.Function.Definitions.Remote.Code.Execution`), High (`Adobe.XML.Entity.Injection`, `PHP.CGI.Argument.Injection`), Medium (`VMware.Server.Directory.Traversal`, `Oxide.WebServer.Directory.Traversal`, `MS.IIS.WebDAV.Authentication.Bypass`, `Majordomo2.Directory.Traversal`, `Drupal.Core.database.inc.expandArguments.SQL.Injection`) y Low (`HTTP.Referer.Header.XSS`).

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/10.png" width="600"> </p>

En **Forward Traffic**, todo el tráfico sigue como `Accept (UTM Allowed)`, bajo la política `offensiveTOdefensive (2)`. Es coherente con que el perfil estaba en Monitor: el IPS registra la firma, pero no corta la conexión.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/11.png" width="600"> </p>

**Por qué aparece como HTTP.** El escaneo exhaustivo (`-A --script vuln`) prueba scripts de reconocimiento sobre el puerto 80, así que genera peticiones HTTP hacia WordPress; de ahí que el tráfico se clasifique como `HTTP` en el log, aunque el origen sea Nmap y no un navegador.

## 3. Cambio del perfil de IPS a modo Block

Se entra al mismo perfil `all_default_pass` para comprobar que, en efecto, estaba en modo Monitor para todas las firmas (`All Attributes`).

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/12.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/13.png" width="600"> </p>

Se cambia la acción a **Block**.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/14.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/15.png" width="600"> </p>

## 4. Segunda ronda: repetición del comando 3 con IPS en Block

```bash
sudo nmap -sS -p- -A -T5 --min-rate 5000 --script vuln -v 192.168.20.10
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/16.png" width="600"> </p>

**Reacción del firewall.** De nuevo 29 eventos, pero ahora las firmas de Intrusion Prevention aparecen con acción **Dropped** en lugar de Detected.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/17.png" width="600"> </p>

El log de Intrusion Prevention confirma la acción `dropped` en cada entrada.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/18.png" width="600"> </p>

Y en **Forward Traffic**, el resultado cambia de `Accept` a **`Deny (Deny: UTM Blocked)`**, bajo la misma política `offensiveTOdefensive (2)`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/prueba/01/19.png" width="600"> </p>

## 5. Comparación

|Estado del perfil IPS|Acción en el log de IPS|Resultado en Forward Traffic|
|---|---|---|
|Monitor|`detected`|`Accept (UTM Allowed)`|
|Block|`dropped`|`Deny (Deny: UTM Blocked)`|




