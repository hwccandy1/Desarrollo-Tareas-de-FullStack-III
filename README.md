# reserva-cancha.openfeign
# Arquitectura OpenFeign entre serviciocanchas y servicioreservas 
#Constanza y Corina
## Objetivo

Implementar la comunicación entre microservicios utilizando:

- `serviciocanchas` (puerto `8081`)
- `servicioreservas` (puerto `8082`)
- Interacción bidireccional mediante Spring Cloud OpenFeign
- Uso de Fallback + Resilience4j para garantizar tolerancia a fallos

------------------------------------

## Estructura FEIGN en servicioreservas

1) `ServicioreservasApplication.java`
   - `@SpringBootApplication`
   - `@EnableFeignClients`
   - Método `main(...)` encargado de iniciar la aplicación

2) `client/CanchaClient.java`
   - `@FeignClient(name = "canchas-service", url = "${canchas.service.url}", fallback = CanchaClientFallback.class)`
   - Endpoint `GET /api/canchas/{id}`
   - Endpoint `GET /api/canchas`

3) `client/CanchaClientFallback.java`
   - `@Component`
   - `obtenerCancha(...) -> null`
   - `listarCanchas() -> Collections.emptyList()`

4) `services/ReservaServicesImpl.java`
   - Inyección con `@Autowired private CanchaClient canchaClient;`
   - `crearReserva()` valida la existencia de la cancha antes de registrar (uso de FEIGN)
   - `listarCanchas()` obtiene información a través de FEIGN

5) `application.properties`
   - `canchas.service.url=http://localhost:8081`
   - `feign.circuitbreaker.enabled=true`
   - `spring.cloud.circuitbreaker.resilience4j.enabled=true`

------------------------------------

## Estructura FEIGN en serviciocanchas

1) `ServiciocanchasApplication.java`
   - `@SpringBootApplication`
   - `@EnableFeignClients`
   - Método `main(...)`

2) `client/ReservaClient.java`
   - `@FeignClient(name = "reservas-service", url = "${reservas.service.url}", fallback = ReservaClientFallback.class)`
   - Endpoint `GET /api/reservas/cancha/{canchaId}`

3) `client/ReservaClientFallback.java`
   - `@Component`
   - `obtenerReservasPorCancha(...) -> Collections.emptyList()`

4) `services/ServicesImpl.java`
   - Inyección con `@Autowired private ReservaClient reservaClient;`
   - `obtenerReservasPorCancha(canchaId)` delega la consulta a `reservaClient`

5) `application.properties`
   - `reservas.service.url=http://localhost:8082`

------------------------------------

## Endpoints expuestos

### serviciocanchas

- `GET /api/canchas`
- `POST /api/canchas`
- `GET /api/canchas/{id}`
- `GET /api/canchas/{id}/reservas` (consulta a reservas mediante FEIGN)

### servicioreservas

- `GET /api/reservas`
- `POST /api/reservas`
- `GET /api/reservas/cancha/{id}`
- `GET /api/reservas/canchas` (consulta a canchas mediante FEIGN)

------------------------------------

## Dependencias clave (pom.xml)

- `spring-cloud-starter-openfeign`
- `spring-cloud-starter-circuitbreaker-resilience4j`
- `spring-boot-starter-web`
- `spring-boot-starter-data-jpa`
- `mysql-connector-j`

------------------------------------

## Flujo de validación (Postman)

1. `GET http://localhost:8081/api/canchas`
2. `POST http://localhost:8081/api/canchas`
3. `GET http://localhost:8082/api/reservas`
4. `POST http://localhost:8082/api/reservas` (incluyendo `canchaId`)
5. `GET http://localhost:8081/api/canchas/{id}/reservas` (integración FEIGN hacia reservas)
6. `GET http://localhost:8082/api/reservas/canchas` (integración FEIGN hacia canchas)

------------------------------------

## Notas

- El archivo `pom.xml` fue configurado con Spring Boot `3.2.12` y Spring Cloud `2023.0.5` para asegurar compatibilidad.
- Se definió correctamente `mainClass` en `spring-boot-maven-plugin`.
- Se implementó configuración de FEIGN con fallback y circuit breaker para mejorar la resiliencia del sistema.

