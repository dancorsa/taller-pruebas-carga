# Registro de Defectos – Pruebas de Carga y Rendimiento

**Curso:** Testing y Validación de Software  
**Proyecto:** Taller Pruebas de Carga – JMeter  
**Estudiante:** [Tu nombre]  
**Fecha de ejecución:** 2026-06-10

---

## Introducción

Este documento recopila los defectos identificados durante la ejecución de pruebas de rendimiento con JMeter (Baseline y Load). Cada defecto se documenta garantizando trazabilidad, análisis técnico y propuesta de mejora.

---

## Defecto PERF-01 — Agotamiento de puertos TCP bajo carga de 100 VUs en Windows

- **Capa afectada:** Infraestructura de red / Entorno de prueba (Windows)
- **Escenario:** Load Test (100 VUs, con pacing de 200ms)
- **SLO definido:** Error rate < 1%
- **Resultado esperado:** Tasa de errores menor al 1% con 100 VUs concurrentes
- **Resultado obtenido:** Error rate ≈ 56% (`java.net.BindException: Address already in use`)

### Evidencia (JMeter Summary Report)

```
summary = 210810 in 00:12:00 = 292.7/s  Avg: 17  Min: 1  Max: 1059  Err: 118728 (56.32%)
Tipo de error: Non HTTP response code: java.net.BindException
              Non HTTP response message: Address already in use: connect
```

### Causa Técnica

Windows asigna puertos efímeros en el rango 49152–65535 (16,383 puertos disponibles). Cada petición HTTP que no reutiliza la conexión TCP abre un nuevo puerto local. Cuando el servidor cierra la conexión, el puerto permanece en estado `TIME_WAIT` durante ~240 segundos (valor por defecto en Windows). Con 100 VUs generando 5 req/s cada uno (500 req/s total), se necesitan `500 × 240 = 120,000` puertos simultáneamente, lo que supera en 7x el límite del sistema operativo.

### Impacto

- Hace inválidas las métricas de error rate del Load Test bajo estas condiciones
- Los resultados de latencia (Avg=17ms) son válidos solo para el 44% de peticiones que lograron conectarse
- Requiere ejecutar pruebas en Linux o con configuración especial de Windows para resultados fiables

### Propuesta de Mejora

1. **Reducir TIME_WAIT en Windows (requiere admin):** Configurar `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\TcpTimedWaitDelay = 30` (segundos). Con 30s: `500 × 30 = 15,000 < 16,383` → problema resuelto.
2. **Ejecutar en Linux/Docker:** Usar un contenedor JMeter en Linux donde el rango de puertos y TIME_WAIT no son una limitación práctica.
3. **Configurar keep-alive en JMeter:** Agregar la propiedad `httpclient4.time_to_live=7200` para reutilización persistente de conexiones TCP.

### Estado

Abierto

### Prioridad

Alta (bloquea validez de resultados Load Test en Windows)

---

## Defecto PERF-02 — Respuesta HTTP 406 por header Accept incompatible

- **Capa afectada:** Cliente de prueba (JMeter) / API (Spring Boot)
- **Escenario:** Baseline Test (primera ejecución con header `Accept: application/json`)
- **SLO definido:** Error rate < 1%
- **Resultado obtenido:** Error rate = 100% (HTTP 406 Not Acceptable)

### Evidencia (JMeter JTL)

```
timeStamp,...,responseCode,responseMessage,...,failureMessage
1781145441681,...,406,,...,Se esperaba HTTP 200
```

### Causa Técnica

El endpoint `POST /register` está declarado como `produces = MediaType.TEXT_PLAIN_VALUE`. El JMX original incluía el header `Accept: application/json`, que es incompatible con el tipo de contenido producido. Spring Boot responde con HTTP 406 cuando el cliente no acepta el tipo de respuesta del servidor.

### Propuesta de Mejora

Cambiar el header `Accept` en el JMX a `Accept: */*` para aceptar cualquier tipo de contenido, o a `Accept: text/plain` para alinearlo con la definición del endpoint.

### Estado

Resuelto (fix aplicado: `Accept: */*` en todos los JMX)

### Prioridad

Crítica (causaba 100% de errores en todos los escenarios)

---

## Defecto PERF-03 — Error HTTP 400 por caracteres UTF-8 en nombres del CSV

- **Capa afectada:** Datos de prueba / Codificación HTTP
- **Escenario:** Todos los escenarios (cualquier CSV con acentos)
- **SLO definido:** Error rate < 1%
- **Resultado obtenido:** Error rate ≈ 75% (mezcla de HTTP 400 y HTTP 200)

### Evidencia (JMeter JTL)

```
timeStamp,...,responseCode,...,failureMessage
1781145799035,...,400,...,Se esperaba HTTP 200  (nombre: "Ana García")
1781145799177,...,200,...,                       (nombre: "Carlos Lopez")
```

### Causa Técnica

Los nombres en el CSV contenían caracteres acentuados (á, é, ó, ú, ñ). Al enviarse en el cuerpo JSON vía JMeter en Windows, la codificación no era UTF-8 válida para Spring Boot, causando errores de parseo JSON. Spring retorna HTTP 400 cuando el cuerpo de la petición no puede deserializarse correctamente.

### Propuesta de Mejora

Usar únicamente caracteres ASCII en los archivos CSV de datos de prueba, o configurar explícitamente `Content-Type: application/json; charset=UTF-8` en el Header Manager de JMeter y asegurar que los archivos CSV se guarden en UTF-8 sin BOM.

### Estado

Resuelto (fix aplicado: nombres ASCII en todos los CSV)

### Prioridad

Alta (causaba errores intermitentes difíciles de diagnosticar)

---

## Resumen de Defectos

| ID | Escenario | SLO violado | Resultado obtenido | Estado | Prioridad |
|----|-----------|------------|-------------------|--------|-----------|
| PERF-01 | Load Test (100 VUs) | Error rate < 1% | 56.32% (BindException TCP) | Abierto | Alta |
| PERF-02 | Todos (primera ejecución) | Error rate < 1% | 100% (HTTP 406) | Resuelto | Crítica |
| PERF-03 | Todos (CSV con acentos) | Error rate < 1% | 75% (HTTP 400 mezclado) | Resuelto | Alta |

---

*Defectos documentados para la Unidad 5 – Testing y Validación de Software*  
*Universidad de La Sabana – Maestría en Ingeniería de Software – 2026*
