# Plan de Experimento Científico: Asimilación de Datos mediante Observation Nudging (WRF-DA) en el Valle de Tulum, San Juan

Este documento establece la estructura metodológica, el flujo de trabajo técnico y el cronograma para ejecutar una prueba del modelo WRF de manera reproducible en un entorno Linux. El objetivo central de la investigación es cuantificar el impacto y la mejora que aporta la incorporación de observaciones de superficie locales mediante *Observation Nudging* (utilizando el formato `LITTLE_R`) frente a una predicción convencional sin asimilación.

---

## Esquema General del Experimento

El flujo metodológico del experimento se estructura de la siguiente manera:

```
                Datos GFS / FNL / ERA5
                          │
                   WPS + REAL.EXE
                          │
             ┌────────────┴────────────┐
             │                         │
        Experimento A             Experimento B
       (WRF de Control)        (WRF + Obs Nudging)
             │                         │
             │                  Archivo LITTLE_R
             │                   generado desde
             │              estaciones PGICH San Juan
             │                         │
             └────────────┬────────────┘
                          │
                Comparación de Resultados
                          │
                Validación con Observaciones
                          │
                 Análisis Estadístico
                          │
                     Conclusiones
```

---

## Objetivo General
Determinar y cuantificar el impacto de la asimilación de datos observacionales superficiales de la red del PGICH mediante *Observation Nudging* utilizando un archivo en formato `LITTLE_R`, sobre la precisión de los pronósticos meteorológicos del modelo *Weather Research and Forecasting* (WRF) en el ámbito geográfico del Valle de Tulum, provincia de San Juan, Argentina.

---

## Etapas del Plan de Trabajo

### Etapa 1. Preparación del Ambiente Local
La ejecución se realizará de forma local en la notebook del investigador. Al tratarse de hardware portátil (ej. procesador Intel Core i3 de 13ra Gen con 8 GB de RAM o configuraciones superiores), se optimizarán los recursos sin necesidad de servidores externos.

* **Sistema Operativo:** Entorno Linux nativo o WSL2 (recomendado Ubuntu 22.04 LTS o superior).
* **Compiladores de Base:** GNU Compiler Collection (`gcc`, `gfortran`, `g++`).
* **Librerías y Dependencias Requeridas:**
    * `NetCDF` (versiones C y Fortran con soporte de grandes archivos).
    * `MPICH` o `OpenMPI` (para la paralelización por procesos, configurando el uso máximo de núcleos físicos estables).
    * Librerías gráficas de compresión para datos GRIB2: `Jasper`, `libpng`, `zlib`.
    * `HDF5` (requerido para el soporte NetCDF4).
* **Módulos del Modelo:** compilación exitosa de `WPS` y `WRF` (opción *em_real*).

### Etapa 2. Diseño y Configuración del Dominio de Estudio
Para aislar correctamente el Valle de Tulum y mitigar los efectos de borde generados por la compleja topografía de la Precordillera y la Cordillera de los Andes, se plantea una configuración de tres dominios anidados (*one-way* o *two-way nesting* según disponibilidad de memoria RAM):

* **Dominio 1 (D01 - Región Oeste de Argentina):** Resolución espacial de 9 km. Captura la dinámica sinóptica continental.
* **Dominio 2 (D02 - Provincia de San Juan):** Resolución espacial de 3 km. Actúa como transición hacia la escala local.
* **Dominio 3 (D03 - Valle de Tulum):** Resolución espacial de 1 km. Escala convectiva local que permite resolver la topografía fina y los gradientes térmicos del valle.

### Etapa 3. Selección del Caso de Estudio
Se debe seleccionar un evento meteorológico crítico y bien documentado que afecte significativamente a la región de San Juan. Se sugieren tres tipos de fenómenos:
1.  **Evento de Viento Zonda:** Fenómeno de viento catabático seco y cálido de fuerte impacto local.
2.  **Tormenta Convectiva Estival:** Eventos de precipitación intensa repentina en el Valle de Tulum.
3.  **Ingreso de Frente Frío / Ola de Calor:** Extremos térmicos estacionales.

* **Ventana Temporal del Experimento:** Se fija un período de simulación continuo de **48 a 72 horas**. Una ventana extendida permite evaluar correctamente la persistencia temporal del efecto del *nudging* y su tasa de degradación una vez que cesa el guiado meteorológico.

### Etapa 4. Provisión de Datos Iniciales y de Frontera
Para asegurar el rigor científico, ambos experimentos compartirán exactamente la misma base sinóptica:
* **Fuentes recomendadas:** Datos del modelo global GFS (con resolución de 0.25°) o reanálisis ERA5.
* **Ejecución:** Procesamiento idéntico mediante la secuencia estándar: `geogrid.exe` $ightarrow$ `ungrib.exe` $ightarrow$ `metgrid.exe`.

### Etapa 5. Generación e Inyección del Archivo `LITTLE_R`
Este componente representa el núcleo innovador de la tesis. Consiste en la ingesta y el formateo de las lecturas horarias de la red del PGICH (Plan de Gestión Integral de la Cuenca del Río San Juan):
* **Variables de Superficie a Formatear:** Temperatura (T), Humedad Relativa o Razón de Mezcla (Q), Presión Superficial (PSFC), Velocidad del Viento (U, V) y Dirección del Viento.
* **Flujo de Conversión:**
    1.  Estructuración de datos crudos mediante scripts en Python (Pandas) al formato ASCII estándar de `LITTLE_R` (bloques con encabezado de estación, registros de altura/superficie, línea de fin de reporte).
    2.  Procesamiento opcional con `obsgrid.exe` para control de calidad del dato.
    3.  Transformación al formato estructurado de asimilación directa mediante la utilidad oficial de WRF en Perl (`RT_fdda_reformat_obsnud.pl`).
    4.  Ubicación del archivo resultante en el directorio `/run` renombrado bajo la estructura estándar: `OBS_DOMAIN101` (o el índice correspondiente al dominio donde se aplicará el nudging).

### Etapa 6. Experimento A (Simulación de Control - WRF Solo)
Simulación meteorológica convencional que servirá como línea de base (Baseline).
* **Configuración:** Bloque `&fdda` apagado por completo en el `namelist.input` (`obs_nudge_opt = 0`).
* **Ejecución:** Correr `real.exe` y posteriormente `wrf.exe`.
* **Salida:** Almacenamiento seguro de los archivos netCDF netos (`wrfout_d03_*`) bajo un directorio etiquetado como `CONTROL`.

### Etapa 7. Experimento B (Simulación con Asimilación - WRF Nudging)
Simulación dinámica modificada por la fuerza de guiado observacional.
* **Configuración:** Modificar el `namelist.input` activando las variables de *Observation Nudging*:
    ```fortran
    &fdda
    obs_nudge_opt          = 1, 1, 1,      ! Activación por dominio
    max_obs                = 150000,       ! Umbral máximo de observaciones
    fdda_start             = 0., 0., 0.,   ! Inicio de asimilación (minutos)
    fdda_end               = 1440., 1440., ! Fin de asimilación según caso
    obs_nudge_wind         = 1, 1, 1,      ! Asimilación de componentes de viento
    obs_nudge_temp         = 1, 1, 1,      ! Asimilación de temperatura
    obs_nudge_mois         = 1, 1, 1,      ! Asimilación de humedad
    obs_rinxy              = 30., 15., 5., ! Radio de influencia horizontal decreciente (km)
    obs_twindo             = 0.5, 0.5, 0.5,! Ventana temporal de peso (horas)
    /
    ```
* **Ejecución:** Asegurar la presencia del archivo `OBS_DOMAIN10x`. Correr `wrf.exe`.
* **Validación de Lectura:** Monitorear los archivos de log (`rsl.out.0000`) para certificar que el modelo asimila activamente las estaciones mediante la instrucción `CALL IN4DOB`. Guardar salidas netCDF en el directorio `NUDGING`.

### Etapa 8. Definición de Variables de Control y Comparación
Se evaluarán las variables clave de diagnóstico en superficie y capa límite, que corresponden a los sensores físicos disponibles en las estaciones terrestres:
* **Temperatura a 2 metros (`T2`)**
* **Humedad Específica / Razón de Mezcla a 2 metros (`Q2`)**
* **Presión en Superficie (`PSFC`)**
* **Componentes del Viento a 10 metros (`U10`, `V10`)**
* **Precipitación Acumulada (`RAINC` Convectiva, `RAINNC` No Convectiva)**

### Etapa 9. Extracción de Datos y Postprocesamiento con Python
Automatización de la lectura de salidas espaciales mediante código científico en Python (`xarray`, `wrf-python`, `netCDF4`):
1.  **Geolocalización:** Localizar los índices matriciales del punto de grilla $(i, j)$ más cercano a las coordenadas geográficas reales de cada estación del PGICH (Pocito, Caucete, Capital, etc.).
2.  **Slicing Temporal:** Extraer las series temporales de las variables simuladas tanto del bloque `CONTROL` como del bloque `NUDGING` para esos puntos de grilla específicos.
3.  **Alineación:** Consolidar un DataFrame estructurado en Pandas que unifique en una misma estampa temporal: `[Fecha/Hora | Valor Observado Real | Valor Simulado Control | Valor Simulado Nudging]`.

### Etapa 10. Validación y Contraste Estadístico
Para cada estación meteorológica considerada en el Valle de Tulum, se contrastarán los vectores temporales calculando métricas de error analítico:

* **Error Medio (ME / Bias):** Mide la tendencia del modelo a sobreestimar o subestimar la variable.
    $$ME = rac{1}{n}\sum_{i=1}^{n}(X_{sim} - X_{obs})$$
* **Error Absoluto Medio (MAE):** Evalúa la magnitud promedio de los errores sin considerar su signo.
    $$MAE = rac{1}{n}\sum_{i=1}^{n}|X_{sim} - X_{obs}|$$
* **Raíz del Error Cuadrático Medio (RMSE):** Penaliza de forma más severa los errores grandes o desvíos extremos.
    $$RMSE = \sqrt{rac{1}{n}\sum_{i=1}^{n}(X_{sim} - X_{obs})^2}$$
* **Coeficiente de Correlación de Pearson ($R$):** Evalúa la correspondencia en las tendencias y ciclos diurnos.

### Etapa 11. Visualización Científica y Gráficos Compilatorios
Se generarán figuras de nivel de publicación académica utilizando `matplotlib` y `seaborn`:
* **Gráficos de Líneas Temporales:** Comparación directa de las tres curvas (Obs, Control, Nudging) a lo largo de las 48-72 horas para evaluar desfases temporales.
* **Boxplots de Error Absoluto:** Distribución del error por experimento para identificar la variabilidad del desvío.
* **Mapas de Campos de Diferencia:** Plots espaciales del Dominio 3 que muestren la resta directa de matrices ($\Delta = 	ext{Campo}_{	ext{Nudging}} - 	ext{Campo}_{	ext{Control}}$). Esto visualizará el radio de impacto geográfico real alrededor de los oasis y valles de San Juan.
* **Histogramas de Frecuencia del Error:** Verificación de la normalidad de los residuos y reducción del sesgo.

### Etapa 12. Estructura para la Discusión de Resultados
El análisis del capítulo de resultados de la tesis deberá dar respuesta científica a los siguientes interrogantes metodológicos:
* ¿El uso de *Observation Nudging* reduce de forma estadísticamente significativa el RMSE en variables críticas como temperatura superficial y viento?
* ¿Qué variable física responde con mayor sensibilidad al esquema de asimilación local?
* ¿Cómo varía la efectividad del nudging con el paso de las horas? ¿Existe una degradación notable del impacto a las 12, 24 o 36 horas del inicio?
* ¿Cómo influye la compleja topografía de la precordillera de San Juan en el comportamiento del radio de influencia horizontal configurado en el modelo?
* ¿Se logra corregir el sesgo sistemático (Bias) de temperatura que suelen presentar los modelos globales en zonas áridas y montañosas?

### Etapa 13. Formulación de Conclusiones Generales
La conclusión debe validar la hipótesis planteada en la tesis. Un resultado esperado se estructuraría bajo la premisa de que la incorporación de la red local PGICH optimiza las condiciones de escala convectiva en el Valle de Tulum, disminuyendo el error absoluto en las primeras horas de pronóstico y demostrando el valor operativo de mantener y explotar redes de monitoreo densas en la provincia.

---

## Cronograma de Ejecución Propuesto (8 Semanas)

| Semana | Actividad Técnica | Entregable / Producto Tangible |
| :---: | :--- | :--- |
| **1** | Configuración, compilación del entorno numérico WRF/WPS en la distribución de Linux local. Verificación de dependencias. | Binarios ejecutables funcionales (`wrf.exe`, `real.exe`, `wps.exe`). |
| **2** | Diseño, optimización y prueba del namelist del dominio anidado triple centrado en San Juan (`namelist.wps`). | Archivo `namelist.wps` validado y mapas de geogrid controlados. |
| **3** | Descarga de condiciones de contorno e iniciales (GFS/ERA5) para el caso de estudio seleccionado. Procesamiento en WPS. | Archivos de entrada intermedia `met_em*` generados. |
| **4** | Simulación del Experimento A (Control sin asimilación). Monitoreo de estabilidad y consumo de hardware. | Archivos de salida netCDF `wrfout_d03_control`. |
| **5** | Extracción, limpieza y conversión de las observaciones horarias de la red PGICH al formato estándar `LITTLE_R` mediante Python y Perl. | Archivos de observaciones asimilables `OBS_DOMAIN10x`. |
| **6** | Configuración de las variables FDDA en `namelist.input`. Simulación del Experimento B (Nudging activo). | Archivos de salida netCDF `wrfout_d03_nudging`. |
| **7** | Desarrollo de scripts en Python para la extracción puntual, emparejamiento geográfico y cálculo de métricas estadísticas (RMSE, MAE, Bias). | DataFrames consolidados y tablas de resumen estadístico. |
| **8** | Generación de la galería gráfica final (series temporales, mapas de diferencias, boxplots) y redacción formal del capítulo de resultados de la tesis. | Borrador final del capítulo metodológico y de resultados. |

---

## Recomendación Metodológica Final
Para garantizar la viabilidad de una tesis de licenciatura y evitar la saturación en la fase de depuración de código, **es fundamental consolidar firmemente un único caso de estudio** (por ejemplo, un evento severo de viento Zonda bien documentado). No se debe intentar procesar múltiples meses o casos simultáneos hasta que todo el *pipeline* automatizado (desde la conversión a `LITTLE_R` hasta el cálculo del RMSE en Python) funcione perfectamente y de manera reproducible en la notebook. Una vez estandarizado este flujo, la extensión a nuevos casos meteorológicos se simplificará notablemente.
