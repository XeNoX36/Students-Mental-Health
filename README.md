# Analysis on Students Mental Health in Asian Schools
## Schema

```sql
select *
from Mental_health
```
## Edit Column Names
```sql
exec sp_rename
'Mental_health.[deptype]', 'Dep_type', 'COLUMN'
```
## delete columns
```sql
alter table Mental_health
drop column japanese, english, partner,friends,profess,doctor,reli,alone
,others,internet,[ Phone];
alter table Mental_health
drop column parents, relative
```
## delete empty columns
```sql
delete from Mental_health 
where inter_dom is null
```
## 1. Are international students more likely to report depression than domestic students?
```sql
select inter_dom, count(dep) Dep_count
from Mental_health
where Dep = 'yes'
group by inter_dom
```
## 2. Does language proficiency (Japanese/English) influence mental health outcomes?
```sql
select Japanese_cate, COUNT(Dep) Depression
from Mental_health
where Japanese_cate is not null
and Dep = 'yes'
group by Japanese_cate
go
select English_cate, COUNT(Dep) Depression
from Mental_health
where English_cate is not null
and Dep = 'yes'
group by English_cate
-- student with high Eglish proficiency show a great depression tendencies
```
## 3. What are the most common causes of depression among students?
```sql
select AVG(apd) apd, AVG(AHome) AHome, AVG(APH)APH, AVG(ACS)ACS, AVG(AGuilt)AGuilt, 
AVG(AMiscell)AMiscell, AVG(ToAS)ToAS
from Mental_health
where Dep = 'yes' and Dep_type = 'major'
-- Academic stress, Miscellenous reasons, Academic Performance
```
## 4. How does support from friends, parents, professionals relate to depression severity? 
```sql
select DepSev, count(Friends_bi) Friend_Sup
from Mental_health
where Friends_bi = 'yes' 
group by DepSev
go
select DepSev, count(Parents_bi) Parent_Sup
from Mental_health
where Parents_bi = 'yes'
group by DepSev
go
select DepSev, count(Professional_bi) Prof_Sup
from Mental_health
where Professional_bi = 'yes'
group by DepSev
-- Most supports came from Parents then Friends
-- Students with Severe depression required support from partents and professionals
```
## 5. Is there a gender difference in these support-seeking behaviors?
```sql
select Gender, count(Friends_bi) Friend_Sup
from Mental_health
where Friends_bi = 'yes'
group by Gender
go
select Gender, count(Parents_bi) Parent_Sup
from Mental_health
where Parents_bi = 'yes'
group by Gender
go
select Gender, count(Professional_bi) Prof_Sup
from Mental_health
where Professional_bi = 'yes'
group by Gender;
-- in all criterias Females Utilized support than Males
```
## 6. How does duration of stay affect mental health or social connections?
```sql
with cte as(
select Stay_Cate, ToSC, 
row_number() over(partition by Stay_Cate order by Tosc desc) Row_Num
from Mental_health)
select *
from cte
where Row_Num <= 10
-- students with Medium and short stay had high social connection score
```
## 7. Does having an intimate partner or being religious reduce depression symptoms?
```sql
select ToDep, count(Religion) religion
from Mental_health
where Religion = 'yes'
group by ToDep
order by ToDep desc
go
select ToDep, count(Intimate) intimate
from Mental_health
where Intimate = 'yes'
group by ToDep
order by ToDep desc
-- Reduced depression symptoms was seen in most students who were religious and had intimate partner
```
## 8. What coping strategies are most commonly associated with lower depression severity?
```sql
select DepSev, count(Partner_bi) partners-- Professional_bi, religion_bi, Alone_bi, Internet_bi, Others_bi
from Mental_health
where DepSev = 'min' and Partner_bi = 'yes'
group by DepSev
go
select DepSev, count(Professional_bi) Professional_bi-- , religion_bi, Alone_bi, Internet_bi, Others_bi
from Mental_health
where DepSev = 'min' and Professional_bi = 'yes'
group by DepSev
go
select DepSev, count(religion_bi) religion_bi-- , , Alone_bi, Internet_bi, Others_bi
from Mental_health
where DepSev = 'min' and religion_bi = 'yes'
group by DepSev
go
select DepSev, count(Alone_bi) Alone_bi-- , , , Internet_bi, Others_bi
from Mental_health
where DepSev = 'min' and Alone_bi = 'yes'
group by DepSev
go
select DepSev, count(Internet_bi) Internet_bi-- , , , Internet_bi, Others_bi
from Mental_health
where DepSev = 'min' and Internet_bi = 'yes'
group by DepSev;
-- partner support is the most commonly associated coping strategy
```
## 9. Are students with higher academic stress also reporting higher depression?
```sql
with cte as(
select ToAS, DepSev,
ROW_NUMBER() over(order by ToAS desc)RN
from Mental_health)
select *
from cte
where RN <= 5 
-- No, Higher academic stress is not associated to higher depression 
```

