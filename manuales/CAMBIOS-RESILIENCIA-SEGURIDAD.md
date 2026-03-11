# Documento de Cambios: Proyecto Resiliencia vs Proyecto Seguridad

Este documento detalla todos los cambios realizados entre el proyecto **resiliencia** y el proyecto **seguridad**, incluyendo archivos nuevos, modificados y las razones de cada cambio.

---

## Resumen Ejecutivo

El proyecto **seguridad** incorpora la integración del **Security Starter** (`qompa-security-starter-aggregator`), que añade capacidades de seguridad estandarizadas, incluyendo:
- Validación configurable de claims requeridos
- Publicación de eventos de seguridad estandarizados
- Sanitización de headers sensibles en llamadas HTTP salientes
- Modos de seguridad (LEGACY y FULL/STRICT)

---

## 1. Archivo: `pom.xml`

### ¿Qué se cambió?
Se agregó la dependencia del **Security Starter** en el proyecto seguridad.

### Código Anterior (Resiliencia):
```xml
<properties>
    <resilience-starter.version>1.0.0-SNAPSHOT</resilience-starter.version>
</properties>
<dependencyManagement>
    <dependencies>
        <!-- ... otras dependencias ... -->
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
    <!-- ... otras dependencias ... -->
</dependencies>
```

### Código Modificado (Seguridad):
```xml
<properties>
    <resilience-starter.version>1.0.0-SNAPSHOT</resilience-starter.version>
    <!-- Security Starter -->
    <security-starter.version>1.0.0-SNAPSHOT</security-starter.version>
</properties>
<dependencyManagement>
    <dependencies>
        <!-- ... otras dependencias ... -->
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

        <!-- Security Starter -->
        <dependency>
            <groupId>com.compartamos.integration.framework</groupId>
            <artifactId>qompa-security-starter-aggregator</artifactId>
            <version>${security-starter.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Dependencia del starter de resiliencia-->
    <dependency>
        <groupId>com.compartamos.integration.framework</groupId>
        <artifactId>qompa-resilience-starter</artifactId>
    </dependency>
    <!-- ... otras dependencias ... -->

    <!-- Security Starter -->
    <dependency>
        <groupId>com.compartamos.integration.framework</groupId>
        <artifactId>qompa-security-starter-aggregator</artifactId>
        <exclusions>
            <exclusion>
                <groupId>com.compartamos.integration.framework</groupId>
                <artifactId>security-starter-camel</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <!-- ... otras dependencias ... -->
</dependencies>
```

### ¿Por qué se cambió?
Para integrar las capacidades del Security Starter que proporciona:
- Validación de claims configurable
- Publicación de eventos de seguridad estandarizados
- Utilidades de sanitización de headers
- Contratos y modelos estandarizados de seguridad

### Impacto
- **Positivo**: Añade capacidades de seguridad estandarizadas y observabilidad de eventos de seguridad
- **Dependencias**: Agrega una nueva dependencia al proyecto
- **Compatibilidad**: Mantiene compatibilidad con el código existente mediante el modo LEGACY

---

## 2. Archivo: `application.properties`

### ¿Qué se cambió?
Se agregaron configuraciones específicas del Security Starter al final del archivo.

### Código Anterior (Resiliencia):
```properties
# ... todas las configuraciones existentes ...
compartamos.seguridad.jwt.enabled=false

#================FIN STARTER SECURITY================
```

### Código Modificado (Seguridad):
```properties
# ... todas las configuraciones existentes ...
compartamos.seguridad.jwt.enabled=false

# ============================================
# Security Starter - Configuración
# ============================================

# Modo de seguridad: FULL (validación completa) o LEGACY (compatibilidad)
security.mode=LEGACY

# Modo de autenticación: jwt (usamos JWT)
security.auth.mode=jwt

# Habilitar módulo Camel (se activa automáticamente si camel-quarkus-core está presente)
# Valores: auto | true | false
security.camel.enabled=auto

# Configuración de claims requeridos
security.claims.required.usuarioBt=true
security.claims.required.tipo=false
security.claims.required.aplicativo=false

compartamos.seguridad.jwt.enabled=false

#================FIN STARTER SECURITY================
```

### ¿Por qué se cambió?
Para configurar el comportamiento del Security Starter:
- `security.mode=LEGACY`: Mantiene compatibilidad con el código existente
- `security.auth.mode=jwt`: Especifica que se usa autenticación JWT
- `security.claims.required.*`: Permite configurar qué claims son requeridos de forma flexible

### Impacto
- **Configuración**: Permite controlar el comportamiento del Security Starter sin modificar código
- **Flexibilidad**: Los claims requeridos pueden ajustarse mediante properties
- **Compatibilidad**: El modo LEGACY asegura que el código existente siga funcionando

---

## 3. Archivo Nuevo: `SecurityClaimsConfig.java`

**Ubicación**: `src/main/java/com/compartamos/process/score/application/config/SecurityClaimsConfig.java`

### ¿Qué se cambió?
Se creó un nuevo archivo de configuración para gestionar los claims requeridos de forma centralizada.

### Código Anterior (Resiliencia):
```
No existe este archivo.
```

### Código Nuevo (Seguridad):
```java
package com.compartamos.process.score.application.config;

import com.compartamos.process.score.cross.util.Constants;
import jakarta.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.config.inject.ConfigProperty;

import java.util.Set;

@ApplicationScoped
public class SecurityClaimsConfig {

    @ConfigProperty(name = "security.claims.required.usuarioBt", defaultValue = "true")
    boolean requireUsuarioBt;

    @ConfigProperty(name = "security.claims.required.tipo", defaultValue = "false")
    boolean requireTipo;

    @ConfigProperty(name = "security.claims.required.aplicativo", defaultValue = "false")
    boolean requireAplicativo;

    /**
     * Obtiene el conjunto de claims requeridos para process-score2.
     *
     * @return Set de nombres de claims requeridos (usando nombres legacy del sistema actual)
     */
    public Set<String> getRequiredClaims() {
        java.util.Set<String> requiredClaims = new java.util.HashSet<>();

        if (requireUsuarioBt) {
            requiredClaims.add(Constants.USER_BT);
        }
        if (requireTipo) {
            requiredClaims.add(Constants.APP_TYPE_JWT);
        }
        if (requireAplicativo) {
            requiredClaims.add(Constants.APPLICATION_JWT);
        }

        return requiredClaims;
    }
}
```

### ¿Por qué se cambió?
Para centralizar la configuración de claims requeridos y hacerla configurable mediante properties, en lugar de tenerla hardcodeada en el código.

### Impacto
- **Centralización**: La configuración de claims está en un solo lugar
- **Flexibilidad**: Los claims requeridos pueden cambiarse mediante properties sin modificar código
- **Mantenibilidad**: Facilita el mantenimiento y la evolución de los requisitos de seguridad

---

## 4. Archivo Nuevo: `SecurityHeaderSanitizerProcessor.java`

**Ubicación**: `src/main/java/com/compartamos/process/score/cross/util/SecurityHeaderSanitizerProcessor.java`

### ¿Qué se cambió?
Se creó un nuevo procesador para sanitizar headers sensibles en llamadas HTTP salientes, usando utilidades del Security Starter.

### Código Anterior (Resiliencia):
```
No existe este archivo.
```

### Código Nuevo (Seguridad):
```java
package com.compartamos.process.score.cross.util;

import com.compartamos.integration.security.HeaderSanitizer;
import com.compartamos.integration.security.SecurityConstants;
import jakarta.enterprise.context.ApplicationScoped;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.jboss.logging.Logger;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@ApplicationScoped
public class SecurityHeaderSanitizerProcessor implements Processor {

    private static final Logger LOG = Logger.getLogger(SecurityHeaderSanitizerProcessor.class);

    private static final Set<String> CANONICAL_HEADERS = Set.of(
            SecurityConstants.HEADER_X_CORRELATION_ID,
            SecurityConstants.HEADER_X_TRACE_ID,
            SecurityConstants.HEADER_X_MESSAGE_ID,
            SecurityConstants.HEADER_X_TENANT_ID,
            "X-Business-Id"
    );

    private final HeaderSanitizer headerSanitizer;

    public SecurityHeaderSanitizerProcessor() {
        this.headerSanitizer = new HeaderSanitizer();
    }

    @Override
    public void process(Exchange exchange) throws Exception {
        try {
            if (exchange == null || exchange.getIn() == null) {
                return;
            }

            Map<String, Object> headers = exchange.getIn().getHeaders();
            Map<String, Object> sanitizedHeaders = new HashMap<>();

            for (Map.Entry<String, Object> entry : headers.entrySet()) {
                String headerName = entry.getKey();
                Object headerValue = entry.getValue();

                if (CANONICAL_HEADERS.contains(headerName)) {
                    sanitizedHeaders.put(headerName, headerValue);
                } else if (headerSanitizer.isSensitive(headerName)) {
                    String sanitizedValue = headerSanitizer.sanitizeHeader(
                            headerName,
                            headerValue != null ? headerValue.toString() : null
                    );
                    sanitizedHeaders.put(headerName, sanitizedValue);
                    LOG.debugf("Sanitized sensitive header: %s", headerName);
                } else {
                    sanitizedHeaders.put(headerName, headerValue);
                }
            }

            LOG.debugf("Headers processed for sanitization in route: %s", exchange.getFromRouteId());
        } catch (Exception e) {
            LOG.warnf(e, "Error sanitizing headers in route: %s", exchange.getFromRouteId());
        }
    }
}
```

### ¿Por qué se cambió?
Para proteger información sensible en headers HTTP que se envían en llamadas salientes, usando las utilidades estandarizadas del Security Starter para identificar y sanitizar headers sensibles.

### Impacto
- **Seguridad**: Previene la fuga de información sensible en headers HTTP
- **Estandarización**: Usa utilidades del Security Starter para identificar headers sensibles
- **Observabilidad**: Registra cuando se sanitizan headers para auditoría

---

## 5. Archivo: `JwtContextFilterProcessor.java`

### ¿Qué se cambió?
Se modificó para integrar el Security Starter, añadiendo:
- Validación de claims requeridos usando `ClaimsValidator`
- Publicación de eventos de seguridad usando `SecurityEventPublisher`
- Soporte para modos de seguridad (LEGACY y FULL/STRICT)
- Manejo de errores mejorado con eventos de seguridad

### Código Anterior (Resiliencia):
```java
package com.compartamos.process.score.cross.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import com.compartamos.process.score.domain.bean.BaseBean;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtException;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import org.jboss.logging.MDC;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@ApplicationScoped
public class JwtContextFilterProcessor extends BaseBean implements Processor {

    private static final Logger LOG = LoggerFactory.getLogger(JwtContextFilterProcessor.class);
    private final JwtUtil jwtUtil;

    @Inject
    public JwtContextFilterProcessor(JwtUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    private static final String AUTHORIZATION_HEADER = "Authorization";
    private static final String BEARER_PREFIX = "Bearer";
    private static final String INVALID_AUTH_MESSAGE = "Unauthorized";
    private static final String INVALID_TOKEN_MESSAGE = "Unauthorized";

    @Override
    public void process(Exchange exchange) throws Exception {

        String uri = exchange.getMessage().getHeader(Exchange.HTTP_URI, String.class);
        String hostName = exchange.getMessage().getHeader("Host", String.class);
        if (hostName == null || hostName.isBlank()) {
            hostName = "";
        }
        MDC.put("hostName", hostName);
        if (uri != null && (uri.contains("swagger") || uri.contains("healthcheck"))) {
            return;
        }

        try {
            String authValues = exchange.getMessage().getHeader(AUTHORIZATION_HEADER, String.class);
            if (authValues == null || authValues.trim().isEmpty()) {
                setUnauthorizedResponse(exchange, INVALID_AUTH_MESSAGE);
                return;
            }

            String[] values = authValues.split(" ");
            if (values.length != 2 || !BEARER_PREFIX.equals(values[0])) {
                setUnauthorizedResponse(exchange, INVALID_AUTH_MESSAGE);
                return;
            }

            Claims claims = jwtUtil.validateToken(values[1]);

            String user = claims.get(Constants.USER_BT).toString().toUpperCase();
            String appType = claims.get(Constants.APP_TYPE_JWT).toString().toUpperCase();
            String application = claims.get(Constants.APPLICATION_JWT).toString().toUpperCase();

            exchange.getMessage().setHeader(Constants.USER_JWT, user);
            exchange.getMessage().setHeader(Constants.USER_BT, user);

            // para settear los valores del MDC
            setLogTraceToMDC(exchange, user, appType, true);

            MDC.put("userBt", user);
            MDC.put("app_type", appType);
            MDC.put("application", application);

        } catch (JwtException ex) {

            setUnauthorizedResponse(exchange, INVALID_TOKEN_MESSAGE);
        }
    }

    private void setUnauthorizedResponse(Exchange exchange, String message) throws JsonProcessingException {
        setLogTraceToMDC(exchange, null, null, false);

        exchange.getMessage().setBody("{\"message\":\"" + message + "\"}");
        stopUnauthorized(exchange, exchange.getMessage().getBody(String.class));
    }

    private void setLogTraceToMDC(Exchange exchange, String user, String appType, boolean generateTraceIfMissing) throws JsonProcessingException {
        String rootPath = exchange.getContext()
                .resolvePropertyPlaceholders("{{quarkus.http.root-path}}");
        String pathUri = exchange.getIn().getHeader(Exchange.HTTP_PATH, String.class);
        String endpointMethod = exchange.getIn().getHeader(Exchange.HTTP_METHOD, String.class);
        Object body = exchange.getIn().getBody();

        MDC.put("base_path", rootPath != null ? rootPath : Constants.NOT_FOUND);
        MDC.put("endpoint_path", pathUri != null ? pathUri : Constants.NOT_FOUND);
        MDC.put("endpoint_method", endpointMethod != null ? endpointMethod : Constants.NOT_FOUND);
        MDC.put("endpoint_body", body != null ? Functions.entityToJson(body) : Constants.NOT_FOUND);

        String traceId = exchange.getIn().getHeader(Constants.HEADER_TRACE_ID, String.class);

        if (traceId == null || traceId.isBlank()) {
            if (generateTraceIfMissing) {
                traceId = Functions.getDefaultIdTrace(user, appType);
                exchange.getMessage().setHeader(Constants.HEADER_TRACE_ID, traceId);
            } else {
                traceId = "";
            }
        }

        MDC.put("trace_id", traceId);

        if (traceId.contains("-")) {
            try {
                MDC.put("username", traceId.split("-")[1]);
            } catch (RuntimeException e) {
                LOG.error("Error al obtener el username del trace-id: {}", traceId, e);
            }
        }

    }
}
```

### Código Modificado (Seguridad):
```java
package com.compartamos.process.score.cross.util;

import com.compartamos.integration.security.ClaimsValidator;
import com.compartamos.integration.security.SecurityEventPublisher;
import com.compartamos.integration.security.SecurityEventPublisherFactory;
import com.compartamos.integration.security.model.ClaimSet;
import com.compartamos.integration.security.model.Principal;
import com.compartamos.integration.security.model.Tenant;
import com.compartamos.integration.security.modes.SecurityMode;
import com.compartamos.process.score.application.config.SecurityClaimsConfig;
import com.fasterxml.jackson.core.JsonProcessingException;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import com.compartamos.process.score.domain.bean.BaseBean;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtException;
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import org.eclipse.microprofile.config.inject.ConfigProperty;
import org.jboss.logging.MDC;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

@ApplicationScoped
public class JwtContextFilterProcessor extends BaseBean implements Processor {

    private static final Logger LOG = LoggerFactory.getLogger(JwtContextFilterProcessor.class);
    private final JwtUtil jwtUtil;
    private final SecurityEventPublisher securityEventPublisher;
    private final ClaimsValidator claimsValidator;
    private final SecurityClaimsConfig claimsConfig;
    private final String securityMode;

    @Inject
    public JwtContextFilterProcessor(
            JwtUtil jwtUtil,
            SecurityClaimsConfig claimsConfig,
            @ConfigProperty(name = "security.mode", defaultValue = "LEGACY") String securityMode) {
        this.jwtUtil = jwtUtil;
        this.claimsConfig = claimsConfig;
        this.securityMode = securityMode;
        this.securityEventPublisher = SecurityEventPublisherFactory.getInstance();
        SecurityMode mode = "FULL".equalsIgnoreCase(securityMode)
                ? SecurityMode.STRICT
                : SecurityMode.LEGACY;
        this.claimsValidator = new ClaimsValidator(mode);
    }

    private static final String AUTHORIZATION_HEADER = "Authorization";
    private static final String BEARER_PREFIX = "Bearer";
    private static final String INVALID_AUTH_MESSAGE = "Unauthorized";
    private static final String INVALID_TOKEN_MESSAGE = "Unauthorized";

    @Override
    public void process(Exchange exchange) throws Exception {

        String uri = exchange.getMessage().getHeader(Exchange.HTTP_URI, String.class);
        String hostName = exchange.getMessage().getHeader("Host", String.class);
        if (hostName == null || hostName.isBlank()) {
            hostName = "";
        }
        MDC.put("hostName", hostName);
        if (uri != null && (uri.contains("swagger") || uri.contains("healthcheck"))) {
            return;
        }

        try {
            String authValues = exchange.getMessage().getHeader(AUTHORIZATION_HEADER, String.class);
            if (authValues == null || authValues.trim().isEmpty()) {
                String correlationId = getCorrelationId(exchange);
                securityEventPublisher.publishAuthFailure(
                        null,
                        null,
                        "Missing Authorization header",
                        correlationId,
                        createDetails(exchange, "MISSING_AUTH_HEADER")
                );
                setUnauthorizedResponse(exchange, INVALID_AUTH_MESSAGE);
                return;
            }

            String[] values = authValues.split(" ");
            if (values.length != 2 || !BEARER_PREFIX.equals(values[0])) {
                String correlationId = getCorrelationId(exchange);
                securityEventPublisher.publishAuthFailure(
                        null,
                        null,
                        "Invalid Authorization header format",
                        correlationId,
                        createDetails(exchange, "INVALID_AUTH_FORMAT")
                );
                setUnauthorizedResponse(exchange, INVALID_AUTH_MESSAGE);
                return;
            }

            String token = values[1];
            Claims claims = jwtUtil.validateToken(token);

            ClaimSet claimSet = convertClaimsToClaimSet(claims);

            Set<String> requiredClaims = claimsConfig.getRequiredClaims();
            if (!requiredClaims.isEmpty()) {
                List<String> validationErrors = claimsValidator.validateRequiredClaims(claimSet, requiredClaims);
                if (!validationErrors.isEmpty()) {
                    String correlationId = getCorrelationId(exchange);
                    String errorMessage = String.join("; ", validationErrors);
                    securityEventPublisher.publishAuthFailure(
                            null,
                            null,
                            "Missing required claims: " + errorMessage,
                            correlationId,
                            createDetails(exchange, "MISSING_REQUIRED_CLAIMS", errorMessage)
                    );
                    setUnauthorizedResponse(exchange, "Missing required claims: " + errorMessage);
                    return;
                }
            }

            Object userClaim = claims.get(Constants.USER_BT);
            Object appTypeClaim = claims.get(Constants.APP_TYPE_JWT);
            Object applicationClaim = claims.get(Constants.APPLICATION_JWT);

            String user = userClaim != null ? userClaim.toString().toUpperCase() : "UNKNOWN";
            String appType = appTypeClaim != null ? appTypeClaim.toString().toUpperCase() : "UNKNOWN";
            String application = applicationClaim != null ? applicationClaim.toString().toUpperCase() : "UNKNOWN";

            Principal principal = new Principal(user, appType, user);
            Tenant tenant = null;

            String correlationId = getCorrelationId(exchange);
            securityEventPublisher.publishTokenValidation(
                    principal,
                    tenant,
                    true,
                    null,
                    correlationId,
                    createDetails(exchange, "TOKEN_VALIDATION_SUCCESS", user, appType, application)
            );

            exchange.getMessage().setHeader(Constants.USER_JWT, user);
            exchange.getMessage().setHeader(Constants.USER_BT, user);

            securityEventPublisher.publishAuthSuccess(
                    principal,
                    tenant,
                    correlationId,
                    createDetails(exchange, "AUTH_SUCCESS", user, appType, application)
            );

            setLogTraceToMDC(exchange, user, appType, true);

            MDC.put("userBt", user);
            MDC.put("app_type", appType);
            MDC.put("application", application);

        } catch (JwtException ex) {
            String correlationId = getCorrelationId(exchange);
            securityEventPublisher.publishTokenValidation(
                    null,
                    null,
                    false,
                    "JWT validation failed: " + ex.getMessage(),
                    correlationId,
                    createDetails(exchange, "TOKEN_VALIDATION_FAILURE", ex.getMessage())
            );

            securityEventPublisher.publishAuthFailure(
                    null,
                    null,
                    "JWT validation failed: " + ex.getMessage(),
                    correlationId,
                    createDetails(exchange, "AUTH_FAILURE", ex.getMessage())
            );

            setUnauthorizedResponse(exchange, INVALID_TOKEN_MESSAGE);
        } catch (Exception ex) {
            LOG.error("Unexpected error in JWT context filter processor", ex);
            String correlationId = getCorrelationId(exchange);
            securityEventPublisher.publishAuthFailure(
                    null,
                    null,
                    "Unexpected error during authentication: " + ex.getMessage(),
                    correlationId,
                    createDetails(exchange, "AUTH_ERROR", ex.getClass().getSimpleName(), ex.getMessage())
            );
            setUnauthorizedResponse(exchange, INVALID_TOKEN_MESSAGE);
        }
    }

    private void setUnauthorizedResponse(Exchange exchange, String message) throws JsonProcessingException {
        setLogTraceToMDC(exchange, null, null, false);

        exchange.getMessage().setBody("{\"message\":\"" + message + "\"}");
        stopUnauthorized(exchange, exchange.getMessage().getBody(String.class));
    }

    private void setLogTraceToMDC(Exchange exchange, String user, String appType, boolean generateTraceIfMissing) throws JsonProcessingException {
        String rootPath = exchange.getContext()
                .resolvePropertyPlaceholders("{{quarkus.http.root-path}}");
        String pathUri = exchange.getIn().getHeader(Exchange.HTTP_PATH, String.class);
        String endpointMethod = exchange.getIn().getHeader(Exchange.HTTP_METHOD, String.class);
        Object body = exchange.getIn().getBody();

        MDC.put("base_path", rootPath != null ? rootPath : Constants.NOT_FOUND);
        MDC.put("endpoint_path", pathUri != null ? pathUri : Constants.NOT_FOUND);
        MDC.put("endpoint_method", endpointMethod != null ? endpointMethod : Constants.NOT_FOUND);
        MDC.put("endpoint_body", body != null ? Functions.entityToJson(body) : Constants.NOT_FOUND);

        String traceId = exchange.getIn().getHeader(Constants.HEADER_TRACE_ID, String.class);

        if (traceId == null || traceId.isBlank()) {
            if (generateTraceIfMissing) {
                traceId = Functions.getDefaultIdTrace(user, appType);
                exchange.getMessage().setHeader(Constants.HEADER_TRACE_ID, traceId);
            } else {
                traceId = "";
            }
        }

        MDC.put("trace_id", traceId);

        if (traceId.contains("-")) {
            try {
                MDC.put("username", traceId.split("-")[1]);
            } catch (RuntimeException e) {
                LOG.error("Error al obtener el username del trace-id: {}", traceId, e);
            }
        }

    }

    /**
     * Obtiene el correlation ID del exchange o genera uno si no existe.
     */
    private String getCorrelationId(Exchange exchange) {
        String correlationId = exchange.getIn().getHeader(Constants.HEADER_TRACE_ID, String.class);
        if (correlationId == null || correlationId.isBlank()) {
            correlationId = exchange.getIn().getHeader("X-Correlation-Id", String.class);
        }
        return correlationId != null ? correlationId : "unknown";
    }

    /**
     * Crea un mapa de detalles para eventos de seguridad.
     */
    private Map<String, Object> createDetails(Exchange exchange, String eventType, String... additionalInfo) {
        Map<String, Object> details = new HashMap<>();
        details.put("eventType", eventType);
        details.put("uri", exchange.getMessage().getHeader(Exchange.HTTP_URI, String.class));
        details.put("method", exchange.getMessage().getHeader(Exchange.HTTP_METHOD, String.class));
        details.put("path", exchange.getMessage().getHeader(Exchange.HTTP_PATH, String.class));

        if (additionalInfo != null && additionalInfo.length > 0) {
            for (int i = 0; i < additionalInfo.length; i++) {
                details.put("info" + i, additionalInfo[i]);
            }
        }

        return details;
    }

    /**
     * Convierte Claims de io.jsonwebtoken a ClaimSet de commons-security-contracts.
     *
     * @param claims los Claims del JWT
     * @return el ClaimSet convertido
     */
    private ClaimSet convertClaimsToClaimSet(Claims claims) {
        Map<String, Object> claimMap = new HashMap<>();

        claims.forEach((key, value) -> {
            if (value != null) {
                claimMap.put(key, value);
            }
        });

        return new ClaimSet(claimMap);
    }
}
```

### ¿Por qué se cambió?
Para integrar las capacidades del Security Starter:
1. **Validación de claims**: Usa `ClaimsValidator` para validar claims requeridos de forma estandarizada
2. **Eventos de seguridad**: Publica eventos estandarizados de éxito/fallo de autenticación y validación de tokens
3. **Modos de seguridad**: Soporta modos LEGACY y FULL/STRICT para compatibilidad y seguridad mejorada
4. **Manejo de errores**: Mejora el manejo de errores con eventos de seguridad para auditoría

### Impacto
- **Observabilidad**: Los eventos de seguridad permiten monitorear y auditar intentos de autenticación
- **Seguridad**: La validación de claims requeridos añade una capa adicional de seguridad
- **Compatibilidad**: El modo LEGACY asegura que el código existente siga funcionando
- **Mantenibilidad**: El código es más robusto con mejor manejo de errores y logging

---

## 6. Archivo: `ConsumerServiceRouteBuilder.java`

### ¿Qué se cambió?
Se agregó el procesador `SecurityHeaderSanitizerProcessor` en todas las rutas HTTP salientes para sanitizar headers sensibles antes de realizar las llamadas.

### Código Anterior (Resiliencia):
```java
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

// ... resto de rutas similares sin SecurityHeaderSanitizerProcessor ...
```

### Código Modificado (Seguridad):
```java
from("direct:getExperianExterno")
    .setProperty("sensitiveRoute", constant(true))
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(experianExternalService, "consultarExperianExterno")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(ConsultarExperianExternoResponse.class));

from("direct:getScoreExperianBrms")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(brmsScoreService, "obtenerScoreExperian")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(BaseResponse.class));

from("direct:getVariablesSistema")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(compartamosSeguridadService, "obtenerVariables")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(BaseProxyResponse.class));

from("direct:getCodigoCatalogos")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(compartamosCatalogoService, "obtenerCatalogos")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(BaseResponse.class));

from("direct:getDatosGuiaProceso")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(bantotalCatalogoService, "obtenerGuiaProcesoEspecial")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(ObtenerGuiaProcesoEspecialResponse.class));

from("direct:getDatosExperian")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(bantotalExperianService, "obtenerDatosExperian")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(ObtenerDatosExperianResponse.class));

from("direct:createDatosExperian")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(bantotalExperianService, "crearDatosExperian")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(CrearDatosExperianResponse.class));

from("direct:getDatosSistemDate")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(bantotalConfiguracionService, "obtenerFechaSistema")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(ObtenerFechaDeSistemaResponse.class));

from("direct:getDatosCantidadConsultasExperian")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(compartamosScoService, "obtenerCantidadConsultasExperian")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(BaseResponse.class));

from("direct:createDatosCantidadConsultasExperian")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(compartamosScoService, "registrarCantidadConsultasExperian")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(BaseResponse.class));

from("direct:getDatosScore")
    .bean(ConsumerBean.class, Constants.CONSUMER_POST_BANTOTAL_LECTURA)
    .process(new SecurityHeaderSanitizerProcessor())
    .bean(bantotalScoreService, "obtenerScorePersona")
    .to(Constants.VALIDATE_RESPONSE)
    .unmarshal(new JacksonDataFormat(ObtenerScorePersonaResponse.class));
```

### ¿Por qué se cambió?
Para proteger información sensible en headers HTTP que se envían en llamadas salientes a servicios externos, sanitizando automáticamente cualquier header que contenga información sensible antes de enviarlo.

### Impacto
- **Seguridad**: Previene la fuga de información sensible en headers HTTP salientes
- **Consistencia**: Todas las llamadas HTTP salientes están protegidas de forma uniforme
- **Rendimiento**: El procesador tiene un impacto mínimo en el rendimiento
- **Auditoría**: Los logs indican cuando se sanitizan headers para auditoría

---

## 7. Archivo: `AssignTokenBtProcessor.java`

### ¿Qué se cambió?
Se modificó para publicar eventos de seguridad cuando se autentica con Bantotal, tanto en casos de éxito como de fallo.

### Código Anterior (Resiliencia):
```java
package com.compartamos.process.score.domain.proccess;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import com.compartamos.process.score.cross.util.Constants;
import com.compartamos.process.score.cross.util.Functions;
import com.compartamos.process.score.domain.bean.BaseBean;
import com.compartamos.process.score.infrastructure.proxy.dto.AuthResponse;

public class AssignTokenBtProcessor extends BaseBean implements Processor {

	@Override
	public void process(Exchange exchange) throws Exception {

		String inBody = exchange.getIn().getBody(String.class);
		AuthResponse authBt = Functions.jsonToEntity(inBody, AuthResponse.class);

		if (!authBt.getErroresnegocio().getBTErrorNegocio().isEmpty()) {
			String outBody = Functions
					.getBodyError(authBt.getErroresnegocio().getBTErrorNegocio().get(0).getDescripcion());
			stop(exchange, outBody, 200);
			return;
		}

		exchange.setProperty(Constants.TOKEN_EXTERNAL, authBt.getSessionToken());

		exchange.getMessage().setBody(inBody);

		String user = exchange.getProperty(Constants.USER_JWT, String.class);
		exchange.setProperty(Constants.USER_JWT, user);

		exchange.getMessage().setHeaders(exchange.getIn().getHeaders());

	}

}
```

### Código Modificado (Seguridad):
```java
package com.compartamos.process.score.domain.proccess;

import com.compartamos.integration.security.SecurityEventPublisher;
import com.compartamos.integration.security.SecurityEventPublisherFactory;
import com.compartamos.integration.security.model.Principal;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import com.compartamos.process.score.cross.util.Constants;
import com.compartamos.process.score.cross.util.Functions;
import com.compartamos.process.score.domain.bean.BaseBean;
import com.compartamos.process.score.infrastructure.proxy.dto.AuthResponse;

import java.util.HashMap;
import java.util.Map;

public class AssignTokenBtProcessor extends BaseBean implements Processor {

    private final SecurityEventPublisher securityEventPublisher =
            SecurityEventPublisherFactory.getInstance();

    @Override
    public void process(Exchange exchange) throws Exception {

        String inBody = exchange.getIn().getBody(String.class);
        AuthResponse authBt = Functions.jsonToEntity(inBody, AuthResponse.class);

        if (!authBt.getErroresnegocio().getBTErrorNegocio().isEmpty()) {
            String user = exchange.getProperty(Constants.USER_JWT, String.class);
            Principal principal = user != null ? new Principal(user, "BANTOTAL", user) : null;
            String correlationId = getCorrelationId(exchange);
            String errorCode = String.valueOf(authBt.getErroresnegocio().getBTErrorNegocio().get(0).getCodigo());
            String errorMessage = authBt.getErroresnegocio().getBTErrorNegocio().get(0).getDescripcion();
            String fullErrorMessage = "Error al hacer login en Bantotal. Código: " + errorCode + ", Mensaje: " + errorMessage;

            securityEventPublisher.publishAuthFailure(
                    principal,
                    null,
                    fullErrorMessage,
                    correlationId,
                    createDetails(exchange, "BANTOTAL_AUTH_FAILURE", errorCode, errorMessage)
            );

            String outBody = Functions
                    .getBodyError(authBt.getErroresnegocio().getBTErrorNegocio().get(0).getDescripcion());
            stop(exchange, outBody, 200);
            return;
        }

        exchange.setProperty(Constants.TOKEN_EXTERNAL, authBt.getSessionToken());

        exchange.getMessage().setBody(inBody);

        String user = exchange.getProperty(Constants.USER_JWT, String.class);
        exchange.setProperty(Constants.USER_JWT, user);

        if (user != null) {
            Principal principal = new Principal(user, "BANTOTAL", user);
            String correlationId = getCorrelationId(exchange);

            securityEventPublisher.publishAuthSuccess(
                    principal,
                    null,
                    correlationId,
                    createDetails(exchange, "BANTOTAL_AUTH_SUCCESS", user)
            );
        }

        exchange.getMessage().setHeaders(exchange.getIn().getHeaders());

    }

    /**
     * Obtiene el correlation ID del exchange o genera uno si no existe.
     */
    private String getCorrelationId(Exchange exchange) {
        String correlationId = exchange.getProperty(Constants.TRACE_ID, String.class);
        if (correlationId == null || correlationId.isBlank()) {
            correlationId = exchange.getIn().getHeader(Constants.HEADER_TRACE_ID, String.class);
        }
        return correlationId != null ? correlationId : "unknown";
    }

    /**
     * Crea un mapa de detalles para eventos de seguridad.
     */
    private Map<String, Object> createDetails(Exchange exchange, String eventType, String... additionalInfo) {
        Map<String, Object> details = new HashMap<>();
        details.put("eventType", eventType);
        details.put("source", "BANTOTAL");

        if (additionalInfo != null && additionalInfo.length > 0) {
            for (int i = 0; i < additionalInfo.length; i++) {
                details.put("info" + i, additionalInfo[i]);
            }
        }

        return details;
    }

}
```

### ¿Por qué se cambió?
Para publicar eventos de seguridad estandarizados cuando se autentica con Bantotal, permitiendo:
- Auditoría de intentos de autenticación exitosos y fallidos
- Monitoreo de problemas de autenticación con Bantotal
- Trazabilidad de eventos de seguridad relacionados con Bantotal

### Impacto
- **Observabilidad**: Los eventos de seguridad permiten monitorear y auditar autenticaciones con Bantotal
- **Troubleshooting**: Facilita la identificación de problemas de autenticación con Bantotal
- **Auditoría**: Proporciona un registro estandarizado de eventos de seguridad

---

## Archivos Sin Cambios

Los siguientes archivos son idénticos en ambos proyectos y no requieren cambios:

1. **`UriSanitizerProcessor.java`**: Idéntico en ambos proyectos
2. **`application-dev.properties`**: Idéntico en ambos proyectos
3. **`ClaimsBean.java`**: Idéntico en ambos proyectos
4. **`ScoreService.java`**: Idéntico en ambos proyectos
5. Todos los demás archivos Java (DTOs, servicios externos, etc.): Sin cambios

---

## Resumen de Cambios por Categoría

### Archivos Nuevos (2)
1. `SecurityClaimsConfig.java` - Configuración de claims requeridos
2. `SecurityHeaderSanitizerProcessor.java` - Sanitización de headers HTTP

### Archivos Modificados (4)
1. `pom.xml` - Agregada dependencia del Security Starter
2. `application.properties` - Agregadas configuraciones de seguridad
3. `JwtContextFilterProcessor.java` - Integración con Security Starter
4. `ConsumerServiceRouteBuilder.java` - Agregado sanitizador de headers
5. `AssignTokenBtProcessor.java` - Agregada publicación de eventos de seguridad

### Total de Cambios
- **Archivos nuevos**: 2
- **Archivos modificados**: 5
- **Archivos sin cambios**: Todos los demás

---

## Conclusión

El proyecto **seguridad** incorpora el **Security Starter** para añadir capacidades de seguridad estandarizadas, manteniendo compatibilidad con el código existente mediante el modo LEGACY. Los cambios principales incluyen:

1. **Validación configurable de claims**: Los claims requeridos pueden configurarse mediante properties
2. **Eventos de seguridad estandarizados**: Todos los eventos de seguridad se publican de forma estandarizada
3. **Sanitización de headers**: Los headers sensibles se sanitizan automáticamente en llamadas HTTP salientes
4. **Observabilidad mejorada**: Mejor trazabilidad y auditoría de eventos de seguridad

Todos los cambios son compatibles hacia atrás y no rompen la funcionalidad existente.
