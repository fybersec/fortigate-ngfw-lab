# Servicio web: WordPress en Docker

> **Fase:** 0 (Entorno) **Máquina:** `debian-server` (Red B, 192.168.20.10) 
> **Objetivo:** dejar publicado en el puerto 80 un servicio web con login (WordPress) que sirva como objetivo de las pruebas. 
> **Estado de red en este documento:** Debian ya está en Defensive_Net y sale a internet por la política `DefensivetoWAN`.
> **Fuentes de los pasos:** (1) guía oficial de Docker para instalar Docker Engine en Debian: [Instalacion](https://docs.docker.com/engine/install/debian/) y (2) el taller de Docker de Aula de Software Libre, sección "Creación de contenedores automatizada": [Instalacion](https://aulasoftwarelibre.github.io/taller-de-docker/docker-compose/#creacion-de-contenedores-automatizada)

## 1. Cambio de alcance

El plan original era un sitio estático servido por el Apache que instala Debian. Se cambió a **WordPress en Docker**.

**Contraseña débil intencional.** La cuenta de administrador de WordPress se creó con una contraseña fácil a propósito, para que una herramienta de fuerza bruta (wpscan) pueda encontrarla y ver cómo responde el firewall.

## 2. Preparación del sistema

**Fuente.** Guía oficial de Docker para Debian [Link de Guia](https://docs.docker.com/engine/install/debian/), pasos previos a la instalación.

Se actualiza el índice de paquetes y el sistema. No había nada pendiente.

```bash
apt update
apt full-upgrade
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/1.png" width="600"> </p>

Se instalan `ca-certificates` y `curl`, necesarios para descargar la clave del repositorio de Docker.

```bash
sudo apt install ca-certificates curl
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/2.png" width="600"> </p>

## 3. Instalación de Docker Engine

**Fuente.** Guía oficial de Docker para Debian [Link de Guia](https://docs.docker.com/engine/install/debian/), método de instalación con el repositorio `apt` de Docker.

Se usa el repositorio oficial de Docker y no el paquete `docker.io` de Debian.

### 3.1 Repositorio

Se descarga la clave GPG, se le dan permisos de lectura y se declara el repositorio. `Suites` y `Architectures` se rellenan solos a partir del sistema (en esta máquina: `trixie` y `amd64`).

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/3.png" width="600"> </p>

### 3.2 Paquetes

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/4.png" width="600"> </p>

### 3.3 Interfaz creada por Docker

Al instalarse, Docker crea en el host la interfaz puente `docker0`. Aparece en estado DOWN porque todavía no hay contenedores. `ens33` conserva la IP 192.168.20.10.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/5.png" width="600"> </p>

## 4. Liberar el puerto 80 y reducir servicios

Este paso **no viene de ninguna de las dos fuentes**; es una decisión propia del laboratorio.

El instalador de Debian dejó Apache activo (componente "Web server", ver `00-entorno-servidor.md`). Se detiene y deshabilita para que el puerto 80 lo use el contenedor. También se detiene y deshabilita CUPS (servicio de impresión).

```bash
systemctl stop apache2
systemctl disable apache2
systemctl stop cups
systemctl disable cups
ss -tuln | tail -n3
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/6.png" width="600"> </p>

Tras esto, los únicos sockets TCP en escucha son los del puerto 22 (IPv4 e IPv6), más el UDP 5353.

## 5. Despliegue con Docker Compose

**Fuente.** Taller de Docker de Aula de Software Libre, sección "Creación de contenedores automatizada" [Link de Taller](https://aulasoftwarelibre.github.io/taller-de-docker/docker-compose/#creacion-de-contenedores-automatizada). El `docker-compose.yaml` y el comando `docker compose up -d` son los del taller, con los cambios de la tabla siguiente.

| Elemento                               | En el taller                  | En este laboratorio                              |
| -------------------------------------- | ----------------------------- | ------------------------------------------------ |
| Directorio de trabajo                  | `~/Sites/wordpress`           | `~/wordpress_docker`                             |
| Puerto publicado                       | `8080:80`                     | `80:80`                                          |
| Credenciales de la base y de WordPress | Valores de ejemplo del taller | Valores propios (se muestran como `<REDACTADO>`) |

### 5.1 Archivo `docker-compose.yaml`

```yaml
services:
    db:
        image: mariadb:10
        volumes:
            - data:/var/lib/mysql
        environment:
            - MYSQL_ROOT_PASSWORD=<REDACTADO>
            - MYSQL_DATABASE=<REDACTADO>
            - MYSQL_USER=<REDACTADO>
            - MYSQL_PASSWORD=<REDACTADO>
    web:
        image: wordpress:6
        depends_on:
            - db
        volumes:
            - ./target:/var/www/html
        environment:
            - WORDPRESS_DB_USER=<REDACTADO>
            - WORDPRESS_DB_PASSWORD=<REDACTADO>
            - WORDPRESS_DB_HOST=db
            - WORDPRESS_DB_NAME=<REDACTADO>
        ports:
            - 80:80

volumes:
    data:
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/7.png" width="600"> </p>

|Parámetro|Qué hace|
|---|---|
|`image: mariadb:10`|Contenedor de la base de datos, versión 10 de MariaDB.|
|`image: wordpress:6`|Contenedor de WordPress, versión 6.|
|`volumes: data:/var/lib/mysql`|Volumen con nombre: los datos de la base sobreviven si se recrea el contenedor.|
|`volumes: ./target:/var/www/html`|Los archivos de WordPress se guardan en `~/wordpress_docker/target` del host.|
|`depends_on: db`|Arranca primero la base de datos.|
|`WORDPRESS_DB_HOST=db`|WordPress encuentra la base por el nombre del servicio, dentro de la red de Docker.|
|`ports: 80:80`|Publica el puerto 80 del contenedor en el puerto 80 del host.|
|Sin `ports` en `db`|La base de datos no se publica fuera de Docker.|

### 5.2 Arranque

```bash
docker compose up -d
```

La descarga de las imágenes tardó unos 200 segundos cada una, a través de la política `DefensivetoWAN` con NAT. Se crean la red, el volumen y los dos contenedores. Compose nombra los recursos con el nombre del directorio como prefijo (`wordpress_docker_...`).

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/8.png" width="600"> </p>

## 6. Verificación de puertos

```bash
ss -tuln
```

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/9.png" width="600"> </p>

| Puerto       | Protocolo | Estado               | Origen                                             |
| ------------ | --------- | -------------------- | -------------------------------------------------- |
| 22           | TCP       | LISTEN (IPv4 e IPv6) | SSH                                                |
| 80           | TCP       | LISTEN (IPv4 e IPv6) | WordPress (contenedor `web`)                       |

No hay nada escuchando en el puerto 3306: la base de datos solo es accesible dentro de la red de Docker.

## 7. Instalación de WordPress

Desde Firefox en la propia máquina Debian se abre `http://localhost/wp-admin/install.php`. Idioma: Español.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/10.png" width="600"> </p>

Datos del sitio:

| Campo                     | Valor                                                                                                                                                       |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Título del sitio          | `fortigate-ngfw-lab`                                                                                                                                        |
| Usuario                   | No se documenta                                                                                                                                             |
| Contraseña                | No se documenta. WordPress la calificó como **"Muy débil"** y se marcó la casilla de confirmar el uso de una contraseña débil (intencional, ver sección 1). |
| Correo                    | `fortigate-ngfw-lab@test.com` (ficticio)                                                                                                                    |
| Visibilidad en buscadores | Sin marcar                                                                                                                                                  |

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/11.png" width="600"> </p>

Tras la instalación se accede al escritorio de administración. El panel indica la versión **6.9.4** y avisa de que hay una versión más reciente (7.1.3) y 6 actualizaciones pendientes. No se actualizó.

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/12.png" width="600"> </p>

## 8. Verificación desde Kali a través del firewall

Desde Kali (Red A) se abre `http://192.168.20.10`. Se muestra el blog de WordPress ("¡Hola, mundo!").

<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/weapp/13.png" width="600"> </p>

|#|Prueba|Resultado|Política validada|
|---|---|---|---|
|1|Kali → `http://192.168.20.10` (puerto 80)|Carga el blog de WordPress|offensiveTOdefensive|

Esto confirma que el servicio es alcanzable de punta a punta con la política base (servicio ALL, sin perfiles de seguridad). Es solo una comprobación de funcionamiento.

## 9. Estado final, decisiones y límites

|Elemento|Estado|
|---|---|
|Apache del sistema|Detenido y deshabilitado|
|CUPS|Detenido y deshabilitado|
|Docker Engine|Instalado desde el repositorio oficial|
|Contenedores|`wordpress_docker-db-1` y `wordpress_docker-web-1` en ejecución|
|Servicio web|WordPress 6.9.4 en `http://192.168.20.10` (puerto 80)|

- **WordPress sin actualizar** (6.9.4, con 7.1.3 disponible).
- **Sin HTTPS.** El sitio se sirve solo por HTTP (puerto 80).
- **Base de datos no expuesta** fuera de Docker.

## Créditos y fuentes

- **Docker Inc.**, guía oficial "Install Docker Engine on Debian": [Link](https://docs.docker.com/engine/install/debian/). Base de las secciones 2 y 3 (prerrequisitos, repositorio `apt` y paquetes).
- **Aula de Software Libre**, "Taller de Docker", sección "Creación de contenedores automatizada": [Link](https://aulasoftwarelibre.github.io/taller-de-docker/docker-compose/#creacion-de-contenedores-automatizada). Base de la sección 5 (archivo `docker-compose.yaml`, `docker compose up -d`) y de la sección 9.1. El taller está publicado bajo licencia **CC BY-NC 4.0**; aquí se adapta con los cambios indicados en la sección 5, con fines educativos y sin uso comercial.
- Imágenes oficiales en Docker Hub: [Link](https://hub.docker.com/_/wordpress) y [Link](https://hub.docker.com/_/mariadb).
