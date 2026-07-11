# Limpieza de Datos

Todas las transformaciones se realizaron en SQL, directamente en
Snowflake, sobre las tablas crudas cargadas en el schema `RAW` de
`COMEX_TECH_DB`.

## Tabla de calidad de datos

| # | Tabla afectada | Problema detectado | Impacto | Decisión tomada | Justificación |
|---|---|---|---|---|---|
| 1 | `COMEX_SECTORES_PAISES` | Columna `VALOR` con formato numérico argentino: coma como separador decimal y punto como separador de miles (ej. `'2.346,49'`) | La carga con tipo `FLOAT` fallaba (`Numeric value not recognized`); una primera conversión ingenua (`REPLACE(',', '.')`) generaba números inválidos con doble punto en valores ≥ 1.000 | Cargar `VALOR` como texto, y convertir con `TRY_CAST(REPLACE(REPLACE(VALOR, '.', ''), ',', '.') AS FLOAT)` (primero se quitan los puntos de miles, luego se reemplaza la coma decimal) | Snowflake espera punto decimal simple; había que resolver ambos separadores en el orden correcto para no perder ni corromper valores grandes |
| 2 | `COMEX_SECTORES_PAISES` | Encoding corrupto (carácter `�`) en 56 nombres de país y en la columna `TIPO_SECTOR` (ej. `Canad�`, `Espa�a`, `tecnol�gica`) | Nombres de país y categorías ilegibles/incorrectos para el análisis y el dashboard | Corrección manual país por país con `UPDATE ... CASE WHEN`, mapeando cada valor corrupto a su forma correcta | El carácter de reemplazo indica pérdida irreversible del dato original; no existe una fórmula automática, se corrigió con conocimiento del nombre real de cada país |
| 3 | `COMEX_SECTORES_PAISES` | Error de tipeo en la fuente original: `'Angora'` en lugar de `'Angola'` | País inexistente en los datos | Corregido con `UPDATE` puntual | Angora no es un país; es un error de carga en la fuente oficial |
| 4 | `COMEX_SECTORES_PAISES` | Nulos | — | 0 encontrados | No requirió acción |
| 5 | `COMEX_SECTORES_PAISES` | Duplicados exactos | — | 0 encontrados | No requirió acción |
| 6 | `COMEX_SECTORES_PAISES` | Completitud de años 2013-2024 | — | Todos los años presentes, entre 1.100 y 1.213 filas por año, sin vacíos significativos | Verificado con `GROUP BY AÑO` |
| 7 | `EXPORT_INDICES`, `IMPORT_INDICES` | Columnas numéricas cargadas con precisión excesiva e inconsistente (`NUMBER(38,14)`, `NUMBER(38,6)`, etc.), incluyendo artefactos de coma flotante en años recientes (ej. `104.58635699999999` en vez de `104.586357`) | Valores difíciles de leer e inconsistentes entre años | Recreación de la tabla (`CREATE OR REPLACE TABLE ... AS SELECT`) con `ROUND(columna, 6)` en todas las columnas numéricas | Se confirmó que 6 decimales es la máxima precisión real presente en la fuente (INDEC amplió la precisión reportada a partir de 2015); redondear a un número de decimales igual o mayor al máximo real no pierde información, solo agrega ceros de relleno en años con menor precisión original |
| 8 | Las 3 tablas | Columnas con nombres largos y en formato técnico (ej. `OP_EXPO_IMPO`, `ICA_INDICE_VALOR_NIVEL_GENERAL`) | Dificultad de lectura y mantenimiento de las queries | Renombrado a nombres cortos y descriptivos en español (ej. `TIPO_OPERACION`, `VALOR_GENERAL`) | Mejora la legibilidad del esquema y de las queries de análisis |

## Nota sobre nombres de columna con tilde

La columna `AÑO` se mantuvo con tilde por decisión de estilo. Esto
requiere referenciarla entre comillas dobles en cualquier query
(`"AÑO"`), ya que Snowflake no reconoce caracteres especiales en
identificadores sin comillas.

## Nota sobre nomenclatura de países

Se verificó que la mayoría de los valores que a primera vista parecen
"errores" en la columna de país (zonas francas, ciudades puntuales,
categorías como "Indeterminado (América)", nombres oficiales distintos
a los coloquiales como "Corea Republicana") son en realidad categorías
válidas del sistema de nomenclatura aduanera oficial argentino
(INDEC/AFIP), y se mantuvieron sin modificar.
