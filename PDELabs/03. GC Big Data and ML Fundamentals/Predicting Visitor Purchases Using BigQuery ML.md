
Los objetivos de este laboratorio serán:
- Buscar y usar los datasets públicos de BigQuery
- Consultar y explorar el dataset _ecommerce_
- Crear un dataset de entrenamiento y validación para la predicción por lotes.
- Crear un modelo de regresión logística para clasificación en BigQuery ML
- Evaluar el funcionamiento de nuestro modelo de ML
- Predecir y hacer un ranking de los visitantes que harán una compra según sus probabilidades

# 1) Explorar el dataset

Esta sentencia define la variable visitors como el número total de visitantes diferentes a la web y purchasers como el número de visitors que finalmente compran algo para calcular el ratio de compra por visitante.

```sql
WITH visitors AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_visitors
FROM `data-to-insights.ecommerce.web_analytics`
),

purchasers AS(
SELECT
COUNT(DISTINCT fullVisitorId) AS total_purchasers
FROM `data-to-insights.ecommerce.web_analytics`
WHERE totals.transactions IS NOT NULL
)

SELECT
  total_visitors,
  total_purchasers,
  total_purchasers / total_visitors AS conversion_rate
FROM visitors, purchasers
```

La siguiente sentencia selecciona los 5 productos más vendidos.
```sql
SELECT
  p.v2ProductName,
  p.v2ProductCategory,
  SUM(p.productQuantity) AS units_sold,
  ROUND(SUM(p.localProductRevenue/1000000),2) AS revenue
FROM `data-to-insights.ecommerce.web_analytics`,
UNNEST(hits) AS h,
UNNEST(h.product) AS p
GROUP BY 1, 2
ORDER BY revenue DESC
LIMIT 5;
```

**¿Qué hace el UNNEST?** 
Esta tabla parece contener datos anidados, donde hay un array de objetos llamado "hits", y dentro de cada objeto "hit" hay un array de productos.
Cuando utilizas UNNEST(hits) AS h y UNNEST(h.product) AS p, estás descomponiendo estos arrays anidados en filas separadas. Esto permite que cada fila de la tabla resultante represente un solo producto vendido, en lugar de un array de productos.

Y por último esta sentencia selecciona a los visitantes que compraron y han vuelto a hacerlo.

```sql
WITH all_visitor_stats AS (
SELECT
  fullvisitorid, # 741,721 unique visitors
  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit
  FROM `data-to-insights.ecommerce.web_analytics`
  GROUP BY fullvisitorid
)

SELECT
  COUNT(DISTINCT fullvisitorid) AS total_visitors,
  will_buy_on_return_visit
FROM all_visitor_stats
GROUP BY will_buy_on_return_visit
```

__Seguir esta tarde__