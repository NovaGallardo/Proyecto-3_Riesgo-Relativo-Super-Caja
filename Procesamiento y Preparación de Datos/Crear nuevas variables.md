# Crear nuevas variables

### Proporción de Pagos Retrasados Más de 60 Días

```sql
SELECT
user_id,
number_times_delayed_payment_loan_60_89_days / NULLIF((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue), 0) AS proportion_delayed_payment_60_89_days
FROM loans_detail
```

<aside>
💡 Esta variable indica la proporción de retrasos de pagos significativos (60-89 días) en relación con todos los retrasos.  Un valor alto (cercano a 1) indica que casi todos los retrasos del cliente están en la categoría de 60-89 días. Esto podría sugerir que el cliente está en una situación financiera inestable y está en riesgo de caer en retrasos más largos.

</aside>

### Proporción de Pagos Retrasados Más de 30 Días

```sql
SELECT 
  user_id,
  (number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days) / NULLIF((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue), 0) AS proportion_delayed_payment_30_89_days
FROM loans_detail
```

<aside>
💡 Esta variable muestra la proporción de todos los retrasos significativos (30-89 días).  Un valor alto (cercano a 1) indica que casi todos los retrasos del cliente están en las categorías de 30-89 días. Esto podría sugerir que el cliente tiene problemas frecuentes pero no extremos de liquidez.

</aside>

### Total de Pagos Retrasados

```sql
SELECT 
  user_id,
  (number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue) AS total_delayed_payments
FROM loans_detail
```

<aside>
💡 Esta variable proporciona una visión general de cuántas veces un cliente ha retrasado sus pagos en total. Un número alto puede indicar un alto riesgo de incumplimiento.

</aside>

### Indicador de Pagos Retrasados Más de 90 Días

```sql
SELECT 
  user_id,
  CASE 
    WHEN more_90_days_overdue > 0 THEN 1 
    ELSE 0 
  END AS indicator_more_90_days_overdue
FROM loans_detail
```

<aside>
💡 Esta variable binaria indica si el cliente alguna vez ha tenido un retraso en el pago superior a 90 días. Es una señal fuerte de riesgo de crédito si esta variable es 1.

</aside>

### Indicador de Uso de Líneas de Crédito No Aseguradas por Activos Personales

```sql
SELECT 
  user_id,
  CASE 
    WHEN using_lines_not_secured_personal_assets > 0 THEN 1 
    ELSE 0 
  END AS indicator_using_lines_not_secured
FROM loans_detail
```

<aside>
💡 Esta variable binaria indica si el cliente está utilizando líneas de crédito que no están aseguradas por activos personales. Esto puede sugerir una mayor dependencia de créditos no garantizados, lo que podría incrementar el riesgo de incumplimiento.

</aside>

### Logaritmo del Ratio de Deuda

```sql
SELECT 
  user_id,
  LOG(NULLIF(debt_ratio, 0)) AS log_debt_ratio
FROM loans_detail
```

<aside>
💡 La transformación logarítmica del ratio de deuda puede ayudar a manejar la distribución sesgada de los datos de deuda, haciendo que los modelos de predicción sean más robustos.
Puede ser más fácil diferenciar entre clientes con diferentes niveles de ratio de deuda después de la transformación.
Por ejemplo:

- Cliente A: `debt_ratio` = 0.5 (50% de deuda respecto a los ingresos)
- Cliente B: `debt_ratio` = 2 (200% de deuda respecto a los ingresos)

Interpretación:

- **Cliente A**: Un `log_debt_ratio` de -0.693 indica que el `debt_ratio` original es menor que 1 (deuda menor que los ingresos).
- **Cliente B**: Un `log_debt_ratio` de 0.693 indica que el `debt_ratio` original es mayor que 1 (deuda mayor que los ingresos).
</aside>

### Escalado del Ratio de Deuda

```sql
SELECT 
  user_id,
  (debt_ratio - MIN(debt_ratio) OVER()) / (MAX(debt_ratio) OVER() - MIN(debt_ratio) OVER()) AS scaled_debt_ratio
FROM loans_detail
```

<aside>
💡 Escalar el ratio de deuda normaliza la variable, lo que puede ser útil para algoritmos de aprendizaje automático que requieren que las características estén en una escala similar.

</aside>

### Interacción entre Debt Ratio y Número de Pagos Retrasados (30-59 días)

```sql
SELECT 
  user_id,
  debt_ratio * number_times_delayed_payment_loan_30_59_days AS interaction_debt_ratio_delayed_30_59
FROM loans_detail
```

<aside>
💡 Esta variable de interacción puede capturar efectos combinados que no son evidentes cuando se consideran las variables individualmente. Por ejemplo, un cliente con un alto ratio de deuda y múltiples retrasos de 30-59 días puede tener un riesgo significativamente mayor que lo que indican las variables por separado.

</aside>

Tabla completa

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail_clean` AS

SELECT 
  user_id,
  debt_ratio,
  using_lines_not_secured_personal_assets,
  CAST (number_times_delayed_payment_loan_30_59_days AS int64) AS number_times_delayed_payment_loan_30_59_days,
  CAST (number_times_delayed_payment_loan_60_89_days AS int64) AS number_times_delayed_payment_loan_60_89_days,
  CAST (more_90_days_overdue AS int64) AS more_90_days_overdue,
  CAST ((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue) AS int64)AS total_delayed_payments,

  CASE
    WHEN (number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days) / NULLIF((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue), 0) IS NULL THEN 0
    ELSE (number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days) / NULLIF((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue), 0)
  END AS proportion_delayed_payment_30_89_days,

  CASE
    WHEN number_times_delayed_payment_loan_60_89_days / NULLIF((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue), 0) IS NULL THEN 0
    ELSE number_times_delayed_payment_loan_60_89_days / NULLIF((number_times_delayed_payment_loan_30_59_days + number_times_delayed_payment_loan_60_89_days + more_90_days_overdue), 0)
    END AS proportion_delayed_payment_60_89_days,

 CASE 
    WHEN more_90_days_overdue > 0 THEN 1 
    ELSE 0 
  END AS indicator_more_90_days_overdue,
   CASE 
    WHEN using_lines_not_secured_personal_assets > 0 THEN 1 
    ELSE 0 
  END AS indicator_using_lines_not_secured,
  
  CASE
    WHEN LOG(NULLIF(debt_ratio, 0)) IS NULL THEN 0
    ELSE LOG(NULLIF(debt_ratio, 0))
    END AS log_debt_ratio,

 (debt_ratio - MIN(debt_ratio) OVER()) / (MAX(debt_ratio) OVER() - MIN(debt_ratio) OVER()) AS scaled_debt_ratio,
 

FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail_winsorized`
```
