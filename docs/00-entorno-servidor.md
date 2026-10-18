# Servidor víctima (Debian)

> **Fase:** 0 (Entorno)
> **Máquina:** Servidor web víctima
> **Estado de red en este documento:** provisional (una sola interfaz en NAT).
> La segmentación definitiva se documenta en `01-topologia.md`.

## 1. Propósito de la máquina

Este servidor simula un servidor real dentro de un entorno laboral. Aloja
un servicio web y actúa como el objetivo que el FortiGate debe proteger
desde la Red B. Desde Kali Linux se ejecutan ataques y pruebas contra él
para observar qué tráfico permite o bloquea el firewall según las políticas
aplicadas en cada fase. Es, en esencia, el punto de escaneo y de entrada
al otro lado del firewall.

## 2. Características de la máquina virtual

| Parámetro                | Valor                       |
| ------------------------ | --------------------------- |
| Hipervisor               | VMware Workstation Pro 26H1 |
| Sistema operativo        | Debian 13                   |
| Hostname                 | `debian-server`             |
| RAM                      | 3 GB                        |
| Procesadores             | 2                           |
| Almacenamiento           | 30 GB                       |
| Interfaces (provisional) | 1 en modo NAT               |
| Entorno gráfico          | GNOME                       |

**Nota sobre los recursos:** estos valores se asignan de forma provisional.
Al ser una máquina virtual, pueden aumentarse si el rendimiento lo requiere.

## 3. Instalación

Instalación con el instalador gráfico de Debian 13.

### 3.1 Arranque y configuración regional

Se inicia desde la ISO y se elige **Graphical install**.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/1.png" width="600"> </p>

Idioma del instalador y del sistema: Español.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/2.png" width="600"> </p>

Mapa de teclado: Ingles Estadounidense

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/3.png" width="600"> </p>

### 3.2 Nombre de la máquina

Hostname configurado: `debian-server`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/4.png" width="600"> </p>

### 3.3 Usuarios y contraseñas

Se define una contraseña para la cuenta de superusuario (root). La
contraseña no se muestra en la documentación.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/5.png" width="600"> </p>

Se crea además un usuario no administrativo. Nombre completo: `Administrator`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/6.png" width="600"> </p>

**Incidencia:** el primer nombre de usuario elegido fue `admin`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/7.png" width="600"> </p>

El instalador lo rechazó por ser un nombre reservado del sistema.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/8.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/9.png" width="600"> </p>


Se utilizó finalmente `debian-admin`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/10.png" width="600"> </p>

Contraseña del usuario definida.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/11.png" width="600"> </p>

### 3.4 Gestor de paquetes

Réplica de Debian: país Estados Unidos, servidor `deb.debian.org`.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/12.png" width="600"> </p>

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/13.png" width="600"> </p>

### 3.5 Selección de software

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/14.png" width="600"> </p>

| Componente                           | Motivo                                                        |
| ------------------------------------ | ------------------------------------------------------------- |
| Entorno de escritorio Debian / GNOME | Es una interfaz mas llamativa, aunque no le daremos mucho uso |
| Web server                           | Despliega el servicio web desde la propia instalación         |
| SSH server                           | Posible administración remota                                 |

### 3.6 Primer arranque

El sistema inicia correctamente en el escritorio GNOME.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/15.png" width="600"> </p>

## 4. Verificación posterior a la instalación

El servicio web quedó activo desde la instalación. Verificación:

```bash
uname -a
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/00-Machines/debian/16.png" width="600"> </p>

El contenido del sitio se documenta en `webapp/instalacion.md`.
