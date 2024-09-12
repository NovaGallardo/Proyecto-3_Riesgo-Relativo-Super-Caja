# Calcular Riesgo Relativo

### RR de default flag según age group

```sql
WITH event_counts AS (
  SELECT
    CASE
      WHEN age_group = 'Adulto Joven' THEN 'Adulto Joven'
      ELSE 'Adulto+Senior'
    END AS group_category,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
    SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
  WHERE age_group IN ('Adulto Joven', 'Adulto', 'Senior')
  GROUP BY group_category
),

relative_risk AS (
  SELECT
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN defaults ELSE 0 END) AS a,
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN non_defaults ELSE 0 END) AS b,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN defaults ELSE 0 END) AS c,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN non_defaults ELSE 0 END) AS d
  FROM event_counts
)

SELECT
  a, b, c, d,
  SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk
FROM relative_risk;
```

| **a** | **b** | **c** | **d** | **relative_risk** |
| --- | --- | --- | --- | --- |
| 170 | 4387 | 513 | 30930 | 2.2865278916697 |

<aside>
💡 Los "Adultos Jóvenes" son más propensos a incumplir en comparación con los "Adultos" y "Seniors".

</aside>

### RR de high debt indicator según age group

```sql
WITH event_counts AS (
  SELECT
    CASE
      WHEN age_group = 'Adulto Joven' THEN 'Adulto Joven'
      ELSE 'Adulto+Senior'
    END AS group_category,
    SUM(CASE WHEN high_debt_indicator = 1 THEN 1 ELSE 0 END) AS defaults,
    SUM(CASE WHEN high_debt_indicator = 0 THEN 1 ELSE 0 END) AS non_defaults
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
  WHERE age_group IN ('Adulto Joven', 'Adulto', 'Senior')
  GROUP BY group_category
),

relative_risk AS (
  SELECT
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN defaults ELSE 0 END) AS a,
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN non_defaults ELSE 0 END) AS b,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN defaults ELSE 0 END) AS c,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN non_defaults ELSE 0 END) AS d
  FROM event_counts
)

SELECT
  a, b, c, d,
  SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk
FROM relative_risk;
```

| **a** | **b** | **c** | **d** | **relative_risk** |
| --- | --- | --- | --- | --- |
| 1654 | 2903 | 15017 | 16426 | 0.7599714398730 |

<aside>
💡 La probabilidad de que los "Adultos Jóvenes" tengan un `high_debt_indicator` es **menor** en comparación con el grupo combinado "Adulto+Senior".

</aside>

### Combinación de RR de default flag y high debt indicator según age group

```sql
WITH event_counts AS (
  SELECT
    CASE
      WHEN age_group = 'Adulto Joven' THEN 'Adulto Joven'
      ELSE 'Adulto+Senior'
    END AS group_category,
    SUM(CASE WHEN default_flag = 1 AND high_debt_indicator = 1 THEN 1 ELSE 0 END) AS defaults_with_high_debt,
    SUM(CASE WHEN default_flag = 0 AND high_debt_indicator = 1 THEN 1 ELSE 0 END) AS non_defaults_with_high_debt,
    SUM(CASE WHEN default_flag = 1 AND high_debt_indicator = 0 THEN 1 ELSE 0 END) AS defaults_without_high_debt,
    SUM(CASE WHEN default_flag = 0 AND high_debt_indicator = 0 THEN 1 ELSE 0 END) AS non_defaults_without_high_debt
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
  WHERE age_group IN ('Adulto Joven', 'Adulto', 'Senior')
  GROUP BY group_category
),

relative_risk AS (
  SELECT
    -- Variables para el grupo "Adulto Joven"
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN defaults_with_high_debt ELSE 0 END) AS a,
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN non_defaults_with_high_debt ELSE 0 END) AS b,
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN defaults_without_high_debt ELSE 0 END) AS e,
    MAX(CASE WHEN group_category = 'Adulto Joven' THEN non_defaults_without_high_debt ELSE 0 END) AS f,
    -- Variables para el grupo "Adulto + Senior"
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN defaults_with_high_debt ELSE 0 END) AS c,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN non_defaults_with_high_debt ELSE 0 END) AS d,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN defaults_without_high_debt ELSE 0 END) AS g,
    MAX(CASE WHEN group_category = 'Adulto+Senior' THEN non_defaults_without_high_debt ELSE 0 END) AS h
  FROM event_counts
),

calculated_rates AS (
  SELECT
    a, b, c, d, e, f, g, h,
    -- Cálculo de tasas de incumplimiento con y sin high_debt_indicator
    SAFE_DIVIDE(a, (a + b)) AS rate_high_debt_adulto_joven,
    SAFE_DIVIDE(e, (e + f)) AS rate_low_debt_adulto_joven,
    SAFE_DIVIDE(c, (c + d)) AS rate_high_debt_adulto_senior,
    SAFE_DIVIDE(g, (g + h)) AS rate_low_debt_adulto_senior
  FROM relative_risk
)

SELECT
  a, b, c, d, e, f, g, h,
  rate_high_debt_adulto_joven,
  rate_low_debt_adulto_joven,
  rate_high_debt_adulto_senior,
  rate_low_debt_adulto_senior,
  SAFE_DIVIDE(rate_high_debt_adulto_joven, rate_high_debt_adulto_senior) AS relative_risk_high_debt,
  SAFE_DIVIDE(rate_low_debt_adulto_joven, rate_low_debt_adulto_senior) AS relative_risk_low_debt
FROM calculated_rates
```

| **a** | **b** | **c** | **d** | **e** | **f** | **g** | **h** | **rate_high_debt_adulto_joven** | **rate_low_debt_adulto_joven** | **rate_high_debt_adulto_senior** | **rate_low_debt_adulto_senior** | **relative_risk_high_debt** | **relative_risk_low_debt** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 73 | 1581 | 285 | 14732 | 97 | 2806 | 228 | 16198 | 0.044135429262394194 | 0.033413709955218737 | 0.01897849104348405 | 0.013880433459150129 | 2.3255499692399075 | 2.407252630370276 |

<aside>
💡 Los "Adultos Jóvenes" con un `high_debt_indicator` tienen una probabilidad de incumplimiento **2.33 veces mayor** que los "Adultos" y "Seniors" con un `high_debt_indicator`.

Los "Adultos Jóvenes" **sin** un `high_debt_indicator` también tienen una probabilidad de incumplimiento **2.41 veces mayor** que los "Adultos" y "Seniors" sin una alta carga de deuda.

Esto sugiere que, incluso sin una alta carga de deuda, los "Adultos Jóvenes" son significativamente más propensos a incumplir que los grupos de mayor edad.

</aside>

### Combinación de RR de default flag y loans

```sql
WITH quartile_grouping AS (
  SELECT
    user_id,
    CASE
      WHEN loans_quartile IN (1, 2) THEN 'Q1+Q2'
      ELSE 'Q3+Q4'
    END AS loans_quartile_group,
    default_flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
),

event_counts AS (
  SELECT
    loans_quartile_group,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
    SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults
  FROM quartile_grouping
  GROUP BY loans_quartile_group
),

relative_risk AS (
  SELECT
    -- Variables para el grupo "Q1+Q2"
    MAX(CASE WHEN loans_quartile_group = 'Q1+Q2' THEN defaults ELSE 0 END) AS a,
    MAX(CASE WHEN loans_quartile_group = 'Q1+Q2' THEN non_defaults ELSE 0 END) AS b,
    -- Variables para el grupo "Q3+Q4"
    MAX(CASE WHEN loans_quartile_group = 'Q3+Q4' THEN defaults ELSE 0 END) AS c,
    MAX(CASE WHEN loans_quartile_group = 'Q3+Q4' THEN non_defaults ELSE 0 END) AS d
  FROM event_counts
)

SELECT
  a, b, c, d,
  SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk
FROM relative_risk
```

| **a** | **b** | **c** | **d** | **relative_risk** |
| --- | --- | --- | --- | --- |
| 466 | 17534 | 217 | 17783 | 2.1474654377880182 |

<aside>
💡 Específicamente, los usuarios en los cuartiles más bajos de `total_loans` son aproximadamente **2.15 veces más propensos** a incumplir en sus pagos en comparación con aquellos en los cuartiles más altos de `total_loans`.

</aside>

### Combinación de RR de default flag y 90 day overdue

```sql
WITH overdue_grouping AS (
  SELECT
    user_id,
    CASE
      WHEN more_90_days_overdue > 0 THEN 'Overdue > 90 days'
      ELSE 'No Overdue > 90 days'
    END AS overdue_group,
    default_flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
),

event_counts AS (
  SELECT
    overdue_group,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults, -- 1 es mal pagador
    SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults -- 0 es buen pagador
  FROM overdue_grouping
  GROUP BY overdue_group
),

relative_risk AS (
  SELECT
    -- Variables para el grupo "Overdue > 90 days"
    MAX(CASE WHEN overdue_group = 'Overdue > 90 days' THEN defaults ELSE 0 END) AS a,
    MAX(CASE WHEN overdue_group = 'Overdue > 90 days' THEN non_defaults ELSE 0 END) AS b,
    -- Variables para el grupo "No Overdue > 90 days"
    MAX(CASE WHEN overdue_group = 'No Overdue > 90 days' THEN defaults ELSE 0 END) AS c,
    MAX(CASE WHEN overdue_group = 'No Overdue > 90 days' THEN non_defaults ELSE 0 END) AS d
  FROM event_counts
)

SELECT
  a, b, c, d,
  SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk
FROM relative_risk
```

| **a** | **b** | **c** | **d** | **relative_risk** |
| --- | --- | --- | --- | --- |
| 627 | 1319 | 56 | 33998 | 195.93174643958304 |

<aside>
💡

Un RR de 195.93 sugiere que los usuarios con retrasos superiores a 90 días tienen aproximadamente **195.93 veces más probabilidades** de incumplir en sus pagos en comparación con aquellos sin tales retrasos. Este valor sugiere una relación extremadamente fuerte entre los retrasos prolongados en los pagos y la probabilidad de incumplimiento.

</aside>

### Combinación de RR de default flag y proportion_delayed_payment_30_89_days

```sql
WITH delayed_payment_grouping AS (
  SELECT
    user_id,
    CASE
      WHEN proportion_delayed_payment_30_89_days > 0.5 THEN 'High Delayed Payment'
      ELSE 'Low Delayed Payment'
    END AS delayed_payment_group,
    default_flag
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
),

event_counts AS (
  SELECT
    delayed_payment_group,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
    SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults
  FROM delayed_payment_grouping
  GROUP BY delayed_payment_group
),

relative_risk AS (
  SELECT
    -- Variables para el grupo "High Delayed Payment"
    MAX(CASE WHEN delayed_payment_group = 'High Delayed Payment' THEN defaults ELSE 0 END) AS a,
    MAX(CASE WHEN delayed_payment_group = 'High Delayed Payment' THEN non_defaults ELSE 0 END) AS b,
    -- Variables para el grupo "Low Delayed Payment"
    MAX(CASE WHEN delayed_payment_group = 'Low Delayed Payment' THEN defaults ELSE 0 END) AS c,
    MAX(CASE WHEN delayed_payment_group = 'Low Delayed Payment' THEN non_defaults ELSE 0 END) AS d
  FROM event_counts
)

SELECT
  a, b, c, d,
  SAFE_DIVIDE(a, (a + b)) / SAFE_DIVIDE(c, (c + d)) AS relative_risk
FROM relative_risk
```

| **a** | **b** | **c** | **d** | **relative_risk** |
| --- | --- | --- | --- | --- |
| 442 | 5505 | 241 | 29812 | 9.268194082305 |

<aside>
💡

Los usuarios con una alta proporción de pagos retrasados entre 30 y 89 días tienen aproximadamente **9.27 veces más probabilidades** de ser malos pagadores (default) en comparación con los usuarios que tienen una baja proporción de pagos retrasados.

</aside>

### Combinación de RR de default flag, salary y number of dependents

```sql
WITH event_counts AS (
  SELECT
    CASE
      WHEN salary_quartile IN (3, 4) THEN 'High Salary'
      ELSE 'Low Salary'
    END AS salary_group,
    CASE
      WHEN number_dependents_clean > 0 THEN 'Has Dependents'
      ELSE 'No Dependents'
    END AS dependents_group,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS defaults,
    SUM(CASE WHEN default_flag = 0 THEN 1 ELSE 0 END) AS non_defaults
  FROM `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data_clean`
  GROUP BY salary_group, dependents_group
),

relative_risk AS (
  SELECT
    -- Variables para el grupo "High Salary" y "Has Dependents"
    MAX(CASE WHEN salary_group = 'High Salary' AND dependents_group = 'Has Dependents' THEN defaults ELSE 0 END) AS a,
    MAX(CASE WHEN salary_group = 'High Salary' AND dependents_group = 'Has Dependents' THEN non_defaults ELSE 0 END) AS b,
    -- Variables para el grupo "High Salary" y "No Dependents"
    MAX(CASE WHEN salary_group = 'High Salary' AND dependents_group = 'No Dependents' THEN defaults ELSE 0 END) AS c,
    MAX(CASE WHEN salary_group = 'High Salary' AND dependents_group = 'No Dependents' THEN non_defaults ELSE 0 END) AS d,
    -- Variables para el grupo "Low Salary" y "Has Dependents"
    MAX(CASE WHEN salary_group = 'Low Salary' AND dependents_group = 'Has Dependents' THEN defaults ELSE 0 END) AS e,
    MAX(CASE WHEN salary_group = 'Low Salary' AND dependents_group = 'Has Dependents' THEN non_defaults ELSE 0 END) AS f,
    -- Variables para el grupo "Low Salary" y "No Dependents"
    MAX(CASE WHEN salary_group = 'Low Salary' AND dependents_group = 'No Dependents' THEN defaults ELSE 0 END) AS g,
    MAX(CASE WHEN salary_group = 'Low Salary' AND dependents_group = 'No Dependents' THEN non_defaults ELSE 0 END) AS h
  FROM event_counts
),

calculated_rates AS (
  SELECT
    a, b, c, d, e, f, g, h,
    -- Cálculo de tasas de incumplimiento para cada grupo
    SAFE_DIVIDE(a, (a + b)) AS rate_high_salary_has_dependents,
    SAFE_DIVIDE(c, (c + d)) AS rate_high_salary_no_dependents,
    SAFE_DIVIDE(e, (e + f)) AS rate_low_salary_has_dependents,
    SAFE_DIVIDE(g, (g + h)) AS rate_low_salary_no_dependents
  FROM relative_risk
)

SELECT
  a, b, c, d, e, f, g, h,
  rate_high_salary_has_dependents,
  rate_high_salary_no_dependents,
  rate_low_salary_has_dependents,
  rate_low_salary_no_dependents,
  SAFE_DIVIDE(rate_high_salary_has_dependents, rate_low_salary_no_dependents) AS relative_risk_high_salary_vs_low_no_dependents,
  SAFE_DIVIDE(rate_high_salary_no_dependents, rate_low_salary_has_dependents) AS relative_risk_high_no_vs_low_dependents
FROM calculated_rates
```

| **a** | **b** | **c** | **d** | **e** | **f** | **g** | **h** | **rate_high_salary_has_dependents** | **rate_high_salary_no_dependents** | **rate_low_salary_has_dependents** | **rate_low_salary_no_dependents** | **relative_risk_high_salary_vs_low_no_dependents** | **relative_risk_high_no_vs_low_dependents** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 109 | 8405 | 96 | 9390 | 227 | 5404 | 251 | 12118 | 0.012802443035001174 | 0.010120177103099304 | 0.04031255549635944 | 0.020292667151750345 | 0.63089011115509763 | 0.25104280734604 |

<aside>
💡

### Tasa de Incumplimiento

1. **rate_high_salary_has_dependents (0.0128)**:
    - Tasa de incumplimiento para aquellos con salario alto y dependientes. Aproximadamente el 1.28% de los usuarios en este grupo incumplen.
2. **rate_high_salary_no_dependents (0.0101)**:
    - Tasa de incumplimiento para aquellos con salario alto y sin dependientes. Aproximadamente el 1.01% de los usuarios en este grupo incumplen.
3. **rate_low_salary_has_dependents (0.0403)**:
    - Tasa de incumplimiento para aquellos con salario bajo y dependientes. Aproximadamente el 4.03% de los usuarios en este grupo incumplen.
4. **rate_low_salary_no_dependents (0.0203)**:
    - Tasa de incumplimiento para aquellos con salario bajo y sin dependientes. Aproximadamente el 2.03% de los usuarios en este grupo incumplen.

### Riesgo Relativo

1. **relative_risk_high_salary_vs_low_no_dependents (0.6309)**:
    - Esto indica que los usuarios con salario alto y dependientes tienen un **37% menor** riesgo de incumplimiento comparado con los usuarios con salario bajo y sin dependientes.
2. **relative_risk_high_no_vs_low_dependents (0.2510)**:
    - Esto indica que los usuarios con salario alto y sin dependientes tienen un **75% menor** riesgo de incumplimiento comparado con los usuarios con salario bajo y con dependientes.
</aside>

### Tabla de Peores Pagadores Ordenada de Mayor a Menor Riesgo

| **Comparación** | **Grupo A** | **Grupo B** | **Riesgo Relativo (RR)** | **Interpretación** |
| --- | --- | --- | --- | --- |
| **Overdue > 90 days vs. No Overdue > 90 days** | Overdue > 90 days | No Overdue > 90 days | 195.9317 | Los que han tenido retrasos de más de 90 días tienen un riesgo de incumplimiento significativamente mayor, siendo el grupo con el riesgo más elevado de incumplimiento. |
| **High Delayed Payment vs. Low Delayed Payment** | High Delayed Payment | Low Delayed Payment | 9.2682 | Un alto porcentaje de pagos retrasados entre 30 y 89 días está fuertemente asociado con un mayor riesgo de incumplimiento, reflejando un riesgo muy elevado. |
| **Low Debt (Adulto Joven) vs. Low Debt (Adulto+Senior)** | Low Debt (Adulto Joven) | Low Debt (Adulto+Senior) | 2.4073 | Los jóvenes con deudas bajas son más propensos a incumplir en comparación con adultos mayores con deudas bajas, mostrando un riesgo relativo alto. |
| **High Debt (Adulto Joven) vs. High Debt (Adulto+Senior)** | High Debt (Adulto Joven) | High Debt (Adulto+Senior) | 2.3255 | Los jóvenes con deudas altas tienen un mayor riesgo de incumplir en comparación con adultos mayores con deudas altas, representando un riesgo considerable. |
| **Adulto Joven vs. Adulto+Senior** | Adulto Joven | Adulto+Senior | 2.2865 | Los adultos jóvenes, en general, presentan un mayor riesgo de incumplimiento en comparación con adultos y seniors, con un riesgo elevado. |
| **Low Loans (Q1+Q2) vs. High Loans (Q3+Q4)** | Q1+Q2 | Q3+Q4 | 2.1475 | Aquellos con menos préstamos tienen un riesgo más alto de incumplimiento en comparación con aquellos con más préstamos, aunque el riesgo es relativamente menor en comparación con otros factores. |
| **High Debt Indicator (Adulto Joven) vs. High Debt Indicator (Adulto+Senior)** | High Debt (Adulto Joven) | High Debt (Adulto+Senior) | 0.7599 | Aunque el riesgo es menor, los adultos jóvenes con altos indicadores de deuda todavía presentan un riesgo de incumplimiento más bajo en comparación con adultos mayores, pero sigue siendo significativo. |
| **High Salary (sin dependientes) vs. Low Salary (con dependientes)** | High Salary (sin dependientes) | Low Salary (con dependientes) | 0.2510 | Las personas con altos ingresos sin dependientes tienen un riesgo de incumplimiento más bajo que aquellos con bajos ingresos y dependientes, pero la diferencia no es tan significativa. |
| **High Salary vs. Low Salary (sin dependientes)** | High Salary (con dependientes) | Low Salary (sin dependientes) | 0.6309 | Las personas con altos ingresos y dependientes tienen un riesgo de incumplimiento menor en comparación con aquellos con bajos ingresos y sin dependientes. |
