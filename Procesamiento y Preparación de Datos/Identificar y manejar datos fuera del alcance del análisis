# Identificar y manejar datos fuera del alcance del análisis

## Standard Deviation de more_90_days_overdue

```sql
SELECT  
STDDEV(more_90_days_overdue) AS standar_deviation_90daysoverdue
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
```

<aside>
💡 Resultado: 4.12136466842672

</aside>

## Standard Deviation de number_times_delayed_payment_loan_30_59_days

```sql
SELECT  
STDDEV(number_times_delayed_payment_loan_30_59_days) AS standar_numbertimesdelayed
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
```

<aside>
💡 Resultado: 4.144020438225871

</aside>

## Correlación more_90_days_overdue y delayed_payment

```sql
SELECT  
CORR(more_90_days_overdue,number_times_delayed_payment_loan_30_59_days) AS correlation_daysoverdue_delayedpayment3059
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
```

<aside>
💡 Resultado: 0.98291680661459857

</aside>

## Correlación more_90_days_overdue y debt_ratio

```sql
SELECT  
CORR(more_90_days_overdue,debt_ratio) AS correlation_daysoverdue_debtratio
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
```

<aside>
💡 Resultado: -0.0082076486652037147

</aside>

## Correlación more_90_days_overdue y number_times_delayed_payment_loan_60_89_days

```sql
SELECT  
CORR(more_90_days_overdue,number_times_delayed_payment_loan_60_89_days) AS correlation_daysoverdue_delayedpayment6089
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
```

<aside>
💡 Resultado: 0.99217552634075257

</aside>

## Correlación more_90_days_overdue y using_lines_not_secured_personal_assets

```sql
SELECT  
CORR(more_90_days_overdue,using_lines_not_secured_personal_assets) AS correlation_daysoverdue_linesnotsecured
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail`
```

<aside>
💡 Resultado: -0.0013609406054606965

</aside>
