# Políticas base

> **Fase:** 2 (Políticas base)
> **Objetivo:** establecer conectividad entre la Red A y la Red B, y salida a internet desde la Red B, con todo el tráfico pasando obligatoriamente por el FortiGate. Aún no hay restricciones: es la línea base.
> **Documento anterior:** `01-topologia.md`.

## 1. Identificación de las interfaces

Antes de crear políticas se asignó **alias** y **rol** a cada interfaz (Network → Interfaces), para identificarlas por nombre en las políticas.

| Interfaz | Alias         | Rol | Conectada a               |
| -------- | ------------- | --- | ------------------------- |
| port1    | WAN           | WAN | Bridge de VMware (salida) |
| port2    | Offensive_LAN | LAN | Kali (Red A)              |
| port3    | Defensive_LAN | LAN | Debian (Red B)            |

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/1.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/2.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/3.png" width="600"> </p>

**Sobre port1.** Está en Bridge: toma IP de mi router personal (192.168.8.218), que a su vez cuelga del router del ISP. El laboratorio queda dentro de mi red doméstica y, para salir, el tráfico atraviesa el FortiGate, mi router y el router del ISP.

## 2. Políticas de firewall

Sin una política que lo permita, el FortiGate descarta el tráfico entre interfaces (denegación implícita). Se crearon tres políticas, que con el máximo que permite la licencia de evaluación.

| # | Nombre | Entrada → Salida | Origen | Destino | Servicio | Acción | NAT |
|---|---|---|---|---|---|---|---|
| 1 | defensiveTOoffensive | port3 → port2 | port3 address | port2 address | ALL | ACCEPT | Activo |
| 2 | offensiveTOdefensive | port2 → port3 | port2 address | port3 address | ALL | ACCEPT | Activo |
| 3 | DefensivetoWAN | port3 → port1 | port3 address | all | ALL | ACCEPT | Activo |

### 2.1 Defensive → Offensive (port3 → port2)

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/4.png" width="600"> </p>

### 2.2 Offensive → Defensive (port2 → port3)

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/5.png" width="600"> </p>

Se crearon dos políticas, una por sentido, porque cada una cubre las conexiones **iniciadas** desde su interfaz de entrada. 

**Incidencia.** Con solo la política Offensive → Defensive, el tráfico iniciado desde Defensive hacia Offensive no recibía respuesta: el ping
quedaba congelado, sin mensaje de error. Es el comportamiento de la denegación implícita. Tras crear la política Defensive → Offensive, respondió. Esto se distingue de "red inaccesible", que genera la propia máquina cuando no tiene ruta.

### 2.3 Defensive → WAN (port3 → port1)

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/6.png" width="600"> </p>

Solo Debian sale a internet. El destino es `all` y el origen es la Red B. La salida se usa únicamente para administración: descargar paquetes y recursos para el servicio web. Las simulaciones de ataque se hacen entre las redes internas. Kali no tiene salida a internet.

**NAT en esta política:** necesario, porque mi router no tiene ruta de vuelta hacia 192.168.20.0/24. 

### 2.4 Vista general

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/7.png" width="600"> </p>

### 2.5 Decisiones y límites

- **Límite de 3 políticas** de la licencia de evaluación. En `03-politicas-restrictivas.md` las políticas existentes se editarán
  en lugar de crear nuevas.
- **Servicio ALL: provisional.** Es la línea base de conectividad y se acota en la fase restrictiva.
- **Sin perfiles de seguridad.** SSL en `no-inspection` y ningún perfil UTM. Es la línea base "sin inspección".

## 3. Configuración de red de las máquinas

Se configuró IP, máscara y gateway de forma **manual** desde la interfaz gráfica de cada sistema, en lugar de `ip route` por terminal, para que la configuración sea persistente.

| Máquina | IP | Máscara | Gateway |
|---|---|---|---|
| Kali (ReconLab) | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Debian (debian-lab) | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/8.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/9.png" width="600"> </p>


## 4. Verificación de conectividad

### 4.1 Hacia el gateway (termina en el FortiGate)

Estos pings tienen como destino la interfaz del propio FortiGate. El TTL de 255 indica que el paquete no pasó por ningún router intermedio. Sirven para demostrar que cada máquina alcanza su gateway; la respuesta depende del acceso PING habilitado en la interfaz, no de las políticas de firewall.

| # | Prueba | Resultado |
|---|---|---|
| 1 | Kali → 192.168.20.1 (FortiGate, port3) | 0% pérdida, TTL 255 |
| 2 | Debian → 192.168.10.1 (FortiGate, port2) | 0% pérdida, TTL 255 |

### 4.2 A través del firewall (validan las políticas)

Con TTL 63, el paquete pasó por un salto (el FortiGate) antes de llegar a su destino: salió con 64 y llegó con 63. Es la evidencia de que el tráfico atravesó el firewall.

| #   | Prueba                        | Resultado               | Política validada                              |
| --- | ----------------------------- | ----------------------- | ---------------------------------------------- |
| 3   | Kali → Debian (192.168.20.10) | 0% pérdida, TTL 63      | offensiveTOdefensive                           |
| 4   | Debian → Kali (192.168.10.10) | 0% pérdida, TTL 63      | defensiveTOoffensive                           |
| 5   | Debian → 8.8.8.8              | 0% pérdida, TTL 117     | DefensivetoWAN y NAT                           |
| 6   | Debian → `google.com`         | 0% pérdida, TTL 109     | DNS - 192.168.8.1                              |
| 7   | Kali → 8.8.8.8                | Se espera sin respuesta | Sin política hacia WAN, el firewall lo bloquea |

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/10.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/12.png" width="600"> </p>

- Las tres políticas validadas: Red A ↔ Red B y Red B → internet.
