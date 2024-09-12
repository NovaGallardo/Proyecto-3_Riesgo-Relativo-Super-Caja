# Identificar y manejar datos discrepantes en variables categóricas

## Estandarizar  loan_type de loans_outstanding

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
