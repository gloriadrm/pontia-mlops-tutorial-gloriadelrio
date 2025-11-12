# 🚀 Proyecto: Automatizaciones CI/CD aplicadas a Modelos de Machine Learning

> "Automatizar no es solo optimizar tiempo, sino asegurar calidad y reproducibilidad en cada entrega."

## 🧩 Descripción del proyecto

Este repositorio implementa un **pipeline completo de CI/CD** que gestiona el ciclo de vida de un modelo de Machine Learning (ML).

El sistema se encarga de:

* Construir y validar el código mediante **pruebas unitarias automáticas**.

* Registrar y versionar modelos de ML en un servidor de **MLflow**.

* Construir una **imagen Docker** con el modelo ganador y sus dependencias.

* **Desplegar automáticamente** el modelo en **Azure Container Instances (ACI)**.

* Exponer una **API REST con FastAPI** para realizar inferencias del modelo desplegado.

El objetivo principal es **automatizar el ciclo de vida del modelo** (entrenamiento → validación → despliegue), garantizando la **reproducibilidad**, la **trazabilidad** y la **disponibilidad** continua.

---

## 📂 Estructura del Repositorio



---

## 🛠️ Tecnologías Utilizadas

Este proyecto se basa en un *stack* de MLOps robusto que garantiza la automatización y la trazabilidad:

| Categoría | Tecnología | Uso Específico | 
| :--- | :--- | :--- | 
| **Integración Continua** | **GitHub Actions** | Automatización completa de los *pipelines* CI/CD, incluyendo *testing*, *build* de Docker y despliegue. | 
| **Modelos & Experimentación** | **Python** | Lenguaje principal de desarrollo y *scripting*. | 
|  | **MLflow** | Plataforma centralizada para **registro, *tracking* y versionado** de los modelos de Machine Learning. | 
| **Despliegue & Contenedores** | **Docker** | Empaquetado del modelo y la API en un entorno reproducible. | 
|  | **Azure Container Registry (ACR)** | Almacenamiento privado de las imágenes Docker listas para producción. | 
|  | **Azure Container Instances (ACI)** | Servicio de cómputo para el despliegue rápido y escalable del contenedor. | 
| **API** | **FastAPI** | Framework moderno y rápido para exponer el modelo como una **API REST** de inferencia. | 
| **Testing** | **Pytest** | Ejecución de pruebas unitarias (`unit_tests/`) y pruebas de modelos (`model_tests/`). | 

---

## 🔁 Flujo del Pipeline CI/CD

El ciclo de vida del modelo se gestiona mediante tres *pipelines* principales en **GitHub Actions**, diseñados para operar de forma secuencial y automatizada:

### 1. `integration.yml` (Integración y Validación del Código)

Se activa con cada *push* o *Pull Request*.

1. **Linting (Flake8):** Verifica el código en `src/` y `scripts/` para asegurar la adherencia a estándares de estilo (PEP 8).

2. **Tests Unitarios:** Ejecuta las pruebas definidas en `unit_tests/` y `model_tests/` usando **Pytest** para validar el código y la lógica del modelo.

3. **Artefactos:** Sube los resultados de los tests (XML y reporte HTML de cobertura) para su revisión.

### 2. `build.yml` (Entrenamiento y Registro del Modelo)

Este *pipeline* se centra en la evaluación del rendimiento y el registro en MLflow.

1. **Entrenamiento y Evaluación:** El *script* principal (`src/main.py`) entrena el modelo y evalúa su rendimiento.

2. **Seguimiento (Tracking):** Los resultados, parámetros y métricas se registran en **MLflow**.

3. **Registro de Modelo:** Si el modelo cumple con un umbral de rendimiento predefinido, se registra y versiona automáticamente en el servidor de **MLflow**.

### 3. `deploy.yml` (Despliegue Continuo)

Se activa después de que un modelo es validado y pasa a un estado de **producción** o **staging** en MLflow.

1. **Consulta de Modelo:** Se obtiene el URI del modelo **ganador** y validado desde MLflow.

2. **Construcción de Imagen Docker:** Se construye la imagen Docker (`deployment/Dockerfile`) que incluye el modelo y la API.

3. **Push a ACR:** La imagen se sube a **Azure Container Registry (ACR)**.

4. **Despliegue a ACI:** Se crea o actualiza el contenedor utilizando **Azure Container Instances (ACI)**.

---

## ⚙️ Cómo ejecutar el proyecto localmente

### 1️⃣ Requisitos previos

Para ejecutar y desplegar el proyecto se necesitan:

* **Python 3.10** o superior.

* **Docker**.

* Cuenta de **Azure** y permisos sobre un **Resource Group**.

### 2️⃣ Configuración de Variables y Secrets en GitHub Actions

Las siguientes variables y *secrets* deben estar configuradas en el repositorio de GitHub para que los pipelines de CI/CD funcionen correctamente:

| Tipo | Nombre | 
| :--- | :--- |
| **Secrets** | `ACR_NAME`, `ACR_USERNAME`, `ACR_PASSWORD`, `AZURE_CREDENTIALS`, `AZURE_RESOURCE_GROUP`, `AZURE_STORAGE_CONNECTION_STRING` | 
| **Variables** | `MODEL_NAME`, `MODEL_ALIAS`, `IMAGE_NAME`, `AZURE_REGION`, `MLFLOW_URL`, `EXPERIMENT_NAME` | 

### 3️⃣ Endpoints principales (FastAPI)

El modelo desplegado se expone a través de los siguientes *endpoints* de la API REST:

| Método | Endpoint | Descripción | 
| :--- | :--- | :--- |
| `GET` | `/health` | Verifica que la API está activa y lista para recibir peticiones. | 
| `POST` | `/predict` | Recibe datos de entrada en formato **JSON** y devuelve la predicción del modelo. | 

---

## ✅ Validación del despliegue

Una vez que el contenedor en ACI esté en estado **`Running`**, la API es accesible públicamente en la URL generada:

`http://<dns-name-label>.<region>.azurecontainer.io:8080`

Puedes comprobar su estado haciendo una llamada sencilla:

```bash
curl http://<dns-name-label>.<region>.azurecontainer.io:8080/health