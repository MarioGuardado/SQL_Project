What issues will you address by cleaning the data?

The main issues with this dataset are values formatted incorrectly, namely prices, revenues, and dates. Additionally, the data set contains in many of its columns a lot of irrelevant values like "(not set)" and "not available in demo dataset". These should be replaced by nulls in order to exclude them from calculations and operations performed on the dataset.


Queries:
----------------------------------------------------------CREATION OF RAW DATA TABLES:
create table rawsessions(
	id serial,
	fullVisitorId varchar(25),
	channelGrouping varchar(20),
	"time" int,
	country varchar(25),
	city varchar(50),
	totalTransactionRevenue int,
	transactions smallint,
	timeonSite smallint,
	pageviews smallint,
	sessionQualityDim smallint,
	ddate int,
	visitId int,
	"type" varchar(6),
	productRefundAmount smallint,
	productQuantity smallint,
	productPrice int,
	productRevenue int,
	SKU varchar(40),
	v2ProductName varchar(70),
	v2ProductCategory varchar(100),
	productVariant varchar(30),
	currencyCode varchar(5),
	itemQuantity smallint,
	itemRevenue int,
	transactionRevenue int,
	transactionId varchar(40),
	pageTitle varchar,
	searchKeyword varchar(10),
	pagePathLevel1 varchar(40),
	eCommerceAction_type smallint,
	eCommerceAction_step smallint,
	eCommerceAction_option varchar(30),
	primary key (id))

create table rawanalytics(
	id serial,
	visitNumber smallint,
	visitId int,
	visitStartTime int,
	"ddate" int,
	fullvisitorId varchar(25),
	userid smallint,
	channelGrouping varchar(20),
	socialEngagementType varchar(20),
	units_sold smallint,
	pageviews smallint,
	timeonsite smallint,
	bounces smallint,
	revenue bigint,
	unit_price int)

create table rawproducts(
	SKU varchar(30) primary key,
	productname varchar(100),
	orderedQuantity int,
	stockLevel int,
	restockingLeadTime smallint,
	sentimentScore real,
	sentimentMagnitude real)

create table rawsales_sku(
	SKU varchar(30) primary key,
	total_ordered smallint)

create table rawsales_report(
	sku varchar(30) primary key,
	total_ordered smallint,
	productname varchar(100),
	stockLevel smallint,
	restockingLeadTime smallint,
	sentimentScore real,
	sentimentMagnitude real,
	ratio real)


----------------------------------------------------------FOR TABLE SESSIONS:

--CREATE CLEAN DATA TABLE "SESSIONS"
create table sessions as 
select * from rawsessions;

--FIND IRRELEVANT "COUNTRY" VALUES:
select distinct country from sessions where country not similar to '[[:alpha:] ]{0,}';

--REMOVE IRRELEVANT "COUNTRY" VALUES
update sessions
set country = case when country = '(not set)' then null
					when country = 'Macedonia (FYROM)' then 'Macedonia'
					when country = 'Myanmar (Burma)' then 'Myanmar'
					else country
					end;

--FIND IRRELEVANT "CITY" VALUES:
select distinct city from sessions 
where city not similar to '[[:alpha:] ]{0,}';

--REMOVE IRRELEVANT "CITY" VALUES
update sessions
set city = null
where city = '(not set)' or city = 'not available in demo dataset';

--FORMAT DOLLAR VALUES

--CHANGE COLUMN DATA TYPE TO REAL
alter table sessions
alter column totalTransactionRevenue type real,
alter column productPrice type real,
alter column productRevenue type real,
alter column transactionRevenue type real;

--DIVIDE COLUMNS VALUES BY 1000000
update sessions
set (totalTransactionRevenue,productPrice,productRevenue,transactionRevenue) =
(totalTransactionRevenue/1000000,productPrice/1000000,productRevenue/1000000,transactionRevenue/1000000);

--FORMAT DATE VALUES

--CHANGE COLUMN DATA TYPE TO TEXT
alter table sessions
alter column ddate type text;

--FORMAT TEXT DATES TO DATE DATA TYPE
alter table sessions
alter column ddate type date
using(ddate::date)

--REMOVE REDUNDANT/IRRELEVANT COLUMNS (productRefundAmount, currencyCode, itemQuantity, itemRevenue, searchKeyword )
alter table sessions
drop column productRefundAmount, 
drop column currencyCode, 
drop column itemQuantity, 
drop column itemRevenue,
drop column searchKeyword;

--CONSOLIDATE PRODUCT CATEGORIES
update sessions
set v2ProductCategory = 
case when v2ProductCategory like '%/' 
		then regexp_replace(v2ProductCategory,'/[[:alnum:][:space:][:punct:]]{0,}','','g')
	when v2ProductCategory = '(not set)' then null
	when v2ProductCategory = '${escCatTitle}' then null
	else v2ProductCategory
	end
-------------------------------------------------------FOR TABLE ANALYTICS:

--CREATE CLEAN DATA TABLE "ANALYTICS"
create table analytics as 
select * from rawanalytics;

--FORMAT DATE VALUES

--CHANGE COLUMN DATA TYPE TO TEXT
alter table analytics
alter column ddate type text;

--FORMAT TEXT DATES TO DATE DATA TYPE
alter table analytics
alter column ddate type date
using(ddate::date)

--FORMAT DOLLAR VALUES

--CHANGE COLUMN DATA TYPE TO REAL
alter table analytics
alter column revenue type real,
alter column unit_price type real;

--DIVIDE COLUMNS VALUES BY 1000000
update analytics
set (revenue,unit_price) = (revenue/1000000,unit_price/1000000);

--REMOVE REDUNDANT/IRRELEVANT COLUMNS (visitStartTime, userid, socialEngagementType, bounces)
alter table analytics
drop column visitStartTime, 
drop column userid, 
drop column socialEngagementType, 
drop column bounces;


-------------------------------------------------------FOR TABLE PRODUCTS:
--CREATE CLEAN DATA TABLE "PRODUCTS"
create table products as 
select * from rawproducts;

--remove leading spaces from productname
update products
set productname = regexp_replace(productname,' ','')
where productname like ' %'


-------------------------------------------------------FOR TABLE SALES_REPORT:
--CREATE CLEAN DATA TABLE "SALES_REPORT"
create table sales_report as 
select * from rawsales_report;

--remove leading spaces from productname
update sales_report
set productname = regexp_replace(productname,' ','')
where productname like ' %'


-------------------------------------------------------FOR TABLE SALES_BY_SKU:
--CREATE CLEAN DATA TABLE "PRODUCTS"
create table sales_sku as 
select * from rawsales_sku;