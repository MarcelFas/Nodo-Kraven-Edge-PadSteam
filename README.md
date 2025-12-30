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

## 🔄 flujo de datos (pipeline)
el sistema sigue una arquitectura lineal y resiliente, asegurando que no se pierdan datos incluso si la base de datos está en mantenimiento:

1. **captura:** los sensores envían tramas json vía **mqtt** al broker local.
2. **orquestación:** **nifi** consume de mqtt, valida el esquema y lo publica en **kafka**.
3. **buffering:** **kafka** actúa como sala de espera, permitiendo que múltiples consumidores (flink, bot, etc.) lean los datos.
4. **persistencia:** los datos finales se guardan en **clickhouse** para consultas analíticas de alta velocidad.



```mermaid
graph LR
    subgraph "planta (campo)"
        A[sensores padsteam] --> B(mosquitto:1883)
    end
    subgraph "nodo edge (kraven)"
        B --> C{apache nifi}
        C --> D[apache kafka]
        D --> E[(clickhouse)]
        D --> F[apache flink]
    end
    subgraph "interfaz"
        E --> G[superset]
        F --> H[ia agent bot]
        H --> I[telegram]
    end
