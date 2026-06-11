# Taller de Pruebas de Carga y Rendimiento con JMeter

**Curso:** Testing y Validación de Software  
**Programa:** Maestría en Ingeniería de Software – Universidad de La Sabana  
**Basado en:** [TYVS-Taller_Pruebas_de_carga](https://github.com/CesarAVegaF312/TYVS-Taller_Pruebas_de_carga)

---

## 1. Descripción

Este taller tiene como propósito enseñar a diseñar, implementar y ejecutar pruebas de rendimiento sobre servicios HTTP/API aplicando buenas prácticas de ingeniería y automatización. Se utiliza **Apache JMeter** como herramienta principal sobre la API de registro de la aplicación **Registraduría**.

### Objetivos académicos

- Comprender los diferentes tipos de prueba de rendimiento: Baseline, Load, Stress, Spike y Soak.
- Configurar planes de prueba JMeter con datos parametrizados desde CSV.
- Definir y validar Objetivos de Nivel de Servicio (SLO).
- Interpretar métricas de rendimiento: latencia promedio, p95, p99, throughput y tasa de errores.
- Identificar cuellos de botella y documentar defectos de rendimiento.

---

## 2. Prerequisitos

| Herramienta | Versión mínima | Enlace |
|-------------|---------------|--------|
| Java JDK | 11+ | https://adoptium.net |
| Apache Maven | 3.8+ | https://maven.apache.org |
| Apache JMeter | 5.6+ | https://jmeter.apache.org/download_jmeter.cgi |
| Git | 2.x | https://git-scm.com |

### Verificar instalaciones

```bash
java -version
mvn -version
jmeter --version
git --version
```

---

## 3. Estructura del Repositorio

```
taller-pruebas-carga/
├── README.md
├── defectos.md                     # Registro de defectos encontrados
├── defectos_template.md            # Plantilla para reportar nuevos defectos
├── perf/
│   ├── scripts/
│   │   ├── registro_personas_baseline.jmx   # Plan de prueba Baseline (10 VUs)
│   │   └── registro_personas_load.jmx        # Plan de prueba Load (hasta 100 VUs)
│   ├── data/
│   │   └── persons.csv                       # Dataset de prueba (30 registros)
│   ├── results/                              # Aquí se guardan los resultados .jtl
│   └── ci/
│       └── github-actions.yml               # Pipeline de CI automatizado
└── cuestionario/
    └── cuestionario.md                       # Cuestionario individual
```

---

## 4. Paso 1 – Clonar y Levantar la Aplicación

La aplicación bajo prueba es **Registraduría**, una API REST Spring Boot que expone el endpoint `POST /register`.

### 4.1 Clonar el repositorio de la aplicación

```bash
git clone https://github.com/CesarAVegaF312/TYVS-Taller_Pruebas_de_carga.git
cd TYVS-Taller_Pruebas_de_carga/registraduria
```

### 4.2 Compilar y ejecutar

```bash
mvn clean spring-boot:run
```

La aplicación quedará disponible en `http://localhost:8080`.

### 4.3 Verificar que la API responde

```bash
curl -X POST http://localhost:8080/register \
  -H "Content-Type: application/json" \
  -d '{"name":"Ana","id":1,"age":30,"gender":"FEMALE","alive":true}'
```

**Respuesta esperada:** `200 OK` con cuerpo que contiene `VALID`.

> **Nota:** Mantén la aplicación corriendo en una terminal durante todo el taller.

---

## 5. Paso 2 – Instalar Apache JMeter

### Windows

1. Descargar `apache-jmeter-5.6.3.zip` desde https://jmeter.apache.org/download_jmeter.cgi
2. Descomprimir en `C:\jmeter\`
3. Agregar `C:\jmeter\bin` a la variable de entorno `PATH`
4. Verificar: `jmeter --version`

### Linux / macOS

```bash
# Linux (Ubuntu/Debian)
wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz
tar -xzf apache-jmeter-5.6.3.tgz -C /opt/
export PATH=$PATH:/opt/apache-jmeter-5.6.3/bin

# macOS (Homebrew)
brew install jmeter
```

---

## 6. Paso 3 – Cargar el Plan de Prueba en JMeter GUI

1. Abrir JMeter: ejecutar `jmeter` (GUI) o `jmeter.bat` en Windows.
2. Ir a **File > Open**.
3. Navegar hasta `perf/scripts/registro_personas_baseline.jmx` y abrirlo.
4. Verificar que el árbol de componentes muestra:
   - `Test Plan`
     - `User Defined Variables`
     - `Thread Group – Baseline 10 VUs`
       - `CSV Data Set Config`
       - `HTTP Header Manager`
       - `HTTP Request – POST /register`
       - `Response Assertion`
       - `Summary Report`
       - `Aggregate Report`

---

## 7. Paso 4 – Configurar Variables del Plan

En el nodo **User Defined Variables** (clic en él en el árbol), verificar o ajustar:

| Variable | Valor por defecto | Descripción |
|----------|------------------|-------------|
| `BASE_URL` | `localhost` | Host de la aplicación |
| `PORT` | `8080` | Puerto de la aplicación |
| `DATA_FILE` | `../data/persons.csv` | Ruta al dataset |

Si la aplicación corre en otro puerto, cambiar `PORT` aquí antes de ejecutar.

---

## 8. Paso 5 – Ejecutar Prueba Baseline y Verificar SLO

### Definición de SLO para este taller

| Métrica | Objetivo (SLO) |
|---------|---------------|
| Latencia p95 | < 300 ms |
| Latencia p99 | < 800 ms |
| Tasa de errores | < 1 % |
| Throughput | > 10 req/s |

### Ejecutar desde GUI

1. Asegurarse de que el Thread Group `Baseline` esté habilitado (clic derecho > Enable).
2. Presionar el botón **Start** (triángulo verde) o usar `Ctrl+R`.
3. Observar en tiempo real el **Summary Report** en el panel derecho.
4. Después de 5 minutos la prueba se detiene automáticamente.

### Ejecutar desde línea de comandos (recomendado)

```bash
jmeter -n \
  -t perf/scripts/registro_personas_baseline.jmx \
  -l perf/results/baseline_$(date +%Y%m%d_%H%M%S).jtl \
  -e -o perf/results/baseline-report/
```

> `-n`: modo no-GUI (headless) | `-t`: test plan | `-l`: log JTL | `-e -o`: generar reporte HTML

---

## 9. Paso 6 – Ejecutar Prueba Load y Comparar con Baseline

```bash
jmeter -n \
  -t perf/scripts/registro_personas_load.jmx \
  -l perf/results/load_$(date +%Y%m%d_%H%M%S).jtl \
  -e -o perf/results/load-report/
```

La prueba Load tiene tres etapas:

| Etapa | VUs inicio | VUs fin | Duración |
|-------|-----------|---------|----------|
| Ramp-up | 10 | 50 | 2 min |
| Sostenido | 50 | 100 | 5 min |
| Ramp-down | 100 | 0 | 2 min |

---

## 10. Paso 7 – Interpretar las Métricas

### Métricas clave en JMeter

| Métrica JMeter | Descripción | Cómo leerla |
|---------------|-------------|-------------|
| **Average** | Latencia promedio de todas las solicitudes | Buena referencia general, sensible a outliers |
| **90% Line (p90)** | 90% de requests respondieron en este tiempo o menos | Ignora el 10% más lento |
| **95% Line (p95)** | Umbral SLO más común | Si supera 300ms, hay problema de rendimiento |
| **99% Line (p99)** | Cubre casi todos los casos extremos | Si supera 800ms, hay cuellos de botella severos |
| **Throughput** | Solicitudes por segundo procesadas | Mide capacidad del sistema |
| **Error %** | Porcentaje de respuestas fallidas | Si supera 1%, el sistema está degradado |

### Tabla comparativa esperada

| Escenario | VUs | p95 esperado | Throughput esperado | Error rate |
|-----------|-----|-------------|-------------------|-----------|
| Baseline | 10 | < 150 ms | > 10 req/s | < 0.1% |
| Load | 100 | < 300 ms | > 50 req/s | < 1% |

### Ejemplo de salida JMeter CLI

```
summary =  15000 in 00:05:00 = 50.0/s  Avg:   180  Min:    45  Max:  1200  Err:   0 (0.00%)
```

- **50.0/s**: throughput alcanzado
- **Avg: 180**: latencia promedio 180 ms
- **Err: 0 (0.00%)**: sin errores

---

## 11. Paso 8 – Exportar Reporte HTML

Los reportes HTML se generan automáticamente con la bandera `-e -o`:

```bash
# El reporte queda en perf/results/baseline-report/index.html
open perf/results/baseline-report/index.html    # macOS
start perf/results/baseline-report/index.html   # Windows
```

El reporte incluye gráficas de: latencia en el tiempo, throughput, percentiles, usuarios activos.

---

## 12. Análisis de Cuellos de Botella

### Señales de alerta

| Señal | Posible causa |
|-------|--------------|
| p95 sube linealmente con VUs | Pool de conexiones de BD saturado |
| Error rate > 1% en load | Límite de threads del servidor alcanzado |
| Throughput se aplana antes del máximo de VUs | CPU o memoria del servidor saturada |
| Latencia sube en soak test | Fuga de memoria o acumulación de conexiones |

### Herramientas complementarias

```bash
# Monitorear CPU y memoria del servidor (Linux)
top -p $(pgrep -f "spring-boot")

# Ver logs del servidor para errores
tail -f registraduria/logs/application.log
```

---

## 13. Registro de Defectos

Al finalizar las pruebas, documentar en [`defectos.md`](./defectos.md) los problemas encontrados usando el formato de [`defectos_template.md`](./defectos_template.md).

**Criterio mínimo:** Al menos 1 escenario debe mostrar violación de SLO.

---

## 14. Entregables

- [ ] Capturas de pantalla del Summary Report (Baseline y Load)
- [ ] Archivos JTL en `perf/results/`
- [ ] Reporte HTML generado
- [ ] `defectos.md` completado con hallazgos propios
- [ ] Análisis escrito de la tabla comparativa (mínimo 200 palabras)

---

## 15. Referencias

- [Apache JMeter User's Manual](https://jmeter.apache.org/usermanual/index.html)
- [JMeter Best Practices](https://jmeter.apache.org/usermanual/best-practices.html)
- [Google SRE Book – SLOs](https://sre.google/sre-book/service-level-objectives/)
- [ISO/IEC 25010 – Performance Efficiency](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010)

---

© Universidad de La Sabana – Facultad de Ingeniería  
Maestría en Ingeniería de Software – 2025
