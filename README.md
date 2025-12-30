# proyecto padsteam - infraestructura edge computing v2

![ubuntu](https://img.shields.io/badge/ubuntu-24.04-orange)
![docker](https://img.shields.io/badge/docker-latest-blue)
![licencia](https://img.shields.io/badge/ambiente-industrial-green)

**desarrollado por:** fabrissio fasabi rivera  
**ubicación:** creditex s.a.a. - planta lima, perú (área de tintorería)  
**nodo:** kraven edge-gateway-01

---

## 🚀 visión general
este proyecto implementa una infraestructura tecnológica escalable para el monitoreo y análisis en tiempo real del proceso **pad steam**. el sistema actúa como un nodo de **edge computing**, procesando datos localmente antes de enviarlos a un servidor central.

## 🏗️ arquitectura tecnológica (stack)
la solución está orquestada totalmente en **docker** con las versiones más recientes:

* **ingesta de datos:** eclipse mosquitto (mqtt) + apache nifi.
* **mensajería distribuida:** apache kafka (bitnami).
* **procesamiento en streaming:** apache flink (pyflink para lógica ml).
* **base de datos (olap):** clickhouse (optimizado para grandes volúmenes).
* **visualización:** apache superset & bot de telegram (ia agente).
* **inteligencia artificial:** ollama (modelos locales para análisis predictivo).

## 📂 estructura del proyecto
hemos organizado el nodo bajo el estándar de "limpieza total":

```text
padsteam/
├── deploy/          # configuraciones y dockerfiles de cada servicio
├── data/            # persistencia de datos (volúmenes de clickhouse/kafka)
├── docs/            # manuales y diagramas técnicos
└── .env             # control maestro de puertos y versiones
