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
| Comprehensive | Sí | Desde el 1 de octubre de 2025 cambia la frecuencia de publicación de los precios (de 24 a 96 periodos al día), por lo que hay que tratarlos para que coincidan con el resto de datos. |
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
 
## Limitaciones y cómo se tratan
- **REData no sirve datos horarios (comprobado el 24/11/2026):** `demanda/evolucion` y `generacion/estructura-generacion` devuelven error 400 ("Los datos solicitados no están disponibles") con `time_trunc=hour`, también con el filtro peninsular (`geo_ids=8741`). Con `time_trunc=day` funcionan. **Decisión:** usar ESIOS (token solicitado el 24/11/2026) como fuente de demanda y generación horarias; REData queda para datos diarios y comprobaciones.
- **Datos provisionales (REData):** anotar la fecha de descarga y volver a descargar los meses recientes antes del análisis final.
- **Cambio a 15 minutos (OMIE, desde 01/10/2025):** agregar a horas con la media de los cuatro cuartos y comprobar que cada hora tiene exactamente cuatro valores.
- **Precio del gas diario (MIBGAS):** repetir el valor del día en sus 24 horas. No recoge variaciones dentro del día.
- **Mecanismo ibérico de tope al gas (mediados de 2022 – finales de 2023):** altera la relación entre el precio del gas y el de la luz en ese periodo; se tendrá en cuenta al interpretar los resultados.
## Pendiente
- Licencias y condiciones de uso de cada fuente (citarlas en el README).
- Comprobar en la primera descarga si cada marca de tiempo indica el inicio o el final del periodo, y en qué zona horaria viene.
 