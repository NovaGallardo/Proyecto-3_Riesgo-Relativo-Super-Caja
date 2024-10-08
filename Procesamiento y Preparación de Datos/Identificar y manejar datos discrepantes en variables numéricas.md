# Identificar y manejar datos discrepantes en variables numéricas

### Identificación de outliers de edad

<aside>
💡 Se utiliza el Z-Score ya que Age tiene una distribución normal

</aside>

```sql
WITH stats AS (
  SELECT
    AVG(age) AS mean_age,
    STDDEV(age) AS stddev_age
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
  WHERE age IS NOT NULL
),
z_scores AS (
  SELECT
    *,
    (age - (SELECT mean_age FROM stats)) / (SELECT stddev_age FROM stats) AS z_score
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
),
flagged AS (
  SELECT
    *,
    CASE
      WHEN z_score > 2 OR z_score < -2 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_flag_age
  FROM z_scores
)
SELECT *
FROM flagged
WHERE outlier_flag_age = 'Outlier';

```

![Screenshot 2024-08-13 at 13.19.32.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/25c85b3e-58a5-48f0-badd-b6b8ccc0fd03/Screenshot_2024-08-13_at_13.19.32.png)

![Screenshot 2024-08-13 at 13.20.31.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/d487fb7a-a2bc-4cec-835c-1df08ccdd841/Screenshot_2024-08-13_at_13.20.31.png)

### Identificación de outliers de last_month_salary

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(last_month_salary, 0.01) OVER() AS P1,
    PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
  WHERE last_month_salary IS NOT NULL
),
outliers AS (
  SELECT 
    *,
    CASE 
      WHEN last_month_salary < P1 THEN 'Outlier'
      WHEN last_month_salary > P99 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_Flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`, (SELECT DISTINCT P1, P99 FROM Percentiles)
  WHERE last_month_salary IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'Outlier'
```

<aside>
💡 Se identificaron 289 outliers en esta categoría

</aside>

![Screenshot 2024-07-29 at 23.09.12.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/d299d193-f970-4d3c-9f81-c51d97afbe63/Screenshot_2024-07-29_at_23.09.12.png)

### Winsorización  o capping de datos

<aside>
💡 Ajustar el capping del P1 al P2, ya que el salario no puede ser equivalente a 1

</aside>

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
    PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
  WHERE last_month_salary IS NOT NULL
),

winzorized AS (
  SELECT
  *,
  CASE
    WHEN last_month_salary < P2 THEN P2
    WHEN last_month_salary > P99 THEN P99
    ELSE last_month_salary
  END AS winzorized_salary
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
)

SELECT
*
FROM winzorized

```

![Screenshot 2024-07-30 at 00.30.13.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/96dd2313-f1e1-402d-8301-455e8b6ad791/Screenshot_2024-07-30_at_00.30.13.png)

Obtener nueva mediana de last_month_salary luego del capping

```sql
WITH OriginalMedian AS (
  SELECT
    PERCENTILE_CONT(last_month_salary, 0.5) OVER() AS original_median
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
  WHERE last_month_salary IS NOT NULL
),
percentiles AS (
  SELECT
    PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
    PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
  WHERE last_month_salary IS NOT NULL
),

winsorized AS (
  SELECT
  *,
  CASE
    WHEN last_month_salary < P2 THEN P2
    WHEN last_month_salary > P99 THEN P99
    ELSE last_month_salary
  END AS winsorized_salary
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
),
MedianWinsorized AS (
  SELECT
    PERCENTILE_CONT(winsorized_salary, 0.5) OVER() AS winsorized_median
  FROM winsorized
)

SELECT DISTINCT 
  (SELECT P2 FROM Percentiles LIMIT 1) AS P2,
  (SELECT P99 FROM Percentiles LIMIT 1) AS P99,
  (SELECT original_median FROM OriginalMedian LIMIT 1) AS original_median,
  (SELECT winsorized_median FROM MedianWinsorized LIMIT 1) AS winsorized_median
FROM percentiles
```

![Screenshot 2024-07-30 at 00.33.58.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/6254f58d-1e4f-4c80-94af-4756cb1fa1cb/Screenshot_2024-07-30_at_00.33.58.png)

<aside>
💡 El hecho de que la mediana no cambie después de aplicar el capping confirma que esta estrategia es adecuada para manejar los outliers en `last_month_salary`. Esto sugiere que los outliers no están influyendo significativamente en la tendencia central de la distribución.

</aside>

### Imputación de nulos

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_clean` AS

WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(last_month_salary, 0.02) OVER() AS P2,
    PERCENTILE_CONT(last_month_salary, 0.99) OVER() AS P99
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
  WHERE last_month_salary IS NOT NULL
),

winsorized AS (
  SELECT
  *,
  CASE
    WHEN last_month_salary < P2 THEN P2
    WHEN last_month_salary > P99 THEN P99
    ELSE last_month_salary
  END AS winsorized_salary
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`, (SELECT DISTINCT P2, P99 FROM percentiles)
)

SELECT
user_id,
age,
sex,
CASE
  WHEN winsorized_salary IS NULL THEN 5400
  ELSE winsorized_salary
END AS last_month_salary_clean,
  
CASE
  WHEN number_dependents IS NULL THEN 0
  ELSE number_dependents
END AS number_dependents_clean

FROM winsorized
```

### Identificación de outliers de more_90_days_overdue

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(more_90_days_overdue, 0.01) OVER() AS P1,
    PERCENTILE_CONT(more_90_days_overdue, 0.998) OVER() AS P998
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE more_90_days_overdue IS NOT NULL
),
outliers AS (
  SELECT 
    *,
    CASE 
      WHEN more_90_days_overdue < P1 THEN 'Outlier'
      WHEN more_90_days_overdue > P998 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_Flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`, (SELECT DISTINCT P1, P998 FROM Percentiles)
  WHERE more_90_days_overdue IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'Outlier'
```

![Screenshot 2024-08-07 at 10.53.41.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/db7476f7-847d-4475-a14f-6c2cde3461d1/Screenshot_2024-08-07_at_10.53.41.png)

### Identificación de number_times_delayed_payment_loan_30_59_days

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(number_times_delayed_payment_loan_30_59_days, 0.01) OVER() AS P1,
    PERCENTILE_CONT(number_times_delayed_payment_loan_30_59_days, 0.998) OVER() AS P998
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE number_times_delayed_payment_loan_30_59_days IS NOT NULL
),
outliers AS (
  SELECT 
    *,
    CASE 
      WHEN number_times_delayed_payment_loan_30_59_days < P1 THEN 'Outlier'
      WHEN number_times_delayed_payment_loan_30_59_days > P998 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_Flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`, (SELECT DISTINCT P1, P998 FROM percentiles)
  WHERE number_times_delayed_payment_loan_30_59_days IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'Outlier'
```

![Screenshot 2024-08-07 at 12.10.13.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/0fb539da-d428-4a9b-ad9c-0251a87779c3/Screenshot_2024-08-07_at_12.10.13.png)

### Identificación de number_times_delayed_payment_loan_60_89_days

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(number_times_delayed_payment_loan_60_89_days, 0.01) OVER() AS P1,
    PERCENTILE_CONT(number_times_delayed_payment_loan_60_89_days, 0.998) OVER() AS P998
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE number_times_delayed_payment_loan_60_89_days IS NOT NULL
),
outliers AS (
  SELECT 
    *,
    CASE 
      WHEN number_times_delayed_payment_loan_60_89_days < P1 THEN 'Outlier'
      WHEN number_times_delayed_payment_loan_60_89_days > P998 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_Flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`, (SELECT DISTINCT P1, P998 FROM percentiles)
  WHERE number_times_delayed_payment_loan_60_89_days IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'Outlier'
```

![Screenshot 2024-08-07 at 12.22.10.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/c68a0ea5-e244-4a18-a160-891a46b96269/Screenshot_2024-08-07_at_12.22.10.png)

### Identificación de using_lines_not_secured_personal_assets

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(using_lines_not_secured_personal_assets, 0.01) OVER() AS P1,
    PERCENTILE_CONT(using_lines_not_secured_personal_assets, 0.998) OVER() AS P998
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE using_lines_not_secured_personal_assets IS NOT NULL
),
outliers AS (
  SELECT 
    *,
    CASE 
      WHEN using_lines_not_secured_personal_assets < P1 THEN 'Outlier'
      WHEN using_lines_not_secured_personal_assets > P998 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_Flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`, (SELECT DISTINCT P1, P998 FROM percentiles)
  WHERE using_lines_not_secured_personal_assets IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'Outlier'
```

![Screenshot 2024-08-07 at 12.16.18.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/81046ed5-dd32-4450-91f7-f0811f632f80/Screenshot_2024-08-07_at_12.16.18.png)

### Identificación de using_lines_not_secured_personal_assets

```sql
WITH percentiles AS (
  SELECT
    PERCENTILE_CONT(debt_ratio, 0.01) OVER() AS P1,
    PERCENTILE_CONT(debt_ratio, 0.998) OVER() AS P998
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE debt_ratio IS NOT NULL
),
outliers AS (
  SELECT 
    *,
    CASE 
      WHEN debt_ratio < P1 THEN 'Outlier'
      WHEN debt_ratio > P998 THEN 'Outlier'
      ELSE 'Within'
    END AS outlier_Flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`, (SELECT DISTINCT P1, P998 FROM percentiles)
  WHERE debt_ratio IS NOT NULL
)
SELECT *
FROM outliers
WHERE outlier_flag = 'Outlier'
```

![Screenshot 2024-08-07 at 12.19.04.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ebb9ff67-9568-4153-82ed-51e9dbc8b9ad/7d7458da-d683-4c4d-bad5-f9b8ef00a0f9/Screenshot_2024-08-07_at_12.19.04.png)

### Imputación de los nulos

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail_winsorized` AS

WITH percentiles_more_90_days_overdue AS (
  SELECT
    PERCENTILE_CONT(more_90_days_overdue, 0.01) OVER() AS P1_90_days_overdue,
    PERCENTILE_CONT(more_90_days_overdue, 0.998) OVER() AS P998_90_days_overdue
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE more_90_days_overdue IS NOT NULL
),
percentiles_30_59_days AS (
  SELECT
    PERCENTILE_CONT(number_times_delayed_payment_loan_30_59_days, 0.01) OVER() AS P1_30_59_days,
    PERCENTILE_CONT(number_times_delayed_payment_loan_30_59_days, 0.998) OVER() AS P998_30_59_days
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE number_times_delayed_payment_loan_30_59_days IS NOT NULL
),
percentiles_60_89_days AS (
  SELECT
    PERCENTILE_CONT(number_times_delayed_payment_loan_60_89_days, 0.01) OVER() AS P1_60_89_days,
    PERCENTILE_CONT(number_times_delayed_payment_loan_60_89_days, 0.998) OVER() AS P998_60_89_days
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE number_times_delayed_payment_loan_60_89_days IS NOT NULL
),
percentiles_using_lines_not_secured AS (
  SELECT
    PERCENTILE_CONT(using_lines_not_secured_personal_assets, 0.01) OVER() AS P1_using_lines_not_secured,
    PERCENTILE_CONT(using_lines_not_secured_personal_assets, 0.998) OVER() AS P998_using_lines_not_secured
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE using_lines_not_secured_personal_assets IS NOT NULL
),
percentiles_debt_ratio AS (
  SELECT
    PERCENTILE_CONT(debt_ratio, 0.01) OVER() AS P1_debt_ratio,
    PERCENTILE_CONT(debt_ratio, 0.998) OVER() AS P998_debt_ratio
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
  WHERE debt_ratio IS NOT NULL
)

SELECT
  user_id,
  CASE
    WHEN SAFE_CAST(more_90_days_overdue AS int64) < P1_90_days_overdue THEN SAFE_CAST(P1_90_days_overdue AS int64)
    WHEN SAFE_CAST(more_90_days_overdue AS int64) > P998_90_days_overdue THEN SAFE_CAST(P998_90_days_overdue AS int64)
    ELSE more_90_days_overdue
  END AS more_90_days_overdue,
  
  CASE
    WHEN number_times_delayed_payment_loan_30_59_days < P1_30_59_days THEN P1_30_59_days
    WHEN number_times_delayed_payment_loan_30_59_days > P998_30_59_days THEN P998_30_59_days
    ELSE number_times_delayed_payment_loan_30_59_days
  END AS number_times_delayed_payment_loan_30_59_days,
  
  CASE
    WHEN number_times_delayed_payment_loan_60_89_days < P1_60_89_days THEN P1_60_89_days
    WHEN number_times_delayed_payment_loan_60_89_days > P998_60_89_days THEN P998_60_89_days
    ELSE number_times_delayed_payment_loan_60_89_days
  END AS number_times_delayed_payment_loan_60_89_days,
  
  CASE
    WHEN using_lines_not_secured_personal_assets < P1_using_lines_not_secured THEN P1_using_lines_not_secured
    WHEN using_lines_not_secured_personal_assets > P998_using_lines_not_secured THEN P998_using_lines_not_secured
    ELSE using_lines_not_secured_personal_assets
  END AS using_lines_not_secured_personal_assets,
  
  CASE
    WHEN debt_ratio < P1_debt_ratio THEN P1_debt_ratio
    WHEN debt_ratio > P998_debt_ratio THEN P998_debt_ratio
    ELSE debt_ratio
  END AS debt_ratio
  
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
CROSS JOIN (SELECT DISTINCT P1_90_days_overdue, P998_90_days_overdue FROM percentiles_more_90_days_overdue)
CROSS JOIN (SELECT DISTINCT P1_30_59_days, P998_30_59_days FROM percentiles_30_59_days)
CROSS JOIN (SELECT DISTINCT P1_60_89_days, P998_60_89_days FROM percentiles_60_89_days)
CROSS JOIN (SELECT DISTINCT P1_using_lines_not_secured, P998_using_lines_not_secured FROM percentiles_using_lines_not_secured)
CROSS JOIN (SELECT DISTINCT P1_debt_ratio, P998_debt_ratio FROM percentiles_debt_ratio)
```
