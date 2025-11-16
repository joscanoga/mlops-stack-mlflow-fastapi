# Stack MLOps con MLflow, Airflow y FastAPI

Este proyecto proporciona una plantilla completa para implementar flujos de trabajo de Machine Learning Operations (MLOps) utilizando un stack moderno basado en contenedores Docker, con gestión de experimentos mediante MLflow, orquestación de pipelines con Apache Airflow y despliegue de modelos a través de una API REST con FastAPI.

## Descripción General

El objetivo de este proyecto es proporcionar una **arquitectura de MLOps reproducible, escalable y lista para producción**. La solución integra las mejores prácticas para el desarrollo, entrenamiento, seguimiento y despliegue de modelos de Machine Learning.

El stack incluye:

- **MLflow Server**: Gestión centralizada de experimentos, tracking de métricas y registro de modelos.
- **Apache Airflow**: Orquestación y programación de pipelines de entrenamiento y procesamiento de datos.
- **FastAPI**: API REST de alto rendimiento para servir predicciones de modelos en producción.
- **PostgreSQL**: Base de datos relacional para almacenamiento de metadatos de Airflow y MLflow.
- **Docker & Docker Compose**: Contenedorización y orquestación de todos los servicios.

## Prerrequisitos

Antes de comenzar, asegúrate de tener instalado lo siguiente en tu sistema:

- **Docker** (v20.10 o superior) y **Docker Compose** (v2.0 o superior): Para construir y gestionar los contenedores.
  - [Instrucciones de instalación de Docker](https://docs.docker.com/get-docker/)
  - [Instrucciones de instalación de Docker Compose](https://docs.docker.com/compose/install/)

- **Git**: Para clonar el repositorio.

- **Variables de Entorno**: El proyecto utiliza variables de entorno para configuración sensible. Asegúrate de crear un archivo `.env` en la raíz del proyecto con las siguientes variables:

```env
# PostgreSQL Configuration
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB_AIRFLOW=airflow_db
POSTGRES_DB_MLFLOW=mlflow_db

# Airflow Configuration
AIRFLOW__WEBSERVER__SECRET_KEY=tu_clave_secreta_aqui
```

> **Nota de Seguridad**: No subas el archivo `.env` al repositorio. Asegúrate de incluirlo en `.gitignore`.

## Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                         STACK MLOps                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    │
│  │   Airflow    │───▶│   MLflow     │◀───│   FastAPI    │    │
│  │  Webserver   │    │   Server     │    │     APP      │    │
│  │  (Port 8080) │    │  (Port 5000) │    │  (Port 8000) │    │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘    │
│         │                   │                    │             │
│  ┌──────▼───────┐          │                    │             │
│  │   Airflow    │          │                    │             │
│  │  Scheduler   │          │                    │             │
│  └──────┬───────┘          │                    │             │
│         │                  │                    │             │
│         └──────────────────┴────────────────────┘             │
│                            │                                   │
│                    ┌───────▼────────┐                         │
│                    │   PostgreSQL   │                         │
│                    │  (Port 5432)   │                         │
│                    └────────────────┘                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Componentes Principales

1. **PostgreSQL** (`postgres_db`): Base de datos que almacena:
   - Metadatos de Airflow (DAGs, ejecuciones, logs)
   - Metadatos de MLflow (experimentos, runs, métricas)

2. **MLflow Server** (`mlflow_server`): Servidor de tracking que:
   - Registra experimentos y parámetros de modelos
   - Almacena métricas y artefactos
   - Proporciona un registro centralizado de modelos

3. **Airflow Webserver** (`airflow-webserver`): Interfaz web para:
   - Monitorizar y gestionar DAGs
   - Visualizar el estado de las ejecuciones
   - Acceder a logs y métricas

4. **Airflow Scheduler** (`airflow-scheduler`): Ejecutor de tareas que:
   - Programa y ejecuta los DAGs según su configuración
   - Coordina las tareas de entrenamiento y procesamiento

5. **FastAPI App** (`fastapi_app`): API REST que:
   - Carga modelos desde MLflow
   - Expone endpoints para predicciones
   - Realiza inferencia en tiempo real

## Cómo Usar

Sigue estos pasos para levantar el stack completo:

### 1. Clonar el Repositorio

```bash
git clone https://github.com/joscanoga/mlops-stack-mlflow-fastapi.git
cd mlops-stack-mlflow-fastapi
```

### 2. Configurar Variables de Entorno

Crea un archivo `.env` en la raíz del proyecto con las credenciales necesarias (ver sección de Prerrequisitos).

### 3. Construir e Iniciar los Contenedores

Ejecuta el siguiente comando para construir las imágenes y levantar todos los servicios:

```bash
docker-compose up --build
```

- `docker-compose up`: Inicia todos los servicios definidos en `docker-compose.yml`.
- `--build`: Fuerza la reconstrucción de las imágenes Docker (recomendado la primera vez o después de cambios en los Dockerfiles).

Para ejecutar en segundo plano (detached mode):

```bash
docker-compose up -d
```

### 4. Acceder a los Servicios

Una vez que los contenedores estén ejecutándose, podrás acceder a:

| Servicio | URL | Credenciales |
|----------|-----|--------------|
| **Airflow Webserver** | http://localhost:8080 | User: `admin` / Password: `admin` |
| **MLflow UI** | http://localhost:5000 | Sin autenticación |
| **FastAPI Swagger Docs** | http://localhost:8000/docs | Sin autenticación |

> **Nota de Seguridad**: Las configuraciones actuales están diseñadas para desarrollo local. **No uses estas configuraciones en producción** sin implementar autenticación y medidas de seguridad adecuadas.

### 5. Verificar el Estado de los Servicios

Puedes verificar que todos los contenedores estén funcionando correctamente con:

```bash
docker-compose ps
```

Para ver los logs de un servicio específico:

```bash
docker-compose logs -f <nombre_servicio>
```

Ejemplos:
```bash
docker-compose logs -f mlflow_server
docker-compose logs -f airflow-scheduler
docker-compose logs -f fastapi_app
```

### 6. Detener los Servicios

Para detener todos los servicios:

- Si están corriendo en primer plano: presiona `Ctrl + C`
- Si están corriendo en segundo plano:

```bash
docker-compose down
```

Para detener y **eliminar volúmenes** (esto borrará datos persistentes):

```bash
docker-compose down -v
```

## Estructura del Proyecto

```
.
├── airflow/                      # Configuración de Apache Airflow
│   ├── dags/                     # DAGs de Airflow (pipelines)
│   │   └── 01_primer_dag.py      # Ejemplo de DAG básico
│   ├── tests/                    # Tests unitarios para DAGs
│   ├── Dockerfile                # Imagen personalizada de Airflow
│   └── requirements.txt          # Dependencias Python para Airflow
│
├── data/                         # Datos de entrada para proyectos
│   └── ejemplo.csv               # Dataset de ejemplo
│
├── docs/                         # Documentación adicional
│   └── Documentacion.md          # Guía detallada del proyecto
│
├── fastapi_app/                  # Aplicación FastAPI
│   ├── main.py                   # Código principal de la API
│   ├── Dockerfile                # Imagen de FastAPI
│   └── requirements.txt          # Dependencias de FastAPI
│
├── mlflow_custom/                # Configuración personalizada de MLflow
│   └── Dockerfile                # Imagen de MLflow con PostgreSQL
│
├── postgres_data/                # Datos persistentes de PostgreSQL (generado)
├── postgres_init/                # Scripts de inicialización de PostgreSQL
│   └── init-databases.sh         # Script para crear múltiples DBs
│
├── scripts/                      # Scripts auxiliares
│   └── script.sql                # Queries SQL de utilidad
│
├── docker-compose.yml            # Orquestación de servicios
├── README.md                     # Esta documentación
└── LICENSE                       # Licencia del proyecto
```

## Archivos Principales

### `docker-compose.yml`
Orquesta todos los servicios del stack. Define:
- Configuración de red entre contenedores
- Puertos expuestos
- Variables de entorno
- Volúmenes para persistencia de datos
- Dependencias entre servicios

### `airflow/Dockerfile`
Imagen personalizada de Airflow que incluye:
- Apache Airflow 2.8.1 con Python 3.10
- Librería TA-Lib compilada desde código fuente
- Dependencias de ML: MLflow, scikit-learn, XGBoost, Optuna
- Conector para Python-Binance

### `fastapi_app/Dockerfile`
Imagen ligera de FastAPI que incluye:
- Python 3.10 sobre Miniconda
- TA-Lib para análisis técnico
- MLflow client para cargar modelos
- FastAPI y Uvicorn como servidor ASGI

### `mlflow_custom/Dockerfile`
Imagen de MLflow extendida con:
- MLflow oficial como base
- Driver `psycopg2-binary` para conectar con PostgreSQL

## Persistencia de Datos

El proyecto utiliza volúmenes de Docker para garantizar la persistencia de datos:

| Volumen | Contenido | Descripción |
|---------|-----------|-------------|
| `postgres_data` | Base de datos PostgreSQL | Almacena metadatos de Airflow y MLflow |
| `mlruns_data` | Artefactos de MLflow | Modelos entrenados, gráficos, archivos |
| `./airflow/dags` | DAGs de Airflow | Sincronizado con tu sistema local |
| `./data` | Datasets | Datos de entrada para entrenamiento |

Cualquier archivo que modifiques en los directorios locales `./airflow/dags` o `./data` se reflejará automáticamente en los contenedores.

## Ejemplo de Uso

### Ejecutar un DAG de Entrenamiento

1. Accede a Airflow: http://localhost:8080
2. Inicia sesión con `admin` / `admin`
3. Busca tu DAG en la lista
4. Activa el toggle para habilitarlo
5. Presiona el botón "▶" para ejecutarlo manualmente

### Consultar Experimentos en MLflow

1. Accede a MLflow: http://localhost:5000
2. Navega por los experimentos registrados
3. Compara métricas, parámetros y visualizaciones
4. Descarga modelos o marca versiones para producción

### Realizar Predicciones con FastAPI

1. Accede a la documentación interactiva: http://localhost:8000/docs
2. Prueba el endpoint `/predict_btc_trend`
3. Envía datos históricos en formato JSON
4. Recibe la predicción del modelo en producción

Ejemplo de request con `curl`:

```bash
curl -X POST "http://localhost:8000/predict_btc_trend" \
     -H "Content-Type: application/json" \
     -d '{
       "data": [
         {
           "open_time": 1634567890000,
           "open": 50000.0,
           "high": 51000.0,
           "low": 49500.0,
           "close": 50500.0,
           "volume": 1000.5
         }
       ]
     }'
```

## Desarrollo y Personalización

### Agregar Nuevas Dependencias

#### Para Airflow:
1. Edita `airflow/requirements.txt`
2. Reconstruye la imagen:
   ```bash
   docker-compose build airflow-scheduler
   docker-compose up -d
   ```

#### Para FastAPI:
1. Edita `fastapi_app/requirements.txt`
2. Reconstruye la imagen:
   ```bash
   docker-compose build fastapi_app
   docker-compose up -d
   ```

### Crear un Nuevo DAG

1. Crea un archivo Python en `airflow/dags/`
2. Define tu DAG siguiendo el ejemplo de `01_primer_dag.py`
3. Airflow detectará automáticamente el nuevo DAG (puede tomar ~30 segundos)

### Modificar la API de FastAPI

1. Edita `fastapi_app/main.py`
2. Reinicia el contenedor:
   ```bash
   docker-compose restart fastapi_app
   ```

## Solución de Problemas

### Los contenedores no inician

**Posibles causas y soluciones:**

- **Docker no está ejecutándose**: Asegúrate de que Docker Desktop esté activo.
- **Puertos ocupados**: Verifica que los puertos 5000, 5432, 8000 y 8080 no estén siendo usados por otras aplicaciones:
  ```bash
  netstat -an | findstr "5000 5432 8000 8080"
  ```
- **Variables de entorno faltantes**: Verifica que el archivo `.env` exista y contenga todas las variables necesarias.

### Airflow no muestra mis DAGs

**Soluciones:**

- Verifica que el archivo esté en `./airflow/dags/`
- Revisa los logs del scheduler:
  ```bash
  docker-compose logs airflow-scheduler
  ```
- Asegúrate de que no haya errores de sintaxis en tu DAG

### MLflow no guarda los experimentos

**Soluciones:**

- Verifica que PostgreSQL esté funcionando:
  ```bash
  docker-compose ps postgres_db
  ```
- Revisa la conexión en los logs de MLflow:
  ```bash
  docker-compose logs mlflow_server
  ```
- Confirma que las credenciales en `.env` sean correctas

### FastAPI no carga el modelo

**Soluciones:**

- Verifica que el modelo esté etiquetado como `production_candidate` en MLflow
- Revisa los logs de FastAPI al iniciar:
  ```bash
  docker-compose logs fastapi_app
  ```
- Asegúrate de que MLflow Server esté accesible desde FastAPI

### Error "Out of Memory"

**Soluciones:**

- Aumenta la memoria asignada a Docker Desktop (Settings → Resources → Memory)
- Reduce el tamaño de batch en tus modelos
- Limita el número de workers de Airflow

## Mejores Prácticas

### Seguridad
- **No uses contraseñas por defecto en producción**
- Implementa autenticación y autorización en todos los servicios
- Usa secretos de Docker/Kubernetes para información sensible
- Habilita HTTPS/TLS para comunicaciones entre servicios

### Monitoreo
- Configura alertas en Airflow para fallos de DAGs
- Implementa logging centralizado (ELK, Loki, etc.)
- Monitoriza el uso de recursos con Prometheus + Grafana

### Escalabilidad
- Migra a Kubernetes para orquestación en producción
- Usa CeleryExecutor o KubernetesExecutor para Airflow
- Implementa balanceo de carga para la API de FastAPI
- Considera usar almacenamiento en la nube (S3, Azure Blob) para artefactos de MLflow

## Tecnologías Utilizadas

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **Apache Airflow** | 2.8.1 | Orquestación de workflows de ML |
| **MLflow** | 2.8.0 | Gestión de experimentos y modelos |
| **FastAPI** | Latest | API REST de alto rendimiento |
| **PostgreSQL** | 13-alpine | Base de datos relacional |
| **Python** | 3.10 | Lenguaje principal del stack |
| **Docker** | 20.10+ | Contenedorización |
| **Docker Compose** | 2.0+ | Orquestación de contenedores |


## Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Haz un fork del repositorio
2. Crea una rama para tu feature (`git checkout -b feature/nueva-funcionalidad`)
3. Haz commit de tus cambios (`git commit -am 'Agrega nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Abre un Pull Request

## Licencia

Este proyecto está bajo la licencia especificada en el archivo [LICENSE](LICENSE).

## Recursos Adicionales

- [Documentación de MLflow](https://mlflow.org/docs/latest/index.html)
- [Documentación de Apache Airflow](https://airflow.apache.org/docs/)
- [Documentación de FastAPI](https://fastapi.tiangolo.com/)
- [Best Practices for MLOps](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)

---

**Autor**: [joscanoga](https://github.com/joscanoga)  
**Repositorio**: [mlops-stack-mlflow-fastapi](https://github.com/joscanoga/mlops-stack-mlflow-fastapi)
