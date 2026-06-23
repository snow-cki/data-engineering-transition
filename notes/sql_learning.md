dense_rank() - good for handling cases where duplicates can arise, used in second highest salary problem


sum() and filter() - better version than sum(case when x then y else end)
SELECT
  SUM(amount) FILTER (WHERE status = 'paid') AS paid_total,
  SUM(amount) FILTER (WHERE status = 'refunded') AS refunded_total
FROM orders;


simple cast to float when calculating percentages - instead of multiplying by 100, multiply by 100.0, prevents losing the decimal point extension




