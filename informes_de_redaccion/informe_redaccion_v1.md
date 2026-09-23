# Informe de redacción v1 — Análisis de estructura del borrador v7

**Documentos analizados:**
- `tesis_borrador_22092026_v7.docx` (v7)
- `anteproyecto_19122025.docx` (anteproyecto)
- `Estructura_de_Tesis-Maestría_en_Informática.doc` (guía de estructura de la Maestría)

**Fecha del informe:** 23/09/2026

**Convención de fuentes:** lo que sale de los documentos se cita como [v7], [Anteproyecto] o [Guía]. Lo marcado como *(conocimiento general)* es de mi conocimiento, no de tus fuentes. Lo marcado como *(sugerencia)* es una hipótesis o propuesta mía.

> **Recordatorio:** verificá en fuente original cualquier cita o dato mencionado antes de usarlo en la tesis.

---

## Veredicto

La estructura de capítulos coincide con la guía y el formato está bien. Los problemas son de **extensión**, de **hilo conductor** y de **consistencia interna**.

---

## 1. Cumplimiento de la guía

### Formato (verificado en el .docx)

A4, Times New Roman 12, interlineado 1,5, márgenes 3 / 2,5 / 2,5 / 2,5 cm, encabezado y pie a 1,25 cm, encabezado con título y autor, pie con capítulo. **Todo cumple.**

Pendiente: los capítulos deben empezar en página impar [Guía]. En mi render (LibreOffice, aproximado) arrancan en las págs. 10, 18, 24 y 34, o sea pares. Revisarlo en Word.

### Extensión (estimación por render, ±15%)

| Cap. | Páginas aprox. | Guía | Estado |
|---|---|---|---|
| I | 7 | 5–10 | ✓ |
| II | 5 | 25–50 | ✗ |
| III | 3 | 10–15 | ✗ |
| IV | 6 | 25–40 | ✗ |
| V | 7 | 10–25 | ✗ |
| VI | 3 | 5–10 | ✗ |

El cuerpo suma unas 31 páginas y la guía pide 100–150. La guía aclara que la estructura es "de referencia", pero la distancia es grande. La proporción 1:2 entre lo revisado y lo propuesto sí se cumple, y de sobra (Cap. II tiene ~5 págs. contra ~16 de Cap. III–V). El problema es el total, no el equilibrio.

### Contenido

- **Cap. I:** las secciones 1.3 y 1.4 no tienen ninguna referencia, y la guía las pide ("con referencias"). La afirmación de 1.4 de que la asimilación es "una de las estrategias más efectivas" tampoco está citada.
- **Cap. II:** es casi textual el Antecedentes + Marco Teórico del anteproyecto, con tiempos verbales de proyecto ("se implementará Git/Docker", "la arquitectura contempla HPC o nube"). No cubre la literatura que necesitan tus hallazgos.
- **Cap. III:** no lleva citas, como pide la guía. Pero define el alcance "conforme al anteproyecto"; la tesis debería sostenerse sola.
- **Índice:** desactualizado, con mojibake ("CapÃtulo") desde el Cap. II, sin subsecciones de II–VI y con páginas que no coinciden. Además hay dos secciones "1.9", y el índice dice "1.8 Contribuciones" cuando el cuerpo tiene "1.8 Breve Esbozo".
- **Faltantes:** lista de siglas (el anteproyecto la tenía; el v7 usa WRFDA, PSFC, dRMSE, QC, etc.) y, como *(sugerencia)*, un resumen (confirmar con el director).

---

## 2. Problema estructural principal: el hilo conductor

La pregunta de investigación (límites de la generalización espacial y topográfica) **no aparece en el v7**.

- El hueco de conocimiento (2.3) es de ingeniería: nadie integró la red con WRF de forma automática.
- Los objetivos, la hipótesis ("mejorar significativamente") y el Cap. III no mencionan el hold-out, la topografía ni los eventos extremos.
- El hallazgo más fuerte (colapso de la generalización en el Zonda, sesgo de altura) vive solo en Cap. V y VI. Nada en I–III lo prepara.
- La sección 6.1 dice "el aporte central no es meteorológico sino arquitectónico", pero la 5.5 declara el resultado de viento "un resultado propio de esta tesis". Son dos tesis distintas.
- "Resolviendo por primera vez" (2.3) no se sostiene con cinco antecedentes. Hay que acotarla o sostenerla con una búsqueda sistemática.

### Objetivos y matriz sin evidencia en Cap. V

Solo el objetivo 5 se valida en Cap. V. No hay resultados de:
- calidad de datos (obj. 1);
- tasa de éxito o tiempos (obj. 2 y 4; solo aparece "~4 min por corrida" en 5.6);
- factibilidad de transferencia (obj. 6).

Además, 6.1 no concluye sobre el obj. 6, y la contribución "transferencia tecnológica" (1.9) contradice a 6.2 y 6.3, donde la Fase 5 está inconclusa.

### Desvíos respecto del anteproyecto sin explicar

- Se eliminó el objetivo de determinar el esquema más eficiente (Nudging vs 3D-Var); ahora es trabajo futuro.
- Las heladas se reemplazaron por frente frío.
- Agregación horaria [Anteproyecto] vs ventana de 30 min [v7, 4.3].
- Dominio de 4 km [v7, 3.3] vs 15 km [v7, 5.4 y 5.5]. ¿Cuál fue el dominio de los experimentos? ¿El "control" es realmente el WRF operativo?
- Plotly/Dash [Anteproyecto] vs Streamlit [v7].

---

## 3. Consistencia interna y datos

### Citas del Cap. II

- El Nudging "eficaz en redes de baja densidad" se apoya en [2], y el 3D-Var "probado en superficie" en [3]. Pero la tabla 2.3 dice que [2] no implementa asimilación y que [3] usa radar. Se contradicen.
- [7] (SciPy) sostiene tres afirmaciones, dos de ellas (HPC/nube, flujos en tiempo real) fuera de lo que esperaría de ese paper. Verificar.
- [13] no lo pude verificar. Chequear que exista tal cual está citado, incluido el DOI.

### Dato técnico *(conocimiento general, verificar en la guía de usuario de WRF)*

Hasta donde sé, el Observation Nudging (FDDA) es parte del núcleo de WRF, no de WRFDA. WRFDA implementa 3D-Var/4D-Var/híbridos. El v7 dice "dos esquemas disponibles en WRFDA" y sus objetivos hablan de compatibilidad con WRFDA, cuando lo implementado (OBS_DOMAIN101) no usa WRFDA.

### Cap. V

- **Los coeficientes de nudging no son comparables entre casos.** Casos 1–2 usan 0.001, el Caso 3 usa 0.0002 y los Casos 4–6 (los del hold-out) usan 0.0001, que 5.3 llama subóptimo. No se justifica para los Casos 5 y 6.
- **Falta el control en los hold-out.** Las tablas de los Casos 4 y 5 muestran solo RMSE nudged. "Colapso de generalización" debería medirse como mejora frente al control en las estaciones hold-out (dRMSE), no como RMSE absoluto. Sin el control no se distingue "el nudging no generaliza" de "el modelo era malo ahí igual".
- **El split no es aleatorio.** Con solo 1–2 estaciones hold-out elegidas a priori y de mayor error de altura, brecha y topografía quedan confundidas. *(Sugerencia)*: validación cruzada dejando una estación afuera (leave-one-station-out).
- **La atribución causal a la resolución de 15 km es hipótesis, no resultado.** No fue testeada.
- **El conteo de viento no cierra.** "No mejora en 4 de 6" enumera los Casos 2, 3 y 6. El Caso 1 mejora 8%, el Caso 5 no reporta viento y el Caso 4 muestra dRMSE≈+0.02.
- **"Mejora consistente de T y RH en 6 casos" no es verificable con las tablas.** El Caso 6 tiene T2 en −0.23 a las 12Z, y los Casos 4 y 5 no muestran control.
- **PSFC "queda explicado" (5.5) es demasiado fuerte.** *(Mi cálculo aproximado, verificar):* cerca de superficie son ~0.1 hPa/m, así que +918 m implicaría del orden de −80 a −100 hPa, no −46. Además, en el Caso 3 el sesgo fue −18/−19, fuera del rango "−29 a −46". Para afirmarlo, aplicar corrección hipsométrica y mostrar que el sesgo residual se anula.
- **"Significativamente" (1.7) no tiene respaldo.** Hay bootstrap en el código (4.5) pero no aparece ningún IC ni test en Cap. V.
- **Faltan** la definición de dRMSE (y su convención de signo), el método de interpolación estación–grilla y una tabla de estaciones con altitud real y del modelo, rol y ubicación. Esta tabla es la pieza central del tema y hoy no existe.
- **5.3 tiene un solo párrafo sin tabla.** Sus valores (27% y 49% a 0.001) difieren de los del Caso 1 con el mismo coeficiente, sin explicación. Además, todo el Cap. V tiene cero figuras.
- **Alcance del "valle de Tulum"** *(conocimiento general, verificar)*: hasta donde sé, Cuesta del Viento y Valle Fértil quedan fuera del valle estricto. Puede afectar cómo se define "generalización espacial".

---

## 4. Marcas de borrador

- Hay notas de trabajo dentro del texto: "al momento de este escrito", "se debe verificar", "se recomienda revisar", "debe re-ejecutarse". Antes de cerrar hay que re-ejecutar el hold-out del Caso 6 y revisar la métrica de correlación (r = −0.997), y luego quitar esas notas.
- Los títulos usan lenguaje de fases ("Cierre de Fase 4", "Inicio de la Fase 5") y "corridas de puesta a punto" figura como tipo de evento. La tesis se organiza por preguntas, no por fases del proyecto.
- La sección 4.5 mezcla un bug de validación con la robustez de la solución. La 5.6 (gestión de HPC ante la dirección) es narrativa de cronograma y no tiene lugar en Cap. V.

---

## 5. Prioridad de trabajo (de más a menos recomendado)

1. **Decidir y alinear el hilo:** hueco, objetivos, hipótesis y Cap. III con lo que realmente se encontró (ver pregunta abierta).
2. **Rehacer la validación de generalización:** dRMSE en hold-out, mismo coeficiente entre casos, estación por estación, con IC. Es lo que sostiene el resultado principal.
3. **Ampliar el Cap. II** hacia lo que se usa en la discusión (representatividad topográfica, nudging en terreno complejo, foehn/Zonda, QC de sensores de bajo costo). Las referencias se buscan y verifican en fuente original; no se toman de memoria.
4. **Limpiar y ampliar Cap. III–V:** tabla de estaciones, figuras, sección de métodos de validación, mover el detalle de código a anexos.
5. **Índice, siglas, numeración y notas de borrador.**

---

## Pregunta abierta

¿Cuál es el aporte central que querés defender?

- **(a)** La arquitectura/pipeline, con el hold-out como hallazgo de validación.
- **(b)** Los límites de generalización del nudging, con la arquitectura como vehículo.

La pregunta de investigación planteada apunta a (b); el v7 hoy está escrito para (a).
