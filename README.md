# Documentación de Pruebas - E-commerce Microservices

Este directorio contiene la documentación completa del proceso de pruebas implementado para la aplicación de e-commerce basada en microservicios.

## Documentos Disponibles

### 1. [REPORTE_PRUEBAS.md](./REPORTE_PRUEBAS.md)
**Reporte Completo del Proceso y Resultados**

Documento formal y completo que incluye:
- Introducción y contexto del proyecto
- Metodología de pruebas implementada
- Proceso completo paso a paso
- Detalle de todas las pruebas implementadas
- Resultados obtenidos y métricas
- Análisis de resultados
- Conclusiones y recomendaciones
- Anexos con comandos y referencias

---

### 2. [TestingDocumentation.md](./TestingDocumentation.md)
**Guía Técnica de Pruebas**

Documentación técnica detallada que incluye:
- Descripción técnica de cada tipo de prueba
- Guías de ejecución paso a paso
- Ejemplos de código y comandos
- Configuración de herramientas
- Mejores prácticas
- Troubleshooting

---

## Resumen Ejecutivo

### Cobertura de Pruebas

| Tipo de Prueba | Cantidad |
|----------------|----------|
| Pruebas Unitarias | 30+ |
| Pruebas de Integración | 5+ |
| Pruebas E2E | 7+ |
| Escenarios de Rendimiento | 10 |
| TOTAL | 52+ |

### Servicios Cubiertos

- User Service (Unit + Integration + E2E)
- Order Service (Unit + Integration + E2E)
- Favourite Service (Unit + Integration + E2E)
- Payment Service (Integration + E2E)
- Shipping Service (Integration + E2E)
- Product Service (E2E)

---

## Inicio Rápido

### Ejecutar Todas las Pruebas

```bash
# Desde la raíz del proyecto
mvn clean test
```

### Ejecutar por Tipo

```bash
# Pruebas unitarias
mvn test -Dtest=*UnitTest

# Pruebas de integración
mvn test -Dtest=*IntegrationTest

# Pruebas E2E
mvn test -Dtest=*E2ETest
```

### Ejecutar Pruebas de Rendimiento

```bash
# Instalar dependencias
pip install -r ci-pipelines/performance-tests/requirements.txt

# Ejecutar con UI
locust -f ci-pipelines/performance-tests/locustfile.py \
  --host=http://localhost:8080 \
  --web-host=0.0.0.0 \
  --web-port=8089
```

---

## Estructura del Proyecto

```
ecommerce-app-devops/
├── docs/
│   ├── README.md                    # Este archivo
│   ├── REPORTE_PRUEBAS.md          # Reporte completo
│   └── TestingDocumentation.md      # Guía técnica
├── src/test/java/com/selimhorri/app/e2e/
│   └── [5 archivos de pruebas E2E]
├── {service}/src/test/java/.../service/
│   ├── *UnitTest.java              # Pruebas unitarias
│   └── *IntegrationTest.java       # Pruebas de integración
└── ci-pipelines/performance-tests/
    ├── locustfile.py                # Tests de rendimiento
    └── requirements.txt
```

---

## Herramientas Utilizadas

| Herramienta | Propósito | Versión |
|------------|-----------|---------|
| JUnit 5 | Framework de pruebas | 5.x |
| Mockito | Mocking | 4.x |
| Spring Boot Test | Pruebas Spring | 2.5.7 |
| TestContainers | Contenedores de prueba | 1.16.0 |
| Locust | Pruebas de rendimiento | Latest |
| Maven | Gestión de dependencias | 3.8+ |

---

## Guías por Tipo de Prueba

### Pruebas Unitarias
- **Ubicación**: `{service}/src/test/java/.../service/*UnitTest.java`
- **Framework**: JUnit 5 + Mockito
- **Ejecución**: `mvn test -Dtest=*UnitTest`
- **Documentación**: Ver sección "Unit Tests" en [TestingDocumentation.md](./TestingDocumentation.md)

### Pruebas de Integración
- **Ubicación**: `{service}/src/test/java/.../service/*IntegrationTest.java`
- **Framework**: Spring Boot Test + TestContainers
- **Ejecución**: `mvn test -Dtest=*IntegrationTest`
- **Documentación**: Ver sección "Integration Tests" en [TestingDocumentation.md](./TestingDocumentation.md)

### Pruebas E2E
- **Ubicación**: `src/test/java/com/selimhorri/app/e2e/`
- **Framework**: Spring Boot Test + TestRestTemplate
- **Ejecución**: `mvn test -Dtest=*E2ETest`
- **Documentación**: Ver sección "E2E Tests" en [TestingDocumentation.md](./TestingDocumentation.md)

### Pruebas de Rendimiento
- **Ubicación**: `ci-pipelines/performance-tests/locustfile.py`
- **Framework**: Locust (Python)
- **Ejecución**: Ver comandos en [TestingDocumentation.md](./TestingDocumentation.md)
- **Documentación**: Ver sección "Performance Tests" en [TestingDocumentation.md](./TestingDocumentation.md)

---

## Mejores Prácticas

1. **Organización**: Seguir la pirámide de pruebas (más unitarias, menos E2E)
2. **Nomenclatura**: Usar nombres descriptivos: `test{Method}_{Scenario}_{ExpectedResult}`
3. **Aislamiento**: Cada prueba debe ser independiente
4. **Limpieza**: Limpiar datos de prueba después de cada ejecución
5. **Documentación**: Documentar casos de prueba complejos

---

## Troubleshooting

### Problemas Comunes

**Pruebas E2E fallan**
- Verificar que todos los servicios estén corriendo
- Verificar registro en Eureka
- Revisar configuración de perfil `test`

**Pruebas de integración fallan**
- Verificar configuración de base de datos
- Revisar TestContainers
- Verificar perfil `test`

**Pruebas de rendimiento no corren**
- Instalar dependencias: `pip install -r requirements.txt`
- Verificar que el host sea accesible
- Revisar puertos y configuración

Para más detalles, ver sección "Troubleshooting" en [TestingDocumentation.md](./TestingDocumentation.md)

---

## Referencias

- [JUnit 5 Documentation](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Locust Documentation](https://docs.locust.io/)
- [Spring Boot Testing](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)

---

*Última actualización: 2024*

