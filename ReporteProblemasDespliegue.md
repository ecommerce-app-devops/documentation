# Reporte de Problemas de Despliegue CI/CD
## Análisis de Fallos en la Implementación de Pipelines

---

## Resumen Ejecutivo

Este documento describe el proceso de implementación de pipelines CI/CD para una aplicación de e-commerce basada en microservicios, los problemas encontrados durante el despliegue y las limitaciones técnicas que impidieron completar exitosamente la implementación. Se detallan dos intentos principales: primero con Jenkins y Minikube local, y posteriormente con Azure Kubernetes Service (AKS) y GitHub Actions.

---

## 1. ¿Cuál fue el objetivo inicial del proyecto de despliegue?

El objetivo principal era implementar un sistema completo de CI/CD para desplegar una aplicación de e-commerce basada en microservicios en un entorno de Kubernetes. La aplicación está compuesta por **10 microservicios** principales:

1. **service-discovery** (Eureka) - Servicio de descubrimiento
2. **cloud-config** - Configuración centralizada
3. **api-gateway** - Puerta de enlace principal
4. **proxy-client** - Cliente proxy
5. **user-service** - Servicio de usuarios
6. **product-service** - Servicio de productos
7. **favourite-service** - Servicio de favoritos
8. **order-service** - Servicio de órdenes
9. **shipping-service** - Servicio de envíos
10. **payment-service** - Servicio de pagos

Adicionalmente, la aplicación requiere servicios auxiliares como:
- Base de datos MySQL (múltiples instancias por servicio)
- Zipkin para tracing distribuido
- Eureka para service discovery

**Requisitos técnicos:**
- Despliegue automatizado mediante pipelines CI/CD
- Orquestación con Kubernetes
- Capacidad para ejecutar múltiples pods simultáneamente
- Recursos suficientes para soportar todos los servicios concurrentemente

---

## 2. ¿Qué problemas se encontraron en el primer intento con Jenkins y Minikube?

### Configuración Inicial

El primer intento de implementación utilizó:
- **Jenkins** como servidor de CI/CD
- **Minikube** como cluster de Kubernetes local
- Máquina local para ejecutar el entorno completo

### Problemas Identificados

#### 2.1. Limitaciones de Recursos del Sistema

**Problema principal:** Minikube consumía demasiados recursos del sistema local, lo que impedía mantener todos los servicios ejecutándose simultáneamente.

**Síntomas observados:**
- El primer despliegue funcionaba correctamente
- Después del primer despliegue, los servicios comenzaban a fallar progresivamente
- Los pods se quedaban en estado `Pending` o `CrashLoopBackOff`
- El sistema se volvía inestable con el tiempo

**Causas técnicas:**
1. **Memoria RAM insuficiente:** 
   - Cada microservicio requiere aproximadamente 512MB-1GB de RAM
   - Minikube requiere recursos adicionales para el control plane
   - Con 10 servicios + bases de datos + infraestructura, se necesitan mínimo 8-12GB de RAM
   - La mayoría de máquinas locales no tienen recursos suficientes

2. **CPU sobrecargada:**
   - Minikube ejecuta un nodo completo en una máquina virtual
   - Los 10 microservicios compiten por recursos CPU limitados
   - La virtualización añade overhead adicional

3. **Almacenamiento:**
   - Múltiples imágenes Docker en cache
   - Volúmenes persistentes para bases de datos
   - Logs y métricas de Kubernetes

#### 2.2. Problemas de Estabilidad

**Comportamiento observado:**
- Los servicios se desplegaban inicialmente
- Después de algunos minutos, comenzaban a fallar
- Los pods eran eliminados por Kubernetes debido a problemas de salud
- Los reinicios de pods consumían aún más recursos

**Impacto:**
- Imposible realizar pruebas E2E completas
- No se podía mantener un entorno estable para desarrollo
- Los despliegues subsecuentes fallaban sistemáticamente

---

## 3. ¿Qué solución se implementó y por qué falló?

### Migración a Azure Kubernetes Service (AKS)

Después de identificar los problemas de recursos con Minikube, se decidió migrar a la nube utilizando:

- **Azure Kubernetes Service (AKS)** como cluster de Kubernetes gestionado
- **GitHub Actions** como plataforma de CI/CD
- **Azure Container Registry (ACR)** para almacenar imágenes Docker

### Configuración Implementada

La configuración de pipelines incluyó:
- Templates reutilizables para build y deploy
- Integración con Azure Container Registry
- Despliegue automático a AKS
- Ejecución de pruebas E2E después del despliegue

### Problemas Encontrados en Azure

#### 3.1. Limitación de Nodos en el Cluster

**Problema:** Azure no permitió crear más de un nodo en el cluster AKS.

**Causas posibles:**
1. **Plan de suscripción Azure:**
   - Suscripciones gratuitas o de estudiante tienen límites estrictos
   - Restricciones de cuotas de recursos
   - Límites en el número de nodos permitidos

2. **Configuración del resource group:**
   - Límites de recursos por región
   - Restricciones de cuotas de CPU/memoria

3. **Políticas de Azure:**
   - Políticas de seguridad que limitan recursos
   - Restricciones de compliance

**Impacto:**
- Un solo nodo no puede soportar todos los servicios
- Sin capacidad de escalado horizontal
- Limitación severa de recursos disponibles

#### 3.2. Limitación en el Número de Pods

**Problema:** Azure no permitió desplegar más de 2 pods simultáneamente.

**Análisis técnico:**
- Con 10 microservicios + bases de datos + servicios auxiliares, se necesitan mínimo 15-20 pods
- Un solo nodo con recursos limitados puede alojar solo unos pocos pods
- Kubernetes requiere recursos para pods del sistema (kube-system)

**Intentos realizados:**
1. Desplegar servicios uno por uno
2. Reducir recursos solicitados por pod
3. Optimizar configuraciones de recursos

**Resultado:**
- Solo 2 pods podían ejecutarse simultáneamente
- Los demás pods quedaban en estado `Pending`
- Imposible ejecutar la aplicación completa

#### 3.3. Limitaciones de Recursos del Nodo Único

Con un solo nodo y recursos limitados:
- **CPU:** Insuficiente para múltiples servicios
- **Memoria:** No alcanza para todos los pods
- **Almacenamiento:** Limitado para volúmenes persistentes

---

## 4. ¿Cuáles fueron las causas raíz de los fallos?

### Análisis de Causas Raíz

#### 4.1. Subestimación de Requisitos de Recursos

**Causa:** No se realizó un análisis adecuado de requisitos de recursos antes de la implementación.

**Evidencia:**
- 10 microservicios + infraestructura requieren recursos significativos
- Cada servicio necesita:
  - CPU: 100-500m (milésimas de CPU)
  - Memoria: 512Mi-1Gi
  - Almacenamiento para logs y datos

**Cálculo estimado:**
```
10 servicios × 1Gi RAM = 10Gi RAM mínimo
10 servicios × 500m CPU = 5 CPU cores mínimo
+ Infraestructura (Eureka, Config, Gateway) = +2Gi RAM, +1 CPU
+ Bases de datos = +4Gi RAM, +2 CPU
+ Overhead Kubernetes = +1Gi RAM, +0.5 CPU

TOTAL MÍNIMO: ~17Gi RAM, ~8.5 CPU cores
```

#### 4.2. Limitaciones de Planes de Suscripción

**Causa:** El plan de suscripción de Azure utilizado tenía restricciones que no se consideraron inicialmente.

**Impacto:**
- Imposible escalar horizontalmente
- Recursos limitados por nodo
- Cuotas insuficientes para el proyecto

#### 4.3. Arquitectura No Optimizada para Recursos Limitados

**Causa:** La arquitectura de microservicios, aunque correcta desde el punto de vista de diseño, requiere más recursos que una aplicación monolítica.

**Consideraciones:**
- Cada microservicio tiene su propio proceso JVM
- Cada servicio requiere su propia base de datos
- Overhead de comunicación entre servicios
- Service discovery y configuración adicional

---

## 5. ¿Qué lecciones aprendidas y recomendaciones se pueden extraer?

### Lecciones Aprendidas

#### 5.1. Planificación de Recursos

**Lección:** Siempre realizar un análisis exhaustivo de requisitos de recursos antes de elegir la infraestructura.

**Recomendaciones:**
1. **Análisis de capacidad:**
   - Calcular recursos necesarios por servicio
   - Considerar overhead de Kubernetes
   - Incluir margen de seguridad (20-30%)

2. **Pruebas de carga:**
   - Probar con un subconjunto de servicios primero
   - Medir consumo real de recursos
   - Escalar gradualmente

3. **Documentación de requisitos:**
   - Documentar requisitos mínimos y recomendados
   - Considerar diferentes escenarios (dev, staging, prod)

#### 5.2. Elección de Infraestructura

**Lección:** La infraestructura debe alinearse con los requisitos del proyecto y las limitaciones presupuestarias.

**Recomendaciones:**

1. **Para desarrollo local:**
   - Usar Docker Compose para desarrollo
   - Considerar Kubernetes solo para staging/producción
   - Usar herramientas como Skaffold para desarrollo local

2. **Para entornos de nube:**
   - Verificar límites de suscripción antes de comenzar
   - Considerar planes de pago si es necesario
   - Evaluar alternativas (AWS EKS, Google GKE, minikube con más recursos)

3. **Estrategias de optimización:**
   - Reducir recursos por pod donde sea posible
   - Usar recursos compartidos (bases de datos compartidas para dev)
   - Implementar auto-scaling horizontal cuando sea posible

#### 5.3. Arquitectura y Diseño

**Lección:** La arquitectura debe balancear principios de diseño con restricciones prácticas.

**Recomendaciones:**
1. **Para proyectos con recursos limitados:**
   - Considerar arquitectura modular en lugar de microservicios puros
   - Compartir infraestructura común (bases de datos, configuración)
   - Usar service mesh para optimizar comunicación

2. **Optimizaciones:**
   - Reducir tamaño de imágenes Docker
   - Usar JVM optimizadas (GraalVM, OpenJ9)
   - Implementar lazy loading de servicios

3. **Monitoreo:**
   - Implementar métricas de recursos desde el inicio
   - Alertas proactivas sobre uso de recursos
   - Dashboards de capacidad

#### 5.4. Gestión de Expectativas

**Lección:** Comunicar claramente las limitaciones y requisitos del proyecto.

**Recomendaciones:**
1. **Documentación temprana:**
   - Documentar requisitos de infraestructura
   - Identificar dependencias y limitaciones
   - Establecer criterios de éxito claros

2. **Comunicación:**
   - Informar sobre limitaciones técnicas
   - Proponer alternativas cuando sea necesario
   - Documentar decisiones arquitectónicas

---

## Proceso Detallado Seguido

### Fase 1: Implementación con Jenkins y Minikube

#### Configuración Inicial
1. Instalación de Jenkins en máquina local
2. Configuración de Minikube con recursos básicos
3. Creación de pipelines en Jenkins para cada servicio
4. Configuración de kubectl para acceso al cluster

#### Desarrollo
1. Creación de pipelines de build para cada microservicio
2. Configuración de despliegue automático a Kubernetes
3. Implementación de pruebas E2E después del despliegue
4. Configuración de monitoreo básico

#### Problemas Encontrados
1. **Primera ejecución:** Despliegue exitoso de todos los servicios
2. **Después de minutos:** Servicios comenzaron a fallar
3. **Diagnóstico:** Falta de recursos (RAM, CPU)
4. **Intentos de solución:**
   - Reducir recursos solicitados por pod
   - Priorizar servicios críticos
   - Desplegar servicios en lotes

#### Resultado
- Imposible mantener un entorno estable
- Decision de migrar a la nube

### Fase 2: Migración a Azure y GitHub Actions

#### Configuración Inicial
1. Creación de cuenta y suscripción en Azure
2. Configuración de Azure Container Registry (ACR)
3. Creación de cluster AKS
4. Configuración de GitHub Actions workflows

#### Desarrollo
1. Migración de pipelines de Jenkins a GitHub Actions
2. Creación de templates reutilizables:
   - `build.yml` - Construcción y publicación de imágenes
   - `deploy-kube.yml` - Despliegue a Kubernetes
   - `run-e2e.yml` - Ejecución de pruebas E2E
   - `pr-stage.yaml` - Pipeline para Pull Requests
   - `pr-dev.yaml` - Análisis de código
3. Configuración de secrets en GitHub
4. Integración con Azure services

#### Problemas Encontrados
1. **Creación de cluster:** Limitación a un solo nodo
2. **Primer despliegue:** Solo 2 pods pudieron ejecutarse
3. **Intentos de solución:**
   - Solicitar aumento de cuotas (rechazado)
   - Optimizar recursos de pods (insuficiente)
   - Considerar planes de pago (no disponible)

#### Resultado
- Imposible desplegar la aplicación completa
- Limitaciones técnicas insuperables con la suscripción actual

---

## Conclusiones

### Problemas Principales Identificados

1. **Recursos insuficientes:** Tanto en local (Minikube) como en nube (Azure con suscripción limitada)
2. **Falta de planificación:** No se realizó un análisis adecuado de requisitos
3. **Limitaciones de suscripción:** Restricciones de Azure no permitieron escalar
4. **Arquitectura compleja:** 10 microservicios requieren recursos significativos

### Estado Actual

- ✅ Pipelines CI/CD configurados y funcionales
- ✅ Integración con Azure Container Registry
- ✅ Templates de despliegue creados
- ❌ Despliegue completo no es posible con recursos actuales
- ❌ Pruebas E2E no pueden ejecutarse completamente

---

## Referencias Técnicas

### Arquitectura de la Aplicación
- **Microservicios:** 10 servicios principales
- **Service Discovery:** Eureka
- **API Gateway:** Spring Cloud Gateway
- **Configuración:** Spring Cloud Config
- **Tracing:** Zipkin

### Infraestructura Intentada
- **Kubernetes:** Minikube (local) / AKS (nube)
- **CI/CD:** Jenkins (local) / GitHub Actions (nube)
- **Container Registry:** Azure Container Registry
- **Bases de datos:** MySQL (múltiples instancias)

### Requisitos Estimados
- **RAM:** ~17GB mínimo
- **CPU:** ~8.5 cores mínimo
- **Almacenamiento:** ~50GB mínimo
- **Nodos:** 3+ nodos recomendados para alta disponibilidad
