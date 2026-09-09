# Informe de avance — Tesis de Maestría en Informática

**Título de la tesis:** Asimilación de observaciones de la red meteorológica del PGICH en el modelo WRF para mejorar pronósticos locales en el Valle de Tulum, San Juan, Argentina

**Autor:** Roberto Nicolás Molina

**Director:** Dr. Oscar Raúl Dölling
**Codirector:** Mg. Héctor Ramón Lepez

**Fecha:** 2026-08-06

---

## 1. Resumen ejecutivo

En este primer avance se completaron las tres primeras fases del plan de tesis: el procesamiento de los datos de las estaciones meteorológicas, la configuración del modelo WRF para la zona de San Juan y la puesta en marcha del esquema de asimilación de datos (Observational Nudging). El sistema ya funciona de principio a fin en una PC local y fue probado en dos fechas reales, mostrando que la asimilación de las observaciones de las estaciones mejora los pronósticos de temperatura y humedad en comparación con una corrida sin asimilación.

La siguiente fase (evaluación con muchos casos) requiere correr decenas de simulaciones para que los resultados sean estadísticamente confiables, y esto está limitado por la infraestructura disponible: hoy las corridas se hacen en una PC compartida, con pocos procesadores y una a la vez. Por eso se solicita acceso a la infraestructura de cómputo de alto rendimiento (HPC) disponible, para poder acelerar esta etapa y cumplir los plazos de la tesis.

## 2. Aporte del trabajo desde la informática

Más allá del resultado meteorológico, la tesis desarrolla un **sistema de software completo y reproducible** que resuelve de forma automatizada todo el circuito que va desde los datos en bruto hasta el informe final. El sistema está compuesto por módulos independientes y reutilizables:

| Módulo | Qué hace |
|---|---|
| **Adquisición automática de datos** | Se conecta a la API de las estaciones EcoWitt y descarga las observaciones sin intervención manual. |
| **Control de calidad** | Detecta y descarta mediciones erróneas o fuera de rango antes de usarlas. |
| **ETL (extracción, transformación y carga)** | Normaliza los datos de distintos formatos a un esquema único y ordenado. |
| **Generación de archivos LITTLE_R** | Módulo de desarrollo propio que convierte las observaciones al formato LITTLE_R (ancho fijo de Fortran) que exige el sistema de asimilación, sin depender de librerías externas. |
| **Automatización del flujo** | Un orquestador (`pipeline_wrf.py`) encadena todos los pasos y corre el modelo con y sin asimilación. |
| **Ejecución reproducible** | Cualquier caso se puede repetir exactamente igual usando la misma configuración y los mismos datos. |
| **Evaluación automática** | Compara el pronóstico contra las observaciones y genera métricas y gráficos automáticamente. |

La siguiente figura muestra la arquitectura del sistema y cómo se conectan estos módulos:

![Arquitectura del sistema desarrollado](figura_arquitectura.png)

*Figura 1. Arquitectura del sistema: flujo de datos desde las estaciones y el forzante global GFS hasta los resultados de validación, orquestado por `pipeline_wrf.py`.*

*Nota: los módulos de generación de archivos LITTLE_R y su conversión al formato de entrada del modelo (OBS_DOMAIN101) fueron implementados íntegramente en este trabajo, replicando las especificaciones de ancho fijo de Fortran del sistema de asimilación de WRF.*

## 3. Estado de avance según el plan metodológico

| Fase | Descripción | Estado |
|---|---|---|
| **Fase 1 — Preparación de datos** | Bajar las observaciones de las 10 estaciones EcoWitt, limpiarlas (control de calidad) y ordenarlas en el formato que el modelo necesita. | Completada |
| **Fase 2 — Configuración del modelo WRF** | Configurar el dominio (zona de San Juan), la resolución, la física y las condiciones de borde (datos GFS). | Completada |
| **Fase 3 — Asimilación de datos** | Implementar el esquema Observational Nudging, que va "corrigiendo" el pronóstico en tiempo real usando las estaciones cada 30 minutos. | Completada |
| **Fase 4 — Evaluación** | Comparar estadísticamente el modelo con asimilación contra el modelo sin asimilación y contra las observaciones reales, en varios días de condiciones climáticas distintas. | En curso (2 casos probados) |
| **Fase 5 — Integración y entrega** | Automatizar todo el circuito y dejar un visor interactivo de resultados. | Pendiente |

En la Fase 4 el objetivo es analizar como mínimo 6 a 10 días con situaciones meteorológicas variadas (viento Zonda, días con lluvia, días estables, frentes fríos) y probar distintas combinaciones de parámetros de la asimilación. Cada combinación se convierte en una corrida del modelo.

## 4. Qué se logró en la PC local

### 4.1 El circuito completo funciona

Se armó un proceso automatizado que va desde la observación hasta el resultado final, sin pasos manuales. En la Figura 1 se puede ver el recorrido completo de los datos a través de los módulos del sistema.

Este circuito se controla con un solo programa (`pipeline_wrf.py`) que prepara los datos, corre el modelo con y sin asimilación, y genera las métricas de error y los gráficos de comparación.

Parte de este circuito —en particular la generación de los archivos LITTLE_R y su conversión al formato que lee el modelo— fue implementada de forma propia en esta tesis, replicando las especificaciones de formato del sistema de asimilación de WRF. Esto constituye un aporte de desarrollo de software del trabajo y queda documentado en el repositorio del proyecto.

### 4.2 Se validó el sistema en dos fechas reales

**Caso 1 — 25 de mayo de 2026 (8 estaciones):**

| Variable | Modelo sin asimilación (control) | Modelo con asimilación | Mejora |
|---|---|---|---|
| Temperatura (T2) | RMSE 6.25 | RMSE 4.94 | Reducción del 21% |
| Humedad relativa | RMSE 23.37 | RMSE 14.84 | Reducción del 36% |
| Viento | RMSE 3.64 | RMSE 3.36 | Reducción del 8% |

**Caso 2 — 5 de julio de 2026 (4 estaciones):**

| Variable | Modelo sin asimilación (control) | Modelo con asimilación | Mejora |
|---|---|---|---|
| Temperatura (T2) | RMSE 8.03 | RMSE 3.70 | Reducción del 54% |
| Humedad relativa | RMSE 33.25 | RMSE 11.13 | Reducción del 67% |
| Viento | RMSE 5.44 | RMSE 6.30 | Sin mejora clara |

El RMSE (raíz del error cuadrático medio) es la medida estándar de cuánto se equivoca el modelo: un valor más bajo significa pronóstico más preciso.

![Comparación modelo vs observaciones — caso 2](validacion_20260705/scatter_4panels.png)

![Tabla de métricas — caso 2](validacion_20260705/tabla_metricas.png)

![Mapa de errores de temperatura — caso 2](validacion_20260705/mapa_errores_t2.png)

### 4.3 Se analizó la sensibilidad de los parámetros

Se probaron distintos valores del coeficiente que controla "cuánta fuerza" tienen las estaciones para corregir al modelo. El mejor resultado se obtuvo con un valor de 0.001, que reduce el error de temperatura un 27% y el de humedad un 49% respecto de no asimilar nada.

**Conclusión preliminar:** la asimilación de las estaciones del PGICH mejora claramente la temperatura y la humedad. El viento y la presión no muestran todavía una mejora consistente, lo que se confirma o descarta cuando se analicen más días.

## 5. Limitaciones de la infraestructura actual

El trabajo se realiza en una PC de escritorio compartida con otros estudiantes (se accede por escritorio remoto), donde el modelo corre dentro de una máquina virtual con las siguientes limitaciones:

- **Poca capacidad de cómputo:** el modelo usa 4 procesadores en paralelo; por el tamaño del dominio, se podría aprovechar muchos más si la máquina lo permitiera.
- **Una corrida a la vez:** los casos se ejecutan en secuencia, uno detrás de otro, sin poder aprovechar el tiempo muerto.
- **Inestabilidad:** el entorno (Docker sobre Windows) se desconecta con frecuencia en medio de las corridas.
- **Recurso compartido:** al ser una PC de varios usuarios, los tiempos de ejecución son impredecibles y no se puede garantizar disponibilidad.

Con esta configuración, cada corrida de 12 horas de simulación tarda unos 4 minutos en ejecutarse. A simple vista parece poco, pero multiplicado por los más de 30 casos que requiere la Fase 4, y sumando los reinicios por caídas del sistema, el avance es muy lento.

## 6. Propuesta de uso de la infraestructura HPC

El cómputo de alto rendimiento (HPC) resuelve las dos limitaciones anteriores:

1. **Más procesadores por corrida:** el modelo WRF puede repartirse en muchos procesadores a la vez. En la infraestructura HPC se planea usar 16 a 32 procesadores por corrida, lo que reduce el tiempo individual de cada una de 4 minutos a menos de 1 minuto.
2. **Varias corridas en paralelo:** los casos son independientes entre sí (cada día y cada combinación de parámetros no depende de los demás), por lo que se pueden correr varios al mismo tiempo en distintas máquinas del clúster. Esto multiplica la cantidad de resultados por día.

**Plan concreto:**
- Preparar una sola vez por fecha los datos de entrada (preprocesamiento WPS + condiciones iniciales), que hoy se recalcula para cada caso.
- Lanzar las corridas de la Fase 4 en paralelo (los 6-10 días × las configuraciones de sensibilidad seleccionadas).
- Ejecutar la validación y generar los gráficos automáticamente al terminar cada corrida.

Con esto, la Fase 4 completa (30+ corridas) se puede terminar en días en lugar de semanas, y con más margen para corregir errores y repetir casos.

## 7. Solicitud y próximos pasos

Se solicita al director considerar y gestionar el acceso a la infraestructura HPC disponible (cuenta de usuario, cuota de cómputo y autorización de uso), con el fin de:

1. Ejecutar las simulaciones restantes de la Fase 4 (evaluación con múltiples casos).
2. Mantener el circuito automatizado que ya funciona, adaptado para correr en esa infraestructura.
3. Avanzar luego a la Fase 5 (automatización completa y visualización de resultados).

**Próximos pasos propuestos:**
- Definir con el director la lista definitiva de días y configuraciones para la Fase 4.
- Adaptar los scripts actuales al entorno HPC (tarea ya planificada, sin impacto en los resultados ya obtenidos).
- Realizar una corrida de prueba en el HPC para validar el entorno y estimar tiempos finales.
