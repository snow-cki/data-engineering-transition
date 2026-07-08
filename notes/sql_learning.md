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

10. self joins
useful concepts for hierarchical data / looking for streaks in time fields

This example allows to find users which made transactions on 3 consecutive days
```
SELECT distinct(t1.user_id) from transactions t1
join transactions t2 on t1.transaction_date + INTERVAL '1 day' = t2.transaction_date
join transactions t3 on t1.transaction_date + INTERVAL '2 days' = t3.transaction_date
order by t1.user_id
```

And this allows to find a manager of an employee in employee table
```
SELECT
  e.first_name || ' ' || e.last_name employee,
  m.first_name || ' ' || m.last_name manager
FROM
  employee e
  INNER JOIN employee m ON m.employee_id = e.manager_id
ORDER BY
  manager;
```

11. working with multiple foregin keys in a single table

If there are multiple columns that point to the same foregin key in a different table, multiple join to the same table should be performed. Data from corresponding joins can then be retrieved by querying correct alias.

```
SELECT 
round( 100.0 * count(*) / ( select count(*) from phone_calls), 1) as international_calls_pct 

FROM phone_calls pc 
join phone_info pci on pc.caller_id = pci.caller_id
join phone_info pri on pc.receiver_id = pri.caller_id
where pci.country_id <> pri.country_id;
```

12. Searching for intersection between two subsets
exists() is useful for such tasks, it allows to check whether row with certain conditions exists. Inside of the call tables from upper scope can be referenced. Here is an example where we look for intersection in 2 groups - users with events that happened both last month and in current month
```
SELECT 
  EXTRACT(MONTH from curr_month.event_date) as month,
  count(distinct user_id) as monthly_active_users
from user_actions curr_month
where extract(year from curr_month.event_date) = 2022 and extract(month from curr_month.event_date) = 7
and exists(
  select last_month.user_id
  from user_actions last_month
  where 
  extract(MONTH from curr_month.event_date - interval '1 MONTH') = extract(MONTH from last_month.event_date ) 
  and curr_month.user_id = last_month.user_id
)
group by month
```
