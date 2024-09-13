## main_user_data

```sql
CREATE OR REPLACE TABLE `erudite-scholar-426802-h9.proyecto3_riesgo.main_user_data` AS

SELECT
i.*,
o.total_loans,
o.real_estate_loans,
o.other_loans,
d.debt_ratio,
d.using_lines_not_secured_personal_assets,
d.number_times_delayed_payment_loan_30_59_days,
d.number_times_delayed_payment_loan_60_89_days,
d.more_90_days_overdue,
d.total_delayed_payments,
d.proportion_delayed_payment_30_89_days,
d.proportion_delayed_payment_60_89_days,
d.indicator_more_90_days_overdue,
d.indicator_using_lines_not_secured,
d.log_debt_ratio,
d.scaled_debt_ratio,
de.default_flag
FROM `erudite-scholar-426802-h9.proyecto3_riesgo.user_info_clean` AS i
LEFT JOIN `erudite-scholar-426802-h9.proyecto3_riesgo.loans_outstanding_clean` AS o
ON i.user_id = o.user_id
LEFT JOIN `erudite-scholar-426802-h9.proyecto3_riesgo.loans_detail_clean` AS d
ON i.user_id = d.user_id
LEFT JOIN `erudite-scholar-426802-h9.proyecto3_riesgo.default` AS de
ON i.user_id = de.user_id

```
