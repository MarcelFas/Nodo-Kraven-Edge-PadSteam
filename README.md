# Proyecto PadSteam - Infraestructura Edge Computing v2

![ubuntu](https://img.shields.io/badge/ubuntu-24.04-orange)
![docker](https://img.shields.io/badge/docker-latest-blue)
![mqtt](https://img.shields.io/badge/bridge-established-green)
![apache flink](https://img.shields.io/badge/apache_flink-latest-original)
![grafana](https://img.shields.io/badge/grafana-latest-orange)

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
### 📂 Estabilidad de Sistema
```text
user@user-default-string:~$ docker ps -a
CONTAINER ID   IMAGE                               COMMAND                  CREATED        STATUS        PORTS                                                                                                NAMES
99587e21ed5c   eclipse-mosquitto:2.0               "/docker-entrypoint.…"   5 days ago     Up 33 hours   0.0.0.0:1883->1883/tcp, [::]:1883->1883/tcp                                                          ps_mosquitto
bb8b32f19b08   grafana/grafana:latest              "/run.sh"                10 days ago    Up 9 days     0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp                                                          ps_grafana
e6ccd9702b60   padsteam-flink-taskmanager          "/docker-entrypoint.…"   5 weeks ago    Up 3 days     6123/tcp, 8081/tcp                                                                                   ps_flink_taskmanager
74671e725b6f   padsteam-flink-jobmanager           "/docker-entrypoint.…"   5 weeks ago    Up 3 days     6123/tcp, 0.0.0.0:8081->8081/tcp, [::]:8081->8081/tcp                                                ps_flink_jobmanager
fc93de1b7b42   padsteam-nifi                       "../scripts/start.sh"    5 weeks ago    Up 33 hours   8000/tcp, 10000/tcp, 0.0.0.0:8443->8443/tcp, [::]:8443->8443/tcp                                     ps_nifi
bb2127d82620   obsidiandynamics/kafdrop:4.1.0      "/kafdrop.sh"            5 weeks ago    Up 33 hours   0.0.0.0:9002->9000/tcp, [::]:9002->9000/tcp                                                          ps_kafdrop
c768bcf49e1f   padsteam-ps_limpiador               "python -u limpieza_…"   5 weeks ago    Up 33 hours                                                                                                        ps_limpiador
bf2d8bdcae6d   bitnami/kafka:3.6.1                 "/opt/bitnami/script…"   5 weeks ago    Up 33 hours   9092/tcp, 0.0.0.0:29092->29092/tcp, [::]:29092->29092/tcp                                            ps_kafka
1cb9dc1260a3   redis:7-alpine                      "docker-entrypoint.s…"   5 weeks ago    Up 33 hours   6379/tcp                                                                                             ps_redis
a1504f71b84a   bitnami/zookeeper:3.8               "/opt/bitnami/script…"   5 weeks ago    Up 33 hours   2181/tcp, 2888/tcp, 3888/tcp, 8080/tcp                                                               ps_zookeeper
32419c8a08cb   clickhouse/clickhouse-server:24.8   "/entrypoint.sh"         5 weeks ago    Up 23 hours   0.0.0.0:8123->8123/tcp, [::]:8123->8123/tcp, 9009/tcp, 0.0.0.0:9005->9000/tcp, [::]:9005->9000/tcp   ps_clickhouse
f596237a347e   ps_report_generator                 "uvicorn main:app --…"   2 months ago   Up 9 days     0.0.0.0:8000->8000/tcp, [::]:8000->8000/tcp                                                          ps_report_generator
666c56a1ca1e   nginx:alpine                        "/docker-entrypoint.…"   2 months ago   Up 9 days     0.0.0.0:8080->80/tcp, [::]:8080->80/tcp                                                              ps_nginx_reports
7e882d45f279   n8nio/n8n:latest                    "tini -- /docker-ent…"   3 months ago   Up 9 days     0.0.0.0:5678->5678/tcp, [::]:5678->5678/tcp                                                          ps_n8n
765c2aa41b85   bot-ai_agent_bot                    "python -u bot.py"       3 months ago   Up 9 days                                                                                                          ai-agent-bot
4ed0d5997c38   ollama/ollama                       "/bin/ollama serve"      3 months ago   Up 9 days     0.0.0.0:11434->11434/tcp, [::]:11434->11434/tcp                                                      ollama-ia
user@user-default-string:~$



CONTAINER ID   NAME                   CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
99587e21ed5c   ps_mosquitto           0.02%     2.191MiB / 62.64GiB   0.00%     133MB / 96.2MB    4.1kB / 0B        1
bb8b32f19b08   ps_grafana             0.28%     952.8MiB / 62.64GiB   1.49%     76.8GB / 90GB     403MB / 1.91GB    52
e6ccd9702b60   ps_flink_taskmanager   3.53%     2.659GiB / 62.64GiB   4.24%     6.62GB / 4.2GB    729kB / 407MB     387
74671e725b6f   ps_flink_jobmanager    1.24%     1.159GiB / 62.64GiB   1.85%     737MB / 471MB     10.3MB / 536MB    159
fc93de1b7b42   ps_nifi                1.29%     1.555GiB / 62.64GiB   2.48%     92.9MB / 87.1MB   4.55MB / 501MB    119
bb2127d82620   ps_kafdrop             0.08%     468.8MiB / 62.64GiB   0.73%     40.2MB / 16.5MB   8.15MB / 26.5MB   50
c768bcf49e1f   ps_limpiador           0.00%     17.14MiB / 62.64GiB   0.03%     1.59MB / 5.4kB    2.88MB / 0B       1
bf2d8bdcae6d   ps_kafka               0.86%     1.077GiB / 62.64GiB   1.72%     262MB / 621MB     2.27MB / 228MB    89
1cb9dc1260a3   ps_redis               0.69%     3.418MiB / 62.64GiB   0.01%     1.55MB / 126B     36.9kB / 0B       6
a1504f71b84a   ps_zookeeper           0.07%     150.4MiB / 62.64GiB   0.23%     4.5MB / 1.84MB    8.19kB / 348kB    69
32419c8a08cb   ps_clickhouse          61.57%    3.049GiB / 62.64GiB   4.87%     3.95GB / 8.2GB    363MB / 179GB     847
f596237a347e   ps_report_generator    0.06%     83.03MiB / 62.64GiB   0.13%     15.7MB / 5.59kB   96.1MB / 0B       4
666c56a1ca1e   ps_nginx_reports       0.00%     12.09MiB / 62.64GiB   0.02%     7.9MB / 9.75kB    7.82MB / 4.1kB    13
7e882d45f279   ps_n8n                 0.15%     403.8MiB / 62.64GiB   0.63%     8.2MB / 427kB     218MB / 57MB      20
765c2aa41b85   ai-agent-bot           0.00%     73.21MiB / 62.64GiB   0.11%     29.5MB / 52.5MB   66.8MB / 0B       14
4ed0d5997c38   ollama-ia              0.00%     30.43MiB / 62.64GiB   0.05%     7.89MB / 126B     37MB / 0B         11
