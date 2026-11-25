# Fuentes de datos — fase Prepare

Evaluación de las fuentes con el criterio **ROCCC** del certificado Google Data Analytics (Reliable, Original, Comprehensive, Current, Cited).

_Fecha de evaluación: 24/11/2026_

## Fuentes

| Fuente | Qué aporta | Acceso |
|---|---|---|
| REData (Red Eléctrica) | Generación por tecnología y demanda | API REST pública, sin token (`apidatos.ree.es`) |
| OMIE | Precio del mercado diario (fuente principal) | Ficheros diarios descargables (omie.es) |
| MIBGAS | Índice diario del precio del gas (€/MWh) | Ficheros anuales descargables (mibgas.es) |

El precio de mercado que también publica REData se usará como **fuente secundaria** para hacer una validación cruzada con OMIE.

## Evaluación ROCCC

### REData (generación y demanda)
| Criterio | Valoración | Razón |
|---|---|---|
| Reliable | Parcial | Los datos recientes son provisionales y se consolidan meses después con las lecturas de los contadores. |
| Original | Sí | Es la API de la propia REE, que mide la generación y la demanda. |
| Comprehensive | Parcial | Cubre todo el periodo de análisis (2022-2025), pero no sirve datos horarios de demanda ni de generación (ver Limitaciones). |
| Current | Sí | Publica los datos con unas horas de retraso. |
| Cited | Sí | Es el operador oficial del sistema eléctrico español; fuente fiable y reconocida. |

### OMIE (precio del mercado diario)
| Criterio | Valoración | Razón |
|---|---|---|
| Reliable | Sí | Es el mercado que fija los precios: el dato es el resultado de la casación, no una estimación. |
| Original | Sí | Es la fuente que fija el precio oficial de la electricidad. |
| Comprehensive | Sí | Cubre todo el periodo 2022-2025 sin huecos. Ojo: desde el 01/10/2025 hay 96 periodos al día y habrá que agregarlos a horas. |
| Current | Sí | Los precios se publican al poco tiempo. |
| Cited | Sí | Es el propio mercado ibérico de electricidad; fuente totalmente reconocida. |

### MIBGAS (precio del gas)
| Criterio | Valoración | Razón |
|---|---|---|
| Reliable | Sí | Es el propio mercado: el precio es el resultado de la negociación, no una medida ni una estimación. |
| Original | Sí | Es fuente primaria y me da los datos que necesito. |
| Comprehensive | Parcial | Los datos son diarios y mi análisis es horario, así que hay que transformarlos. |
| Current | Sí | Se publica de forma prácticamente actualizada. |
| Cited | Sí | Es el mercado oficial del gas en España y Portugal; su índice diario es la referencia que usa el sector. |

## Formato de los ficheros de OMIE (precio del mercado diario)
Revisado a mano el 24/11/2026 con los ficheros del 01/01/2024 y de noviembre de 2025.

- **Ubicación:** acceso a ficheros de omie.es → Mercado Diario → 1. Precios → "Precios horarios del mercado diario en España" (`marginalpdbc`).
- **Nombre del fichero:** `marginalpdbc_AAAAMMDD.1`, un fichero por día. El `.1` parece un número de versión (pendiente de confirmar).
- **Mercado day-ahead:** el fichero de un día se publica el día anterior.
- **Formato:** texto plano, separador `;`, decimales con punto y cada línea termina en `;` (al leerlo con pandas aparece una columna vacía extra).
- **Cabecera y pie:** la primera línea (`MARGINALPDBC;`) y la última (`*`) no son datos y hay que saltarlas.
- **Columnas:** año; mes; día; periodo; precio; precio. Las dos columnas de precio son las de España y Portugal. Coinciden salvo cuando hay separación de mercados (*market splitting*). **Pendiente: confirmar en la documentación de OMIE qué columna es cada país.**
- **Periodos por día (validación con recuento esperado):**

| Tipo de día | Hasta el 30/09/2025 (horario) | Desde el 01/10/2025 (15 minutos) |
|---|---|---|
| Normal | 24 | 96 |
| Cambio de hora de marzo (último domingo) | 23 | 92 |
| Cambio de hora de octubre (último domingo) | 25 | 100 |

## Formato de los ficheros de MIBGAS (precio del gas)
Revisado a mano el fichero de 2024 (descargado el 25/11/2026).

- **Ubicación:** acceso a ficheros de mibgas.es → Año 2024 → Resultados anuales → `MIBGAS_Data_2024.xlsx`
- **Nombre del fichero:** `MIBGAS_Data_YYYY.xlsx`, un fichero por año.
- **Formato:** Excel (.xlsx) con 12 hojas visibles (pendiente de confirmar si hay alguna más). La hoja `Definitions` es el diccionario de datos.
- **Hoja usada:** `MIBGAS Indexes`.
- **Columnas de la hoja `MIBGAS Indexes`:**
  - `Delivery day`: día de entrega del gas (gas day). **Clave para unir** con el precio de la luz del mismo día.
  - `MIBGAS PVB Last Price Index Day-Ahead` y `MIBGAS PVB Average Price Index Day-Ahead`: España (PVB).
  - `MIBGAS VTP Last Price Index Day-Ahead` y `MIBGAS VTP Average Price Index Day-Ahead`: Portugal (VTP).
  - `MIBGAS-ES Index`, `MIBGAS-PT Index`, `MIBGAS LNG-ES Index`, `MIBGAS AVB-ES Index`: índices de todos los productos con entrega ese día.
- **Unidad:** €/MWh **de gas** (energía térmica). No se puede comparar directamente con el €/MWh eléctrico: con un rendimiento de ~50 %, una central de ciclo combinado necesita ~2 MWh de gas por MWh eléctrico (además del coste del CO₂).

### Columna elegida: `MIBGAS PVB Average Price Index Day-Ahead`
- Media ponderada por volumen (VWAP) de la sesión del producto D+1 (o Weekend / D+3). Es más robusta que el último precio (`Last Price`), que depende de una sola operación.
- Se negocia el día anterior al de entrega, igual que el precio de la luz (subasta de OMIE, ~12:00 de D-1). Refleja el precio del gas **conocido** cuando se fijó el precio de la luz.
- Se descarta `MIBGAS-ES Index` porque incluye operaciones intradiarias hechas el propio día de entrega, después de la subasta de la luz → **look-ahead bias** (y **data leakage** en el modelo de la versión 2).

### Peculiaridades
- Filas ordenadas de **más reciente a más antiguo** → ordenar por fecha en la limpieza.
- Hay valor todos los días, pero el **sábado y el domingo tienen el mismo precio** (salen del producto Weekend, negociado el viernes). No es un error. En puentes y festivos se usa el producto D+3.
- Dato **diario**: para unirlo con datos horarios, se repite el valor en todas las horas del día (forward-fill / broadcast).

### Limitaciones
- Sin comprobar la hora de cierre de la sesión D+1 de MIBGAS. Si cierra después de las 12:00, el índice incluiría algunas operaciones posteriores a la subasta de OMIE.
- Sin comprobar si el gas day va de 00:00 a 24:00 o de 06:00 a 06:00 (estándar europeo).
- Sin comprobar todavía si el formato del fichero es igual en 2022, 2023 y 2025.

## Limitaciones y cómo se tratan
- **REData no sirve datos horarios (comprobado el 24/11/2026):** `demanda/evolucion` y `generacion/estructura-generacion` devuelven error 400 ("Los datos solicitados no están disponibles") con `time_trunc=hour`, también con el filtro peninsular (`geo_ids=8741`). Con `time_trunc=day` funcionan. **Decisión:** usar ESIOS (token solicitado el 24/11/2026) como fuente de demanda y generación horarias; REData queda para datos diarios y comprobaciones.
- **Datos provisionales (REData):** anotar la fecha de descarga y volver a descargar los meses recientes antes del análisis final.
- **Cambio a 15 minutos (OMIE, desde 01/10/2025):** agregar a horas con la media de los cuatro cuartos y comprobar que cada hora tiene exactamente cuatro valores.
- **Precio del gas diario (MIBGAS):** repetir el valor del día en sus 24 horas. No recoge variaciones dentro del día.
- **Mecanismo ibérico de tope al gas (mediados de 2022 – finales de 2023):** altera la relación entre el precio del gas y el de la luz en ese periodo; se tendrá en cuenta al interpretar los resultados.

## Pendiente
- Licencias y condiciones de uso de cada fuente (citarlas en el README).
- Comprobar en la primera descarga si cada marca de tiempo indica el inicio o el final del periodo, y en qué zona horaria viene.