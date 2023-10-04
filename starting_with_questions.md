Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
select country, city, sum(totalTransactionRevenue) 
	from sessions
	group by country, city
	having sum(totalTransactionRevenue) > 0
order by sum(totalTransactionRevenue) desc


Answer:
The United States generates by far the highest level of transaction revenue, followed by Israel, Australia, Canada and Switzerland.However, most of the transactions don't have city information, and is thus impossible to accurately determine which cities generate the most revenue. 



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
select city, country, avg(units_sold) avg_sold 
	from sessions join analytics 
	on sessions.visitid = analytics.visitid 
	and sessions.fullvisitorid = analytics.fullvisitorid
	group by city, country
	having avg(units_sold) is not null
order by avg_sold desc



Answer:
The city of Chicago, US has the highest average (5 units) of units of products ordered per visitor. However, if only countries are considered, then Canada has the highest average at 2.625 units of products ordered per visitor. Notably, 105 out of 135 countries in the dataset have no products ordered at all.




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
------------for countries:
with tmp as (
	select country, v2ProductCategory, count(v2ProductCategory) catcount, 
	rank() over(partition by country order by count(v2ProductCategory) desc) from sessions
	where v2ProductCategory is not null
	group by country, v2ProductCategory
	order by country, catcount desc, v2ProductCategory)
		select country, v2ProductCategory most_popular_category from tmp
		where rank = 1
	order by country

------------for cities:
with tmp as (
	select city, v2ProductCategory, count(v2ProductCategory) catcount, 
	rank() over(partition by city order by count(v2ProductCategory) desc) 
	from sessions
	where v2ProductCategory is not null
	group by city, v2ProductCategory
	order by city, catcount desc, v2ProductCategory)
		select city, v2ProductCategory most_popular_category from tmp
		where rank = 1
	order by city

Answer: The product category "Home" is the most popular category in every country and every city, with the exception of Nashville, United States, having "Nest-USA" as its most popular product category.


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
with tmp as (
	select country, sku, v2ProductName prod_name, sum(units_sold) total_sold, rank() over(partition by country order by sum(units_sold) desc ) 
		from sessions join analytics 
			on sessions.visitid = analytics.visitid 
			and sessions.fullvisitorid = analytics.fullvisitorid
		group by country, sku, v2ProductName
		having sum(units_sold) is not null
	order by country)
	select country, prod_name, total_sold from tmp
		where rank = 1
	order by prod_name



Answer:
By far the highest selling product in the dataset and in the United States is "SPF-15 Slim & Slender Lip Balm". Other noteworthy products are "26 oz Double Wall Insulated Bottle" which is the top-selling product in Denmark and Thailand, and "YouTube Men's 3/4 Sleeve Henley" which is the top-selling product in Colombia and Switzerland.




**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
with tmp as (
	select country, visitid, max(totaltransactionrevenue) total_revenue 
		from sessions
		where totaltransactionrevenue is not null
		group by visitid, country)
	select country, sum(total_revenue) total_revenue_country, 
		sum(total_revenue)*100/(select sum(total_revenue) from tmp) percentage_revenue 
		from tmp
		group by country
	order by total_revenue_country desc



Answer:
The United States represents 92% of all the revenue generated from product sales, with Israel coming as a distant second, representing 4.25% of the total revenue. Considering this, it would be advisable to focus most of advertising expenses in the United States.






