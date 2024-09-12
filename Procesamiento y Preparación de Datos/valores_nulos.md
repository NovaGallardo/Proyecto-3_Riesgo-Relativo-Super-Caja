# Análisis y Tratamiento de Valores Nulos en el Proyecto 3

Este documento detalla el análisis de valores nulos en los diferentes datasets del proyecto, así como los métodos utilizados para tratarlos mediante la imputación de valores.

## Consultar valores nulos en “default”

```sql
SELECT 
*,
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.default`
WHERE user_id IS NULL
OR default_flag IS NULL

```

<aside>
💡 “default” no tiene valores nulos.

</aside>

## Consultar valores nulos en “loans_details”

```sql
SELECT 
*,
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
WHERE user_id IS NULL
OR more_90_days_overdue IS NULL
OR using_lines_not_secured_personal_assets IS NULL
OR number_times_delayed_payment_loan_30_59_days IS NULL
OR debt_ratio IS NULL
OR number_times_delayed_payment_loan_60_89_days IS NULL
```

<aside>
💡 Se detectaron 0 valores nulos.

</aside>

## Consultar valores nulos en “loans_outstanding”

```sql
SELECT 
*,
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding`
WHERE user_id IS NULL
OR loan_id IS NULL
OR loan_type IS NULL
```

<aside>
💡 “loans_outstanding” no tiene valores nulos.

</aside>

## Consultar valores nulos en “user_info”

```sql
SELECT 
*,
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
WHERE user_id IS NULL
OR age IS NULL
OR sex IS NULL
OR last_month_salary IS NULL
OR number_dependents IS NULL
```

<aside>
💡 Se detectaron 7199 valores nulos en ”user_info” .

</aside>

## Utilizar métodos de imputación para tratar los valores nulos

### Sacar la correlación entre age e income de “last_month_salary”

```sql
SELECT
CORR (age, last_month_salary) AS correlation_value
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_default`
```

<aside>
El valor de la correlación es: 0.035978007340202761, sugiere que no hay una relación lineal fuerte entre estas dos variables. Al no haber una distribución normal de los datos, se tiene que obtener la median para reemplazar los datos nulos.

</aside>

### Sacar la correlación entre age y dependents de “number_dependents”

```sql
SELECT
 CORR (age, number_dependents) AS dependets
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_default`
```

<aside>
Es valor de la correlación es: -0.21661156571947596

</aside>

### Obtener la median de “last_month_income”

```sql
SELECT
 PERCENTILE_CONT(last_month_salary, 0.5) OVER() AS median_salary
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_default`
WHERE last_month_salary IS NOT NULL
LIMIT 1
```

<aside>
Se pide calcular los percentiles sobre toda la columna de last_month_salary y devuelve solo el percentil 50%, que representa la mediana. Sobre toda la columna, se deja de lado los nulos y se limita la respuesta a solo 1 valor para que no retorne todas las filas de la columna. La mediana es 5400

</aside>

### Obtener la median de “number_dependents”

```sql
SELECT
 PERCENTILE_CONT(number_dependents, 0.5) OVER() AS median_dependents
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_default`
WHERE last_month_salary IS NOT NULL
LIMIT 1
```

<aside>
La mediana es 0

</aside>

## Reemplazar los valores nulos de “last_month_salary” y “number_dependents” y crean una nueva tabla limpia

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_clean` AS

SELECT
user_id,
age,
sex,
CASE
  WHEN last_month_salary IS NULL THEN 5400
  ELSE last_month_salary
END AS last_month_salary_clean,
  
CASE
  WHEN number_dependents IS NULL THEN 0
  ELSE number_dependents
END AS number_dependents_clean

FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info`
```
