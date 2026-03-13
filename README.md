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
* **Procesamiento en streaming:** Motores Python personalizados (Lógica de Analítica y Alarmas).
* **Base de Datos (OLAP):** ClickHouse (Optimizado para Time-Series y Big Data).
* **Visualización:** Apache Superset & Bot de Telegram (IA Agente).
* **Inteligencia Artificial:** Ollama (Modelos locales para análisis predictivo).

---

## 🔐 Acceso y Seguridad de Sistemas
Debido a las políticas de seguridad de NiFi 2.0 y la integración de servicios, se han estandarizado las siguientes credenciales:

| Servicio | URL / Acceso | Usuario | Password |
| :--- | :--- | :--- | :--- |
| **Apache NiFi** | [https://100.71.228.107:8443](https://100.71.228.107:8443) | `admin` | `admin_password_123` |
| **Apache Superset** | [http://100.71.228.107:8088](http://100.71.228.107:8088) | `admin` | `admin123` |
| **ClickHouse** | Puerto `8123` (HTTP) | `default` | `password12345` |
| **MQTT Broker** | Puerto `1883` | `kraven` | `kraven123` |
| **Kafdrop** | [http://100.71.228.107:9002](http://100.71.228.107:9002) | *Acceso Libre* | *N/A* |

### 🛠️ Solución al error "Invalid SNI" en NiFi
Si el acceso es bloqueado, verificar en `docker-compose.yml`:
- `NIFI_WEB_PROXY_HOST=100.71.228.107:8443`
- `NIFI_WEB_HTTPS_HOST=0.0.0.0`
> **Nota:** Si se cambian credenciales, borrar `./deploy/nifi/conf` para regenerar certificados.

---

## 🔄 Pipeline de Procesamiento (Microservicios)
El flujo de datos se procesa en tres capas desacopladas para garantizar cero pérdida de información:

### 1. Ingesta y Aplanamiento (`ps_ingestor`)
Ubicado en `./ingestor`, este microservicio consume de Kafka y realiza:
* **Flattening Dinámico:** Transforma JSONs anidados en filas planas compatibles con ClickHouse.
* **Mapeo de Áreas:** Clasifica sensores en *Lavadoras 1, Lavadoras 2, Vaporizador e Impacta*.
* **Timezone Sync:** Asegura que toda la data se registre en hora exacta de **Lima, Perú**.

### 2. Motor de Alarmas (`ps_alarms`)
Ubicado en `./deploy/flink/jobs`, vigila la estabilidad del proceso:
* **Vigilancia de Umbrales:** Compara `valor` vs `límites` en milisegundos.
* **Severidad:** Clasifica eventos en *Posible Alarma, Baja, Media o Alta* según persistencia.
* **Inicio de Evento:** Registra el `timestamp` original del inicio de la desviación.

### 3. Analítica Avanzada (`ps_analytics`)
Ubicado en `./deploy/flink/jobs`, genera métricas sobre ventanas de 60 min:
* **Estadística Real-Time:** Promedio, Desviación Estándar y Coeficiente de Variación.
* **Dinámica de Señal:** Calcula Velocidad de Cambio (Derivada) y Aceleración.
* **Saturación:** Mide el % de tiempo que una señal opera cerca de sus límites críticos.

---
---

---

## 🗄️ Modelo de Datos (ClickHouse)
La persistencia de datos se organiza en la base de datos `padsteam`, utilizando el motor **MergeTree** para garantizar inserciones de alta velocidad y consultas analíticas eficientes. El diseño sigue un esquema de **Entidad-Atributo-Valor (EAV)** para permitir escalabilidad sin modificar la estructura de las tablas.

### 1. Tablas Principales

| Tabla | Propósito Analítico | Llave de Ordenamiento (`ORDER BY`) |
| :--- | :--- | :--- |
| **`operaciones_maquina`** | Registro de estados (Produciendo/Parada), velocidad, lote y receta. Ideal para KPIs de disponibilidad y OEE. | `(lote, maquina, timestamp)` |
| **`senales_padsteam`** | Telemetría cruda aplanada y clasificada por área de proceso. Base para gráficos de tendencias y líneas de tiempo. | `(lote, desc_variable, timestamp)` |
| **`eventos_alarmas`** | Histórico de desviaciones, gestión de severidad (Baja/Media/Alta) y persistencia de alarmas (MTTR). | `(timestamp, codigo_senal)` |
| **`analitica_senal`** | Métricas calculadas en streaming: Coeficiente de variación, aceleración, error normalizado y niveles de saturación. | `(timestamp, desc_variable)` |

### 2. Diccionario de Áreas (Catálogo Dinámico)
El sistema clasifica automáticamente cada sensor (`registro_id`) en una de las siguientes áreas operativas mediante el microservicio de ingesta:
* **Lavadoras 1 / Lavadoras 2**: Monitoreo de tensiones, temperaturas y caudales de lavado.
* **Vaporizador**: Control crítico de presión y temperatura de vaporización.
* **Impacta**: Gestión de dosificación, volúmenes de químicos y Pick-Up.
* **Otras Áreas**: Sensores de monitoreo de condición (vibración/ambiente) y consumos eléctricos.


## 🤖 Automatización y Orquestación de Reportes (`n8n / IA`)
Esta capa actúa como el puente entre los datos procesados y la toma de decisiones gerenciales, utilizando **n8n** como núcleo de integración y modelos de lenguaje locales:

* **Generación de Reportes Automáticos:** Orquestador avanzado en Python (`report_orchestrator.py`) que automatiza la extracción de datos desde ClickHouse para generar reportes de supervisión diaria en formato **LaTeX** y PDF.
* **Gestión de Turnos Industriales:** Lógica especializada en `shift_operations.py` para la segmentación de KPIs (Productividad, Paradas, Consumos) según los turnos rotativos de la planta Creditex.
* **Analítica de Señales Dinámica:** Implementación de scripts (`signals_analytics.py`) para el cálculo de métricas post-proceso que alimentan los reportes de calidad.
* **Inteligencia Artificial Local:** Integración de **Ollama** para el análisis de causa-raíz, procesando las alertas de Flink para sugerir acciones correctivas basadas en el comportamiento histórico.

## 📊 Visualización y Monitoreo Estratégico (Grafana)
Implementación de un centro de control visual de alta fidelidad, optimizado para entornos industriales críticos:

* **Dashboards de Alta Disponibilidad:** Conexión directa a ClickHouse mediante motor OLAP para visualizaciones en tiempo real sin latencia.
* **Infraestructura de Plugins Modificada:** Configuración de entorno seguro en Docker para la carga de plugins comunitarios esenciales, superando las restricciones de firma mediante variables de entorno:
    * **Flowcharting:** Para la representación esquemática, animada y en tiempo real del flujo de tela en la máquina Pad Steam.
    * **VolkovLabs ECharts:** Para el análisis de correlación multivariable y diagramas de dispersión complejos.
    * **Dynamic Text:** Para la generación de avisos y alertas textuales dinámicas basadas en el estado del proceso.
* **Gestión de Seguridad de Capa:** Implementación de `GF_PLUGINS_SKIP_SIGNATURE_VERIFICATION` para garantizar la interoperabilidad de componentes visuales personalizados sin comprometer la estabilidad del nodo Edge.
---
### 📂 Estructura del Proyecto
La organización del repositorio sigue un estándar de arquitectura de microservicios para facilitar la portabilidad del nodo hacia otros Gateways de la planta:

```text
padsteam/
├── config/                     # Configuraciones maestras (YAML) y credenciales
├── data/                       # Persistencia de volúmenes (Grafana Plugins, CH, Ollama)
├── deploy/                     # Definiciones Docker por servicio (NiFi, Bot IA, Superset)
├── ingestor/                   # Microservicios de ingesta y limpieza de DB (Python)
├── n8n/                        # Orquestador de reportes, analítica y generador LaTeX
│   ├── main.py                 # Punto de entrada del orquestador
│   ├── shift_operations.py     # Lógica de gestión de turnos industriales
│   └── latex_generator.py      # Motor de creación de reportes formales
├── scripts/                    # Utilidades de mantenimiento y automatización
├── www/                        # Archivos estáticos y web logs
├── .env                        # Control maestro de variables de entorno
├── docker-compose.yml          # Orquestador principal de infraestructura
├── docker-compose-grafana.yml  # Orquestador de visualización avanzada
├── docker-compose-flink.yml    # Orquestador de procesamiento streaming
└── docker-compose-web.yml      # Interfaz y servicios adicionales
