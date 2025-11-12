# Lecciones aprendidas

## Contexto del proyecto
 Hemos aprendido a implementar automatizaciones CI/CD que permiten: 
 - Validar el correcto funcionamiento de del códgio (unit_tests)
 - Registrar métricas modelos de los experimentos realizados, según variaciones del código (en este casos modelos de ML), así como, los datos empleados para su entrenamiento en un repositorio de MLFlow. 
 - Construir una imagen docker con todas las dependencias requeridas para el experimento, desplegar el modelo ganador del repositorio de MLFlow en un contendor alojado en Azure y diponibilizarlo a través de una API para su uso. 

## Lo que salió bien
- Pipeline de build y push a ACR estable.
- Tests unitarios en PR obligatorios (status check).

## Lo que costó / Errores y cómo los resolvimos
### 1) Docker context y COPY
- **Síntoma:** `COPY requirements.txt` fallaba.
- **Causa raíz:** Dockerfile en `deployment/` con contexto mal pasado; `..` no funciona en `COPY`.
- **Fix:** `docker build -f deployment/Dockerfile .` (contexto en raíz) y paths coherentes.

### 2) Azure Region vacía → osType `<null>`
- **Síntoma:** `InvalidOsType ... container group '<null>'`.
- **Causa raíz:** `--location` sin valor (secret vacío) descolocó flags.
- **Fix:** Validaciones previas en el paso de Actions (`: "${{ secrets.AZURE_REGION:?...}}"`) y uso de arrays para el comando.
