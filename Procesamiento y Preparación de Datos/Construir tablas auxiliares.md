# Construir tablas auxiliares

<aside>
💡 Al unir todas las tablas, hay 425 filas con nulos debido a la tabla loans_outstanding. Se obtendrá la mediana de total_loans según cada cuartil de last_month_salary y con eso se imputarán los datos. Además, se obtienen los cuartiles de variables importantes.

</aside>

```sql
WITH salary_quartiles AS (
  SELECT
    user_id,
    last_month_salary_clean,
    NTILE(4) OVER (ORDER BY last_month_salary_clean) AS salary_quartile
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data`
),

loans_quartiles AS (
  SELECT
    user_id,
    total_loans,
    NTILE(4) OVER (ORDER BY total_loans) AS loans_quartile
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data`
),

delayed_payments_quartiles AS (
  SELECT
    user_id,
    total_delayed_payments,
    NTILE(4) OVER (ORDER BY total_delayed_payments) AS delayed_payments_quartile
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data`
),

scaled_debt_ratio_quartiles AS (
  SELECT
    user_id,
    scaled_debt_ratio,
    NTILE(4) OVER (ORDER BY scaled_debt_ratio) AS scaled_debt_ratio_quartile
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data`
),

quartile_medians AS (
  SELECT
    salary_quartile,
    PERCENTILE_CONT(total_loans, 0.5) OVER (PARTITION BY salary_quartile) AS median_total_loans
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding_clean` AS o
  JOIN salary_quartiles AS sq ON o.user_id = sq.user_id
),

main_data_with_quartiles AS (
  SELECT
    m.*,
    sq.salary_quartile,
    lq.loans_quartile,
    dpq.delayed_payments_quartile,
    sdq.scaled_debt_ratio_quartile,
    qm.median_total_loans
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data` AS m
  LEFT JOIN salary_quartiles sq ON m.user_id = sq.user_id
  LEFT JOIN loans_quartiles lq ON m.user_id = lq.user_id
  LEFT JOIN delayed_payments_quartiles dpq ON m.user_id = dpq.user_id
  LEFT JOIN scaled_debt_ratio_quartiles sdq ON m.user_id = sdq.user_id
  LEFT JOIN (
    SELECT DISTINCT salary_quartile, median_total_loans
    FROM quartile_medians
  ) qm ON sq.salary_quartile = qm.salary_quartile
),

imputed_data AS (
  SELECT
    user_id,
    age,
    last_month_salary_clean,
    number_dependents_clean,
    COALESCE(total_loans, median_total_loans) AS total_loans,
    CASE
      WHEN real_estate_loans IS NULL THEN 0
      ELSE real_estate_loans
    END AS real_estate_loans,
    other_loans,
    debt_ratio,
    using_lines_not_secured_personal_assets,
    number_times_delayed_payment_loan_30_59_days,
    number_times_delayed_payment_loan_60_89_days,
    more_90_days_overdue,
    total_delayed_payments,
    proportion_delayed_payment_30_89_days,
    proportion_delayed_payment_60_89_days,
    indicator_more_90_days_overdue,
    indicator_using_lines_not_secured,
    log_debt_ratio,
    scaled_debt_ratio,
    CASE
      WHEN debt_ratio > 0.4 THEN 1
      ELSE 0
    END AS high_debt_indicator,
    default_flag,
    salary_quartile,
    loans_quartile,
    delayed_payments_quartile,
    scaled_debt_ratio_quartile
  FROM main_data_with_quartiles
)

SELECT
  SAFE_CAST (user_id AS string) AS user_id,
  age,
  CASE 
    WHEN age BETWEEN 18 AND 34 THEN 'Adulto Joven'
    WHEN age BETWEEN 35 AND 59 THEN 'Adulto'
    ELSE 'Senior' 
  END AS age_group,
  last_month_salary_clean,
  salary_quartile,
  number_dependents_clean,
  CAST (total_loans AS int64) AS total_loans,
  loans_quartile,
  real_estate_loans,
  CASE
    WHEN other_loans IS NULL THEN CAST (total_loans AS int64)
    ELSE CAST (other_loans AS int64)
  END AS other_loans,
  debt_ratio,
  using_lines_not_secured_personal_assets,
  CAST (number_times_delayed_payment_loan_30_59_days AS int64) AS number_times_delayed_payment_loan_30_59_days,
  CAST (number_times_delayed_payment_loan_60_89_days AS int64) AS number_times_delayed_payment_loan_60_89_days,
  more_90_days_overdue,
  CAST (total_delayed_payments AS int64) AS total_delayed_payments,
  delayed_payments_quartile,
  proportion_delayed_payment_30_89_days,
  proportion_delayed_payment_60_89_days,
  indicator_more_90_days_overdue,
  indicator_using_lines_not_secured,
  log_debt_ratio,
  scaled_debt_ratio,
  scaled_debt_ratio_quartile,
  high_debt_indicator,
  default_flag,
  CAST (default_flag AS BOOLEAN) AS boolean_default_flag
FROM imputed_data
```
