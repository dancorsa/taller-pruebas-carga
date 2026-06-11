# Plantilla: Registro de Defecto de Rendimiento

Usa esta plantilla para reportar cada defecto identificado durante las pruebas. Copia el bloque y llena cada campo.

---

## Defecto PERF-XX — [Título breve del defecto]

- **Capa afectada:** [Aplicación / Base de datos / Red / Servidor]
- **Escenario:** [Baseline / Load / Stress / Spike / Soak] ([N] VUs)
- **SLO definido:** [Ej: p95 < 300 ms]
- **Resultado esperado:** [Qué debería ocurrir según el SLO]
- **Resultado obtenido:** [Valor real observado]

### Evidencia

```
[Pegar aquí la salida del Summary Report o Aggregate Report de JMeter]
[También puedes incluir fragmentos de log de la aplicación]
```

### Impacto

[Describir cómo afecta a los usuarios o al sistema. ¿Es visible? ¿Afecta disponibilidad?]

### Causa probable

- [Primera hipótesis técnica]
- [Segunda hipótesis técnica si aplica]

### Propuesta de mejora

- [Acción técnica concreta para resolver el problema]
- [Ajuste de configuración, optimización de código, etc.]

### Estado

[Abierto / En progreso / Resuelto]

### Prioridad

[Crítica / Alta / Media / Baja]

---

## Guía de Prioridades

| Prioridad | Cuándo asignarla |
|-----------|-----------------|
| Crítica | Error rate > 5% o sistema inaccesible bajo carga esperada |
| Alta | Incumplimiento de SLO de latencia bajo carga nominal |
| Media | Degradación leve o solo bajo carga extrema |
| Baja | Problema cosmético o sin impacto en usuarios reales |
