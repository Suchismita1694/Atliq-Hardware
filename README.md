# Atliq-Hardware
Codebasics Resume Project Challenge #4 - Provide Insights to Management in Consumer Goods Domain

## Dataset preview 

```
select * from dim_customer;
select * from dim_product;
select * from fact_gross_price;
select * from fact_manufacturing_cost;
select * from fact_pre_invoice_deductions;
select * from fact_sales_monthly;
```

# SQL Ad-hoc Requests
--1
```
select market from dim_customer
where customer = 'Atliq Exclusive'
and region = 'APAC' 
order by market;
```
--2
```
with cte as 
(select sum(case when fiscal_year = '2020' then 1 else 0 end) as unique_product_2020,
sum(case when fiscal_year = '2021' then 1 else 0 end) as unique_product_2021 from fact_gross_price)
select unique_product_2020, unique_product_2021, 
Round((unique_product_2021-unique_product_2020)/unique_product_2020 * 100,2) as percent_change
from cte
```
--3
```
select segment, count(distinct product_code) as product_count
from dim_product 
group by segment
order by 2 desc
```
--4
```
with cte as
(select segment,
count(distinct case when fiscal_year = '2020' then a.product_code end) as unique_product_2020,
count(distinct case when fiscal_year = '2021' then a.product_code end) as unique_product_2021
from  dim_product a
join fact_gross_price b 
on a.product_code = b.product_code
group by segment)
select segment, unique_product_2020, unique_product_2021,
(unique_product_2021- unique_product_2020) as difference
from cte 
order by 4 desc
```
--5 
```
select a.product_code, product, manufacturing_cost
from dim_product a
join fact_manufacturing_cost b
on a.product_code = b.product_code
where manufacturing_cost in (select max(manufacturing_cost) from fact_manufacturing_cost)
or manufacturing_cost in (select min(manufacturing_cost) from fact_manufacturing_cost)
```
--6
```
select a.customer_code, customer, 
Round(Avg(pre_invoice_discount_pct)*100,2) as average_discount_percentage,
Concat(Round(AVG(pre_invoice_discount_pct)*100, 2), '%') AS average_discount_percentage_with_percent_sign
from dim_customer a
join fact_pre_invoice_deductions b
on a.customer_code = b.customer_code
where fiscal_year = '2021'
and market = 'India'
group by a.customer_code
order by 3 desc
limit 5;
```
--7.
```
select monthname(date), year(date), CONCAT("$",(Round(Sum(sold_quantity * gross_price)/1000000,2))) as gross_sales_amount
from fact_sales_monthly a
join fact_gross_price b
on a.product_code = b.product_code
and a.fiscal_year = b.fiscal_year
join dim_customer c
on a.customer_code =  c.customer_code
where customer = 'Atliq Exclusive'
group by 1,2
order by 2
```
--8.
```
select case when month(date) in (9,10,11) then "Q1"
            when month(date) in (12,1,2) then "Q2"
            when month(date) in (3,4,5) then "Q3"
            else "Q4" end as Quarter ,
            sum(sold_quantity) as total_quantity_sold
from fact_sales_monthly
where fiscal_year = '2020'
group by 1
order by 2 desc;
```
--9.
```
with cte as
(select channel, Round(sum(a.sold_quantity * b.gross_price)/1000000,2) as gross_sales_mln
from fact_sales_monthly a
left join fact_gross_price b
on a.product_code = b.product_code
and a.fiscal_year = b.fiscal_year
join dim_customer
on a.customer_code = dim_customer.customer_code 
where b.fiscal_year = '2021'
group by channel)
select channel, gross_sales_mln, CONCAT(ROUND((gross_sales_mln/(SELECT SUM(gross_sales_mln) FROM CTE))*100, 2), '%') AS percentage
from cte
group by channel 
order by 3 desc;
```
--10
```
with cte as 
(select division, a.product_code, product, sum(sold_quantity) as total_sold_quantity,
dense_rank() over (partition by division order by sum(sold_quantity) desc) as rank_order
from fact_sales_monthly a
join dim_product b
on a.product_code = b.product_code
where fiscal_year = '2021'
group by division, a.product_code)
select *
from cte 
where rank_order <= 3
```
