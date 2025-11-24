# Lecciones aprendidas

## Contexto del proyecto
Hemos aprendido a implementar automatizaciones CI/CD que permiten:  
- Validar el correcto funcionamiento del código mediante **unit tests**.  
- Registrar las métricas de los modelos de los experimentos realizados (en este caso, modelos de *Machine Learning*), así como los datos empleados para su entrenamiento, en un repositorio de **MLflow**.  
- Construir una imagen Docker con todas las dependencias requeridas para el experimento, desplegar el modelo ganador del repositorio de MLflow en un contenedor alojado en **Azure**, y disponibilizarlo a través de una **API** para su uso.

## Lo que costó / Errores y cómo los resolvimos

### 1) Docker context y `COPY`
- **Síntoma:** `COPY requirements.txt` fallaba durante la construcción de la imagen.  
- **Causa raíz:** El `Dockerfile` estaba dentro de la carpeta `deployment/`, pero el contexto del *build* no incluía la raíz del proyecto, por lo que Docker no encontraba el archivo.  
- **Solución:** Ejecutar `docker build -f deployment/Dockerfile .` (con el contexto en la raíz) y ajustar las rutas de `COPY` para que sean coherentes.

### 2) Azure Region vacía → `osType <null>`
- **Síntoma:** `InvalidOsType ... container group '<null>'`.  
- **Causa raíz:** El parámetro `--location` se quedaba sin valor (la variable o secret de `AZURE_REGION` estaba mal referenciada). Esto descolocaba los flags del comando `az container create`.  
- **Solución:** Asegurar que cada valor se define correctamente como *variable* o *secret* en GitHub, según corresponda, y mantener coherencia en todas las referencias.  
- **Mejora adicional:** En el paso *Deploy to Azure Container Instances*, encapsular el comando en un array `CMD=( ... )` para evitar errores por tabulaciones o saltos de línea mal interpretados.
