# 01 - Topología y segmentación de red

> **Fase:** 1 (Topología)
> **Objetivo:** que el FortiGate sea el único camino entre la Red A (Kali)
> y la Red B (Debian).
> **Estado al cierre de este documento:** cada máquina llega a la interfaz
> de su gateway. El tráfico entre redes aún no está permitido; se resuelve
> en `02-politicas-base.md`.

## 1. Diseño

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/Diagrama.png" width="600"> </p>

| Red               | Subred          | Red virtual (VMware)    | Máquina         | Gateway         |
| ----------------- | --------------- | ----------------------- | --------------- | --------------- |
| Red A (ofensiva)  | 192.168.10.0/24 | Offensive_Net (vmnet10) | ReconLab        | FortiGate port2 |
| Red B (defensiva) | 192.168.20.0/24 | Defensive_Net (vmnet19) | debian-lab      | FortiGate port3 |
| Salida / gestión  | 192.168.8.0/24  | VMnet0 (Bridge)         | FortiGate port1 | N/A             |

### 1.1 Decisiones de diseño

- **Una red virtual aislada por segmento.**
- **Tres interfaces en el FortiGate, una por máquina.** 
- **Gateway = IP del FortiGate en cada red (.1).** 
- **port1 en modo Bridge.** El laboratorio vive dentro de mi red
  doméstica: mi router recibe su IP del router del ISP, y el FortiGate
  toma por Bridge una IP de mi router (192.168.8.218). Para salir a
  internet, el tráfico pasa por mi router y luego por el del ISP.

## 2. Creación de las redes virtuales en VMware

Las redes se crean primero en el **Virtual Network Editor**, porque las
máquinas no pueden seleccionarlas si no existen. En ambas se desmarcó
**"Connect a host virtual adapter to this network"**, para que la PC
física no participe en ellas.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/1.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/2.png" width="600"> </p>

En ambas el DHCP local de VMware quedó activo. 

## 3. Asignación de redes a las máquinas virtuales

Con las redes creadas, se asignó cada adaptador en **Custom: Specific
virtual network**.

Kali (ReconLab) en Offensive_Net:

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/3.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/4.png" width="600"> </p>

Debian (debian-lab) en Defensive_Net:

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/5.png" width="600"> </p>

FortiGate: de los 10 adaptadores del OVF se dejaron 3.

| Adaptador | Red virtual | Puerto en FortiOS |
|---|---|---|
| Network Adapter | Bridged (Automatic) | port1 |
| Network Adapter 2 | Offensive_Net | port2 |
| Network Adapter 3 | Defensive_Net | port3 |

## 4. Plan de direccionamiento

| Dispositivo | Interfaz | Red virtual | IP | Asignación |
|---|---|---|---|---|
| FortiGate | port1 | VMnet0 (Bridge) | 192.168.8.218/24 | DHCP del router doméstico |
| FortiGate | port2 | Offensive_Net | 192.168.10.1/24 | Estática |
| FortiGate | port3 | Defensive_Net | 192.168.20.1/24 | Estática |
| Kali | eth0 | Offensive_Net | 192.168.10.128/24 | DHCP de VMware |
| Debian | ens33 | Defensive_Net | 192.168.20.128/24 | DHCP de VMware |

## 5. Configuración del FortiGate por CLI

### 5.1 Estado inicial

port1 tenía IP por DHCP; port2 y port3 estaban en `0.0.0.0`.

Comando usado para listarlas: config system interface.

### 5.2 port2 (Red A)

```
config system interface
edit port2
set mode static
set ip 192.168.10.1 255.255.255.0
next
end
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/23.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/24.png" width="600"> </p>

### 5.3 port3 (Red B)

```
config system interface
edit port3
set mode static
set ip 192.168.20.1 255.255.255.0
next
end
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/25.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/26.png" width="600"> </p>

## 6. Acceso a la GUI

Se accede por HTTPS a la IP de port1. El navegador advierte por el
certificado autofirmado.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/16.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/17.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/18.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/19.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/20.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/21.png" width="600"> </p>


## 7. Incidencia: el ping solo funcionaba en un sentido

Con las IPs configuradas por CLI, el ping **del FortiGate hacia las
máquinas funcionaba**, pero el ping **de las máquinas hacia el FortiGate
no**. Al principio parecía un error de red.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/28.png" width="600"> </p>

Desde la GUI se editaron `port2` y `port3`: se asignó el rol **LAN** y se
habilitó **PING** en el acceso administrativo. Después, Kali y Debian
pudieron hacer ping a su gateway.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/31.png" width="600"> </p>


## 8. Configuración de Kali y Debian

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/9.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/02-politica-base/11.png" width="600"> </p>


## 9. Verificación

| Prueba                              | Resultado                         |
| ----------------------------------- | --------------------------------- |
| FortiGate → Kali (192.168.10.128)   | 5 packets received 0% packet loss |
| FortiGate → Debian (192.168.20.128) | 5 packets received 0% packet loss |
| Kali → gateway (192.168.10.1)       | Responde (tras habilitar PING)    |
| Debian → gateway (192.168.20.1)     | Responde (tras habilitar PING)    |
| Kali ↔ Debian                       | "Red inaccesible"                 |

## 10. Estado actual y pendientes

- Cada red llega a su gateway.
- Kali y Debian no se alcanzan entre sí; el mensaje es "red inaccesible".
- El paso siguiente es permitir el tráfico entre redes, en `02-politicas-base.md`.
