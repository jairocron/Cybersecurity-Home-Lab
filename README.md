# Cybersecurity Home Lab - Fase 1: SOC con Raspberry Pi 5

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![OS](https://img.shields.io/badge/OS-Ubuntu%20Server-orange)
![Docker](https://img.shields.io/badge/Containerization-Docker-blue)

¡Hola! Soy un estudiante de ciberseguridad construyendo este laboratorio para aprender y demostrar mis habilidades en monitoreo y defensa. Este repositorio documenta mi progreso transformando una Raspberry Pi 5 en un SOC compacto.

## 📋 Tabla de Contenidos
- [Resumen de la Fase 1](#resumen-de-la-fase-1)
- [Stack Tecnológico](#stack-tecnológico)
- [Escenario de Pruebas](#escenario-de-pruebas)
- [Evidencia del Laboratorio Operativo](#evidencia-del-laboratorio-operativo)
- [Cómo replicar este laboratorio](#cóme-replicar-este-laboratorio)
  - [Prerrequisitos](#prerrequisitos)
  - [1. Gestión de Servicios Mediante Docker](#1-gestión-de-servicios-mediante-docker)
  - [2. Wazuh (SIEM)](#2-wazuh-siem)
  - [3. Netdata (Monitoreo)](#3-netdata-monitoreo)
  - [4. Control de Tráfico y Privacidad DNS (Pi-hole)](#4-️-control-de-tráfico-y-privacidad-dns-pi-hole)
  - [5. Honeypot (Opcional)](#5-honeypot-opcional)
- [Acceso y Uso](#acceso-y-uso)

---

## Resumen de la Fase 1

En esta primera etapa, he transformado una **Raspberry Pi 5 (16GB RAM)** en un Centro de Operaciones de Seguridad (SOC) ligero utilizando contenedores.

### Stack Tecnológico
* **S.O.:** Ubuntu Server (64-bit)
* **Gestión:** Docker & Portainer
* **SIEM/XDR:** WazuhManager (Bare Metal/System)
* **Monitoreo de Sistema:** Netdata (Docker)
* **Red & DNS:** Pi-hole (Docker)
* **Defensa Activa:** Honeypot (configurado en la RPi)

### Escenario de Pruebas
1. **PC Windows:** Monitorizada mediante agente de Wazuh y Netdata.
2. **Kali Linux (VM):** Utilizada como máquina de ataque para realizar escaneos y pruebas contra el Honeypot.
3. **Wazuh:** Centraliza todas las alertas de seguridad para su análisis.

---

### 📊 Evidencia del Laboratorio Operativo

| Servicio | Visualización |
| :--- | :--- |
| **SIEM Wazuh** | ![Wazuh](./img/wazuh-threat-hunting.png) |
| **DNS Security** | ![Pi-hole](./img/pihole-dns-sinkhole.png) |
| **Stack Docker** | ![Portainer](./img/portainer-stack-core.png) |

---

## Cómo replicar este laboratorio

### Prerrequisitos
- Raspberry Pi 5 (o equivalente) con Ubuntu Server instalado.
- Acceso SSH configurado.

### 1. 📦 Gestión de Servicios mediante Docker

Utilizo **Portainer** como interfaz de orquestación para gestionar el ciclo de vida de los contenedores en el nodo `soc-master`. Actualmente, el stack incluye servicios críticos de seguridad y monitoreo:

*   **Honeypot (Cowrie):** Captura de intentos de intrusión por SSH.
*   **Monitoreo (Netdata):** Visualización de métricas de rendimiento.
*   **Filtrado DNS (Pi-hole):** Control de tráfico de red.

> 📌 **Estado actual:** El stack está desplegado y operativo.
> **Próxima fase:** Optimización de la persistencia de datos y refinamiento de alertas.

#### Instalación Unificada (Docker Compose)

He consolidado la gestión de Portainer, Netdata y Pi-hole en un único archivo `docker-compose.yml`.

1. Clona el repositorio o crea el archivo `docker-compose.yml`.
2. Levanta todos los servicios de soporte:

```bash
docker-compose up -d
```

> **Nota:** El archivo `docker-compose.yml` incluido despliega:
> *   Portainer (Puerto 9443)
> *   Netdata (Puerto 19999)
> *   Pi-hole (Puertos 53 y 80)


### 2. Wazuh (SIEM)

#### 🛠️ Implementación de Wazuh (Bare Metal / Sistema)
A diferencia de otros servicios del laboratorio que corren en Docker (como Pi-hole o Netdata), el stack de Wazuh ha sido instalado directamente sobre el sistema operativo para maximizar el rendimiento y la estabilidad, permitiendo una integración profunda con los recursos de la Raspberry Pi 5.

Para instalar Wazuh en este modo, se recomienda seguir la [documentación oficial de instalación asistida](https://documentation.wazuh.com/current/installation-guide/wazuh-server/step-by-step.html).

### 3. Netdata (Monitoreo)

Netdata ofrece métricas en tiempo real con muy bajo consumo de recursos. **Este servicio se despliega automáticamente con el docker-compose principal.**

Si necesitas modificar la configuración, los volúmenes están mapeados en el archivo compose.


### 4. 🛡️ Control de Tráfico y Privacidad DNS (Pi-hole)

He implementado **Pi-hole** para actuar como el primer escudo de la red del laboratorio. Su función es interceptar consultas DNS maliciosas y bloquear telemetría no deseada antes de que llegue a los endpoints.

**Capacidades configuradas:**
*   Bloqueo basado en listas de reputación (74,000+ dominios).
*   Dashboard centralizado para auditoría de tráfico en tiempo real.

> ⚠️ **Desafío técnico detectado:** Actualmente se observa un bajo volumen de consultas bloqueadas. Se está trabajando en la reconfiguración del DHCP/DNS en el router principal para asegurar que todo el tráfico del laboratorio pase obligatoriamente por este nodo.

**Despliegue:**

Este servicio está incluido en el `docker-compose.yml`. Asegúrate de configurar la variable `WEBPASSWORD` en el archivo antes de desplegar, o busca la contraseña en los logs si no la definiste.

> ⚠️ **Solución de Problemas (Ubuntu/Debian):**
> Si el contenedor falla por "Port 53 already in use", es probable que `systemd-resolved` esté ocupando el puerto. Para solucionarlo:
> ```bash
> sudo systemctl stop systemd-resolved
> sudo systemctl disable systemd-resolved
> # Nota: Esto puede afectar la resolución DNS del host si no se configura un DNS alternativo en /etc/resolv.conf
> ```


### 5. Honeypot (Opcional)

Si deseas añadir un honeypot SSH como Cowrie:

```bash
docker run -p 2222:2222 -d cowrie/cowrie
```

---

## Acceso y Uso

Una vez desplegado el stack, puedes acceder a los servicios:

| Servicio | URL | Credenciales por defecto (si aplican) |
|----------|-----|----------------------------------------|
| **Portainer** | `https://<IP-RPi>:9443` | Definir en primer inicio |
| **Wazuh** | `https://<IP-RPi>` | admin / SecretPassword (revisar logs/docs) |
| **Netdata** | `http://<IP-RPi>:19999` | No requiere auth por defecto |
| **Pi-hole** | `http://<IP-RPi>:80/admin` | Ver logs (`docker logs pihole`) |

> **Nota:** Reemplaza `<IP-RPi>` con la dirección IP local de tu Raspberry Pi.
