What are your risk areas? Identify and describe them.

The main risk area identified is my lack of expertise relating to data types and the operations allowed for each type


QA Process:

To prevent loss of data resulting from incorrect manipulation of columns during the data cleaning process, the raw data tables lacking primary keys, Analytics and Sessions, were created with an additional SERIAL column named id. In cases where data was lost, this column allowed for easy retrieval of raw data into the clean data tables using the following queries:

update Analytics
set column_name = rawanalytics.column_name from rawanalytics
where analytics.id = rawanalytics.id

or 

update sessions
set column_name = rawsessions.column_name from rawsessions
where sessions.id = rawsessions.id

Additionally, no operations were performed on the raw data tables after their creation, except for copying data from them to the clean data tables.

Another issue identified was running queries and not getting the expected results for example, the following query returns no values:

select * from products join sales_sku using(sku)
where products.sku in (
			select sku from sales_sku
			order by total_ordered desc
			limit 20) 
	and (total_ordered/stocklevel) > 0.5 
	and restockingleadtime > 7
order by total_ordered desc

But rewriting the query to the following returns two rows:

select * from products join sales_sku using(sku)
where products.sku in (
			select sku from sales_sku
			order by total_ordered desc
			limit 20) 
	and total_ordered > 0.5*stocklevel
	and restockingleadtime > 7
order by total_ordered desc

meaning that (total_ordered/stocklevel) was not returning the correct reuslt because both values are INTEGERS. Replacing this expression with (total_ordered::numeric/stocklevel) fixed the issue.

