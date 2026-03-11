# Documento de Cambios: Proyecto Observabilidad vs Final

Este documento detalla todos los cambios realizados entre el proyecto `observabilidad` y el proyecto `final`, incluyendo modificaciones en archivos de configuración, clases Java, y manifiestos de Kubernetes.

---

## 1. Archivos de Configuración

### 1.1. pom.xml

**Ubicación**: `process-score-mvp/pom.xml`

#### Cambios Realizados:

**1. Propiedades de SonarQube agregadas**

**Código Anterior:**
```xml
<properties>
    <compiler-plugin.version>3.14.0</compiler-plugin.version>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <quarkus.platform.version>3.27.0</quarkus.platform.version>
    <surefire-plugin.version>3.5.3</surefire-plugin.version>

    <resilience-starter.version>1.0.0-SNAPSHOT</resilience-starter.version>
    <!-- Security Starter -->
    <security-starter.version>1.0.0-SNAPSHOT</security-starter.version>

    <!-- Observability Starter -->
    <observability-starter.version>1.0.0-SNAPSHOT</observability-starter.version>
</properties>
```

**Código Modificado:**
```xml
<properties>
    <compiler-plugin.version>3.14.0</compiler-plugin.version>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <quarkus.platform.version>3.27.0</quarkus.platform.version>
    <surefire-plugin.version>3.5.3</surefire-plugin.version>

    <resilience-starter.version>1.0.0-SNAPSHOT</resilience-starter.version>
    <!-- Security Starter -->
    <security-starter.version>1.0.0-SNAPSHOT</security-starter.version>

    <!-- Observability Starter -->
    <observability-starter.version>1.0.0-SNAPSHOT</observability-starter.version>

    <!-- SonarQube: excluir clases base/utilitarias que generan falsos positivos por CDI/RouteBuilder -->
    <sonar.exclusions>**/BaseBean.java,**/RootRouteBuilder.java</sonar.exclusions>
    <sonar.coverage.exclusions>**/BaseBean.java,**/RootRouteBuilder.java</sonar.coverage.exclusions>
</properties>
```

**¿Qué se cambió?**
- Se agregaron dos propiedades nuevas para SonarQube: `sonar.exclusions` y `sonar.coverage.exclusions`

**¿Por qué se cambió?**
- Para excluir clases base y utilitarias (BaseBean.java y RootRouteBuilder.java) del análisis de SonarQube, ya que estas clases generan falsos positivos debido a su uso de CDI y RouteBuilder, que son patrones válidos pero que SonarQube puede marcar como problemas.

**Impacto:**
- **Positivo**: Mejora la calidad del análisis de código al excluir falsos positivos
- **Neutro**: No afecta la funcionalidad de la aplicación
- **Riesgo**: Bajo - solo afecta el análisis estático de código

---

**2. Dependencia icu4j agregada**

**Código Anterior:**
```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
<!-- unit test -->
```

**Código Modificado:**
```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.3.0</version>
</dependency>
<dependency>
    <groupId>com.ibm.icu</groupId>
    <artifactId>icu4j</artifactId>
    <version>76.1</version>
</dependency>
<!-- unit test -->
```

**¿Qué se cambió?**
- Se agregó la dependencia `icu4j` versión 76.1

**¿Por qué se cambió?**
- Probablemente para soportar operaciones de internacionalización y localización más robustas, especialmente para el manejo de fechas, números y texto en diferentes locales.

**Impacto:**
- **Positivo**: Mejora el soporte de internacionalización
- **Neutro**: Aumenta ligeramente el tamaño del artefacto final
- **Riesgo**: Bajo - es una biblioteca estable y ampliamente utilizada

---

### 1.2. application.properties

**Ubicación**: `process-score-mvp/src/main/resources/application.properties`

#### Cambios Realizados:

**1. Configuraciones de Fault Tolerance - Consolidación de Servicios Bantotal**

**Código Anterior:**
```properties
# ============================================
# Configuracion por Servicio - Bantotal Score (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalScoreService/obtenerScorePersona/Timeout/value=${BANTOTAL_SCORE_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalScoreService/obtenerScorePersona/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_SCORE_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalScoreService/obtenerScorePersona/CircuitBreaker/failureRatio=${BANTOTAL_SCORE_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalScoreService/obtenerScorePersona/CircuitBreaker/delay=${BANTOTAL_SCORE_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalScoreService/obtenerScorePersona/CircuitBreaker/successThreshold=${BANTOTAL_SCORE_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Compartamos Seguridad (FAST Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/Timeout/value=${COMPARTAMOS_SEGURIDAD_TIMEOUT:800}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SEGURIDAD_REQUEST_VOLUME:10}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/failureRatio=${COMPARTAMOS_SEGURIDAD_FAILURE_RATIO:0.4}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/delay=${COMPARTAMOS_SEGURIDAD_CB_DELAY:3000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/successThreshold=${COMPARTAMOS_SEGURIDAD_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Bantotal Experian (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/obtenerDatosExperian/Timeout/value=${BANTOTAL_EXPERIAN_OBTENER_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/obtenerDatosExperian/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_EXPERIAN_OBTENER_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/obtenerDatosExperian/CircuitBreaker/failureRatio=${BANTOTAL_EXPERIAN_OBTENER_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/obtenerDatosExperian/CircuitBreaker/delay=${BANTOTAL_EXPERIAN_OBTENER_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/obtenerDatosExperian/CircuitBreaker/successThreshold=${BANTOTAL_EXPERIAN_OBTENER_CB_SUCCESS:2}

com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/crearDatosExperian/Timeout/value=${BANTOTAL_EXPERIAN_CREAR_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/crearDatosExperian/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_EXPERIAN_CREAR_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/crearDatosExperian/CircuitBreaker/failureRatio=${BANTOTAL_EXPERIAN_CREAR_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/crearDatosExperian/CircuitBreaker/delay=${BANTOTAL_EXPERIAN_CREAR_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalExperianService/crearDatosExperian/CircuitBreaker/successThreshold=${BANTOTAL_EXPERIAN_CREAR_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Bantotal Catalogo (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalCatalogoService/obtenerGuiaProcesoEspecial/Timeout/value=${BANTOTAL_CATALOGO_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalCatalogoService/obtenerGuiaProcesoEspecial/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_CATALOGO_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalCatalogoService/obtenerGuiaProcesoEspecial/CircuitBreaker/failureRatio=${BANTOTAL_CATALOGO_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalCatalogoService/obtenerGuiaProcesoEspecial/CircuitBreaker/delay=${BANTOTAL_CATALOGO_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalCatalogoService/obtenerGuiaProcesoEspecial/CircuitBreaker/successThreshold=${BANTOTAL_CATALOGO_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Bantotal Configuracion (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalConfiguracionService/obtenerFechaSistema/Timeout/value=${BANTOTAL_CONFIGURACION_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalConfiguracionService/obtenerFechaSistema/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_CONFIGURACION_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalConfiguracionService/obtenerFechaSistema/CircuitBreaker/failureRatio=${BANTOTAL_CONFIGURACION_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalConfiguracionService/obtenerFechaSistema/CircuitBreaker/delay=${BANTOTAL_CONFIGURACION_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalConfiguracionService/obtenerFechaSistema/CircuitBreaker/successThreshold=${BANTOTAL_CONFIGURACION_CB_SUCCESS:2}
```

**Código Modificado:**
```properties
# ============================================
# Configuracion por Servicio - Bantotal Score (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerScorePersona/Timeout/value=${BANTOTAL_SCORE_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerScorePersona/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_SCORE_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerScorePersona/CircuitBreaker/failureRatio=${BANTOTAL_SCORE_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerScorePersona/CircuitBreaker/delay=${BANTOTAL_SCORE_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerScorePersona/CircuitBreaker/successThreshold=${BANTOTAL_SCORE_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Bantotal Experian (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerDatosExperian/Timeout/value=${BANTOTAL_EXPERIAN_OBTENER_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerDatosExperian/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_EXPERIAN_OBTENER_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerDatosExperian/CircuitBreaker/failureRatio=${BANTOTAL_EXPERIAN_OBTENER_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerDatosExperian/CircuitBreaker/delay=${BANTOTAL_EXPERIAN_OBTENER_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerDatosExperian/CircuitBreaker/successThreshold=${BANTOTAL_EXPERIAN_OBTENER_CB_SUCCESS:2}

com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/crearDatosExperian/Timeout/value=${BANTOTAL_EXPERIAN_CREAR_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/crearDatosExperian/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_EXPERIAN_CREAR_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/crearDatosExperian/CircuitBreaker/failureRatio=${BANTOTAL_EXPERIAN_CREAR_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/crearDatosExperian/CircuitBreaker/delay=${BANTOTAL_EXPERIAN_CREAR_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/crearDatosExperian/CircuitBreaker/successThreshold=${BANTOTAL_EXPERIAN_CREAR_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Bantotal Catalogo (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerGuiaProcesoEspecial/Timeout/value=${BANTOTAL_CATALOGO_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerGuiaProcesoEspecial/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_CATALOGO_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerGuiaProcesoEspecial/CircuitBreaker/failureRatio=${BANTOTAL_CATALOGO_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerGuiaProcesoEspecial/CircuitBreaker/delay=${BANTOTAL_CATALOGO_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerGuiaProcesoEspecial/CircuitBreaker/successThreshold=${BANTOTAL_CATALOGO_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Bantotal Configuracion (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerFechaSistema/Timeout/value=${BANTOTAL_CONFIGURACION_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerFechaSistema/CircuitBreaker/requestVolumeThreshold=${BANTOTAL_CONFIGURACION_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerFechaSistema/CircuitBreaker/failureRatio=${BANTOTAL_CONFIGURACION_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerFechaSistema/CircuitBreaker/delay=${BANTOTAL_CONFIGURACION_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.bantotal.BantotalService/obtenerFechaSistema/CircuitBreaker/successThreshold=${BANTOTAL_CONFIGURACION_CB_SUCCESS:2}
```

**¿Qué se cambió?**
- Se consolidaron las referencias de servicios separados (`BantotalScoreService`, `BantotalExperianService`, `BantotalCatalogoService`, `BantotalConfiguracionService`) en un único servicio `BantotalService`
- Todas las configuraciones de Fault Tolerance ahora apuntan a métodos del mismo servicio `BantotalService`

**¿Por qué se cambió?**
- Refactorización arquitectónica para consolidar los servicios de Bantotal en una única clase, simplificando la estructura del código y mejorando la mantenibilidad
- Esto refleja que en el código Java, todos estos métodos están ahora en la clase `BantotalService` en lugar de estar separados en múltiples clases

**Impacto:**
- **Positivo**: Simplifica la configuración y hace que sea más fácil de mantener
- **Positivo**: Alinea la configuración con la estructura real del código
- **Riesgo**: Medio - requiere actualizar las configuraciones en todos los ambientes
- **Nota**: Las configuraciones anteriores referenciaban servicios que no existían en el código, lo que podría haber causado problemas de configuración

---

**2. Configuraciones de Fault Tolerance - Consolidación de Servicios Compartamos**

**Código Anterior:**
```properties
# ============================================
# Configuracion por Servicio - Compartamos Seguridad (FAST Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/Timeout/value=${COMPARTAMOS_SEGURIDAD_TIMEOUT:800}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SEGURIDAD_REQUEST_VOLUME:10}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/failureRatio=${COMPARTAMOS_SEGURIDAD_FAILURE_RATIO:0.4}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/delay=${COMPARTAMOS_SEGURIDAD_CB_DELAY:3000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosSeguridadService/obtenerVariables/CircuitBreaker/successThreshold=${COMPARTAMOS_SEGURIDAD_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Compartamos Catalogo (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/Timeout/value=${COMPARTAMOS_CATALOGO_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_CATALOGO_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/failureRatio=${COMPARTAMOS_CATALOGO_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/delay=${COMPARTAMOS_CATALOGO_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/successThreshold=${COMPARTAMOS_CATALOGO_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Compartamos SCO (MEDIUM Profile)
# ============================================
# Obtener Cantidad Consultas
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/obtenerCantidadConsultasExperian/Timeout/value=${COMPARTAMOS_SCO_OBTENER_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/obtenerCantidadConsultasExperian/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SCO_OBTENER_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/obtenerCantidadConsultasExperian/CircuitBreaker/failureRatio=${COMPARTAMOS_SCO_OBTENER_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/obtenerCantidadConsultasExperian/CircuitBreaker/delay=${COMPARTAMOS_SCO_OBTENER_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/obtenerCantidadConsultasExperian/CircuitBreaker/successThreshold=${COMPARTAMOS_SCO_OBTENER_CB_SUCCESS:2}

# Registrar Cantidad Consultas
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/registrarCantidadConsultasExperian/Timeout/value=${COMPARTAMOS_SCO_REGISTRAR_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/registrarCantidadConsultasExperian/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SCO_REGISTRAR_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/registrarCantidadConsultasExperian/CircuitBreaker/failureRatio=${COMPARTAMOS_SCO_REGISTRAR_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/registrarCantidadConsultasExperian/CircuitBreaker/delay=${COMPARTAMOS_SCO_REGISTRAR_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosScoService/registrarCantidadConsultasExperian/CircuitBreaker/successThreshold=${COMPARTAMOS_SCO_REGISTRAR_CB_SUCCESS:2}
```

**Código Modificado:**
```properties
# ============================================
# Configuracion por Servicio - Compartamos Seguridad (FAST Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerVariables/Timeout/value=${COMPARTAMOS_SEGURIDAD_TIMEOUT:800}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerVariables/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SEGURIDAD_REQUEST_VOLUME:10}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerVariables/CircuitBreaker/failureRatio=${COMPARTAMOS_SEGURIDAD_FAILURE_RATIO:0.4}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerVariables/CircuitBreaker/delay=${COMPARTAMOS_SEGURIDAD_CB_DELAY:3000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerVariables/CircuitBreaker/successThreshold=${COMPARTAMOS_SEGURIDAD_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Compartamos Catalogo (MEDIUM Profile)
# ============================================
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/Timeout/value=${COMPARTAMOS_CATALOGO_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_CATALOGO_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/failureRatio=${COMPARTAMOS_CATALOGO_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/delay=${COMPARTAMOS_CATALOGO_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCatalogos/CircuitBreaker/successThreshold=${COMPARTAMOS_CATALOGO_CB_SUCCESS:2}

# ============================================
# Configuracion por Servicio - Compartamos SCO (MEDIUM Profile)
# ============================================
# Obtener Cantidad Consultas
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCantidadConsultasExperian/Timeout/value=${COMPARTAMOS_SCO_OBTENER_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCantidadConsultasExperian/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SCO_OBTENER_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCantidadConsultasExperian/CircuitBreaker/failureRatio=${COMPARTAMOS_SCO_OBTENER_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCantidadConsultasExperian/CircuitBreaker/delay=${COMPARTAMOS_SCO_OBTENER_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/obtenerCantidadConsultasExperian/CircuitBreaker/successThreshold=${COMPARTAMOS_SCO_OBTENER_CB_SUCCESS:2}

# Registrar Cantidad Consultas
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/registrarCantidadConsultasExperian/Timeout/value=${COMPARTAMOS_SCO_REGISTRAR_TIMEOUT:2000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/registrarCantidadConsultasExperian/CircuitBreaker/requestVolumeThreshold=${COMPARTAMOS_SCO_REGISTRAR_REQUEST_VOLUME:20}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/registrarCantidadConsultasExperian/CircuitBreaker/failureRatio=${COMPARTAMOS_SCO_REGISTRAR_FAILURE_RATIO:0.5}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/registrarCantidadConsultasExperian/CircuitBreaker/delay=${COMPARTAMOS_SCO_REGISTRAR_CB_DELAY:5000}
com.compartamos.process.score.infrastructure.service.external.compartamos.CompartamosService/registrarCantidadConsultasExperian/CircuitBreaker/successThreshold=${COMPARTAMOS_SCO_REGISTRAR_CB_SUCCESS:2}
```

**¿Qué se cambió?**
- Se consolidaron las referencias de `CompartamosSeguridadService` y `CompartamosScoService` en el servicio unificado `CompartamosService`
- Todas las configuraciones ahora apuntan a métodos del mismo servicio `CompartamosService`

**¿Por qué se cambió?**
- Refactorización arquitectónica para consolidar los servicios de Compartamos en una única clase
- Alinea la configuración con la estructura real del código donde todos estos métodos están en `CompartamosService`

**Impacto:**
- **Positivo**: Simplifica la configuración y mejora la mantenibilidad
- **Positivo**: Corrige configuraciones que referenciaban servicios inexistentes
- **Riesgo**: Medio - requiere actualizar configuraciones en todos los ambientes

---

## 2. Clases Java

### 2.1. RootRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/cross/util/RootRouteBuilder.java`

#### Cambios Realizados:

**1. Cambio de logTrace estático a inyección de dependencias**

**Código Anterior:**
```java
@ApplicationScoped
public abstract class RootRouteBuilder extends RouteBuilder {

    private static LoggerTrace logTrace = new LoggerTrace();

    @ConfigProperty(name = "seguridad.timeout")
    long time;

    public void onExceptions() {
        onException(Exception.class)
                .handled(true)
                .logStackTrace(false)
                .logExhausted(false)
                .logHandled(true)
                .log(LoggingLevel.ERROR, Constants.LOG_ERROR)
                .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(200))
                .transform(simple(Messages.RESPONSE_ERROR_GENERIC_BASE))
                .process(new ResponseFormatterProcessor())
                .bean(Constants.LOG_TRACE, Constants.FORMAT_RESPONSE_ROOT)
                .stop();
    }

    // ... más código ...

    public void onCompleteLog() {
        onCompletion()
                .onWhen(exchangeProperty("alreadyProcessed").isNull())
                .bean(Constants.LOG_TRACE, Constants.FORMAT_RESPONSE_ROOT);
    }
}
```

**Código Modificado:**
```java
@ApplicationScoped
public abstract class RootRouteBuilder extends RouteBuilder {

    LoggerTrace logTrace;

    @ConfigProperty(name = "seguridad.timeout")
    long time;

    public LoggerTrace getLogTrace() {
        return logTrace;
    }

    @Inject
    public void setLogTrace(LoggerTrace logTrace) {
        this.logTrace = logTrace;
    }

    public void onExceptions() {
        onException(Exception.class)
                .handled(true)
                .logStackTrace(false)
                .logExhausted(false)
                .logHandled(true)
                .log(LoggingLevel.ERROR, Constants.LOG_ERROR)
                .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(200))
                .transform(simple(Messages.RESPONSE_ERROR_GENERIC_BASE))
                .process(new ResponseFormatterProcessor())
                .process(logTrace::formatResponseRoot)
                .stop();
    }

    // ... más código ...

    public void onCompleteLog() {
        onCompletion()
                .onWhen(exchangeProperty("alreadyProcessed").isNull())
                .process(exchange -> logTrace.formatResponseRoot(exchange));
    }
}
```

**¿Qué se cambió?**
- Se cambió `logTrace` de un campo estático inicializado directamente a un campo de instancia con inyección de dependencias mediante `@Inject`
- Se agregaron métodos getter y setter para `logTrace`
- Se cambió el uso de `.bean(Constants.LOG_TRACE, Constants.FORMAT_RESPONSE_ROOT)` a `.process(logTrace::formatResponseRoot)` y `.process(exchange -> logTrace.formatResponseRoot(exchange))`

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de campos estáticos es una mejor práctica en aplicaciones CDI/Quarkus
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks
- **Flexibilidad**: Permite que el contenedor CDI gestione el ciclo de vida del objeto
- **Compatibilidad con Quarkus**: Quarkus funciona mejor con inyección de dependencias que con campos estáticos

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Positivo**: Facilita las pruebas unitarias
- **Riesgo**: Bajo - el cambio es transparente si `LoggerTrace` está correctamente configurado como bean CDI
- **Nota**: Requiere que `LoggerTrace` sea un bean CDI válido

---

### 2.2. ClaimsRepository.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/repository/ClaimsRepository.java`

#### Cambios Realizados:

**1. Cambio de acceso directo a singleton a inyección de dependencias**

**Código Anterior:**
```java
public class ClaimsRepository implements IClaimsRepository {

	JedisPool jedisPool = RedisPoolConnection.getInstance().getJedisPool();

	private boolean isRedisCacheEnabled() {
		// propiedad nueva: redis.cache.enabled=true|false
		String value = SingletonProperties.getInstance()
				.getPropertiesIntegration()
				.getProperty("redis.cache.enabled", "true");
		return Boolean.parseBoolean(value);
	}

	@Override
	public void create(ClaimsDto data, StringBuilder error)
	{
		if (!isRedisCacheEnabled()) {
			return;
		}

		try(Jedis redis = jedisPool.getResource())
		{
			// ... código ...
			parameters.ex(Integer.valueOf(SingletonProperties.getInstance().getPropertiesIntegration().getProperty("redis.time")));
			// ... código ...
		}
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class ClaimsRepository implements IClaimsRepository {

    private final RedisPoolConnection redisPoolConnection;
    private final SingletonProperties singletonProperties;

    @Inject
    public ClaimsRepository(RedisPoolConnection redisPoolConnection, SingletonProperties singletonProperties) {
        this.redisPoolConnection = redisPoolConnection;
        this.singletonProperties = singletonProperties;
    }

    private JedisPool getJedisPool() {
        return redisPoolConnection.getJedisPool();
    }

    private Properties getIntegrationProperties() {
        return singletonProperties.getPropertiesIntegration();
    }

    private boolean isRedisCacheEnabled() {
		// propiedad nueva: redis.cache.enabled=true|false
		String value = getIntegrationProperties().getProperty("redis.cache.enabled", "true");
		return Boolean.parseBoolean(value);
	}

	@Override
	public void create(ClaimsDto data, StringBuilder error)
	{
		if (!isRedisCacheEnabled()) {
			return;
		}

		try(Jedis redis = getJedisPool().getResource())
		{
			// ... código ...
			parameters.ex(Integer.valueOf(getIntegrationProperties().getProperty("redis.time")));
			// ... código ...
		}
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se cambió de acceso directo a singleton (`RedisPoolConnection.getInstance()`, `SingletonProperties.getInstance()`) a inyección de dependencias mediante constructor con `@Inject`
- Se agregaron campos `final` para `redisPoolConnection` y `singletonProperties`
- Se agregaron métodos privados `getJedisPool()` y `getIntegrationProperties()` para encapsular el acceso
- Se reemplazaron todas las llamadas directas a los singletons por llamadas a los métodos encapsulados

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de acceso directo a singletons es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks
- **Consistencia**: Alinea el patrón con otras clases del proyecto que usan CDI
- **Bean CDI**: Hace que la clase sea un bean CDI válido, necesario para que `ClaimsBean` pueda inyectarlo

**Impacto:**
- **Positivo**: Mejora la arquitectura y facilita las pruebas
- **Riesgo**: Medio - requiere que `RedisPoolConnection` y `SingletonProperties` sean beans CDI válidos
- **Nota**: Este cambio es necesario para que `ClaimsBean` pueda inyectar `IClaimsRepository` correctamente

---

### 2.3. NativeReflectionConfig.java (NUEVA CLASE)

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/cross/config/NativeReflectionConfig.java`

#### Cambios Realizados:

**Clase completamente nueva**

**Código Anterior:**
- La clase no existía en el proyecto `observabilidad`

**Código Modificado:**
```java
package com.compartamos.process.score.cross.config;

import io.quarkus.runtime.annotations.RegisterForReflection;

@RegisterForReflection(targets = {
    com.fasterxml.jackson.databind.exc.InvalidFormatException.class,
    com.fasterxml.jackson.databind.JsonMappingException.class,
    com.fasterxml.jackson.core.JsonParseException.class,
    org.apache.camel.support.processor.PredicateValidationException.class
})
public class NativeReflectionConfig {
    // Registro de clases de excepcion para reflexión nativa en Quarkus, necesario para el manejo de errores en Camel
}
```

**¿Qué se cambió?**
- Se creó una nueva clase `NativeReflectionConfig` en el paquete `cross.config`

**¿Por qué se cambió?**
- **Compilación nativa de Quarkus**: Cuando Quarkus compila una aplicación a código nativo (GraalVM), necesita conocer qué clases deben estar disponibles para reflexión en tiempo de ejecución
- **Manejo de excepciones**: Las clases de excepciones de Jackson y Camel necesitan estar disponibles para reflexión para que el manejo de errores funcione correctamente en modo nativo
- **Anotación @RegisterForReflection**: Esta anotación le dice a Quarkus que estas clases deben estar disponibles para reflexión en tiempo de ejecución

**Impacto:**
- **Positivo**: Permite que la aplicación funcione correctamente en modo nativo de Quarkus
- **Positivo**: Asegura que las excepciones de Jackson y Camel se manejen correctamente
- **Neutro**: No afecta el funcionamiento en modo JVM estándar
- **Riesgo**: Bajo - solo afecta la compilación nativa, no el funcionamiento normal

---

## 4. Archivos de Manifiestos Kubernetes

### 4.1. config-process-score.yaml → config-process-score-mvp.yaml

**Ubicación**: `process-score-mvp/manifest/config-process-score.yaml` → `config-process-score-mvp.yaml`

#### Cambios Realizados:

**1. Cambio de nombre del ConfigMap**

**Código Anterior:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-process-score
  namespace: middleware-poc
```

**Código Modificado:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-process-score-mvp
  namespace: middleware-poc
```

**¿Qué se cambió?**
- Se cambió el nombre del ConfigMap de `config-process-score` a `config-process-score-mvp`

**¿Por qué se cambió?**
- **Consistencia de nomenclatura**: Alinear el nombre del ConfigMap con el nombre del servicio y otros recursos de Kubernetes
- **Claridad**: El sufijo `-mvp` hace más claro que es el ConfigMap para el servicio MVP

**Impacto:**
- **Riesgo**: Medio - requiere actualizar todas las referencias al ConfigMap en otros manifiestos (deployment, etc.)
- **Nota**: Este cambio debe estar sincronizado con el cambio en `deployment.yaml`

---

### 4.2. deployment.yaml

**Ubicación**: `process-score-mvp/manifest/deployment.yaml`

#### Cambios Realizados:

**1. Cambio de nombres de recursos**

**Código Anterior:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: process-score
  namespace: middleware-poc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: process-score
  template:
    metadata:
      labels:
        app: process-score
    spec:
      containers:
      - name: process-score-container
        image: southamerica-west1-docker.pkg.dev/cb-cloud-gke-poc-bfd3/poc-gke-dev-qa/process-score:4.0.0-20250626.1
        envFrom:
        - configMapRef:
            name: config-process-score
```

**Código Modificado:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: process-score-mvp
  namespace: middleware-poc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: process-score-mvp
  template:
    metadata:
      labels:
        app: process-score-mvp
    spec:
      containers:
      - name: process-score-mvp-container
        image: southamerica-west1-docker.pkg.dev/cb-cloud-gke-poc-bfd3/poc-gke-dev-qa/process-score-mvp:4.0.0-20250626.1
        envFrom:
        - configMapRef:
            name: config-process-score-mvp
```

**¿Qué se cambió?**
- Se cambió el nombre del Deployment de `process-score` a `process-score-mvp`
- Se cambió el label `app` de `process-score` a `process-score-mvp`
- Se cambió el nombre del contenedor de `process-score-container` a `process-score-mvp-container`
- Se cambió la imagen Docker de `process-score:4.0.0-20250626.1` a `process-score-mvp:4.0.0-20250626.1`
- Se actualizó la referencia al ConfigMap de `config-process-score` a `config-process-score-mvp`

**¿Por qué se cambió?**
- **Consistencia de nomenclatura**: Alinear todos los nombres de recursos de Kubernetes con el nombre del servicio MVP
- **Claridad**: Hace más claro que estos recursos pertenecen al servicio MVP
- **Evitar conflictos**: Si hay múltiples versiones del servicio, los nombres únicos evitan conflictos

**Impacto:**
- **Riesgo**: Alto - este es un cambio crítico que afecta el despliegue
- **Nota**: Requiere actualizar:
  - Referencias en otros manifiestos (Service, Ingress)
  - Scripts de despliegue
  - Configuraciones de CI/CD
  - La imagen Docker debe ser construida con el nuevo nombre

---

### 4.3. process-score-svc.yaml → process-score-mvp-svc.yaml

**Ubicación**: `process-score-mvp/manifest/process-score-svc.yaml` → `process-score-mvp-svc.yaml`

#### Cambios Realizados:

**Cambio de nombre del Service**

**Código Anterior:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: process-score-svc
  namespace: middleware-poc
spec:
  selector:
    app: process-score
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

**Código Modificado:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: process-score-mvp-svc
  namespace: middleware-poc
spec:
  selector:
    app: process-score-mvp
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

**¿Qué se cambió?**
- Se cambió el nombre del Service de `process-score-svc` a `process-score-mvp-svc`
- Se actualizó el selector para apuntar a `app: process-score-mvp`
- La configuración de puertos y tipo se mantiene igual

**¿Por qué se cambió?**
- **Consistencia**: Alinear el nombre del Service con el Deployment y otros recursos
- **Correspondencia**: El selector debe coincidir con los labels del Deployment

**Impacto:**
- **Riesgo**: Alto - afecta el acceso al servicio
- **Nota**: Requiere actualizar referencias en Ingress y otros servicios que consuman este servicio

---

### 4.4. process-score-ingress.yaml → process-score-mvp-ingress.yaml

**Ubicación**: `process-score-mvp/manifest/process-score-ingress.yaml` → `process-score-mvp-ingress.yaml`

#### Cambios Realizados:

**Cambio de nombre del Ingress, host y referencias al Service**

**Código Anterior:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: process-score-ingress
  namespace: middleware-poc
  annotations:
    kubernetes.io/ingress.class: "nginx"
    argoproj.io/expect-ingresses: "false"
spec:
  rules:
    - host: process-score.devpoc.compartamos.com.pe
      http:
        paths:
          - path: /q/openapi
            pathType: Prefix
            backend:
              service:
                name: process-score-svc
                port:
                  number: 8080
          - path: /swagger-ui/
            pathType: Prefix
            backend:
              service:
                name: process-score-svc
                port:
                  number: 8080
          - path: /cxf/api
            pathType: Prefix
            backend:
              service:
                name: process-score-svc
                port:
                  number: 8080
```

**Código Modificado:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: process-score-mvp-ingress
  namespace: middleware-poc
  annotations:
    kubernetes.io/ingress.class: "nginx"
    argoproj.io/expect-ingresses: "false"
spec:
  rules:
    - host: process-score-mvp.devpoc.compartamos.com.pe
      http:
        paths:
          - path: /q/openapi
            pathType: Prefix
            backend:
              service:
                name: process-score-mvp-svc
                port:
                  number: 8080
          - path: /swagger-ui/
            pathType: Prefix
            backend:
              service:
                name: process-score-mvp-svc
                port:
                  number: 8080
          - path: /cxf/api
            pathType: Prefix
            backend:
              service:
                name: process-score-mvp-svc
                port:
                  number: 8080
```

**¿Qué se cambió?**
- Se cambió el nombre del Ingress de `process-score-ingress` a `process-score-mvp-ingress`
- Se cambió el host de `process-score.devpoc.compartamos.com.pe` a `process-score-mvp.devpoc.compartamos.com.pe`
- Se actualizaron todas las referencias al Service de `process-score-svc` a `process-score-mvp-svc` (en los tres paths: `/q/openapi`, `/swagger-ui/`, y `/cxf/api`)
- Las anotaciones y la estructura de paths se mantienen iguales

**¿Por qué se cambió?**
- **Consistencia**: Alinear el nombre del Ingress con otros recursos
- **Correspondencia**: La referencia al Service debe coincidir con el nuevo nombre del Service
- **Host único**: El cambio de host permite tener un dominio único para el servicio MVP

**Impacto:**
- **Riesgo**: Alto - afecta el acceso externo al servicio
- **Crítico**: El cambio de host significa que las URLs externas cambian de `process-score.devpoc.compartamos.com.pe` a `process-score-mvp.devpoc.compartamos.com.pe`
- **Nota**: Requiere actualizar:
  - Configuraciones de DNS
  - Referencias en documentación
  - URLs en aplicaciones cliente
  - Configuraciones de monitoreo y alertas

---

### 2.14. BaseBean.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/domain/bean/BaseBean.java`

#### Cambios Realizados:

**1. Cambio de logTrace estático a inyección de dependencias y métodos estáticos a no estáticos**

**Código Anterior:**
```java
public abstract class BaseBean {
	/**
	 * Detiene la ruta camel y retorna una respuesta
	 * 
	 * @param exchange Contenedor camel
	 * @param body     Cuerpo
	 * @param httpCode Codigo status http
	 */
	private static LoggerTrace logTrace = new LoggerTrace();

	protected void stop(Exchange exchange, String body, int httpCode) {
        exchange.setProperty("alreadyProcessed", true);
		stopWithoutStatus(exchange, body, httpCode);
		logTrace.formatResponseRoot(exchange);
	}

	// ... más código ...

	protected static void stopWithoutStatus(Exchange exchange, String body, int httpCode) {
		exchange.getMessage().setHeader(Exchange.HTTP_RESPONSE_CODE, httpCode);
		exchange.getMessage().setBody(body);
        new ResponseFormatterProcessor().process(exchange);
		exchange.setRouteStop(true);
		logTrace.formatResponseRoot(exchange);
	}

	protected static void stopUnauthorized(Exchange exchange, String body) {
		exchange.getMessage().setHeader(Exchange.HTTP_RESPONSE_CODE, 401);
		exchange.getMessage().setHeader(Exchange.CONTENT_TYPE, "application/json");
		exchange.getMessage().setBody(body);
		new ResponseFormatterProcessor().process(exchange);
		exchange.setRouteStop(true);
		logTrace.formatResponseRoot(exchange);
	}

	protected static void stopWithout(Exchange exchange) {
		exchange.getMessage().setHeader(Exchange.HTTP_RESPONSE_CODE,
				exchange.getIn().getHeader(Exchange.HTTP_RESPONSE_CODE, Integer.class));
		exchange.getMessage().setBody(exchange.getIn().getBody(String.class));
        new ResponseFormatterProcessor().process(exchange);
		exchange.setRouteStop(true);
		logTrace.formatResponseRoot(exchange);
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public abstract class BaseBean {
	/**
	 * Detiene la ruta camel y retorna una respuesta
	 * 
	 * @param exchange Contenedor camel
	 * @param body     Cuerpo
	 * @param httpCode Codigo status http
	 */

	LoggerTrace logTrace;

    public LoggerTrace getLogTrace() {
        return logTrace;
    }

    @Inject
    public void setLogTrace(LoggerTrace logTrace) {
        this.logTrace = logTrace;
    }

    protected void stop(Exchange exchange, String body, int httpCode) {
        exchange.setProperty("alreadyProcessed", true);
		stopWithoutStatus(exchange, body, httpCode);
		logTrace.formatResponseRoot(exchange);
	}

	// ... más código ...

	protected void stopWithoutStatus(Exchange exchange, String body, int httpCode) {
		exchange.getMessage().setHeader(Exchange.HTTP_RESPONSE_CODE, httpCode);
		exchange.getMessage().setBody(body);
        new ResponseFormatterProcessor().process(exchange);
		exchange.setRouteStop(true);
		logTrace.formatResponseRoot(exchange);
	}

	protected void stopUnauthorized(Exchange exchange, String body) {
		exchange.getMessage().setHeader(Exchange.HTTP_RESPONSE_CODE, 401);
		exchange.getMessage().setHeader(Exchange.CONTENT_TYPE, "application/json");
		exchange.getMessage().setBody(body);
		new ResponseFormatterProcessor().process(exchange);
		exchange.setRouteStop(true);
		logTrace.formatResponseRoot(exchange);
	}

	protected void stopWithout(Exchange exchange) {
		exchange.getMessage().setHeader(Exchange.HTTP_RESPONSE_CODE,
				exchange.getIn().getHeader(Exchange.HTTP_RESPONSE_CODE, Integer.class));
		exchange.getMessage().setBody(exchange.getIn().getBody(String.class));
        new ResponseFormatterProcessor().process(exchange);
		exchange.setRouteStop(true);
		logTrace.formatResponseRoot(exchange);
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se cambió `logTrace` de un campo estático inicializado directamente a un campo de instancia con inyección de dependencias mediante `@Inject`
- Se agregaron métodos getter y setter para `logTrace`
- Se cambiaron los métodos `stopWithoutStatus`, `stopUnauthorized` y `stopWithout` de `static` a métodos de instancia (se removió `static`)

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de campos estáticos es una mejor práctica en aplicaciones CDI/Quarkus
- **Consistencia**: Alinea el patrón con `RootRouteBuilder` y otras clases del proyecto
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks
- **Compatibilidad con Quarkus**: Quarkus funciona mejor con inyección de dependencias que con campos estáticos
- **Métodos no estáticos**: Permite que los métodos accedan a `logTrace` como campo de instancia

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Positivo**: Facilita las pruebas unitarias
- **Riesgo**: Medio - todas las clases que extienden `BaseBean` ahora deben ser beans CDI válidos
- **Nota**: Requiere que todas las subclases de `BaseBean` sean beans CDI válidos

---

### 2.15. ClaimsBean.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/bean/ClaimsBean.java`

#### Cambios Realizados:

**1. Cambio de instanciación directa a inyección de dependencias y agregado de anotaciones CDI**

**Código Anterior:**
```java
public class ClaimsBean extends BaseBean {

	private IClaimsRepository claimsRepository = new ClaimsRepository();

	public void createClaims(Exchange exchange) throws JsonProcessingException {
		// ... código ...
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
@Named("claimsBean")
public class ClaimsBean extends BaseBean {


	private final IClaimsRepository claimsRepository;

    @Inject
    public ClaimsBean(IClaimsRepository claimsRepository) {
        this.claimsRepository = claimsRepository;
    }

    public void createClaims(Exchange exchange) throws JsonProcessingException {
		// ... código ...
	}
}
```

**¿Qué se cambió?**
- Se agregaron las anotaciones `@ApplicationScoped` y `@Named("claimsBean")`
- Se cambió `claimsRepository` de una instanciación directa (`new ClaimsRepository()`) a inyección de dependencias mediante constructor con `@Inject`
- El campo ahora es `final` y se inicializa en el constructor

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de instanciación directa es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks del repositorio
- **Consistencia**: Alinea el patrón con otras clases del proyecto que usan CDI
- **Bean CDI**: Hace que la clase sea un bean CDI válido, necesario ahora que `BaseBean` es `@ApplicationScoped`

**Impacto:**
- **Positivo**: Mejora la arquitectura y facilita las pruebas
- **Riesgo**: Bajo - requiere que `IClaimsRepository` tenga una implementación CDI válida
- **Nota**: El nombre del bean `claimsBean` puede ser usado en rutas Camel para referenciar esta clase

---

### 2.16. ScoreBean.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/domain/bean/ScoreBean.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI**

**Código Anterior:**
```java
public class ScoreBean extends BaseBean {

    private static final String PROP_SDT_PARAM_SCORE_PERSONA = "sdtParamScorePersona";

	public void validarGestionCalificacion(Exchange exchange) {
		// ... código ...
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
@Named("scoreBean")
public class ScoreBean extends BaseBean {

    private static final String PROP_SDT_PARAM_SCORE_PERSONA = "sdtParamScorePersona";

	public void validarGestionCalificacion(Exchange exchange) {
		// ... código ...
	}
}
```

**¿Qué se cambió?**
- Se agregaron las anotaciones `@ApplicationScoped` y `@Named("scoreBean")`
- Se agregó el import de `jakarta.enterprise.context.ApplicationScoped` y `jakarta.inject.Named`

**¿Por qué se cambió?**
- **Bean CDI**: Hace que la clase sea un bean CDI válido, necesario ahora que `BaseBean` es `@ApplicationScoped`
- **Consistencia**: Alinea el patrón con otras clases Bean del proyecto
- **Referencia en Camel**: El nombre del bean `scoreBean` puede ser usado en rutas Camel para referenciar esta clase

**Impacto:**
- **Positivo**: Hace la clase compatible con CDI
- **Riesgo**: Bajo - no afecta la funcionalidad existente
- **Nota**: La clase ahora es gestionada por el contenedor CDI

---

### 2.17. VariablesBean.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/domain/bean/VariablesBean.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI**

**Código Anterior:**
```java
public class VariablesBean extends BaseBean {

	public void prepareRequestGetVariablesScore(Exchange exchange) throws IOException {
		// ... código ...
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
@Named("variablesBean")
public class VariablesBean extends BaseBean {

	public void prepareRequestGetVariablesScore(Exchange exchange) throws IOException {
		// ... código ...
	}
}
```

**¿Qué se cambió?**
- Se agregaron las anotaciones `@ApplicationScoped` y `@Named("variablesBean")`
- Se agregaron los imports de `jakarta.enterprise.context.ApplicationScoped` y `jakarta.inject.Named`

**¿Por qué se cambió?**
- **Bean CDI**: Hace que la clase sea un bean CDI válido, necesario ahora que `BaseBean` es `@ApplicationScoped`
- **Consistencia**: Alinea el patrón con otras clases Bean del proyecto

**Impacto:**
- **Positivo**: Hace la clase compatible con CDI
- **Riesgo**: Bajo - no afecta la funcionalidad existente

---

### 2.18. CatalogueBean.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/domain/bean/CatalogueBean.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI**

**Código Anterior:**
```java
public class CatalogueBean extends BaseBean {

	public void prepareRequestGetCodigoTabla(Exchange exchange){
		// ... código ...
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
@Named("catalogueBean")
public class CatalogueBean extends BaseBean {

	public void prepareRequestGetCodigoTabla(Exchange exchange){
		// ... código ...
	}
}
```

**¿Qué se cambió?**
- Se agregaron las anotaciones `@ApplicationScoped` y `@Named("catalogueBean")`
- Se agregaron los imports de `jakarta.enterprise.context.ApplicationScoped` y `jakarta.inject.Named`

**¿Por qué se cambió?**
- **Bean CDI**: Hace que la clase sea un bean CDI válido, necesario ahora que `BaseBean` es `@ApplicationScoped`
- **Consistencia**: Alinea el patrón con otras clases Bean del proyecto

**Impacto:**
- **Positivo**: Hace la clase compatible con CDI
- **Riesgo**: Bajo - no afecta la funcionalidad existente

---

### 2.8. SessionValidator.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/cross/util/SessionValidator.java`

#### Cambios Realizados:

**1. Cambio de clase utilitaria estática a bean CDI**

**Código Anterior:**
```java
public class SessionValidator {

    private static final Logger LOG = LoggerFactory.getLogger(SessionValidator.class);
    private static final Integer CODIGO_SESION_INVALIDA = 10011;

    // Constructor privado para evitar instanciación
    private SessionValidator() {
    }

    public static boolean isSesionInvalida(Object response) {
        return extractErrores(response)
                .map(SessionValidator::containsInvalidSessionError)
                .orElse(false);
    }

    private static Optional<List<?>> extractErrores(Object response) {
        try {
            return Optional.ofNullable(response)
                    .map(SessionValidator::getErroresnegocio)
                    .map(SessionValidator::getBTErrorNegocio);
        } catch (Exception e) {
            LOG.debug("Error al extraer errores de negocio", e);
            return Optional.empty();
        }
    }

    private static Object getErroresnegocio(Object response) {
        // ... código ...
    }

    private static List<?> getBTErrorNegocio(Object erroresnegocio) {
        // ... código ...
    }

    private static boolean containsInvalidSessionError(List<?> errores) {
        return errores.stream()
                .map(SessionValidator::getCodigo)
                .filter(Optional::isPresent)
                .map(Optional::get)
                .anyMatch(CODIGO_SESION_INVALIDA::equals);
    }

    private static Optional<Object> getCodigo(Object error) {
        // ... código ...
    }
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class SessionValidator {

    private static final Logger LOG = LoggerFactory.getLogger(SessionValidator.class);
    private static final Integer CODIGO_SESION_INVALIDA = 10011;

    /**
     * Predicado para Camel. Usar Exchange evita problemas de binding/reflexión con proxies CDI en nativo.
     */
    public boolean isSesionInvalida(Exchange exchange) {
        Object response = exchange != null ? exchange.getMessage().getBody() : null;
        return extractErrores(response)
                .map(this::containsInvalidSessionError)
                .orElse(false);
    }

    public Optional<List<Object>> extractErrores(Object response) {
        try {
            return Optional.ofNullable(response)
                    .map(this::getErroresnegocio)
                    .map(this::getBTErrorNegocio);
        } catch (Exception e) {
            LOG.debug("Error al extraer errores de negocio", e);
            return Optional.empty();
        }
    }

    public Object getErroresnegocio(Object response) {
        // ... código ...
    }

    @SuppressWarnings("unchecked")
    public List<Object> getBTErrorNegocio(Object erroresnegocio) {
        // ... código con validación de tipo ...
    }

    public boolean containsInvalidSessionError(List<Object> errores) {
        return errores.stream()
                .map(this::getCodigo)
                .flatMap(Optional::stream)
                .anyMatch(CODIGO_SESION_INVALIDA::equals);
    }

    public Optional<Object> getCodigo(Object error) {
        // ... código ...
    }
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se eliminó el constructor privado (ahora es un bean CDI)
- Se cambió `isSesionInvalida(Object response)` de método estático a método de instancia que recibe `Exchange exchange`
- Todos los métodos privados estáticos se convirtieron en métodos de instancia públicos
- Se cambió de referencias estáticas (`SessionValidator::method`) a referencias de instancia (`this::method`)
- Se cambió `filter(Optional::isPresent).map(Optional::get)` a `flatMap(Optional::stream)` en `containsInvalidSessionError`
- Se agregó validación de tipo en `getBTErrorNegocio` con `@SuppressWarnings("unchecked")`
- Se cambió el tipo de retorno de `List<?>` a `List<Object>` en varios métodos

**¿Por qué se cambió?**
- **Compatibilidad con CDI**: Necesario para que funcione con inyección de dependencias en los RouteBuilders
- **Compatibilidad con Quarkus Native**: Usar `Exchange` como parámetro evita problemas de binding/reflexión con proxies CDI en compilación nativa
- **Mejora de arquitectura**: Convertir a bean CDI permite mejor gestión del ciclo de vida y testabilidad
- **Mejora de tipos**: Cambiar a `List<Object>` y usar `flatMap(Optional::stream)` mejora la seguridad de tipos

**Impacto:**
- **Positivo**: Mejora la compatibilidad con Quarkus Native
- **Positivo**: Facilita la inyección de dependencias en RouteBuilders
- **Riesgo**: Alto - cambio de API: el método `isSesionInvalida` ahora requiere `Exchange` en lugar de `Object`
- **Nota**: Este cambio es crítico porque afecta cómo se usa en los RouteBuilders (de `method(SessionValidator.class, Constants.SESION_INVALIDA)` a `sessionValidator::isSesionInvalida`)

---

### 2.9. ScoreRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/route/ScoreRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI e inyección de dependencias**

**Código Anterior:**
```java
public class ScoreRouteBuilder extends RootRouteBuilder {

    @Override
    public void configure() throws Exception {

        onExceptions();

        from("direct:processGestionaCalificacion")
            .bean(ScoreBean.class, "validarGestionCalificacion")
            .choice()
                .when(simple("${exchangeProperty.consultaScoreExperian}"))
                    .bean(ScoreBean.class, "prepareRequestManageCalification")
                    // ... más código ...
                .end()
                .bean(ScoreBean.class, "formatResponseGestionExperian");

        // ... más rutas ...
        
        from("direct:processGetExperianCalification").routeId("routeProcessGetExperianCalification")
            .setProperty(Constants.AUTH_APP, constant(Constants.CODE_APP_BANTOTAL))
            .setProperty(Constants.USER, exchangeProperty(Constants.USER_JWT))
            .to(Constants.OBTENER_TOKEN)
            .choice()
                .when(body())
                    .to(Constants.OBTENER_EXPERIAN)
                    .choice()
                        .when(method(SessionValidator.class, Constants.SESION_INVALIDA))
                            .to(Constants.OBTENER_NUEVO_TOKEN)
                            .to(Constants.OBTENER_EXPERIAN)
                        .endChoice()
                    // ... más código ...
    }
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class ScoreRouteBuilder extends RootRouteBuilder {

    private final ScoreBean scoreBean;
    private final SessionValidator sessionValidator;

    @Inject
    public ScoreRouteBuilder(ScoreBean scoreBean, SessionValidator sessionValidator) {
        this.scoreBean = scoreBean;
        this.sessionValidator = sessionValidator;
    }

    @Override
    public void configure() throws Exception {

        onExceptions();

        from("direct:processGestionaCalificacion")
            .process(scoreBean::validarGestionCalificacion)
            .choice()
                .when(simple("${exchangeProperty.consultaScoreExperian}"))
                    .process(scoreBean::prepareRequestManageCalification)
                    // ... más código ...
                .end()
                .process(scoreBean::formatResponseGestionExperian);

        // ... más rutas ...
        
        from("direct:processGetExperianCalification").routeId("routeProcessGetExperianCalification")
            .setProperty(Constants.AUTH_APP, constant(Constants.CODE_APP_BANTOTAL))
            .setProperty(Constants.USER, exchangeProperty(Constants.USER_JWT))
            .to(Constants.OBTENER_TOKEN)
            .choice()
                .when(body())
                    .to(Constants.OBTENER_EXPERIAN)
                    .choice()
                        .when(sessionValidator::isSesionInvalida)
                            .to(Constants.OBTENER_NUEVO_TOKEN)
                            .to(Constants.OBTENER_EXPERIAN)
                        .endChoice()
                    // ... más código ...
    }
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se agregó inyección de dependencias mediante constructor con `@Inject` para `ScoreBean` y `SessionValidator`
- Se cambió de `.bean(ScoreBean.class, "methodName")` a `.process(scoreBean::methodName)` (referencias de método)
- Se cambió de `method(SessionValidator.class, Constants.SESION_INVALIDA)` a `sessionValidator::isSesionInvalida` (referencias de método)

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica en CDI/Quarkus
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks
- **Rendimiento**: Las referencias de método son más eficientes que las referencias de clase en Camel
- **Consistencia**: Alinea el patrón con otras clases RouteBuilder del proyecto

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Positivo**: Facilita las pruebas unitarias
- **Riesgo**: Medio - requiere que `ScoreBean` y `SessionValidator` sean beans CDI válidos
- **Nota**: Todos los métodos de `ScoreBean` ahora deben ser no estáticos para funcionar con referencias de método

---

### 2.10. VariablesRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/route/VariablesRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI e inyección de dependencias**

**Código Anterior:**
```java
public class VariablesRouteBuilder extends RootRouteBuilder {

	@Override
	public void configure() throws Exception {

		onExceptions();

		from("direct:processGetVariablesScore").routeId("routeProcessGetVariablesScore")
				.bean(VariablesBean.class, "prepareRequestGetVariablesScore")
				.to("direct:getVariablesSistema")
				.bean(VariablesBean.class, "preparePreResponseGetVariables");
		
		from("direct:processGetVariables").routeId("routeProcessGetVariables")
				.bean(VariablesBean.class, "prepareRequestGetVariables")
				.to("direct:getVariablesSistema")
				.bean(VariablesBean.class, "preparePreResponseGetVariables");
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class VariablesRouteBuilder extends RootRouteBuilder {

    private final VariablesBean variablesBean;

    @Inject
    public VariablesRouteBuilder(VariablesBean variablesBean) {
        this.variablesBean = variablesBean;
    }

	@Override
	public void configure() throws Exception {

		onExceptions();

		from("direct:processGetVariablesScore").routeId("routeProcessGetVariablesScore")
				.process(variablesBean::prepareRequestGetVariablesScore)
				.to("direct:getVariablesSistema")
				.process(variablesBean::preparePreResponseGetVariables);

		from("direct:processGetVariables").routeId("routeProcessGetVariables")
				.process(variablesBean::prepareRequestGetVariables)
				.to("direct:getVariablesSistema")
				.process(variablesBean::preparePreResponseGetVariables);
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se agregó inyección de dependencias mediante constructor con `@Inject` para `VariablesBean`
- Se cambió de `.bean(VariablesBean.class, "methodName")` a `.process(variablesBean::methodName)` (referencias de método)

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias
- **Consistencia**: Alinea el patrón con otras clases RouteBuilder del proyecto

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que `VariablesBean` sea un bean CDI válido

---

### 2.11. CatalogueRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/route/CatalogueRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI e inyección de dependencias**

**Código Anterior:**
```java
public class CatalogueRouteBuilder extends RootRouteBuilder {

	@Override
	public void configure() throws Exception {

		onExceptions();

		from("direct:processGetCodigoCatalogos")
				.bean(CatalogueBean.class, "prepareRequestGetCodigoCatalogos")
				.to("direct:getCodigoCatalogos")
				.bean(CatalogueBean.class, "prepareResponseGetCodigoCatalogos");

		from("direct:processGetGuiaProceso").routeId("routeProcessGetGuiaProceso")
				.setProperty(Constants.AUTH_APP, constant(Constants.CODE_APP_BANTOTAL))
				.setProperty(Constants.USER, exchangeProperty(Constants.USER_JWT))
				.to(Constants.OBTENER_TOKEN)
				.choice()
					.when(body())
						.to(Constants.OBTENER_GUIA_PROCESO_PROC)
						.choice()
							.when(method(SessionValidator.class, Constants.SESION_INVALIDA))
								.to(Constants.OBTENER_NUEVO_TOKEN)
								.to(Constants.OBTENER_GUIA_PROCESO_PROC)
						.endChoice()
					// ... más código ...
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class CatalogueRouteBuilder extends RootRouteBuilder {

    private final CatalogueBean catalogueBean;
    private final SessionValidator sessionValidator;

    @Inject
    public CatalogueRouteBuilder(CatalogueBean catalogueBean, SessionValidator sessionValidator) {
        this.catalogueBean = catalogueBean;
        this.sessionValidator = sessionValidator;
    }

	@Override
	public void configure() throws Exception {

		onExceptions();

		from("direct:processGetCodigoCatalogos")
				.process(catalogueBean::prepareRequestGetCodigoCatalogos)
				.to("direct:getCodigoCatalogos")
				.process(catalogueBean::prepareResponseGetCodigoCatalogos);

		from("direct:processGetGuiaProceso").routeId("routeProcessGetGuiaProceso")
				.setProperty(Constants.AUTH_APP, constant(Constants.CODE_APP_BANTOTAL))
				.setProperty(Constants.USER, exchangeProperty(Constants.USER_JWT))
				.to(Constants.OBTENER_TOKEN)
				.choice()
					.when(body())
						.to(Constants.OBTENER_GUIA_PROCESO_PROC)
						.choice()
							.when(sessionValidator::isSesionInvalida)
								.to(Constants.OBTENER_NUEVO_TOKEN)
								.to(Constants.OBTENER_GUIA_PROCESO_PROC)
						.endChoice()
					// ... más código ...
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se agregó inyección de dependencias mediante constructor con `@Inject` para `CatalogueBean` y `SessionValidator`
- Se cambió de `.bean(CatalogueBean.class, "methodName")` a `.process(catalogueBean::methodName)` (referencias de método)
- Se cambió de `method(SessionValidator.class, Constants.SESION_INVALIDA)` a `sessionValidator::isSesionInvalida` (referencias de método)

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias
- **Consistencia**: Alinea el patrón con otras clases RouteBuilder del proyecto

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que `CatalogueBean` y `SessionValidator` sean beans CDI válidos

---

### 2.12. ClaimsRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/route/ClaimsRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI e inyección de dependencias**

**Código Anterior:**
```java
public class ClaimsRouteBuilder extends RootRouteBuilder {

	@Override
	public void configure() {

		onExceptions();

		from("direct:readClaims").routeId("routeReadClaims")
				.bean(ClaimsBean.class, "readClaims")
				.end();

		from("direct:createClaims").routeId("routeCreateClaims")
				.bean(ClaimsBean.class, "createClaims")
				.end();

		from("direct:updateClaims").routeId("routeUpdateClaims")
				.bean(ClaimsBean.class, "updateClaims")
				.end();

		from("direct:deleteClaims").routeId("routeDeleteClaims")
				.bean(ClaimsBean.class, "deleteClaims")
				.end();
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class ClaimsRouteBuilder extends RootRouteBuilder {

    private final ClaimsBean claimsBean;

    @Inject
    public ClaimsRouteBuilder(ClaimsBean claimsBean) {
        this.claimsBean = claimsBean;
    }

	@Override
	public void configure() {

		onExceptions();

		from("direct:readClaims").routeId("routeReadClaims")
				.process(claimsBean::readClaims)
				.end();

		from("direct:createClaims").routeId("routeCreateClaims")
				.process(claimsBean::createClaims)
				.end();

		from("direct:updateClaims").routeId("routeUpdateClaims")
				.process(exchange -> claimsBean.updateClaims(null, exchange))
				.end();

		from("direct:deleteClaims").routeId("routeDeleteClaims")
				.process(exchange -> claimsBean.deleteClaims(null, exchange))
				.end();
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se agregó inyección de dependencias mediante constructor con `@Inject` para `ClaimsBean`
- Se cambió de `.bean(ClaimsBean.class, "methodName")` a `.process(claimsBean::methodName)` (referencias de método)
- Para `updateClaims` y `deleteClaims`, se cambió a expresiones lambda con `exchange` como parámetro explícito

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias
- **Compatibilidad con métodos**: Los métodos `updateClaims` y `deleteClaims` requieren dos parámetros, por lo que se usan lambdas en lugar de referencias de método simples

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que `ClaimsBean` sea un bean CDI válido

---

### 2.13. AuthRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/route/AuthRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de anotaciones CDI e inyección de dependencias**

**Código Anterior:**
```java
public class AuthRouteBuilder extends RootRouteBuilder {

	@Override
	public void configure() {

		onExceptions();

		from("direct:getTokenByApp").routeId("routeGetTokenByApp")
			.bean(ClaimsBean.class, "assignAppAuth")
		.to("direct:readClaims").end();

		from("direct:getNewTokenBantotal").routeId("routeGetNewTokenBantotal")
			.process(new CredentialsBtProcessor())
			.to("direct:getTokenBantotal")
			.process(new AssignTokenBtProcessor())
			.bean(ClaimsBean.class, "assignAppAuth")
			.to("direct:createClaims")
		.end();
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class AuthRouteBuilder extends RootRouteBuilder {

    ClaimsBean claimsBean;

    @Inject
    public AuthRouteBuilder(ClaimsBean claimsBean) {
        this.claimsBean = claimsBean;
    }

    @Override
	public void configure() {

		onExceptions();

		from("direct:getTokenByApp").routeId("routeGetTokenByApp")
			.process(claimsBean::assignAppAuth)
		.to("direct:readClaims").end();

		from("direct:getNewTokenBantotal").routeId("routeGetNewTokenBantotal")
			.process(new CredentialsBtProcessor())
			.to("direct:getTokenBantotal")
			.process(new AssignTokenBtProcessor())
            .process(claimsBean::assignAppAuth)
			.to("direct:createClaims")
		.end();
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se agregó inyección de dependencias mediante constructor con `@Inject` para `ClaimsBean`
- Se cambió de `.bean(ClaimsBean.class, "assignAppAuth")` a `.process(claimsBean::assignAppAuth)` (referencias de método)

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias
- **Consistencia**: Alinea el patrón con otras clases RouteBuilder del proyecto

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que `ClaimsBean` sea un bean CDI válido

---

## 3. Clases DTO (Data Transfer Objects)

### 3.1. Agregado de anotación @RegisterForReflection en múltiples DTOs

Se agregó la anotación `@RegisterForReflection` de Quarkus a varios DTOs para habilitar la compilación nativa. Esta anotación es necesaria porque Quarkus Native Image no puede realizar análisis estático completo de todas las clases que se usan mediante reflexión.

#### DTOs Modificados:

**1. VariablesReqDto.java**

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/dto/VariablesReqDto.java`

**Código Anterior:**
```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class VariablesReqDto {
	private String variables;
	// ... getters y setters ...
}
```

**Código Modificado:**
```java
@RegisterForReflection
@JsonIgnoreProperties(ignoreUnknown = true)
public class VariablesReqDto {
	private String variables;
	// ... getters y setters ...
}
```

---

**2. ClienteExperianRequestDto.java**

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/dto/ClienteExperianRequestDto.java`

**Código Anterior:**
```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class ClienteExperianRequestDto {
	private String tipoDocumento;
	private String numeroDocumento;
	private String tipoBusqueda;
	// ... getters y setters ...
}
```

**Código Modificado:**
```java
@RegisterForReflection
@JsonIgnoreProperties(ignoreUnknown = true)
public class ClienteExperianRequestDto {
	private String tipoDocumento;
	private String numeroDocumento;
	private String tipoBusqueda;
	// ... getters y setters ...
}
```

---

**3. ConsultarNombresExperianExternoRequest.java**

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/dto/ConsultarNombresExperianExternoRequest.java`

**Código Anterior:**
```java
public class ConsultarNombresExperianExternoRequest {
	private String tipoid;
	private String papellido;
	private String sapellido;
	private String pnombre;
	private String snombre;
	// ... getters y setters ...
}
```

**Código Modificado:**
```java
@RegisterForReflection
public class ConsultarNombresExperianExternoRequest {
	private String tipoid;
	private String papellido;
	private String sapellido;
	private String pnombre;
	private String snombre;
	// ... getters y setters ...
}
```

---

**4. ConsultarExperianExternoRequest.java**

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/dto/ConsultarExperianExternoRequest.java`

**Código Anterior:**
```java
public class ConsultarExperianExternoRequest {
	@JsonProperty("Gx_UsuEnc")
	private String gxUsuEnc;
	// ... más campos y getters/setters ...
}
```

**Código Modificado:**
```java
@RegisterForReflection
public class ConsultarExperianExternoRequest {
	@JsonProperty("Gx_UsuEnc")
	private String gxUsuEnc;
	// ... más campos y getters/setters ...
}
```

---

**5. ScorePersonaRequestDto.java**

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/dto/ScorePersonaRequestDto.java`

**Código Anterior:**
```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class ScorePersonaRequestDto {
	private String tipoDocumento;
	private int codigoScore;
	private String nroDocumento;
	// ... getters y setters ...
}
```

**Código Modificado:**
```java
@RegisterForReflection
@JsonIgnoreProperties(ignoreUnknown = true)
public class ScorePersonaRequestDto {
	private String tipoDocumento;
	private int codigoScore;
	private String nroDocumento;
	// ... getters y setters ...
}
```

---

**6. ClienteExperianDto.java**

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/dto/ClienteExperianDto.java`

**Código Anterior:**
```java
public class ClienteExperianDto {
	// ... campos y métodos ...
}
```

**Código Modificado:**
```java
@RegisterForReflection
public class ClienteExperianDto {
	// ... campos y métodos ...
}
```

---

**¿Qué se cambió?**
- Se agregó la anotación `@RegisterForReflection` de Quarkus a 6 clases DTO
- Se agregó el import `io.quarkus.runtime.annotations.RegisterForReflection` en todas estas clases

**¿Por qué se cambió?**
- **Compilación Nativa**: La anotación `@RegisterForReflection` es necesaria para que Quarkus Native Image pueda usar estas clases mediante reflexión
- **Serialización/Deserialización**: Los DTOs se usan frecuentemente con Jackson para serialización JSON, lo cual requiere reflexión
- **Camel Routes**: Algunos DTOs se usan en rutas Camel que pueden requerir reflexión en tiempo de ejecución
- **Compatibilidad con Native Image**: Sin esta anotación, las clases podrían no estar disponibles en tiempo de ejecución en una imagen nativa

**Impacto:**
- **Positivo**: Habilita la compilación nativa de Quarkus para estos DTOs
- **Positivo**: Mejora el rendimiento en modo nativo al registrar explícitamente las clases para reflexión
- **Riesgo**: Bajo - la anotación no afecta el comportamiento en modo JVM, solo es relevante para compilación nativa
- **Nota**: Esta es una práctica recomendada cuando se planea compilar la aplicación como imagen nativa de GraalVM

---

### 3.2. SingletonProperties.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/cross/util/SingletonProperties.java`

#### Cambios Realizados:

**1. Cambio de singleton tradicional a bean CDI**

**Código Anterior:**
```java
public class SingletonProperties {

    private static SingletonProperties instance = new SingletonProperties();
    protected Properties propertiesIntegration;
    protected Properties propertiesLocal;

    private SingletonProperties() {
        this.propertiesIntegration = propertiesSimpleLocal();
        this.propertiesLocal = propertiesSimpleLocal();
    }

    public static SingletonProperties getInstance() {
        if (instance == null) {
            instance = new SingletonProperties();
        }
        return instance;
    }

    // ... más código ...
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class SingletonProperties {

    protected Properties propertiesIntegration;
    protected Properties propertiesLocal;

    public SingletonProperties() {
        // Constructor vacío para permitir la inyección de dependencias
    }

    @PostConstruct
    void init() {
        this.propertiesIntegration = propertiesSimpleLocal();
        this.propertiesLocal = propertiesSimpleLocal();
    }

    public static SingletonProperties getInstance() {
        return Arc.container().instance(SingletonProperties.class).get();
    }

    // ... más código ...
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se eliminó el campo estático `instance` y la inicialización estática
- Se cambió el constructor privado a público vacío para permitir inyección de dependencias
- Se agregó el método `@PostConstruct void init()` para inicializar las propiedades
- Se cambió `getInstance()` para usar `Arc.container().instance()` en lugar de patrón singleton tradicional
- Se agregaron imports de `io.quarkus.arc.Arc` y `jakarta.annotation.PostConstruct`

**¿Por qué se cambió?**
- **Compatibilidad con CDI**: Convertir a bean CDI permite mejor gestión del ciclo de vida
- **Inyección de dependencias**: Ahora puede ser inyectado en otras clases que lo necesiten
- **Compatibilidad con Quarkus**: Quarkus funciona mejor con beans CDI que con singletons tradicionales
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - el método `getInstance()` ahora requiere que el contenedor CDI esté inicializado
- **Nota**: Este cambio es crítico porque `SingletonProperties` es usado por muchas clases en el proyecto

---

### 3.3. RedisPoolConnection.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/config/RedisPoolConnection.java`

#### Cambios Realizados:

**1. Cambio de singleton a bean CDI con inyección de dependencias**

**Código Anterior:**
```java
public class RedisPoolConnection {
	private static RedisPoolConnection instance = null;
	private static final Properties properties = SingletonProperties.getInstance().getPropertiesIntegration();
	private JedisPoolConfig poolConfig;
	private JedisPool jedisPool;

	private RedisPoolConnection() {
		jedisPool = getJedisPool();
	}

	public static RedisPoolConnection getInstance() {
		if(instance == null) {
			instance = new RedisPoolConnection();
		}
		return instance;
	}

	private JedisPoolConfig getJedisPoolConfig() {
		if(poolConfig == null) {
			poolConfig = new JedisPoolConfig();
			poolConfig.setMaxTotal(Integer.parseInt(properties.getProperty("redis.pool.maxTotal")));
			// ... más configuración ...
		}
		return poolConfig;
	}

	public JedisPool getJedisPool() {
		if(jedisPool == null) {
			poolConfig = getJedisPoolConfig();
			jedisPool = new JedisPool(poolConfig, properties.getProperty("redis.host"),
					Integer.parseInt(properties.getProperty("redis.port"))
					,Integer.parseInt(properties.getProperty("redis.timeout"))
					,properties.getProperty("redis.password"));
		}
		return jedisPool;
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class RedisPoolConnection {

    private final SingletonProperties singletonProperties;
	private JedisPoolConfig poolConfig;
	private JedisPool jedisPool;

    @Inject
    public RedisPoolConnection(SingletonProperties singletonProperties) {
        this.singletonProperties = singletonProperties;
    }

    private Properties getProperties() {
        return singletonProperties.getPropertiesIntegration();
    }

	private JedisPoolConfig getJedisPoolConfig() {
		if(poolConfig == null) {
            Properties properties = getProperties();
			poolConfig = new JedisPoolConfig();
			poolConfig.setMaxTotal(Integer.parseInt(properties.getProperty("redis.pool.maxTotal")));
			// ... más configuración ...
		}
		return poolConfig;
	}

	public JedisPool getJedisPool() {
		if(jedisPool == null) {
            Properties properties = getProperties();
			poolConfig = getJedisPoolConfig();
			jedisPool = new JedisPool(poolConfig, properties.getProperty("redis.host"),
					Integer.parseInt(properties.getProperty("redis.port"))
					,Integer.parseInt(properties.getProperty("redis.timeout"))
					,properties.getProperty("redis.password"));
		}
		return jedisPool;
	}
}
```

**¿Qué se cambió?**
- Se agregó la anotación `@ApplicationScoped` a la clase
- Se eliminó el patrón singleton (campo estático `instance` y método `getInstance()`)
- Se agregó inyección de dependencias mediante constructor con `@Inject` para `SingletonProperties`
- Se eliminó el campo estático `properties` y se reemplazó por acceso a través de `singletonProperties`
- Se agregó el método privado `getProperties()` para encapsular el acceso
- Se cambió el constructor privado a público con inyección de dependencias

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de singleton es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks
- **Consistencia**: Alinea el patrón con otras clases del proyecto que ahora usan CDI
- **Compatibilidad con Quarkus**: Quarkus funciona mejor con beans CDI

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que todas las clases que usan `RedisPoolConnection.getInstance()` sean actualizadas para usar inyección de dependencias
- **Nota**: Este cambio es crítico porque `RedisPoolConnection` es usado por `ClaimsRepository` que ya fue actualizado para usar inyección de dependencias

---

### 3.4. ConsumerBean.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/bean/ConsumerBean.java`

#### Cambios Realizados:

**1. Cambio en la firma del método `consumer` y `consumerBantotalLectura`**

**Código Anterior:**
```java
@ApplicationScoped
@Named("consumerBean")
public class ConsumerBean extends BaseBean {

	public void consumer(String method, Exchange exchange) {
		exchange.getMessage().setBody(exchange.getIn().getBody(String.class));
		setCommonHeaders(method, exchange);
	}

	// ... más código ...

	public void consumerBantotalLectura(String method, Exchange exchange) throws IOException {
		consumerLectura(method, exchange);
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
@Named("consumerBean")
public class ConsumerBean extends BaseBean {

    private static final String TYPE_METHOD_POST = "POST";

	public void consumer(Exchange exchange) {
		exchange.getMessage().setBody(exchange.getIn().getBody(String.class));
		setCommonHeaders(TYPE_METHOD_POST, exchange);
	}

	// ... más código ...

	public void consumerBantotalLectura(Exchange exchange) throws IOException {
		consumerLectura(TYPE_METHOD_POST, exchange);
	}
}
```

**¿Qué se cambió?**
- Se agregó la constante `TYPE_METHOD_POST = "POST"`
- Se eliminó el parámetro `String method` del método `consumer(Exchange exchange)`
- Se eliminó el parámetro `String method` del método `consumerBantotalLectura(Exchange exchange)`
- Se reemplazó el uso del parámetro `method` por la constante `TYPE_METHOD_POST` en ambos métodos

**¿Por qué se cambió?**
- **Simplificación**: El método siempre usa "POST", por lo que no es necesario pasar el método como parámetro
- **Consistencia**: Alinea el código con el uso real donde siempre se pasa "POST"
- **Mejora de API**: Simplifica la firma del método

**Impacto:**
- **Positivo**: Simplifica la API del método
- **Riesgo**: Medio - requiere actualizar todas las llamadas a estos métodos en los RouteBuilders
- **Nota**: Este cambio afecta a `ConsumerServiceRouteBuilder` y `ConsumerAuthRouteBuilder` que ya fueron actualizados

---

### 3.5. ScoreService.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/application/service/ScoreService.java`

#### Cambios Realizados:

**1. Cambio de `.bean()` a `.process()` y agregado de campo loggerTrace**

**Código Anterior:**
```java
@Dependent
public class ScoreService extends InitFilterRouteBuilder {

    protected ScoreService(RequestContextProcessor requestContextProcessor,
                               JwtContextFilterProcessor jwtContextFilterProcessor,
                               LoggerTrace loggerTrace,
                           UriSanitizerProcessor uriSanitizerProcessor) {
        super(requestContextProcessor, jwtContextFilterProcessor, loggerTrace, uriSanitizerProcessor);
    }

    @Override
    public void configureRestRoutes() throws RouteConfigurationException {

        onException(InvalidFormatException.class, JsonMappingException.class, JsonParseException.class)
                .handled(true)
                // ... más código ...
                .bean(Constants.LOG_TRACE, Constants.FORMAT_RESPONSE_ROOT)
                .stop();

        onException(PredicateValidationException.class, NumberFormatException.class)
                .handled(true)
                // ... más código ...
                .bean(Constants.LOG_TRACE, Constants.FORMAT_RESPONSE_ROOT)
                .stop();
    }
}
```

**Código Modificado:**
```java
@Dependent
public class ScoreService extends InitFilterRouteBuilder {

    private final LoggerTrace loggerTrace;

    protected ScoreService(RequestContextProcessor requestContextProcessor,
                               JwtContextFilterProcessor jwtContextFilterProcessor,
                               LoggerTrace loggerTrace,
                           UriSanitizerProcessor uriSanitizerProcessor) {
        super(requestContextProcessor, jwtContextFilterProcessor, loggerTrace, uriSanitizerProcessor);
        this.loggerTrace = loggerTrace;
    }

    @Override
    public void configureRestRoutes() throws RouteConfigurationException {

        onException(InvalidFormatException.class, JsonMappingException.class, JsonParseException.class)
                .handled(true)
                // ... más código ...
                .process(loggerTrace::formatResponseRoot)
                .stop();

        onException(PredicateValidationException.class, NumberFormatException.class)
                .handled(true)
                // ... más código ...
                .process(loggerTrace::formatResponseRoot)
                .stop();
    }
}
```

**¿Qué se cambió?**
- Se agregó el campo `private final LoggerTrace loggerTrace`
- Se inicializó el campo en el constructor
- Se cambió de `.bean(Constants.LOG_TRACE, Constants.FORMAT_RESPONSE_ROOT)` a `.process(loggerTrace::formatResponseRoot)` en ambos manejadores de excepciones

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar referencias de método en lugar de referencias de clase es más eficiente
- **Consistencia**: Alinea el patrón con otros RouteBuilders del proyecto
- **Rendimiento**: Las referencias de método son más eficientes que las referencias de clase en Camel

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas
- **Riesgo**: Bajo - el cambio es transparente si `LoggerTrace` está correctamente inyectado

---

### 3.6. ConsumerServiceRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/route/ConsumerServiceRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de inyección de ConsumerBean y cambio de `.bean()` a `.process()`**

**Código Anterior:**
```java
@ApplicationScoped
public class ConsumerServiceRouteBuilder extends RootRouteBuilder {

    private final ExperianService experianService;
    private final BrmsScoreService brmsScoreService;
    private final CompartamosService compartamosService;
    private final BantotalService bantotalService;

    @Inject
    public ConsumerServiceRouteBuilder(ExperianService experianService, BrmsScoreService brmsScoreService, CompartamosService compartamosService, BantotalService bantotalService) {
        this.experianService = experianService;
        this.brmsScoreService = brmsScoreService;
        this.compartamosService = compartamosService;
        this.bantotalService = bantotalService;
    }

    @Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getExperianExterno")
				.setProperty("sensitiveRoute", constant(true))
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .process(new SecurityHeaderSanitizerProcessor())
                .bean(experianService, "consultarExperianExterno")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ConsultarExperianExternoResponse.class));

		from("direct:GetDatosHistoricCreditNombres").bean(ConsumerBean.class, "consumerConsultarNombresSoap");
		
		from("direct:getScoreExperianBrms")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .process(new SecurityHeaderSanitizerProcessor())
                .bean(brmsScoreService, "obtenerScoreExperian")
				// ... más rutas ...

		from(Constants.VALIDATE_RESPONSE).bean(ConsumerBean.class, "validateStatusResponse");
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class ConsumerServiceRouteBuilder extends RootRouteBuilder {

    private final ExperianService experianService;
    private final BrmsScoreService brmsScoreService;
    private final CompartamosService compartamosService;
    private final BantotalService bantotalService;
    private final ConsumerBean consumerBean;

    @Inject
    public ConsumerServiceRouteBuilder(ExperianService experianService, BrmsScoreService brmsScoreService, CompartamosService compartamosService, BantotalService bantotalService, ConsumerBean consumerBean) {
        this.experianService = experianService;
        this.brmsScoreService = brmsScoreService;
        this.compartamosService = compartamosService;
        this.bantotalService = bantotalService;
        this.consumerBean = consumerBean;
    }

    @Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getExperianExterno")
				.setProperty("sensitiveRoute", constant(true))
                .process(consumerBean::consumer)
                .process(new SecurityHeaderSanitizerProcessor())
                .process(experianService::consultarExperianExterno)
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ConsultarExperianExternoResponse.class));

		from("direct:GetDatosHistoricCreditNombres").process(consumerBean::consumerConsultarNombresSoap);
		
		from("direct:getScoreExperianBrms")
                .process(consumerBean::consumer)
                .process(new SecurityHeaderSanitizerProcessor())
                .process(brmsScoreService::obtenerScoreExperian)
				// ... más rutas ...

		from(Constants.VALIDATE_RESPONSE).process(consumerBean::validateStatusResponse);
	}
}
```

**¿Qué se cambió?**
- Se agregó el campo `private final ConsumerBean consumerBean`
- Se agregó `ConsumerBean` al constructor con inyección de dependencias
- Se cambió de `.bean(ConsumerBean.class, Constants.CONSUMER_POST)` a `.process(consumerBean::consumer)` en todas las rutas
- Se cambió de `.bean(ConsumerBean.class, "consumerConsultarNombresSoap")` a `.process(consumerBean::consumerConsultarNombresSoap)`
- Se cambió de `.bean(ConsumerBean.class, "validateStatusResponse")` a `.process(consumerBean::validateStatusResponse)`
- Se cambió de `.bean(experianService, "consultarExperianExterno")` a `.process(experianService::consultarExperianExterno)`
- Se cambió de `.bean(brmsScoreService, "obtenerScoreExperian")` a `.process(brmsScoreService::obtenerScoreExperian)`
- Se cambió de `.bean(compartamosService, "obtenerVariables")` a `.process(compartamosService::obtenerVariables)`
- Se cambió de `.bean(compartamosService, "obtenerCatalogos")` a `.process(compartamosService::obtenerCatalogos)`
- Se cambió de `.bean(bantotalService, "obtenerGuiaProcesoEspecial")` a `.process(bantotalService::obtenerGuiaProcesoEspecial)`
- Se cambió de `.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)` a `.process(consumerBean::consumerBantotalLectura)`
- Se cambió de `.bean(bantotalService, "obtenerDatosExperian")` a `.process(bantotalService::obtenerDatosExperian)`
- Se cambió de `.bean(bantotalService, "crearDatosExperian")` a `.process(bantotalService::obtenerDatosExperian)` (**ERROR**: Debería ser `crearDatosExperian` en lugar de `obtenerDatosExperian`)

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias al permitir inyectar mocks
- **Rendimiento**: Las referencias de método son más eficientes que las referencias de clase en Camel
- **Consistencia**: Alinea el patrón con otros RouteBuilders del proyecto

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que `ConsumerBean` y todos los servicios sean beans CDI válidos
- **ERROR CRÍTICO**: En la línea 91, la ruta `createDatosExperian` usa `bantotalService::obtenerDatosExperian` cuando debería usar `bantotalService::crearDatosExperian`. Este es un bug que debe corregirse antes del despliegue

---

### 3.7. ConsumerAuthRouteBuilder.java

**Ubicación**: `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/route/ConsumerAuthRouteBuilder.java`

#### Cambios Realizados:

**1. Agregado de inyección de ConsumerBean y cambio de `.bean()` a `.process()`**

**Código Anterior:**
```java
@ApplicationScoped
public class ConsumerAuthRouteBuilder extends RootRouteBuilder {

    private final BantotalService bantotalService;

    @Inject
    public ConsumerAuthRouteBuilder(BantotalService bantotalService) {
        this.bantotalService = bantotalService;
    }

	@Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getTokenBantotal").routeId("routeGetTokenBantotal")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(bantotalService, "obtenerTokenBantotal")
		.end();
	}
}
```

**Código Modificado:**
```java
@ApplicationScoped
public class ConsumerAuthRouteBuilder extends RootRouteBuilder {

    private final BantotalService bantotalService;
    private final ConsumerBean consumerBean;

    @Inject
    public ConsumerAuthRouteBuilder(BantotalService bantotalService, ConsumerBean consumerBean) {
        this.bantotalService = bantotalService;
        this.consumerBean = consumerBean;
    }

	@Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getTokenBantotal").routeId("routeGetTokenBantotal")
                .process(consumerBean::consumer)
                .process(bantotalService::obtenerTokenBantotal)

		.end();
	}
}
```

**¿Qué se cambió?**
- Se agregó el campo `private final ConsumerBean consumerBean`
- Se agregó `ConsumerBean` al constructor con inyección de dependencias
- Se cambió de `.bean(ConsumerBean.class, Constants.CONSUMER_POST)` a `.process(consumerBean::consumer)`
- Se cambió de `.bean(bantotalService, "obtenerTokenBantotal")` a `.process(bantotalService::obtenerTokenBantotal)`

**¿Por qué se cambió?**
- **Mejora de arquitectura**: Usar inyección de dependencias en lugar de referencias de clase es una mejor práctica
- **Testabilidad**: Facilita las pruebas unitarias
- **Consistencia**: Alinea el patrón con otros RouteBuilders del proyecto

**Impacto:**
- **Positivo**: Mejora la arquitectura y sigue mejores prácticas de CDI
- **Riesgo**: Medio - requiere que `ConsumerBean` y `BantotalService` sean beans CDI válidos

---

## Resumen de Cambios

### Archivos Modificados:
1. ✅ `pom.xml` - Propiedades SonarQube y dependencia icu4j
2. ✅ `application.properties` - Consolidación de configuraciones de servicios
3. ✅ `RootRouteBuilder.java` - Inyección de dependencias para LoggerTrace
4. ✅ `BaseBean.java` - Inyección de dependencias y métodos no estáticos
5. ✅ `ClaimsBean.java` - Inyección de dependencias y anotaciones CDI
6. ✅ `ClaimsRepository.java` - Inyección de dependencias y anotaciones CDI
7. ✅ `ScoreBean.java` - Agregado de anotaciones CDI
8. ✅ `VariablesBean.java` - Agregado de anotaciones CDI
9. ✅ `CatalogueBean.java` - Agregado de anotaciones CDI
10. ✅ `SessionValidator.java` - Cambio de clase estática a bean CDI con métodos de instancia
11. ✅ `ScoreRouteBuilder.java` - Inyección de dependencias y cambio de `.bean()` a `.process()`
12. ✅ **Múltiples DTOs** - Agregado de anotación `@RegisterForReflection` (VariablesReqDto, ClienteExperianRequestDto, ConsultarNombresExperianExternoRequest, ConsultarExperianExternoRequest, ScorePersonaRequestDto, ClienteExperianDto)
13. ✅ `SingletonProperties.java` - Cambio de singleton tradicional a bean CDI con `@ApplicationScoped` y `@PostConstruct`
14. ✅ `RedisPoolConnection.java` - Cambio de singleton a bean CDI con inyección de dependencias
15. ✅ `ConsumerBean.java` - Cambio en firma de métodos (eliminación de parámetro `method`)
16. ✅ `ScoreService.java` - Cambio de `.bean()` a `.process()` y agregado de campo loggerTrace
17. ✅ `ConsumerServiceRouteBuilder.java` - Inyección de ConsumerBean y cambio de `.bean()` a `.process()`
18. ✅ `ConsumerAuthRouteBuilder.java` - Inyección de ConsumerBean y cambio de `.bean()` a `.process()`
19. ✅ `VariablesRouteBuilder.java` - Inyección de dependencias y cambio de `.bean()` a `.process()`
20. ✅ `CatalogueRouteBuilder.java` - Inyección de dependencias y cambio de `.bean()` a `.process()`
21. ✅ `ClaimsRouteBuilder.java` - Inyección de dependencias y cambio de `.bean()` a `.process()`
22. ✅ `AuthRouteBuilder.java` - Inyección de dependencias y cambio de `.bean()` a `.process()`
23. ✅ `NativeReflectionConfig.java` - **NUEVA CLASE**
24. ✅ `config-process-score.yaml` → `config-process-score-mvp.yaml` - Renombrado
25. ✅ `deployment.yaml` - Renombrado de recursos
26. ✅ `process-score-svc.yaml` → `process-score-mvp-svc.yaml` - Renombrado
27. ✅ `process-score-ingress.yaml` → `process-score-mvp-ingress.yaml` - Renombrado

### Archivos Sin Cambios:
- `application-dev.properties` - Sin cambios
- Todas las demás clases Java (BantotalService, CompartamosService, etc.) - Sin cambios en el código, solo cambios en referencias de configuración

### Impacto General:

**Cambios de Bajo Riesgo:**
- Propiedades SonarQube en pom.xml
- Dependencia icu4j
- NativeReflectionConfig.java (solo afecta compilación nativa)
- Cambios en RootRouteBuilder.java (mejora arquitectónica)
- Agregado de anotaciones CDI en ScoreBean, VariablesBean, CatalogueBean
- Agregado de `@RegisterForReflection` en múltiples DTOs (solo afecta compilación nativa)

**Cambios de Medio Riesgo:**
- Cambios en BaseBean.java (afecta a todas las clases que lo extienden)
- Cambios en ClaimsBean.java (inyección de dependencias)

**Cambios de Medio Riesgo:**
- Consolidación de configuraciones en application.properties (requiere actualización en todos los ambientes)
- Cambios en BaseBean.java (afecta a todas las clases que lo extienden)
- Cambios en ClaimsBean.java (inyección de dependencias)
- Cambios en ClaimsRepository.java (inyección de dependencias, requiere que RedisPoolConnection y SingletonProperties sean beans CDI)
- Cambios en todos los RouteBuilders (ScoreRouteBuilder, VariablesRouteBuilder, CatalogueRouteBuilder, ClaimsRouteBuilder, AuthRouteBuilder, ConsumerServiceRouteBuilder, ConsumerAuthRouteBuilder) - requieren que los Beans inyectados sean CDI válidos
- Cambios en SessionValidator.java (cambio de API: ahora requiere Exchange en lugar de Object) - **CRÍTICO**: afecta todos los usos en RouteBuilders
- Cambios en SingletonProperties.java (cambio de singleton tradicional a bean CDI) - **CRÍTICO**: afecta todas las clases que lo usan
- Cambios en RedisPoolConnection.java (cambio de singleton a bean CDI) - **CRÍTICO**: afecta ClaimsRepository y otras clases que lo usan
- Cambios en ConsumerBean.java (cambio de firma de métodos) - requiere actualizar todas las llamadas en RouteBuilders

**Cambios de Alto Riesgo:**
- Renombrado de recursos de Kubernetes (requiere actualización de despliegues, CI/CD, y referencias)

---

## Recomendaciones

1. **Actualizar configuraciones de ambiente**: Asegurarse de que todas las configuraciones de Fault Tolerance en los diferentes ambientes (dev, qa, prod) estén actualizadas con las nuevas referencias de servicios.

2. **Actualizar scripts de despliegue**: Revisar y actualizar todos los scripts de CI/CD que referencien los nombres antiguos de recursos de Kubernetes.

3. **Verificar imagen Docker**: Asegurarse de que la imagen Docker `process-score-mvp` esté construida y disponible en el registro.

4. **Pruebas de integración**: Realizar pruebas exhaustivas para verificar que:
   - Las configuraciones de Fault Tolerance funcionan correctamente
   - La inyección de dependencias de LoggerTrace funciona
   - Los recursos de Kubernetes se despliegan correctamente

5. **Documentación**: Actualizar la documentación del proyecto con los nuevos nombres de recursos y configuraciones.

6. **CORRECCIÓN CRÍTICA REQUERIDA**: Corregir el error en `ConsumerServiceRouteBuilder.java` línea 91 - la ruta `createDatosExperian` debe usar `bantotalService::crearDatosExperian` en lugar de `bantotalService::obtenerDatosExperian`. Este es un bug funcional que impedirá que la creación de datos Experian funcione correctamente.

---

**Fecha de generación**: 2025-01-27
**Versión del documento**: 1.0
