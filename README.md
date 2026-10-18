<p align="center"> <img src="https://github.com/fybersec/fortigate-ngfw-lab/blob/main/evidencias/docs/01-topologia/Diagrama.png" width="900"> </p>


# Laboratorio FortiGate NGFW

Laboratorio para practicar la configuración y comprobación de un firewall FortiGate (NGFW), tras obtener la certificación NSE 2 de Fortinet. El objetivo no fue solo levantar el entorno, sino **comprobar** que las reglas configuradas realmente se respetan: se configura una política o un perfil de seguridad y luego se ataca desde Kali para ver si el firewall reacciona como se espera.

**Metodología.** Este laboratorio se hizo con el objetivo explícito de depender lo menos posible de la IA y lo más posible de documentación oficial. Las configuraciones del FortiGate se buscaron en docs.fortinet.com, el Fortinet Training Institute y canales de ingenieros en YouTube; la instalación de Docker y WordPress siguió documentación oficial y un tutorial específico se usó como apoyo puntual: redacción, orden de la documentación, diagnóstico de errores guiado y revisión antes de publicar, nunca para generar configuraciones del FortiGate de memoria.

## Topología

|Red|Subred|Máquina|Interfaz del FortiGate|
|---|---|---|---|
|Red A (ofensiva)|192.168.10.0/24|Kali (`ReconLab`)|port2 (Offensive_LAN)|
|Red B (defensiva)|192.168.20.0/24|Debian (`debian-server`)|port3 (Defensive_LAN)|
|Salida / gestión|192.168.8.0/24 (Bridge)|—|port1 (WAN)|

El FortiGate es el único camino entre Kali y Debian: cada red vive en su propia red virtual aislada de VMware, sin adaptador del host conectado. Detalle completo en `docs/01-topologia.md`.

## Herramientas

|Herramienta|Uso en el laboratorio|
|---|---|
|FortiGate-VM64 (FortiOS v8.0.1)|Firewall NGFW, objeto de las pruebas|
|VMware Workstation Pro|Hipervisor|
|Kali Linux|Máquina atacante (Nmap, wpscan)|
|Debian 13|Máquina víctima, aloja el servicio web|
|Docker / Docker Compose|Despliegue del servicio web|
|WordPress 6 + MariaDB 10|Servicio web objetivo, con login|
|Nmap|Reconocimiento|
|Archivo EICAR|Prueba del perfil de AntiVirus|

## Resumen de fases

| Documento                           | Contenido                                                       |
| ----------------------------------- | --------------------------------------------------------------- |
| `docs/00-entorno-fortigate.md`      | Despliegue e instalación del FortiGate-VM                       |
| `docs/00-entorno-servidor.md`       | Instalación de Debian                                           |
| `servicio-web/instalacion.md`       | WordPress en Docker sobre Debian                                |
| `docs/01-topologia.md`              | Segmentación de red y direccionamiento                          |
| `docs/02-politicas-base.md`         | Políticas de conectividad, sin restricciones                    |
| `docs/03-politicas-restrictivas.md` | DoS Policy, perfil de AntiVirus y perfiles de IPS               |
| `pruebas/01-reconocimiento.md`      | Nmap contra Debian, IPS en Monitor y en Block                   |
| `pruebas/04-contenido-malicioso.md` | Descarga del EICAR, AntiVirus                                   |

## Resultado: ataque → sin regla / con regla

| Ataque                      | Sin control                                            | Con control                                        | Control que marcó la diferencia               |
| --------------------------- | ------------------------------------------------------ | -------------------------------------------------- | --------------------------------------------- |
| Reconocimiento (Nmap)       | Tráfico aceptado, firmas solo registradas (`detected`) | Tráfico bloqueado (`dropped`, `Deny: UTM Blocked`) | Perfil de IPS, acción Monitor → Block         |
| Contenido malicioso (EICAR) | —                                                      | Descarga bloqueada (`High Security Alert`)         | Perfil de AntiVirus en `defensiveTOoffensive` |
## Cambios respecto a la estructura original

- `00-entorno.md` se dividió en `00-entorno-servidor.md` y `00-entorno-fortigate.md`.
- Se agregaron las carpetas `evidencia/servidor/` y `evidencia/topologia/`.
- `servicio-web/` quedó solo con `instalacion.md` y `capturas/` (sin README propio).
- **Se eliminó `SECURITY.md`** del repositorio.
- El servicio web pasó de un sitio estático a WordPress en Docker (ver `servicio-web/instalacion.md`).
- La prueba de "acceso web" se retiró del plan original de pruebas.

## Referencias y créditos

- Fortinet, documentación oficial de FortiOS 8.0: [Link](https://docs.fortinet.com)
- Docker Inc., instalación de Docker Engine en Debian: [Link](https://docs.docker.com/engine/install/debian/)
- Aula de Software Libre, "Taller de Docker": [Link](https://aulasoftwarelibre.github.io/taller-de-docker/docker-compose/)
- EICAR, archivo de prueba de AntiVirus: [Link](https://www.eicar.org/download-anti-malware-testfile/)
- Video: [Link](https://www.youtube.com/watch?v=yJC3Par77bY)
- Video: [Link](https://www.youtube.com/watch?v=4z-R5nJmb1w)
