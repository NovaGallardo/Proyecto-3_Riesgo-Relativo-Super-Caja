# Identificar y manejar valores duplicados

## Identificar valores duplicados en “default”

```sql
SELECT user_id
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.default`
GROUP BY user_id
HAVING COUNT (*) > 1
```

<aside>
💡 No se encontraron valores duplicados

</aside>

## Identificar valores duplicados en “loans_detail”

```sql
SELECT user_id
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
GROUP BY user_id
HAVING COUNT (*) > 1
```

<aside>
💡 No se encontraron duplicados.

</aside>

## Identificar valores duplicados en “user_info_clean”

```sql
SELECT loan_id
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_clean`
GROUP BY loan_id
HAVING COUNT (*) > 1
```

<aside>
💡 No se encontraron duplicados.

</aside>

## Identificar valores duplicados en “loans_outstanding”

```sql
SELECT loan_id
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding`
GROUP BY loan_id
HAVING COUNT (*) > 1
```

<aside>
💡 No se encontraron duplicados.

</aside>

```sql
SELECT 
  user_id, 
  COUNT(*) AS veces_repetido
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding`
GROUP BY user_id
```

<aside>
💡 Se encontraron que 35575 user_ids tienen valores repetidos

</aside>

Limpieza de los users repetidos

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding_clean` AS

WITH NormalizedLoans AS (
  SELECT *,
    CASE
      WHEN LOWER(loan_type) = 'others' THEN 'other'
      ELSE LOWER(loan_type)
    END AS loan_type_normalized
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding`
)

SELECT
  user_id,
  COUNT(*) AS total_loans,
  COUNT(CASE WHEN loan_type_normalized = 'real estate' THEN 1 END) AS real_estate_loans,
  COUNT(CASE WHEN loan_type_normalized = 'other' THEN 1 END) AS other_loans
FROM NormalizedLoans
GROUP BY user_id
```

<aside>
💡 Se eliminó la columna loan_id y se contó la cantidad de loans por user_id y el tipo de loan. De 305335 filas se pasaron a 35575

</aside>
