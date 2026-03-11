# Documento de Cambios: Migración a Resiliencia

## Resumen Ejecutivo

Este documento detalla todos los cambios realizados en el proyecto `process-score-mvp` durante la migración desde la versión base (`migracion`) hacia la versión con capacidades de resiliencia (`resiliencia`). La migración introduce un framework de resiliencia que proporciona circuit breakers, timeouts, retries y métricas para las llamadas HTTP a servicios externos.

---

## 1. Archivos de Configuración

### 1.1. `pom.xml`

**Ubicación:** `process-score-mvp/pom.xml`

#### ¿Qué se cambió?

Se agregaron nuevas dependencias y propiedades relacionadas con el framework de resiliencia.

#### Código Anterior:

```xml
<properties>
    <compiler-plugin.version>3.14.0</compiler-plugin.version>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <quarkus.platform.version>3.27.0</quarkus.platform.version>
    <surefire-plugin.version>3.5.3</surefire-plugin.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.quarkus.platform</groupId>
            <artifactId>quarkus-bom</artifactId>
            <version>${quarkus.platform.version}</version>
            <type>pom</type>
            <scope>import</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.commons</groupId>
                    <artifactId>commons-lang3</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.quarkus.platform</groupId>
            <artifactId>quarkus-camel-bom</artifactId>
            <version>${quarkus.platform.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.apache.camel.quarkus</groupId>
        <artifactId>camel-quarkus-core</artifactId>
    </dependency>
    <!-- ... resto de dependencias ... -->
</dependencies>
```

#### Código Modificado:

```xml
<properties>
    <compiler-plugin.version>3.14.0</compiler-plugin.version>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <quarkus.platform.version>3.27.0</quarkus.platform.version>
    <surefire-plugin.version>3.5.3</surefire-plugin.version>

    <resilience-starter.version>1.0.0-SNAPSHOT</resilience-starter.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.quarkus.platform</groupId>
            <artifactId>quarkus-bom</artifactId>
            <version>${quarkus.platform.version}</version>
            <type>pom</type>
            <scope>import</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.commons</groupId>
                    <artifactId>commons-lang3</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.quarkus.platform</groupId>
            <artifactId>quarkus-camel-bom</artifactId>
            <version>${quarkus.platform.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>com.compartamos.integration.framework</groupId>
            <artifactId>qompa-resilience-starter</artifactId>
            <version>${resilience-starter.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>io.quarkus</groupId>
                    <artifactId>quarkus-resteasy-reactive</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Dependencia del starter de resiliencia-->
    <dependency>
        <groupId>com.compartamos.integration.framework</groupId>
        <artifactId>qompa-resilience-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-micrometer</artifactId>
    </dependency>

    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-smallrye-fault-tolerance</artifactId>
    </dependency>

    <!-- Fin de dependencia del starter de resiliencia-->

    <dependency>
        <groupId>org.apache.camel.quarkus</groupId>
        <artifactId>camel-quarkus-core</artifactId>
    </dependency>
    <!-- ... resto de dependencias ... -->
</dependencies>
```

#### ¿Por qué se cambió?

Para integrar el framework de resiliencia (`qompa-resilience-starter`) que proporciona:
- Circuit breakers para evitar llamadas a servicios fallidos
- Timeouts configurables por servicio
- Retries automáticos con backoff
- Métricas de Prometheus para monitoreo
- Anotaciones declarativas para aplicar políticas de resiliencia

#### Impacto:

- **Positivo:** 
  - Mejora la estabilidad y disponibilidad del servicio
  - Reduce el impacto de fallos en servicios externos
  - Facilita el monitoreo y observabilidad
  - Permite configurar políticas de resiliencia por servicio
- **Consideraciones:**
  - Requiere configuración adicional en `application.properties`
  - Aumenta la complejidad del proyecto
  - Necesita validación de las políticas configuradas

---

### 1.2. `application.properties`

**Ubicación:** `process-score-mvp/src/main/resources/application.properties`

#### ¿Qué se cambió?

Se agregaron configuraciones globales y específicas por servicio para el framework de resiliencia, incluyendo timeouts, circuit breakers y métricas.

#### Código Anterior:

```properties
# [END] BANTOTAL - SOLO CONSULTA
```

#### Código Modificado:

```properties
# [END] BANTOTAL - SOLO CONSULTA


# Resilience Starter - Configuracion Global
resilience.enabled=true
resilience.mode=STRICT
resilience.deployment-environment=${DEPLOYMENT_ENV:production}

# MicroProfile Fault Tolerance - Configuracion Global
mp.fault.tolerance.metrics.enabled=true

# Metricas (Prometheus)
quarkus.micrometer.export.prometheus.path=/metrics
quarkus.micrometer.registry-enabled-default=true

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

# ... (configuraciones similares para otros servicios: BantotalExperianService, BantotalCatalogoService, BantotalConfiguracionService, BantotalAuthService, CompartamosCatalogoService, ExperianExternalService, BrmsScoreService, CompartamosScoService)
```

#### ¿Por qué se cambió?

Para configurar políticas de resiliencia específicas por servicio, permitiendo:
- Timeouts personalizados según el tipo de servicio (FAST, MEDIUM)
- Circuit breakers con umbrales de fallo configurables
- Delays y thresholds para la recuperación de circuit breakers
- Variables de entorno para ajustar valores sin recompilar

#### Impacto:

- **Positivo:**
  - Control granular sobre el comportamiento de resiliencia
  - Configuración externa mediante variables de entorno
  - Perfiles diferenciados (FAST para servicios rápidos, MEDIUM para servicios normales)
- **Consideraciones:**
  - Muchas propiedades nuevas que deben ser gestionadas
  - Requiere conocimiento de los valores óptimos por servicio
  - Necesita monitoreo para ajustar los valores

---

### 1.3. `application-dev.properties`

**Ubicación:** `process-score-mvp/src/main/resources/application-dev.properties`

#### ¿Qué se cambió?

Se agregaron configuraciones de logging para el framework de resiliencia en modo desarrollo.

#### Código Anterior:

```properties
# [END] BANTOTAL - SOLO CONSULTA
```

#### Código Modificado:

```properties
# [END] BANTOTAL - SOLO CONSULTA

# Logging más detallado
quarkus.log.category."com.compartamos.integration.framework.resilience".level=DEBUG
quarkus.log.category."org.eclipse.microprofile.faulttolerance".level=DEBUG
```

#### ¿Por qué se cambió?

Para habilitar logging detallado del framework de resiliencia y MicroProfile Fault Tolerance durante el desarrollo, facilitando la depuración y el entendimiento del comportamiento de los circuit breakers, retries y timeouts.

#### Impacto:

- **Positivo:**
  - Facilita el debugging durante el desarrollo
  - Permite entender el comportamiento de las políticas de resiliencia
- **Consideraciones:**
  - Puede generar mucho log en producción si se usa en ese ambiente
  - Solo debe estar activo en desarrollo

---

## 2. Capa de Infraestructura

### 2.1. `ConsumerAuthRouteBuilder.java`

**Ubicación:** `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/route/ConsumerAuthRouteBuilder.java`

#### ¿Qué se cambió?

Se refactorizó para usar el servicio inyectado `BantotalAuthService` en lugar de `recipientList` directo, aplicando políticas de resiliencia a la llamada de autenticación.

#### Código Anterior:

```java
public class ConsumerAuthRouteBuilder extends RootRouteBuilder {

	@Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getTokenBantotal").routeId("routeGetTokenBantotal")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.BANTOTAL_URI + "odwsbt_Authenticate_v1?Execute" + Constants.AND_BRIDGE_ENDPOINT))
		.end();
	}
	
}
```

#### Código Modificado:

```java
@ApplicationScoped
public class ConsumerAuthRouteBuilder extends RootRouteBuilder {

    @Inject
    BantotalAuthService bantotalAuthService;

	@Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getTokenBantotal").routeId("routeGetTokenBantotal")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(bantotalAuthService, "obtenerTokenBantotal")
		.end();
	}
	
}
```

#### ¿Por qué se cambió?

Para aplicar políticas de resiliencia (timeout, circuit breaker, retry) a la llamada de autenticación con Bantotal mediante el servicio `BantotalAuthService`, mejorando la estabilidad del proceso de obtención de tokens.

#### Impacto:

- **Positivo:**
  - Protección automática contra fallos en la autenticación
  - Reintentos automáticos en caso de fallos transitorios
  - Circuit breaker para evitar llamadas repetidas a un servicio fallido
- **Consideraciones:**
  - Requiere el servicio `BantotalAuthService` creado
  - La anotación `@ApplicationScoped` es necesaria para la inyección de dependencias

---

### 2.2. `ConsumerServiceRouteBuilder.java`

**Ubicación:** `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/proxy/route/ConsumerServiceRouteBuilder.java`

#### ¿Qué se cambió?

Se refactorizó para usar servicios inyectados en lugar de `recipientList` directo. Las llamadas HTTP ahora se delegan a servicios especializados que aplican políticas de resiliencia.

#### Código Anterior:

```java
public class ConsumerServiceRouteBuilder extends RootRouteBuilder {

	@Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getExperianExterno")
				.setProperty("sensitiveRoute", constant(true))
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.EXPERIAN_EXTERNO_URI + Constants.BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ConsultarExperianExternoResponse.class));

		from("direct:getScoreExperianBrms")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.BRMS_URI_FUSE_SCORE_EXPERIAN + Constants.BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:getVariablesSistema")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.COMPARTAMOS_URI_SEGURIDAD + "api/variables/obtenervariables" + Constants.BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseProxyResponse.class));

		from("direct:getCodigoCatalogos")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.COMPARTAMOS_URI_CATALOGO + "api/catalogo/obtenercatalogos" + Constants.BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:getDatosGuiaProceso")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
				.recipientList(constant(Constants.BANTOTAL_URI_CATALOGUE_GET_SPECIAL_PROCESS_GUIDE + Constants.AND_BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerGuiaProcesoEspecialResponse.class));

		from("direct:getDatosExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.BANTOTAL_URI_EXPERIAN + "ObtenerDatosExperian" + Constants.AND_BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerDatosExperianResponse.class));

		from("direct:createDatosExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.BANTOTAL_URI_EXPERIAN + "CrearDatosExperian" + Constants.AND_BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(CrearDatosExperianResponse.class));

		from("direct:getDatosSistemDate")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
				.recipientList(constant(Constants.BANTOTAL_URI_CONFIG_GET_SYSTEM_DATE + Constants.AND_BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerFechaDeSistemaResponse.class));

		from("direct:getDatosCantidadConsultasExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.COMPARTAMOS_URI_SCO + "api/Experian/ObtenerCantidadConsultasExperian" + Constants.BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:createDatosCantidadConsultasExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
				.recipientList(constant(Constants.COMPARTAMOS_URI_SCO + "api/Experian/RegistrarCantidadConsultasExperian" + Constants.BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:getDatosScore")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
				.recipientList(constant(Constants.BANTOTAL_URI_GET_SCORE + Constants.AND_BRIDGE_ENDPOINT))
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerScorePersonaResponse.class));

		from(Constants.VALIDATE_RESPONSE).bean(ConsumerBean.class, "validateStatusResponse");
	}
}
```

#### Código Modificado:

```java
@ApplicationScoped
public class ConsumerServiceRouteBuilder extends RootRouteBuilder {

    @Inject
    ExperianExternalService experianExternalService;

    @Inject
    BrmsScoreService brmsScoreService;

    @Inject
    CompartamosSeguridadService compartamosSeguridadService;

    @Inject
    CompartamosCatalogoService compartamosCatalogoService;

    @Inject
    CompartamosScoService compartamosScoService;

    @Inject
    BantotalExperianService bantotalExperianService;

    @Inject
    BantotalCatalogoService bantotalCatalogoService;

    @Inject
    BantotalConfiguracionService bantotalConfiguracionService;

    @Inject
    BantotalScoreService bantotalScoreService;

	@Override
	public void configure() {

		onExceptions();
		onInterceptLog();

		from("direct:getExperianExterno")
				.setProperty("sensitiveRoute", constant(true))
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(experianExternalService, "consultarExperianExterno")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ConsultarExperianExternoResponse.class));

		from("direct:getScoreExperianBrms")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(brmsScoreService, "obtenerScoreExperian")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:getVariablesSistema")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(compartamosSeguridadService, "obtenerVariables")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseProxyResponse.class));

		from("direct:getCodigoCatalogos")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(compartamosCatalogoService, "obtenerCatalogos")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:getDatosGuiaProceso")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
                .bean(bantotalCatalogoService, "obtenerGuiaProcesoEspecial")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerGuiaProcesoEspecialResponse.class));

		from("direct:getDatosExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(bantotalExperianService, "obtenerDatosExperian")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerDatosExperianResponse.class));

		from("direct:createDatosExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(bantotalExperianService, "crearDatosExperian")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(CrearDatosExperianResponse.class));

		from("direct:getDatosSistemDate")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
                .bean(bantotalConfiguracionService, "obtenerFechaSistema")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerFechaDeSistemaResponse.class));

		from("direct:getDatosCantidadConsultasExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(compartamosScoService, "obtenerCantidadConsultasExperian")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:createDatosCantidadConsultasExperian")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST)
                .bean(compartamosScoService, "registrarCantidadConsultasExperian")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(BaseResponse.class));

		from("direct:getDatosScore")
				.bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
                .bean(bantotalScoreService, "obtenerScorePersona")
				.to(Constants.VALIDATE_RESPONSE)
				.unmarshal(new JacksonDataFormat(ObtenerScorePersonaResponse.class));

		from(Constants.VALIDATE_RESPONSE).bean(ConsumerBean.class, "validateStatusResponse");
	}
}
```

#### ¿Por qué se cambió?

Para separar la lógica de llamadas HTTP en servicios especializados que aplican políticas de resiliencia (circuit breakers, timeouts, retries) de forma declarativa mediante anotaciones. Esto mejora:
- **Separación de responsabilidades:** Cada servicio externo tiene su propia clase
- **Aplicación de resiliencia:** Las anotaciones `@ResilientHttpOutbound`, `@Timeout`, `@CircuitBreaker`, `@Retry` se aplican automáticamente
- **Testabilidad:** Los servicios pueden ser mockeados fácilmente
- **Mantenibilidad:** Cambios en políticas de resiliencia se hacen en un solo lugar

#### Impacto:

- **Positivo:**
  - Mejor organización del código
  - Aplicación automática de políticas de resiliencia
  - Facilita el testing unitario
  - Permite reutilización de servicios
- **Consideraciones:**
  - Requiere crear múltiples clases de servicio
  - Aumenta el número de archivos en el proyecto
  - Necesita inyección de dependencias (CDI)

---

## 3. Nuevos Servicios Externos

Se crearon 10 nuevos servicios en la carpeta `infrastructure/service/external/` para encapsular las llamadas HTTP a servicios externos con políticas de resiliencia aplicadas.

### 3.1. `BantotalScoreService.java`

**Ubicación:** `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/service/external/bantotal/BantotalScoreService.java`

#### ¿Qué se cambió?

**Archivo nuevo.** Servicio que encapsula la llamada HTTP a Bantotal para obtener el score de una persona, aplicando políticas de resiliencia.

#### Código:

```java
@ApplicationScoped
@Named
public class BantotalScoreService {

	private static final Logger LOG = Logger.getLogger(BantotalScoreService.class);

	@Inject
	ProducerTemplate producerTemplate;

	@ConfigProperty(name = "bantotal.wsobtenerscorepersona.v1.obtenerscore.uri")
	String bantotalUriScore;

	@ResilientHttpOutbound(
		policyRef = "bantotal-score",
		tier = ResilienceProfile.MEDIUM,
		endpointAlias = "bantotal.score.obtenerScorePersona",
		dependency = "BANTOTAL",
		owner = "team-process-score"
	)
	@Idempotent
	@Timeout(2000)
	@Retry(
		maxRetries = 2,
		delay = 500,
		jitter = 200
	)
	@CircuitBreaker(
		requestVolumeThreshold = 20,
		failureRatio = 0.5,
		delay = 5000,
		successThreshold = 2
	)
	public void obtenerScorePersona(Exchange exchange) {
		String uriCompleta = bantotalUriScore + Constants.AND_BRIDGE_ENDPOINT;

		try {
			Exchange responseExchange = producerTemplate.request(uriCompleta, ex -> {
				String requestBody = exchange.getMessage().getBody(String.class);
				ex.getIn().setBody(requestBody != null ? requestBody : "");
				ex.getIn().setHeaders(exchange.getMessage().getHeaders());
				for (Map.Entry<String, Object> property : exchange.getProperties().entrySet()) {
					ex.setProperty(property.getKey(), property.getValue());
				}
			});

			String responseBody = responseExchange.getIn().getBody(String.class);
			exchange.getIn().setBody(responseBody != null ? responseBody : "");
			exchange.getIn().setHeaders(responseExchange.getIn().getHeaders());

			Integer responseCode = responseExchange.getIn().getHeader(Exchange.HTTP_RESPONSE_CODE, Integer.class);
			exchange.getIn().setHeader(Exchange.HTTP_RESPONSE_CODE, responseCode != null ? responseCode : 200);

		} catch (Exception e) {
			LOG.errorf("Unexpected error calling Bantotal Score: %s", e.getMessage(), e);
			throw new RuntimeException("Error calling Bantotal Score service", e);
		}
	}
}
```

#### ¿Por qué se creó?

Para encapsular la llamada HTTP a Bantotal Score con:
- **Timeout de 2000ms:** Evita esperas indefinidas
- **Retry con 2 intentos:** Reintenta automáticamente en caso de fallo transitorio
- **Circuit Breaker:** Abre el circuito si hay más del 50% de fallos en 20 requests
- **Idempotencia:** Garantiza que llamadas duplicadas no causen efectos secundarios
- **Métricas:** Expone métricas para monitoreo

#### Impacto:

- **Positivo:**
  - Protección automática contra fallos del servicio externo
  - Mejora la disponibilidad del servicio
  - Facilita el monitoreo mediante métricas
- **Consideraciones:**
  - Requiere configuración en `application.properties`
  - Los valores de timeout y circuit breaker deben ajustarse según el comportamiento real

---

### 3.2. `CompartamosSeguridadService.java`

**Ubicación:** `process-score-mvp/src/main/java/com/compartamos/process/score/infrastructure/service/external/compartamos/CompartamosSeguridadService.java`

#### ¿Qué se cambió?

**Archivo nuevo.** Servicio que encapsula la llamada HTTP a Compartamos Seguridad para obtener variables del sistema, usando un perfil FAST (timeout más corto).

#### Código:

```java
@ApplicationScoped
@Named
public class CompartamosSeguridadService {

	private static final Logger LOG = Logger.getLogger(CompartamosSeguridadService.class);

	@Inject
	ProducerTemplate producerTemplate;

	@ConfigProperty(name = "compartamos.uri_seguridad")
	String compartamosUriSeguridad;

	@ResilientHttpOutbound(
		policyRef = "compartamos-seguridad",
		tier = ResilienceProfile.FAST,
		endpointAlias = "compartamos.seguridad.obtenerVariables",
		dependency = "COMPARTAMOS_SEGURIDAD",
		owner = "team-process-score"
	)
	@Idempotent
	@Timeout(800)
	@Retry(
		maxRetries = 2,
		delay = 300,
		jitter = 100
	)
	@CircuitBreaker(
		requestVolumeThreshold = 10,
		failureRatio = 0.4,
		delay = 3000,
		successThreshold = 2
	)
	public void obtenerVariables(Exchange exchange) {
		String uriCompleta = compartamosUriSeguridad + "api/variables/obtenervariables" + Constants.BRIDGE_ENDPOINT;

		try {
			Exchange responseExchange = producerTemplate.request(uriCompleta, ex -> {
				String requestBody = exchange.getMessage().getBody(String.class);
				ex.getIn().setBody(requestBody != null ? requestBody : "");
				ex.getIn().setHeaders(exchange.getMessage().getHeaders());
				for (Map.Entry<String, Object> property : exchange.getProperties().entrySet()) {
					ex.setProperty(property.getKey(), property.getValue());
				}
			});

			String responseBody = responseExchange.getIn().getBody(String.class);
			exchange.getIn().setBody(responseBody != null ? responseBody : "");
			exchange.getIn().setHeaders(responseExchange.getIn().getHeaders());

			Integer responseCode = responseExchange.getIn().getHeader(Exchange.HTTP_RESPONSE_CODE, Integer.class);
			exchange.getIn().setHeader(Exchange.HTTP_RESPONSE_CODE, responseCode != null ? responseCode : 200);

		} catch (Exception e) {
			LOG.errorf("Unexpected error calling Compartamos Seguridad: %s", e.getMessage(), e);
			throw new RuntimeException("Error calling Compartamos Seguridad service", e);
		}
	}
}
```

#### ¿Por qué se creó?

Para servicios que deben responder rápidamente (perfil FAST):
- **Timeout de 800ms:** Más estricto que servicios MEDIUM
- **Circuit Breaker más sensible:** Se abre con 40% de fallos en 10 requests
- **Retry más rápido:** Delay de 300ms vs 500ms

#### Impacto:

- **Positivo:**
  - Respuesta más rápida en caso de fallo
  - Protección temprana contra servicios degradados
- **Consideraciones:**
  - Puede abrir el circuito más frecuentemente si el servicio es lento
  - Requiere que el servicio externo sea realmente rápido

---

### 3.3. Otros Servicios Externos Creados

Se crearon los siguientes servicios adicionales con estructura similar:

1. **`BantotalExperianService.java`** - Servicios para obtener y crear datos Experian en Bantotal
2. **`BantotalCatalogoService.java`** - Servicio para obtener guías de proceso especial
3. **`BantotalConfiguracionService.java`** - Servicio para obtener fecha del sistema
4. **`BantotalAuthService.java`** - Servicio para autenticación con Bantotal
5. **`BrmsScoreService.java`** - Servicio para obtener score Experian desde BRMS
6. **`CompartamosCatalogoService.java`** - Servicio para obtener catálogos
7. **`CompartamosScoService.java`** - Servicios para obtener y registrar cantidad de consultas Experian
8. **`ExperianExternalService.java`** - Servicio para consultar Experian externo

Todos siguen el mismo patrón:
- Anotaciones de resiliencia (`@ResilientHttpOutbound`, `@Timeout`, `@CircuitBreaker`, `@Retry`, `@Idempotent`)
- Uso de `ProducerTemplate` para realizar llamadas HTTP
- Manejo de errores con logging
- Propagación de headers y propiedades del Exchange

---

## 4. Archivos Sin Cambios

Los siguientes archivos fueron comparados y **no presentan diferencias** entre las versiones `migracion` y `resiliencia`:

### 4.1. Capa de Aplicación
- `ScoreService.java` - Sin cambios
- `MainRouteBuilder.java` - Sin cambios
- `ScoreRouteBuilder.java` - Sin cambios
- `AuthRouteBuilder.java` - Sin cambios (verificado)
- `CatalogueRouteBuilder.java` - Sin cambios (verificado)
- `ClaimsRouteBuilder.java` - Sin cambios (verificado)
- `VariablesRouteBuilder.java` - Sin cambios (verificado)

### 4.2. Capa de Dominio
- `ScoreBean.java` - Sin cambios
- `BaseBean.java` - Sin cambios
- `CatalogueBean.java` - Sin cambios
- `VariablesBean.java` - Sin cambios
- `AggregationProperties.java` - Sin cambios

### 4.3. Capa de Infraestructura
- `ConsumerBean.java` - Sin cambios (verificado)
- `ClaimsRepository.java` - Sin cambios
- Todos los DTOs en `infrastructure/proxy/dto/` - Sin cambios

### 4.4. Capa Transversal (Cross)
- Todos los archivos en `cross/exception/` - Sin cambios
- Todos los archivos en `cross/util/` - Sin cambios

### 4.5. Configuración
- `quarkus.properties` - Sin cambios
- Archivos YAML en `manifest/` - Sin cambios
- `openapi.yaml` - Sin cambios

---

## 5. Resumen de Cambios por Categoría

### 5.1. Archivos Modificados
1. `pom.xml` - Agregadas dependencias de resiliencia
2. `application.properties` - Agregadas configuraciones de resiliencia
3. `application-dev.properties` - Agregado logging de resiliencia
4. `ConsumerAuthRouteBuilder.java` - Refactorizado para usar servicio inyectado `BantotalAuthService`
5. `ConsumerServiceRouteBuilder.java` - Refactorizado para usar servicios inyectados

### 5.2. Archivos Nuevos
1. `BantotalScoreService.java`
2. `BantotalExperianService.java`
3. `BantotalCatalogoService.java`
4. `BantotalConfiguracionService.java`
5. `BantotalAuthService.java`
6. `BrmsScoreService.java`
7. `CompartamosSeguridadService.java`
8. `CompartamosCatalogoService.java`
9. `CompartamosScoService.java`
10. `ExperianExternalService.java`

### 5.3. Archivos Sin Cambios
- Todos los demás archivos del proyecto (más de 100 archivos Java, DTOs, utilidades, etc.)

---

## 6. Impacto General de la Migración

### 6.1. Beneficios
1. **Resiliencia:** El sistema es más robusto ante fallos de servicios externos
2. **Observabilidad:** Métricas de Prometheus para monitoreo
3. **Configurabilidad:** Políticas de resiliencia configurables por servicio
4. **Separación de responsabilidades:** Cada servicio externo tiene su propia clase
5. **Testabilidad:** Servicios pueden ser mockeados fácilmente

### 6.2. Consideraciones
1. **Complejidad:** Aumenta el número de clases y configuraciones
2. **Configuración:** Requiere ajuste de valores de timeout y circuit breakers
3. **Dependencias:** Nueva dependencia del `resilience-starter`
4. **Monitoreo:** Necesita monitoreo activo para ajustar políticas

### 6.3. Recomendaciones
1. Monitorear las métricas de circuit breakers y ajustar valores según comportamiento real
2. Realizar pruebas de carga para validar los timeouts configurados
3. Documentar las políticas de resiliencia por servicio
4. Establecer alertas basadas en las métricas de Prometheus

---

## 7. Conclusión

La migración introduce un framework de resiliencia robusto que mejora significativamente la estabilidad y disponibilidad del servicio `process-score-mvp`. Los cambios son principalmente en la capa de infraestructura, manteniendo intacta la lógica de negocio y las rutas de aplicación. La implementación sigue buenas prácticas de separación de responsabilidades y permite configuración granular por servicio.

---

**Fecha de generación:** $(date)
**Versión del documento:** 1.0
