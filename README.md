# SQL-Data-Analysis-LEGO-Database-

The project dataset is from LEGO Catalog Database- Source-https://rebrickable.com/downloads/

In this project, I used Rebrickable LEGO Data model and created database in MSSQL server to analyse real scenarios of answering questions about data using SQL. Have used advanced SQL queries to explore complex problems using JOINS,CTEs and WINDOW functions.

Used SSIS to import files into database.

Queries used:

--What are the number of parts/per theme
 create view dbo.analytics_main as (

 select s.set_num, s.name as set_nam,s.year,s.theme_id, cast(s.num_parts as numeric) num_parts,
 t.name as theme_name,
 t.parent_id,p.name as parent_theme_name
 from dbo.sets s
 left join [dbo].[themes] t
	on s.theme_id=t.id
	left join [dbo].[themes] p
	on t.parent_id=p.id)
(select * 
	from dbo.analytics_main)

WITH cte_ThemeCount
AS (
select cast(num_parts as numeric) num_parts,[dbo].[themes].name,parent_id
from [dbo].[sets]
left join [dbo].[themes]
on [dbo].[sets].[theme_id]=[dbo].[themes].id)
SELECT sum(num_parts) as Total_Number_of_parts,name FROM cte_ThemeCount
where parent_id is not null
group by name
order by 1 desc



---What is the number of parts per year
select year,sum(num_parts) as total_num_parts
from [dbo].[analytics_main]
where parent_theme_name is not null
group by year
order by 2 desc


----How may sets where created in each centuary in the dataset

select count(set_num) as total_set_num,centuary
from dbo.analytics_main
--where parent_theme_name is not null
group by centuary
order by 1 desc


---what percentage of sets ever released in the 21st centuary were Trains Themed
WITH CTE_Percentage AS(
Select count(set_num)as total_set_num,centuary,theme_name
from [dbo].[analytics_main]
where centuary = '21st_Centuary'
group by theme_name,Centuary)

select sum(total_set_num) as TotalSets,sum(sets_percentage) as Setpercentage
from(
select Centuary, theme_name,total_set_num,
sum(total_set_num) OVER()as total, Cast((1.00*total_set_num / sum(total_set_num) OVER()) as decimal(5,4))*100 sets_percentage
from CTE_Percentage)h
where theme_name like '%train%'


---What was the popular theme by year in terms of sets released in the 21st centuary
WITH CTE_Percentage AS(
Select count(set_num)as total_set_num,centuary,theme_name,year
from [dbo].[analytics_main]
where centuary = '21st_Centuary'
group by theme_name,Centuary,year)

Select * 
from(
Select count(set_num) as TotalSetNum,centuary,theme_name,
	year,ROW_NUMBER() OVER(partition by year order by count(set_num) desc) rownum
from [dbo].[analytics_main]
where centuary = '21st_Centuary' 
group by theme_name,Centuary,year) g
where rownum =1
order by TotalSetNum desc


--what is the most produced color of lego ever in terms of quantity of parts?

Create view itsdef as (
select c.id,c.name as color_name,inv.inventory_id,inv.quantity,parts.part_num,parts.name
from colors c
left join inventory_parts inv
on c.id=inv.color_id
left join parts
on inv.part_num=parts.part_num)

select max(QtySum) as result,color_name from(
select sum(quantity) as QtySum,id,color_name
from itsdef
group by id,color_name)main
group by color_name
order by result desc
