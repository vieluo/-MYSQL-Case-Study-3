# 8 Weeks SQL Challenge - 3th Week

### This [case study](https://8weeksqlchallenge.com/case-study-2/)  is provided by Danny Ma 

### This is the continuation of a case study, see the description, problem statement, tables and the first questions in [Repository](https://github.com/vieluo/-MYSQL-Case-Study-3/blob/main/CaseStudy3_DataAnalysisQuestions.md)

The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:
* Monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
* Upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
* Upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
* Once a customer churns they will no longer make payments

First we are going to create a table call 'num'

```sql
CREATE TABLE num (num INTEGER);

INSERT INTO num
  (num)
VALUES
  ('0'),
  ('1'),
  ('2'),
  ('3'),
  ('4'),
  ('5'),
  ('6'),
  ('7'),
  ('8'),
  ('9'),
  ('10'),
  ('11');
```

We are going to join 2 tables together and get how much months left to pay for each plan
```sql
CREATE TABLE consolidated AS (
WITH temp AS (
WITH temp1 AS (
SELECT *, 
LEAD(start_date,1) OVER(PARTITION BY customer_id ORDER BY start_date, plan_id) AS new_plan_start_date 
/* take the inicial date of next plan, then we can calculate the months for each plan */
FROM subscriptions
WHERE plan_id <> 0 AND start_date <= '2020-12-31'
)
SELECT *, CASE
WHEN new_plan_start_date IS NOT NULL
THEN TIMESTAMPDIFF(month, start_date,new_plan_start_date)
ELSE TIMESTAMPDIFF(month, start_date,'2020-12-31')
END AS payment_num
FROM temp1
)
SELECT temp.customer_id, temp.plan_id, plans.plan_name, plans.price, temp.start_date, temp.new_plan_start_date, temp.payment_num
FROM temp
LEFT JOIN plans
ON temp.plan_id = plans.plan_id
)
```

If churn, the price will be 0, if user opts for annual plan, the paymente number will be 1
```sql
UPDATE consolidated
SET payment_num = 0 /* when repeat the row, can repeat only once, if put payment_num = 1, the row will be repeted twice */
WHERE plan_id = 3
```
```sql
UPDATE consolidated
SET price = 0
WHERE plan_id = 4
```

Repeat rows according to column value.
```sql
WITH temp2 AS (
SELECT *
FROM consolidated
JOIN num 
ON consolidated.payment_num >= num.num
ORDER BY customer_id, plan_id, num
)
SELECT *, DATE_ADD(start_date, INTERVAL num MONTH) AS payment_date, num+1 AS payment_order
FROM temp2
WHERE NOT plan_id = 4
```

Organize data as needed.
```sql
SELECT customer_id, plan_id, plan_name, price, payment_date, payment_order
FROM temp1
```

```sql
WITH temp1 AS (
SELECT *,LAG(payment_date) OVER (partition by customer_id order by payment_date) AS last_payment
FROM payment_order_2020
ORDER BY customer_id, plan_id
)
SELECT customer_id, plan_id, plan_name, payment_date, payment_order, CASE WHEN
MONTH (last_payment) = MONTH (payment_date)
THEN price - (LAG(price) OVER (partition by customer_id order by payment_date))
ELSE price
END AS actual_price
FROM temp1
```

Result:
customer_id | plan_id | plan_name | payment_date | payment_order | actual_price
| - | - | ------------- | ---------- | - | ------ |
| 1 | 1 | basic monthly | 2020-08-08 | 1 | 9.90   |
| 1 | 1 | basic monthly | 2020-09-08 | 2 | 9.90   |
| 1 | 1 | basic monthly | 2020-10-08 | 3 | 9.90   |
| 1 | 1 | basic monthly | 2020-11-08 | 4 | 9.90   |
| 1 | 1 | basic monthly | 2020-12-08 | 5 | 9.90   |
| 2 | 3 | pro annual    | 2020-09-27 | 1 | 199.00 |
....
| 15 | 2 | pro monthly   | 2020-03-24 | 1 | 19.90  |
| 15 | 2 | pro monthly   | 2020-04-24 | 2 | 19.90  |
| 16 | 1 | basic monthly | 2020-06-07 | 1 | 9.90   |
| 16 | 1 | basic monthly | 2020-07-07 | 2 | 9.90   |
| 16 | 1 | basic monthly | 2020-08-07 | 3 | 9.90   |
| 16 | 1 | basic monthly | 2020-09-07 | 4 | 9.90   |
| 16 | 1 | basic monthly | 2020-10-07 | 5 | 9.90   |
| 16 | 3 | pro annual    | 2020-10-21 | 1 | 189.10 |