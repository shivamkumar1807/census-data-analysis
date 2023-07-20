# census-data-analysis

select * from project.dbo.Data1;
select * from project.dbo.Data2;


select count(*) from project..Data1;
select count(*) from project..Data2

---dataset for Bihar and Jharkhand

select * from project..Data1 where state in ('Jharkhand' , 'Bihar');

--- population of India

select sum(population) as Population from project..Data2;

--- avg growth

select state,avg(growth)*100 as avg_growth from project..Data1 group by state;


--- sex ratio

select state,round(avg(Sex_Ratio),0) as avg_Sex_Ratio from project..Data1 group by state
order by avg_Sex_Ratio desc;


--- avg literacy rate

select state,round(avg(Literacy),0) as avg_literacy_Ratio from project..Data1
group by state having round(avg(Literacy),0)>90 order by avg_Literacy_Ratio desc;


---top 3 states showing highest growth rate

select top 3 state,avg(growth)*100 as avg_growth from project..Data1 group by state order by avg_growth desc;


---bottom 3 states showing lowest sex ratio
select top 3 state,round(avg(Sex_Ratio),0) as avg_Sex_Ratio from project..Data1 group by state
order by avg_Sex_Ratio asc;
  
 --- top and bottom 3 states in literacy state
 drop table if exists #topstates; 
 create table #topstates
 ( state nvarchar(255),
   topstates float
  )

insert into #topstates 
select state,round(avg(Literacy),0) avg_Literacy_Ratio from project..Data1 
group by state order by avg_Literacy_Ratio desc;

select top 3 * from #topstates order by #topstates.topstate desc;


 drop table if exists #bottomstates;
 create table #bottomstates
 ( state nvarchar(255),
   bottomstates float
  )

insert into #bottomstates 
select state,round(avg(Literacy),0) avg_Literacy_Ratio from project..Data1 
group by state order by avg_Literacy_Ratio desc;

select top 3 * from #bottomstates order by #bottomstates. bottomstate desc ;

--- union operator 

select * from (
select top 3 * from #topstates order by #topstates. topstate desc) as a

union

select * from (
select top 3 * from #bottomstates order by #bottomstates. bottomstate asc) as b;


--- states starting with letter a or b
select distinct state from project..Data1 where lower(state) like 'a%' or lower(state) like 'b%'


--- states ending with letter a or b
select distinct state from project..Data1 where lower(state) like '%a' or lower(state) like '%d'


--- joining both tables

--total males and females

select d.state, sum(d.males) total_males, sum(d.females) total_females from
(select c.district,c.state, round(c.population/(c.sex_ratio+1),0) males, round((c.population*c.sex_ratio)/(c.sex_ratio+1),0) females from
(select a.district,a.state,a.sex_ratio/1000 sex_ratio,b.population from project..data1 a inner join project..data2 b on a.district=b.district)c)d
group by d.state order by d.state asc;


--- total literacy rate

select c.state,sum(literate_people) total_literate_pop,sum(illiterate_people) total_lliterate_pop from 
(select d.district,d.state,round(d.literacy_ratio*d.population,0) literate_people,
round((1-d.literacy_ratio)* d.population,0) illiterate_people from
(select a.district,a.state,a.literacy/100 literacy_ratio,b.population from project..data1 a 
inner join project..data2 b on a.district=b.district) d) c
group by c.state



---population in the previous census

select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from project..data1 a inner join project..data2 b on a.district=b.district)d)e
group by e.state)m

--- population vs area
select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from project..data1 a inner join project..data2 b on a.district=b.district)d)e
group by e.state)m

select sum(area_km2) total_area from project..data2

-- population vs area

select (g.total_area/g.previous_census_population)  as previous_census_population_vs_area, (g.total_area/g.current_census_population) as 
current_census_population_vs_area from
(select q.*,r.total_area from (

select '1' as keyy,n.* from
(select sum(m.previous_census_population) previous_census_population,sum(m.current_census_population) current_census_population from(
select e.state,sum(e.previous_census_population) previous_census_population,sum(e.current_census_population) current_census_population from
(select d.district,d.state,round(d.population/(1+d.growth),0) previous_census_population,d.population current_census_population from
(select a.district,a.state,a.growth growth,b.population from project..data1 a inner join project..data2 b on a.district=b.district) d) e
group by e.state)m) n) q inner join (

select '1' as keyy,z.* from (
select sum(area_km2) total_area from project..data2)z) r on q.keyy=r.keyy)g

--window 

output top 3 districts from each state with highest literacy rate


select a.* from
(select district,state,literacy,rank() over(partition by state order by literacy desc) rnk from project..data1) a

where a.rnk in (1,2,3) order by state
