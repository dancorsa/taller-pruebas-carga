# Dashboards – Visualización de Resultados JMeter

Esta carpeta contiene configuraciones y guías para visualizar los resultados de las pruebas de rendimiento.

---

## Opción 1: Reporte HTML integrado de JMeter (recomendado)

JMeter genera un dashboard HTML completo de forma automática con el flag `-e -o`. No requiere instalación adicional.

### Generar el reporte

```bash
# Baseline
jmeter -n \
  -t perf/scripts/registro_personas_baseline.jmx \
  -l perf/results/baseline.jtl \
  -e -o perf/results/baseline-report/

# Load
jmeter -n \
  -t perf/scripts/registro_personas_load.jmx \
  -l perf/results/load.jtl \
  -e -o perf/results/load-report/
```

### Abrir el reporte

```bash
# Windows
start perf/results/baseline-report/index.html

# macOS
open perf/results/baseline-report/index.html

# Linux
xdg-open perf/results/baseline-report/index.html
```

### Qué contiene el reporte HTML

| Sección | Descripción |
|---------|-------------|
| **Statistics** | Tabla con avg, p90, p95, p99, throughput y error rate por sampler |
| **Response Times Over Time** | Gráfica de latencia en el tiempo – útil para detectar degradación en soak |
| **Transactions Per Second** | Throughput en el tiempo – muestra si el sistema satura |
| **Active Threads Over Time** | Evolución de VUs activos |
| **Response Time Percentiles** | Distribución de percentiles al final de la prueba |
| **Response Time Distribution** | Histograma de tiempos de respuesta |
| **Errors** | Lista y porcentaje de errores ocurridos |

---

## Opción 2: Generar reporte desde un JTL existente

Si ya tienes el archivo `.jtl` y quieres regenerar el dashboard:

```bash
jmeter -g perf/results/baseline.jtl \
  -o perf/results/baseline-report-v2/
```

---

## Opción 3: Personalizar umbrales del reporte (jmeter-report.properties)

El archivo `jmeter-report.properties` en esta carpeta personaliza los colores y umbrales del dashboard HTML.

Para aplicarlo:

```bash
jmeter -n \
  -t perf/scripts/registro_personas_baseline.jmx \
  -l perf/results/baseline.jtl \
  -q perf/dashboards/jmeter-report.properties \
  -e -o perf/results/baseline-report/
```

---

## Métricas clave a capturar en capturas de pantalla

Para el informe de ejecución, toma capturas de estas secciones:

1. **Statistics** – muestra p95, p99, error% y throughput en tabla
2. **Response Times Over Time** – muestra tendencia de latencia
3. **Transactions Per Second** – muestra throughput en el tiempo

---

## Estructura de resultados esperada

```
perf/results/
├── baseline.jtl                  # Datos crudos del baseline
├── load.jtl                      # Datos crudos del load test
├── baseline-report/
│   ├── index.html                # Dashboard HTML principal
│   ├── content/
│   │   └── js/graphs.js
│   └── sbadmin2-1.0.7/
└── load-report/
    └── index.html
```

> **Nota:** Los archivos generados en `results/` no se versionan en Git (ver `.gitignore`). Solo el directorio vacío se mantiene con `.gitkeep`.
