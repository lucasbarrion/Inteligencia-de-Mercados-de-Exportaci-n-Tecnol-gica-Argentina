# Inteligencia-de-Mercados-de-Exportaci-n-Tecnol-gica-Argentina


Análisis de comercio exterior argentino con foco en sectores de intensidad
tecnológica, construido como caso de consultoría para una empresa ficticia
de exportación de base tecnológica ("Andes Tech Export"), usando datos
reales y abiertos del gobierno argentino (2013-2024).

El problema de negocio

Andes Tech Export es una pyme argentina de servicios y productos de
base tecnológica que evalúa su estrategia de expansión internacional.
La Gerencia Comercial necesita decidir en qué mercados y sectores de
intensidad tecnológica enfocar sus esfuerzos de exportación — hoy alterna
entre oportunismo (ferias, contactos puntuales) y poca data dura sobre
tendencias reales de comercio exterior argentino en tecnología.

Stakeholders


Gerente Comercial/Exportaciones
Dirección General
Área de Producto/I+D


Objetivos de negocio


Identificar sectores de intensidad tecnológica con mejor desempeño
exportador argentino en los últimos años.
Detectar mercados/países con crecimiento sostenido de demanda en
esos sectores.
Entender si Argentina crece por precio o por volumen en sus
exportaciones tecnológicas.
Dar una base objetiva de datos para decidir dónde invertir esfuerzo
comercial.


Preguntas de negocio


¿Cómo evolucionaron las exportaciones argentinas por nivel de
intensidad tecnológica (alta/media/baja) en los últimos años?
¿Qué países/regiones concentran la demanda de productos argentinos
de mayor intensidad tecnológica?
¿El crecimiento de las exportaciones tecnológicas se explica por
mayor volumen o por mejores precios?
¿Cómo se compara Argentina con la tendencia regional/mundial en
esos mismos sectores?
¿Hay mercados con alto crecimiento de importaciones tecnológicas
globales, pero baja presencia argentina (oportunidad)?
A nivel de comercio exterior general del país, ¿el crecimiento se
explica por mayor volumen o por mejores precios internacionales?


KPIs


Variación interanual de exportaciones por intensidad tecnológica
Participación de mercado por sector/país
Índice de precio vs. cantidad exportada
Concentración geográfica de exportaciones
Ranking de sectores por crecimiento sostenido


Fuentes de datos

Datos oficiales, abiertos y reales — sin uso de Kaggle ni datasets
sintéticos.

DatasetFuentePeríodoFilasComercio exterior por intensidad tecnológica y paísDEyEN / Subsecretaría de Ciencia y Tecnología, en base a INDEC2013-2024~14.159Índices de valor, precio y cantidad de exportacionesSubsecretaría de Programación Macroeconómica, INDEC1990-202536Índices de valor, precio y cantidad de importacionesSubsecretaría de Programación Macroeconómica, INDEC1990-202536

Licencia: Creative Commons Attribution 4.0.

Clasificación de intensidad tecnológica (OCDE/Lall)

NivelSectores incluidos (ejemplos)Alta TecnologíaAeroespacial, Farmacéutica, Computadoras y máquinas de oficina, Electrónica y comunicaciones, Instrumentos científicosMedia Alta TecnologíaMaquinaria eléctrica, Vehículos a motor, Químicos (excl. farmacéuticos), Otros equipos de transporte, Maquinaria no eléctricaMedia Baja TecnologíaCombustibles/petróleo refinado, Productos de goma y plástico, Minerales no metálicos, Construcción de barcos, Metales básicos, Productos fabricados en metalBaja TecnologíaManufactura y reciclaje, Madera/Pulpa/Papel/Impresión, Alimentos/Bebidas/Tabaco, Textil y Prendas de vestir

Stack técnico


Snowflake (SQL): limpieza, transformación y análisis exploratorio
Power BI: dashboard final
GitHub: documentación y publicación


Estructura del repositorio


README.md — este archivo: contexto de negocio y objetivos
docs/limpieza_datos.md — tabla de calidad de datos, problemas
encontrados y decisiones tomadas
docs/hallazgos_eda.md — preguntas de negocio respondidas con
queries SQL y hallazgos
sql/ — queries de limpieza y análisis (a agregar)


Limitaciones del análisis


El dataset de intensidad tecnológica tiene mayor rezago de
publicación que las cifras generales de comercio exterior de INDEC
(2024 es el último año disponible en esta fuente específica).
Los índices de valor/precio/cantidad usan una clasificación sectorial
distinta (tipo de uso económico) a la de intensidad tecnológica
(OCDE/Lall) — se presentan como secciones complementarias del
dashboard, no cruzadas entre sí.
Datos agregados a nivel país, no por empresa — no permiten identificar
compradores/clientes específicos.
