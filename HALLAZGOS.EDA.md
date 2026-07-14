# Hallazgos del Análisis Exploratorio (EDA)

Cada sección responde una de las preguntas de negocio definidas en el
README, con su query SQL y el hallazgo correspondiente. Todas las
queries corren sobre las tablas ya limpias en el schema `RAW` de
`COMEX_TECH_DB` (ver `limpieza_datos.md`).

---

## Pregunta 1: ¿Cómo evolucionaron las exportaciones argentinas por nivel de intensidad tecnológica en los últimos años?

```sql
SELECT
    "AÑO",
    SECTOR_INTENSIDAD_TECNOLOGICA,
    SUM(VALOR) AS valor_total_exportado
FROM COMEX_TECH_DB.RAW.COMEX_SECTORES_PAISES
WHERE TIPO_OPERACION = 'Exportaciones'
GROUP BY "AÑO", SECTOR_INTENSIDAD_TECNOLOGICA
ORDER BY "AÑO", SECTOR_INTENSIDAD_TECNOLOGICA;
```

- Argentina exporta abrumadoramente productos de **Baja Tecnología**:
  en 2024, este segmento representó ~USD 36.376 millones, mientras que
  **Alta Tecnología** apenas alcanzó ~USD 1.590 millones — 22 veces
  menos.
- La brecha se **agrandó** en el período: Alta Tecnología cayó 39,1%
  entre 2013 y 2024, mientras que Baja Tecnología creció 13,7%.
- Caída marcada en 2020 (efecto COVID) en Media Alta Tecnología, con
  recuperación posterior; Baja Tecnología fue el segmento más
  resiliente ese año.

**Qué significa esto:** el perfil exportador tecnológico argentino es
chico y con tendencia de caída relativa. Puede leerse como señal de
alerta estructural o como oportunidad de nicho poco explotado, dado el
bajo número de jugadores relevantes en el segmento.

---

## Pregunta 2: ¿Qué países concentran la demanda de exportaciones argentinas de Alta Tecnología?

```sql
SELECT
    PAIS_DESTINO_ORIGEN,
    SUM(VALOR) AS valor_total_exportado
FROM COMEX_TECH_DB.RAW.COMEX_SECTORES_PAISES
WHERE TIPO_OPERACION = 'Exportaciones'
  AND SECTOR_INTENSIDAD_TECNOLOGICA = 'ALTA TECNOLOGIA'
GROUP BY PAIS_DESTINO_ORIGEN
ORDER BY valor_total_exportado DESC
LIMIT 15;
```

Top 5:

| País | Valor total (USD M, 2013-2024) |
|---|---|
| Brasil | 2.654 |
| Estados Unidos | 2.554 |
| Uruguay | 1.764 |
| República Federal de Alemania | 1.394 |
| Chile | 982 |

**Qué significa esto:** concentración regional fuerte — 9 de los 15
principales compradores son países de América Latina. Brasil y
Estados Unidos lideran con una diferencia de apenas 4% entre ellos.
Uruguay, sumando el país y la zona franca "Zonamérica", se acerca al
podio (USD 2.225M combinado). Europa tiene solo 3 representantes en el
top 15 (Alemania, España, Francia). Estrategia sugerida: consolidar
base regional probada mientras se explora margen de crecimiento en
EE.UU. y Alemania.

---

## Pregunta 3: ¿El crecimiento de las exportaciones se explica por precio o por volumen?

```sql
SELECT
    "AÑO",
    VALOR_GENERAL,
    PRECIO_GENERAL,
    CANTIDAD_GENERAL
FROM COMEX_TECH_DB.RAW.EXPORT_INDICES
WHERE "AÑO" IN (2013, 2024)
ORDER BY "AÑO";
```

Ajuste por inflación en dólares (deflactado con CPI de EE.UU., fuente
FRED — serie CPIAUCNS, promedio anual):

```sql
SELECT
    e."AÑO",
    e.PRECIO_GENERAL AS precio_nominal,
    c.CPI_PROMEDIO_ANUAL AS cpi,
    ROUND(e.PRECIO_GENERAL * (base.cpi_base / c.CPI_PROMEDIO_ANUAL), 3) AS precio_real_ajustado
FROM COMEX_TECH_DB.RAW.EXPORT_INDICES e
JOIN COMEX_TECH_DB.RAW.CPI_EEUU c ON e."AÑO" = c."AÑO"
CROSS JOIN (
    SELECT CPI_PROMEDIO_ANUAL AS cpi_base
    FROM COMEX_TECH_DB.RAW.CPI_EEUU
    WHERE "AÑO" = 2013
) base
WHERE e."AÑO" IN (2013, 2024)
ORDER BY e."AÑO";
```

| Índice | 2013 | 2024 | Variación % nominal |
|---|---|---|---|
| Valor | 219,700 | 230,520 | +4,9% |
| Precio | 190,500 | 182,267 | -4,3% |
| Cantidad | 115,300 | 126,473 | +9,7% |

| | Precio 2013 | Precio 2024 | Variación % |
|---|---|---|---|
| Nominal (sin ajustar) | 190,500 | 182,267 | -4,3% |
| **Real (ajustado por inflación en USD)** | 190,500 | 135,358 | **-29,0%** |

**Qué significa esto:** el crecimiento de las exportaciones argentinas
se explicó enteramente por mayor volumen, no por precio — los precios
internacionales cayeron. En términos nominales la caída de precio
parece moderada (-4,3%), pero ajustada por inflación en dólares la
caída real fue de casi 30%. Argentina tuvo que exportar
considerablemente más cantidad solo para compensar una pérdida real de
precio mucho mayor a la que muestran los números nominales. Depender
de mejores precios internacionales no es una estrategia viable en este
contexto; el camino de crecimiento pasa por volumen, diferenciación de
producto o nichos con menor presión de precios commoditizados.

---

## Pregunta 4: ¿Cómo se compara Argentina con la tendencia general del propio país? (benchmark interno entre sectores, 2013 → 2024)

```sql
WITH por_sector_anio AS (
    SELECT
        SECTOR_INTENSIDAD_TECNOLOGICA,
        "AÑO",
        SUM(VALOR) AS valor_total
    FROM COMEX_TECH_DB.RAW.COMEX_SECTORES_PAISES
    WHERE TIPO_OPERACION = 'Exportaciones'
      AND "AÑO" IN (2013, 2024)
    GROUP BY SECTOR_INTENSIDAD_TECNOLOGICA, "AÑO"
)
SELECT
    SECTOR_INTENSIDAD_TECNOLOGICA,
    MAX(CASE WHEN "AÑO" = 2013 THEN valor_total END) AS valor_2013,
    MAX(CASE WHEN "AÑO" = 2024 THEN valor_total END) AS valor_2024,
    ROUND(
        (MAX(CASE WHEN "AÑO" = 2024 THEN valor_total END) - MAX(CASE WHEN "AÑO" = 2013 THEN valor_total END))
        / MAX(CASE WHEN "AÑO" = 2013 THEN valor_total END) * 100, 1
    ) AS variacion_pct
FROM por_sector_anio
GROUP BY SECTOR_INTENSIDAD_TECNOLOGICA
ORDER BY variacion_pct DESC;
```

| Sector | 2013 (USD M) | 2024 (USD M) | Variación % |
|---|---|---|---|
| Media Baja Tecnología | 7.316 | 11.231 | **+53,5%** |
| Baja Tecnología | 31.988 | 36.376 | **+13,7%** |
| Media Alta Tecnología | 16.823 | 13.448 | **-20,1%** |
| Alta Tecnología | 2.612 | 1.590 | **-39,1%** |

**Qué significa esto:** relación inversa clara y sistemática — cuanto
más alta la intensidad tecnológica, peor el desempeño exportador
argentino en la década. El crecimiento exportador estuvo impulsado por
Media Baja Tecnología (combustibles, metales básicos), no por
manufactura sofisticada. Este es el hallazgo central que amarra la
narrativa del proyecto: Argentina no logró revertir su perfil
exportador de baja sofisticación tecnológica en 12 años — la brecha se
profundizó.

---

## Pregunta 5: ¿Hay mercados con crecimiento fuerte pero baja presencia argentina? (oportunidades de expansión)

```sql
WITH periodo1 AS (
    SELECT PAIS_DESTINO_ORIGEN, AVG(VALOR) AS promedio_periodo1
    FROM COMEX_TECH_DB.RAW.COMEX_SECTORES_PAISES
    WHERE TIPO_OPERACION = 'Exportaciones'
      AND SECTOR_INTENSIDAD_TECNOLOGICA = 'ALTA TECNOLOGIA'
      AND "AÑO" BETWEEN 2013 AND 2016
    GROUP BY PAIS_DESTINO_ORIGEN
),
periodo2 AS (
    SELECT PAIS_DESTINO_ORIGEN, AVG(VALOR) AS promedio_periodo2
    FROM COMEX_TECH_DB.RAW.COMEX_SECTORES_PAISES
    WHERE TIPO_OPERACION = 'Exportaciones'
      AND SECTOR_INTENSIDAD_TECNOLOGICA = 'ALTA TECNOLOGIA'
      AND "AÑO" BETWEEN 2021 AND 2024
    GROUP BY PAIS_DESTINO_ORIGEN
)
SELECT
    COALESCE(p1.PAIS_DESTINO_ORIGEN, p2.PAIS_DESTINO_ORIGEN) AS pais,
    ROUND(COALESCE(p1.promedio_periodo1, 0), 3) AS promedio_periodo1,
    ROUND(COALESCE(p2.promedio_periodo2, 0), 3) AS promedio_periodo2,
    ROUND(COALESCE(p2.promedio_periodo2, 0) - COALESCE(p1.promedio_periodo1, 0), 3) AS diferencia_absoluta
FROM periodo1 p1
FULL OUTER JOIN periodo2 p2 ON p1.PAIS_DESTINO_ORIGEN = p2.PAIS_DESTINO_ORIGEN
WHERE COALESCE(p2.promedio_periodo2, 0) < 20
ORDER BY diferencia_absoluta DESC
LIMIT 15;
```

Top 5 (excluyendo categorías administrativas del sistema aduanero):

| País | Promedio 2013-2016 (USD M) | Promedio 2021-2024 (USD M) | Diferencia |
|---|---|---|---|
| Corea Republicana | 3,4 | 11,3 | +7,9 (más que se triplicó) |
| Costa Rica | 4,5 | 9,7 | +5,2 |
| Filipinas | 2,7 | 5,9 | +3,2 |
| Honduras | 1,9 | 4,8 | +2,9 |
| Egipto | 2,7 | 5,6 | +2,9 |

**Qué significa esto:** Corea del Sur es el hallazgo más relevante —
economía tecnológicamente sofisticada que más que triplicó sus compras
a Argentina, señal de que existen productos/nichos argentinos
genuinamente competitivos a nivel internacional. Se identifica además
un clúster centroamericano (Costa Rica, Honduras, Nicaragua) y del
sudeste asiático (Filipinas, Tailandia, Indonesia) en expansión
sostenida y con menor competencia relativa que los mercados
tradicionales (Brasil, EE.UU.).

---

## Pregunta 6: ¿El comercio exterior general del país crece por precio o por volumen? (sección de importaciones)

```sql
SELECT
    "AÑO",
    VALOR_GENERAL,
    PRECIO_GENERAL,
    CANTIDAD_GENERAL
FROM COMEX_TECH_DB.RAW.IMPORT_INDICES
WHERE "AÑO" IN (2013, 2024)
ORDER BY "AÑO";
```

| Índice | 2013 | 2024 | Variación % |
|---|---|---|---|
| Valor | 331,700 | 270,772 | **-18,4%** |
| Precio | 140,900 | 134,542 | -4,5% |
| Cantidad | 235,400 | 201,255 | **-14,5%** |

**Qué significa esto:** las importaciones argentinas cayeron fuerte en
el período (-18,4% en valor), explicado sobre todo por menor volumen
(-14,5%), no por precios más bajos. Contraste relevante: mientras las
exportaciones crecieron por mayor volumen, las importaciones cayeron
por menor volumen — patrón consistente con una economía que redujo su
apertura comercial en la década, posiblemente por restricciones
cambiarias. Dato relevante para Andes Tech Export si depende de
insumos importados para producir.

---

## Resumen ejecutivo del EDA

1. El perfil exportador tecnológico argentino es chico y se achicó más
   en la última década (-39,1% en Alta Tecnología).
2. La demanda está concentrada regionalmente (Brasil, Uruguay, Chile)
   con Estados Unidos y Alemania como principales socios extra-región.
3. El crecimiento exportador general del país se explicó por volumen,
   no por precio — los precios reales cayeron ~29% en la década.
4. Las importaciones cayeron por menor apertura comercial, no por
   precios más bajos.
5. Existen mercados concretos de oportunidad con crecimiento sostenido
   y baja saturación: Corea del Sur, Centroamérica y sudeste asiático.
