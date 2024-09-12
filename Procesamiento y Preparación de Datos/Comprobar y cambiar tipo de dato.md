# Comprobar y cambiar tipo de dato

<aside>
💡 Cambiar el tipo de dato de last_month_salary de FLOAT a INTEGER

</aside>

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_clean` AS

SELECT
user_id,
age,
sex,
SAFE_CAST(last_month_salary_clean AS int64) AS last_month_salary_clean,
number_dependents_clean
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_clean`
```
