RappiPlus análisis de desempeño

Análisis integral del desempeño del servicio **RappiPlus**, combinando limpieza de datos, KPIs de rentabilidad, un funnel de conversión, retención por cohortes y un experimento A/B, con el objetivo de apoyar decisiones de negocio basadas en datos.

## Fuentes de datos

| Fuente | Contenido |
|---|---|
| `rappiplus_orders_raw.csv` | Pedidos, precios, descuentos e ingreso |
| `rappiplus_catalog.csv` | Costos de productos, categorías y proveedores |
| `rappiplus_marketing_spend.csv` | Inversión en marketing por canal y país |
| `events` / `users` / `user_activity` (SQL) | Comportamiento del usuario en la plataforma |
| `experiment_checkout_ui.csv` | Resultados del experimento A/B en el checkout |

## Estructura del proyecto

**Paso 1 — Calidad de datos**
Limpieza de las tres tablas de Python (`orders`, `catalog`, `marketing`): conversión de tipos, estandarización de texto (incluyendo acentos), eliminación de duplicados, tratamiento de valores inválidos y outliers (winsorización), e imputación de nulos deduciendo valores desde otras columnas siempre que fue posible, antes de recurrir a mediana/moda o a `'Desconocido'`.

**Paso 2 — Rentabilidad del negocio**
Cálculo de ingreso, costo (vía `merge` con el catálogo), margen bruto y profit neto (restando marketing), ticket promedio y producto más vendido.

**Paso 3 — Funnel de conversión**
Consulta SQL que mide la conversión de cada paso del funnel (`first_visit` → `purchase`) contra el primer evento, identificando el punto de mayor fricción.

**Paso 4 — Retención por cohortes**
Consulta SQL que agrupa usuarios por mes de registro y mide su retención en las semanas 1, 2 y 3 posteriores.

**Paso 5 — Prueba A/B**
Prueba z sobre proporciones para evaluar si un nuevo diseño del checkout mejora la tasa de conversión frente al diseño actual.

**Paso 6 — Dashboard**
Visualización de los resultados en Power BI (fuera del alcance de este notebook).

## Principales hallazgos

| Área | Resultado |
|---|---|
| Ingreso total | $9,627,993.65 |
| Costo total | $3,834,482.61 |
| Margen bruto | 60.2% |
| Profit neto (tras marketing) | 30.3% |
| Ticket promedio | $385.95 |
| Conversión final del funnel | 80.04% |
| Mayor caída del funnel | `begin_checkout` → `add_payment_info` (-12.29 pp) |
| Retención por cohorte | Estable entre 40% y 44% en las 5 cohortes y las 3 semanas |
| Prueba A/B (checkout) | Sin diferencia significativa (p = 0.416); lift de +0.60 pp por debajo del mínimo detectable (~2 pp) |

**Conclusión:** RappiPlus es un negocio rentable, con un funnel sólido y una retención estable. El mayor punto de oportunidad es consistente entre dos secciones distintas del análisis: el checkout, tanto por ser la mayor caída del funnel como por ser la etapa evaluada en la prueba A/B.

## Recomendaciones accionables

1. **Investigar la fricción en el checkout** (`begin_checkout` → `add_payment_info`), el mayor punto de pérdida de todo el funnel.
2. **Revisar el margen por producto, no solo el ingreso.** Sneakers-Urban-42 y Jacket-Winter-M generan un ingreso similar (~$520K cada uno), pero Sneakers convierte ~93% de ese ingreso en profit frente a solo ~27% de Jacket-Winter-M, por su costo unitario más alto.
3. **Ampliar la muestra del experimento A/B** antes de descartar el nuevo diseño del checkout: con el tamaño de muestra actual, el experimento solo detecta de forma confiable diferencias de 2 puntos porcentuales o más, y el lift observado (0.60 pp) está por debajo de ese umbral.

## Cómo ejecutar

1. Instalar dependencias: `pandas`, `numpy`, `matplotlib`, `seaborn`, `sqlalchemy` (u otro conector a la base de datos usada para las consultas SQL de los Pasos 3 y 4).
2. Colocar los archivos CSV en el directorio esperado por el notebook.
3. Configurar la conexión a la base de datos para las consultas SQL (tablas `events`, `users`, `user_activity`).
4. Ejecutar las celdas en orden (Kernel → Restart & Run All).
