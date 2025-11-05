# Documentación de Pipelines CI/CD

Este documento explica qué hace cada pipeline implementado en el proyecto.

## Estructura General

Los pipelines están organizados en dos niveles:
- **Templates reutilizables** en `ci-pipelines/.github/workflows/` (workflows que pueden ser llamados por otros)
- **Pipelines específicos** en cada servicio (workflows que llaman a los templates)

---

## Pipelines Templates (Reutilizables)

### 1. build.yml
**Ubicación**: `ci-pipelines/.github/workflows/build.yml`  
**Tipo**: Template reutilizable (workflow_call)

**Propósito**: Construir y publicar imágenes Docker al Azure Container Registry (ACR)

**Proceso**:
1. **Checkout del código**: Obtiene el código del repositorio
2. **Autenticación Docker Hub**: Inicia sesión en Docker Hub
3. **Configuración Docker Buildx**: Configura el builder de Docker
4. **Autenticación ACR**: Inicia sesión en Azure Container Registry
5. **Extracción de nombre de imagen**: Extrae el nombre del servicio desde el nombre del repositorio
6. **Determinación de tag**: Convierte el tag según la rama:
   - `main` → `prod`
   - `develop` → `stage`
   - Otros → se mantiene el tag original
7. **Construcción de imagen Docker**: Construye la imagen sin hacer push (para escaneo)
8. **Push a ACR**: Construye y publica la imagen al Azure Container Registry con el tag determinado
9. **Deploy a Kubernetes**: Llama al workflow `deploy-kube.yml` para desplegar en Kubernetes

**Entradas**:
- `image_name`: Nombre de la imagen (requerido)
- `image_tag`: Tag de la imagen (default: dev)
- `dockerfile_path`: Ruta al Dockerfile (default: ./Dockerfile)
- `context`: Contexto de build (default: .)

**Salidas**:
- `image_name`: Nombre de la imagen extraído
- `final_tag`: Tag final utilizado
- `service_name`: Nombre del servicio

---

### 2. deploy-kube.yml
**Ubicación**: `ci-pipelines/.github/workflows/deploy-kube.yml`  
**Tipo**: Template reutilizable (workflow_call)

**Propósito**: Desplegar una imagen Docker en un cluster de Kubernetes (AKS)

**Proceso**:
1. **Checkout del código**: Obtiene el código del repositorio
2. **Extracción de nombre de servicio**: Extrae el nombre del servicio desde el nombre de la imagen
3. **Determinación de namespace**: Define el namespace de Kubernetes (actualmente siempre usa 'dev')
4. **Autenticación Azure**: Inicia sesión en Azure
5. **Instalación kubectl**: Instala kubectl
6. **Obtención de credenciales AKS**: Obtiene las credenciales del cluster AKS
7. **Clonación de repositorio de infraestructura**: Clona el repositorio de infraestructura con los manifiestos de Kubernetes
8. **Verificación de imagen en ACR**: Verifica que la imagen con el tag especificado existe en ACR
9. **Creación de secret ACR**: Asegura que el secret para autenticar con ACR existe en el namespace
10. **Creación de ConfigMap**: Asegura que el ConfigMap con variables de entorno existe
11. **Creación de secret MySQL**: Asegura que el secret de MySQL existe
12. **Actualización de manifiesto**: Actualiza el manifiesto de Kubernetes con la nueva imagen
13. **Aplicación de manifiesto**: Aplica el manifiesto actualizado en Kubernetes
14. **Enforcement de imagen**: Fuerza la imagen final como medida de seguridad
15. **Espera de rollout**: Espera a que el deployment se complete exitosamente
16. **Diagnósticos en caso de fallo**: Si falla, muestra información de diagnóstico

**Entradas**:
- `image_name`: Nombre de la imagen (requerido)
- `image_tag`: Tag de la imagen (requerido)
- `namespace`: Namespace de Kubernetes (requerido)
- `manifest_path`: Ruta al manifiesto (opcional)

---

### 3. run-e2e.yml
**Ubicación**: `ci-pipelines/.github/workflows/run-e2e.yml`  
**Tipo**: Template reutilizable (workflow_call y workflow_dispatch)

**Propósito**: Ejecutar pruebas end-to-end (E2E) contra servicios desplegados en Kubernetes

**Proceso**:
1. **Checkout de templates**: Clona el repositorio de templates CI
2. **Autenticación Azure**: Inicia sesión en Azure
3. **Instalación kubectl**: Instala kubectl
4. **Obtención de credenciales AKS**: Obtiene las credenciales del cluster AKS
5. **Obtención de IP del API Gateway**: Obtiene la IP pública del API Gateway desde Kubernetes
6. **Verificación de servicios en Eureka**: Verifica que todos los servicios están registrados en Eureka:
   - API-GATEWAY
   - USER-SERVICE
   - PRODUCT-SERVICE
   - PAYMENT-SERVICE
   - ORDER-SERVICE
   - SHIPPING-SERVICE
   - FAVOURITE-SERVICE
   - PROXY-CLIENT
7. **Actualización de colección Postman**: Actualiza la colección de Postman con la IP dinámica del API Gateway
8. **Actualización de test K6**: Actualiza el test de rendimiento K6 con la IP dinámica
9. **Instalación Node.js**: Instala Node.js versión 18
10. **Instalación K6**: Instala K6 para pruebas de rendimiento
11. **Ejecución de prueba de rendimiento**: Ejecuta el test K6
12. **Instalación Newman**: Instala Newman (ejecutor de Postman)
13. **Ejecución de pruebas Postman**: Ejecuta la colección de Postman con Newman

**Entradas**:
- `namespace`: Namespace donde están desplegados los servicios (default: dev)

**Triggers**:
- Manual (workflow_dispatch)
- Llamado desde otros workflows (workflow_call)

---

### 4. remote-e2e.yml
**Ubicación**: `ci-pipelines/.github/workflows/remote-e2e.yml`  
**Tipo**: Template reutilizable (workflow_call)

**Propósito**: Wrapper que llama al workflow `run-e2e.yml` para ejecutar pruebas E2E remotas

**Proceso**:
- Simplemente llama al workflow `run-e2e.yml` pasando los parámetros necesarios

**Uso**: Se utiliza como intermediario para simplificar la llamada desde otros workflows

---

### 5. pr-stage.yaml
**Ubicación**: `ci-pipelines/.github/workflows/pr-stage.yaml`  
**Tipo**: Template reutilizable (workflow_call)

**Propósito**: Pipeline para Pull Requests a la rama `stage`

**Proceso**:
1. **Build y Deploy**: Llama al workflow `build.yml` con:
   - `image_tag: stage`
   - Esto construye la imagen, la publica en ACR con tag `stage` y la despliega en Kubernetes
2. **Ejecución de E2E**: Después del build exitoso, llama al workflow `remote-e2e.yml` para ejecutar pruebas E2E

**Entradas**:
- `image_name`: Nombre de la imagen (requerido)

**Uso**: Se llama desde los workflows de cada servicio cuando se hace un PR a la rama `stage`

---

### 6. pr-dev.yaml
**Ubicación**: `ci-pipelines/.github/workflows/pr-dev.yaml`  
**Tipo**: Template reutilizable (workflow_call)

**Propósito**: Pipeline para Pull Requests a la rama `develop` - Análisis de código con SonarCloud

**Proceso**:
1. **Sanitización de nombres**: Sanitiza el nombre de la rama y del proyecto para SonarCloud
2. **Checkout del código**: Obtiene el código del repositorio
3. **Configuración Java 11**: Configura Java 11 para compilar
4. **Configuración SonarCloud**: 
   - Descarga el archivo `sonar-project.properties` desde los templates
   - Configura el proyecto de SonarCloud con el nombre y key sanitizados
5. **Ejecución de pruebas unitarias**: Ejecuta `mvn test`
6. **Ejecución de pruebas de integración**: Ejecuta `mvn verify`

**Entradas**:
- `projectKeyBase`: Base para el key del proyecto (default: nombre del repositorio)
- `projectNameBase`: Base para el nombre del proyecto (default: nombre del repositorio)

**Uso**: Se llama desde los workflows de cada servicio cuando se hace un PR a la rama `develop`

---

### 7. release.yml
**Ubicación**: `ci-pipelines/.github/workflows/release.yml`  
**Tipo**: Template reutilizable (workflow_call)

**Propósito**: Generar releases automáticos usando semantic-release

**Proceso**:
1. **Checkout del código**: Obtiene el código con historial completo
2. **Configuración Node.js**: Configura Node.js (default: versión 22)
3. **Instalación de dependencias**: Ejecuta `npm ci`
4. **Verificación de token**: Verifica que el token de GitHub está disponible
5. **Ejecución de semantic-release**: Ejecuta `npx semantic-release` para generar releases automáticos basados en commits convencionales

**Entradas**:
- `release_branch`: Rama para releases (default: main)
- `node_version`: Versión de Node.js (default: 22)

**Uso**: Se utiliza para generar versiones automáticamente basándose en los commits

---

## Pipelines Específicos de Servicios

### 1. build.yml (en cada servicio)
**Ubicación**: `{service}/.github/workflows/build.yml`  
**Ejemplo**: `cloud-config/.github/workflows/build.yml`

**Propósito**: Pipeline principal de cada servicio que se ejecuta en push o manualmente

**Triggers**:
- `workflow_dispatch`: Ejecución manual
- `push` a ramas: `main`, `stage`, `develop`

**Proceso**:
- Llama al template `build.yml` pasando:
  - `image_name`: Nombre del repositorio
  - `image_tag`: Nombre de la rama (main, stage, develop)
  - `dockerfile_path`: ./Dockerfile
  - `context`: .

**Resultado**: Construye, publica y despliega el servicio automáticamente

---

### 2. pr-to-stage.yml (en cada servicio)
**Ubicación**: `{service}/.github/workflows/pr-to-stage.yml`  
**Ejemplo**: `cloud-config/.github/workflows/pr-to-stage.yml`

**Propósito**: Pipeline que se ejecuta cuando se hace un Pull Request a la rama `stage`

**Trigger**:
- `pull_request` a la rama `stage`

**Proceso**:
- Llama al template `pr-stage.yaml` que:
  1. Construye y despliega con tag `stage`
  2. Ejecuta pruebas E2E

**Resultado**: Valida que el código funciona antes de mergear a `stage`

---

### 3. pr-to-develop.yml (en cada servicio)
**Ubicación**: `{service}/.github/workflows/pr-to-develop.yml`  
**Ejemplo**: `cloud-config/.github/workflows/pr-to-develop.yml`

**Propósito**: Pipeline que se ejecuta cuando se hace un Pull Request a la rama `develop`

**Trigger**:
- `pull_request` a la rama `develop`

**Proceso**:
- Llama al template `pr-dev.yaml` que:
  1. Ejecuta análisis de código con SonarCloud
  2. Ejecuta pruebas unitarias
  3. Ejecuta pruebas de integración

**Resultado**: Valida la calidad del código antes de mergear a `develop`

---

## Flujo Completo de CI/CD

### Flujo de Desarrollo

```
1. Desarrollador crea PR a 'develop'
   └─> pr-to-develop.yml
       └─> pr-dev.yaml
           ├─> SonarCloud analysis
           ├─> Unit tests
           └─> Integration tests

2. PR mergeado a 'develop'
   └─> build.yml
       └─> build.yml (template)
           ├─> Build Docker image
           ├─> Push to ACR (tag: stage)
           └─> Deploy to Kubernetes (namespace: dev)

3. Desarrollador crea PR a 'stage'
   └─> pr-to-stage.yml
       └─> pr-stage.yaml
           ├─> Build and deploy (tag: stage)
           └─> Run E2E tests

4. PR mergeado a 'stage'
   └─> build.yml
       └─> build.yml (template)
           ├─> Build Docker image
           ├─> Push to ACR (tag: stage)
           └─> Deploy to Kubernetes (namespace: dev)

5. Merge a 'main'
   └─> build.yml
       └─> build.yml (template)
           ├─> Build Docker image
           ├─> Push to ACR (tag: prod)
           └─> Deploy to Kubernetes (namespace: dev)
```

---

## Resumen por Pipeline

| Pipeline | Trigger | Acción Principal | Resultado |
|----------|---------|-------------------|-----------|
| **build.yml** (template) | workflow_call | Build + Push + Deploy | Imagen en ACR y desplegada en K8s |
| **deploy-kube.yml** | workflow_call | Deploy a Kubernetes | Servicio desplegado en AKS |
| **run-e2e.yml** | workflow_call/dispatch | Ejecutar pruebas E2E | Validación de servicios |
| **remote-e2e.yml** | workflow_call | Wrapper para E2E | Llama a run-e2e.yml |
| **pr-stage.yaml** | workflow_call | Build + Deploy + E2E | Validación para stage |
| **pr-dev.yaml** | workflow_call | SonarCloud + Tests | Análisis de calidad |
| **release.yml** | workflow_call | Semantic Release | Generación de versiones |
| **build.yml** (servicio) | push/main/stage/develop | Build del servicio | Despliegue automático |
| **pr-to-stage.yml** | PR a stage | Validación stage | Validación antes de merge |
| **pr-to-develop.yml** | PR a develop | Validación develop | Análisis de código |

---

## Configuración Requerida

Todos los pipelines requieren los siguientes secrets configurados en GitHub:

- `DOCKERHUB_USERNAME`: Usuario de Docker Hub
- `DOCKERHUB_TOKEN`: Token de Docker Hub
- `ACR_USERNAME`: Usuario de Azure Container Registry
- `ACR_PASSWORD`: Contraseña de Azure Container Registry
- `ACR_NAME`: Nombre del Azure Container Registry
- `AZURE_CLIENT_ID`: Client ID de Azure
- `AZURE_CLIENT_SECRET`: Client Secret de Azure
- `AZURE_TENANT_ID`: Tenant ID de Azure
- `AZURE_SUBSCRIPTION_ID`: Subscription ID de Azure
- `RESOURCE_GROUP`: Resource Group de Azure
- `GH_TOKEN`: Token de GitHub para clonar repositorios privados

