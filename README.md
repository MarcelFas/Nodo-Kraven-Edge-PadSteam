# Proyecto PadSteam - Infraestructura Edge Computing v2

![ubuntu](https://img.shields.io/badge/ubuntu-24.04-orange)
![docker](https://img.shields.io/badge/docker-latest-blue)
![licencia](https://img.shields.io/badge/ambiente-industrial-green)

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
* **Visualización:** Apache Superset & Bot de Telegram.
* **Inteligencia Artificial:** Ollama (Modelos locales Llama 3).

---

## 📂 Estructura del Proyecto
Organizado bajo el estándar de "Limpieza Total" para facilitar la replicación en los 6 nodos de planta:

```text
padsteam/
├── deploy/          # Configuraciones, XMLs y Dockerfiles de cada servicio
├── data/            # Persistencia de datos (ClickHouse, Kafka, Zookeeper)
├── docs/            # Manuales, diagramas Mermaid y documentación técnica
├── .env             # Control maestro de puertos y versiones
└── docker-compose.yml
