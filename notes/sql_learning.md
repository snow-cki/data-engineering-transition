1.
`dense_rank()` - good for handling cases where duplicates can arise, used in second highest salary problem, also does not skip numbers

2.
`sum()` and `filter()` - better version than sum(case when x then y else end)
```
SELECT
  SUM(amount) FILTER (WHERE status = 'paid') AS paid_total,
  SUM(amount) FILTER (WHERE status = 'refunded') AS refunded_total
FROM orders;
```

3.
simple cast to float when calculating percentages - instead of multiplying by 100, multiply by 100.0, prevents losing the decimal point extension

4.
`year()` method is not always accessible in every DB engine, but EXTRACT(YEAR from x) is standard SQL

5.
rolling average - do not try to use lag(), usually there's a dedicated AVG
`avg(tweet_count) over(partition by user_id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)`
rolling average from 2 previous and current row

6.
count() with CASE - count counts EVERY non-NULL value, so even providing 0 still bumps up the counter

correct way would be to use else NULL

```
SELECT 
  round(count(CASE WHEN signup_action='Confirmed' then 1 ELSE NULL END) * 1.0 / count(*), 2) as confirm_rate
  FROM emails e join texts t
  on e.email_id = t.email_id and signup_action is not NULL
```

7. Look for customers that have contracts for all categories, exists not needed, simple distinct counts do the job

```
SELECT cc.customer_id FROM customer_contracts cc join products p on cc.product_id = p.product_id group by cc.customer_id
having count(distinct p.product_category) = (select count(distinct product_category) from products)
```


8. Merging historical data with new data, for example deltas, new incoming data
Usual way is to union all old rows to historical rows. If there's a need to drop duplicate rows, union should be used.


9. Horizontal merging of data
While vertical merging is done with union all, horizontal is usually done with joins - if data from the same table is needed CTEs come in handy. Example:

with highest as (
  SELECT 
    ticker,
    to_char(date, 'Mon-YYYY') as highest_mth,
    max(open) as highest_open,
    row_number() over (partition by ticker order by open desc) as rn
  FROM stock_prices 
  group by ticker, highest_mth, open
  order by ticker
),
lowest as (
  SELECT 
    ticker,
    to_char(date, 'Mon-YYYY') as lowest_mth,
    min(open) as lowest_open,
    row_number() over (partition by ticker order by open asc) as rn
  FROM stock_prices 
  group by ticker, lowest_mth, open
  order by ticker
)
SELECT
h.ticker,
h.highest_mth,
h.highest_open,
l.lowest_mth,
l.lowest_open
from highest h join lowest l on h.ticker = l.ticker
where h.rn = 1 and l.rn=1

we transform stock data to just display highest and lowest as columns

