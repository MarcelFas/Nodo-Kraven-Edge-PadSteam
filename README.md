# Proyecto PadSteam - Infraestructura Edge Computing v2

![ubuntu](https://img.shields.io/badge/ubuntu-24.04-orange)
![docker](https://img.shields.io/badge/docker-latest-blue)
![licencia](https://img.shields.io/badge/ambiente-industrial-green)
![mqtt](https://img.shields.io/badge/bridge-established-green)

**Desarrollado por:** Fabrissio Fasabi Rivera  
**Ubicación:** Creditex S.A.A. - Planta Lima, Perú (Área de Tintorería)  
**Nodo:** Kraven Edge-Gateway-01

---

## 🚀 Visión General
Este proyecto implementa una infraestructura tecnológica escalable para el monitoreo y análisis en tiempo real del proceso **Pad Steam**. El sistema actúa como un nodo de **Edge Computing**, procesando datos localmente antes de enviarlos a un servidor central.

## 🏗️ Arquitectura Tecnológica (Stack)
La solución está orquestada totalmente en **Docker** con las versiones más recientes:

* **Ingesta de datos:** Eclipse Mosquitto (MQTT) + Apache NiFi 2.0.
* **Mensajería distribuida:** Apache Kafka (Bitnami) + Kafdrop (Visor).
* **Procesamiento en streaming:** Apache Flink (ML Logic).
* **Base de Datos (OLAP):** ClickHouse (Optimizado para Big Data).
* **Visualización:** Apache Superset & Bot de Telegram (IA Agente).
* **Inteligencia Artificial:** Ollama (Modelos locales para análisis predictivo).

---

## 🔐 Acceso y Seguridad de NiFi 2.0
Debido a las políticas estrictas de seguridad de NiFi 2.0, el acceso requiere una configuración de Proxy específica para evitar el error `Invalid SNI`.

* **URL de Acceso:** [https://100.71.228.107:8443/nifi](https://100.71.228.107:8443/nifi)
* **Credenciales Maestras:**
    * **Usuario:** `admin`
    * **Password:** `admin_password_123`

### 🛠️ Solución al error "Invalid SNI"
Si el acceso es bloqueado (Error 400), verificar que el `docker-compose.yml` contenga las siguientes variables de entorno:
- `NIFI_WEB_PROXY_HOST=100.71.228.107:8443`
- `NIFI_WEB_HTTPS_HOST=0.0.0.0`

> **Nota Crítica:** Si se cambian las credenciales o el Host, es imperativo borrar la carpeta `./deploy/nifi/conf` para que NiFi regenere los certificados `.p12` correctamente en el próximo arranque.

---

## 📡 Arquitectura de Mensajería (MQTT Bridge)
El nodo Kraven gestiona dos flujos de datos simultáneos mediante **Eclipse Mosquitto 2.0.22**:

### 1. Ingesta de Campo (Portenta X8)
El hardware de campo publica datos hacia el nodo Kraven utilizando credenciales locales.
* **IP Servidor:** `172.25.10.236`
* **Puerto:** `1883`
* **Credenciales:** `kraven / kraven123`
* **Tópicos:** `Creditex/Lima/PadSteam` y `Creditex/Lima/PadSteamv3`

### 2. Salida a la Nube (Bridge Creditex)
El nodo actúa como Gateway, reenviando los datos automáticamente al bróker central remoto.
* **Endpoint Remoto:** `181.177.239.92:31621`
* **Client ID:** `kraven_edge_LIMA_01`
* **Credenciales Cloud:** `controlsi / controlsi!`
* **Regla de Reenvío:** Replicación automática de toda la rama `Creditex/#`.

## 📂 Estructura del Proyecto
Organizado bajo el estándar de "Limpieza Total" para facilitar la replicación en los 6 nodos de planta:

```text
padsteam/
├── deploy/          # Configuraciones, XMLs y Dockerfiles de cada servicio
│   ├── nifi/        # Drivers JDBC y configuración de seguridad
│   ├── clickhouse/  # Configuración de usuarios y red
│   └── mosquitto/   # Configuración del bróker MQTT
├── data/            # Persistencia de datos (ClickHouse, Kafka, Zookeeper)
├── docs/            # Manuales, diagramas Mermaid y documentación técnica
├── .env             # Control maestro de puertos y versiones
└── docker-compose.yml

