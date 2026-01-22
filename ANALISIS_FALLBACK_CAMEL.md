# Análisis: ¿Las Invocaciones de Rutas Camel Aplican Fallback?

## Respuesta Directa

**Sí, las invocaciones desde rutas Camel SÍ aplican Fallback**, pero con **consideraciones importantes** sobre cuándo y cómo se ejecuta.

---

## 🔍 Análisis Técnico

### 1. Cómo Funciona la Integración CDI + Camel

#### Invocación desde Camel
```java
// En ConsumerServiceRouteBuilder.java
from("direct:GetDatosHistoricCreditNombres")
    .bean(ConsumerBean.class, "consumerConsultarNombresSoap");
```

#### Bean con Anotaciones
```java
@ApplicationScoped
@Named("consumerBean")
public class ConsumerBean {
    
    @Timeout(2000)
    @CircuitBreaker(...)
    @Retry(...)
    @Fallback(fallbackMethod = "consumerConsultarNombresSoapFallback")
    public void consumerConsultarNombresSoap(Exchange exchange) {
        // Lógica que puede fallar
        throw new RuntimeException("Fallo consultando Experian SOAP");
    }
    
    public void consumerConsultarNombresSoapFallback(Exchange exchange) {
        // Método de fallback
    }
}
```

### 2. Mecanismo de Funcionamiento

#### ✅ Por Qué SÍ Funciona

1. **CDI Proxy**: Cuando Camel invoca `.bean(ConsumerBean.class, "methodName")`, Quarkus busca el bean en el contexto CDI.

2. **Bean Anotado con @ApplicationScoped**: Como `ConsumerBean` está anotado con `@ApplicationScoped`, Quarkus crea un **proxy CDI** que envuelve la instancia real.

3. **Interceptores de MicroProfile Fault Tolerance**: Las anotaciones `@Timeout`, `@CircuitBreaker`, `@Retry` y `@Fallback` funcionan mediante **interceptores CDI** que se ejecutan antes y después de la invocación del método.

4. **Orden de Ejecución**:
   ```
   Camel Route
       ↓
   .bean(ConsumerBean.class, "consumerConsultarNombresSoap")
       ↓
   CDI Proxy (Quarkus)
       ↓
   Interceptor @Timeout
       ↓
   Interceptor @CircuitBreaker
       ↓
   Interceptor @Retry
       ↓
   Método Real: consumerConsultarNombresSoap()
       ↓ (si falla)
   Interceptor @Fallback
       ↓
   Método Fallback: consumerConsultarNombresSoapFallback()
   ```

### 3. Cuándo se Ejecuta el Fallback

El `@Fallback` se ejecuta cuando:

1. ✅ **Timeout**: El método excede el tiempo configurado (`@Timeout(2000)`)
2. ✅ **Circuit Breaker Abierto**: El Circuit Breaker está en estado OPEN
3. ✅ **Excepción después de Retry**: Después de agotar los reintentos del `@Retry`
4. ✅ **Excepción no manejada**: Cualquier excepción no capturada dentro del método

#### ⚠️ Consideración Importante

El método `consumerConsultarNombresSoap` tiene un `try-catch` que captura excepciones y las convierte en `RuntimeException`:

```java
try {
    // Lógica HTTP
} catch (Exception e) {
    LOG.warn("Error consultando Experian SOAP", e);
    throw new RuntimeException("Fallo consultando Experian SOAP", e);
}
```

**Esto es correcto** porque:
- La excepción se lanza **después** de que el interceptor puede procesarla
- El `@Fallback` captura la `RuntimeException` y ejecuta el método de fallback

### 4. Diferencia con Otros Métodos

#### Métodos SIN Fallback (Solo Preparan el Exchange)
```java
@Timeout(2000)
@CircuitBreaker(...)
@Retry(...)
// SIN @Fallback
public void consumerBrmsScoreExperian(Exchange exchange) {
    consumer("POST", exchange);  // Solo prepara el Exchange
}
```

**Flujo**:
```
Ruta Camel
    ↓
.bean(ConsumerBean.class, "consumerBrmsScoreExperian")
    ↓
CDI Proxy → Interceptores → Método (prepara Exchange)
    ↓
.recipientList(...)  // La llamada HTTP real se hace aquí
    ↓
Si falla → onException() de Camel maneja el error
```

**Por qué no tienen Fallback**:
- Estos métodos solo **preparan** el Exchange
- La llamada HTTP real se hace en el siguiente paso de la ruta (`.recipientList()`)
- Si falla, el error se maneja con `onException()` de Camel
- El Fallback no aplica porque el error ocurre **fuera** del método anotado

#### Método CON Fallback (Hace la Llamada HTTP Directamente)
```java
@Timeout(2000)
@CircuitBreaker(...)
@Retry(...)
@Fallback(fallbackMethod = "consumerConsultarNombresSoapFallback")
public void consumerConsultarNombresSoap(Exchange exchange) {
    // Hace la llamada HTTP DIRECTAMENTE dentro del método
    URLConnection conn = url.openConnection();
    // ...
    // Si falla, lanza RuntimeException
}
```

**Flujo**:
```
Ruta Camel
    ↓
.bean(ConsumerBean.class, "consumerConsultarNombresSoap")
    ↓
CDI Proxy → Interceptores → Método (hace HTTP directamente)
    ↓
Si falla → @Fallback intercepta → Ejecuta fallback
    ↓
Exchange tiene respuesta de fallback
```

**Por qué SÍ tiene Fallback**:
- El método hace la llamada HTTP **dentro** del método anotado
- Si falla, la excepción ocurre **dentro** del contexto del interceptor
- El `@Fallback` puede capturar la excepción y ejecutar el método de fallback
- La ruta Camel continúa con la respuesta del fallback

---

## 📊 Comparación: Métodos con y sin Fallback

| Aspecto | Métodos SIN Fallback | Método CON Fallback |
|---------|---------------------|---------------------|
| **Responsabilidad** | Solo preparan Exchange | Hacen llamada HTTP directamente |
| **Llamada HTTP** | En siguiente paso de ruta (`.recipientList()`) | Dentro del método anotado |
| **Manejo de Error** | `onException()` de Camel | `@Fallback` + `onException()` |
| **Ejemplo** | `consumerBrmsScoreExperian` | `consumerConsultarNombresSoap` |

---

## ✅ Verificación: ¿Realmente Funciona?

### Evidencia en el Código

1. **Bean Anotado Correctamente**:
   ```java
   @ApplicationScoped  // ← Habilita proxy CDI
   @Named("consumerBean")
   public class ConsumerBean { ... }
   ```

2. **Método con Fallback**:
   ```java
   @Fallback(fallbackMethod = "consumerConsultarNombresSoapFallback")
   public void consumerConsultarNombresSoap(Exchange exchange) { ... }
   ```

3. **Método Fallback Implementado**:
   ```java
   public void consumerConsultarNombresSoapFallback(Exchange exchange) {
       ResilienceContext.markFallback();  // ← Marca que se ejecutó fallback
       // Respuesta degradada
   }
   ```

4. **Invocación desde Camel**:
   ```java
   .bean(ConsumerBean.class, "consumerConsultarNombresSoap")
   ```

### Condiciones para que Funcione

✅ **SÍ funciona si**:
- El bean está anotado con `@ApplicationScoped` o `@Dependent` (CDI)
- El método está anotado con `@Fallback`
- La excepción se lanza **dentro** del método anotado
- La excepción no es capturada antes de llegar al interceptor

❌ **NO funciona si**:
- El bean no está en el contexto CDI
- La excepción ocurre **después** de que el método retorna (en siguiente paso de ruta)
- La excepción es capturada y manejada dentro del método sin relanzarla

---

## 🎯 Respuesta Final

### ¿Las invocaciones de rutas Camel aplican Fallback?

**Sí, PERO solo cuando**:

1. ✅ El método anotado con `@Fallback` hace la operación que puede fallar **dentro del método mismo**
2. ✅ La excepción se lanza dentro del contexto del método anotado
3. ✅ El bean está correctamente anotado con `@ApplicationScoped` (o `@Dependent`)

### ¿Por qué solo un método tiene Fallback?

**Porque solo `consumerConsultarNombresSoap` cumple las condiciones**:

- ✅ Hace la llamada HTTP **directamente** dentro del método
- ✅ Lanza excepción si falla
- ✅ La excepción ocurre dentro del contexto del interceptor

**Los demás métodos NO tienen Fallback porque**:

- ❌ Solo preparan el Exchange
- ❌ La llamada HTTP se hace en el siguiente paso de la ruta
- ❌ El error ocurre fuera del contexto del interceptor
- ❌ El error se maneja con `onException()` de Camel

---

## 📝 Recomendaciones

### Para Aplicar Fallback en Otros Métodos

Si se quisiera aplicar Fallback a otros métodos, habría que:

1. **Opción 1**: Mover la llamada HTTP dentro del método anotado
   ```java
   @Fallback(fallbackMethod = "fallback")
   public void consumerBrmsScoreExperian(Exchange exchange) {
       // Hacer llamada HTTP aquí directamente
       // En lugar de solo preparar el Exchange
   }
   ```

2. **Opción 2**: Usar `onException()` de Camel con manejo de fallback
   ```java
   onException(Exception.class)
       .handled(true)
       .process(fallbackProcessor)  // Procesador de fallback
       .stop();
   ```

### Mejores Prácticas

1. ✅ **Fallback en métodos que hacen operaciones directamente**: Como `consumerConsultarNombresSoap`
2. ✅ **onException() de Camel para errores en rutas**: Para errores que ocurren después de `.bean()`
3. ✅ **Combinar ambos**: Fallback para errores del método, `onException()` para errores de la ruta

---

## 🔬 Validación Práctica

Para verificar que el Fallback funciona:

1. **Simular fallo en `consumerConsultarNombresSoap`**:
   - Desconectar red
   - Hacer request inválido
   - Verificar que se ejecuta `consumerConsultarNombresSoapFallback`

2. **Verificar en logs**:
   - Buscar `ResilienceContext.markFallback()` en logs
   - Verificar respuesta con `respuesta: "SERVICIO_NO_DISPONIBLE"`

3. **Verificar en métricas**:
   - Métrica `resilience_fallback_executions_total` debe incrementar

---

**Conclusión**: El Fallback **SÍ se aplica** en las invocaciones desde rutas Camel, pero solo cuando la operación que puede fallar ocurre dentro del método anotado, no cuando ocurre en pasos posteriores de la ruta.
