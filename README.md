# Analysis on Students Mental Health in Asian Schools
## Project Description
```sql
inter_dom – Student type: International (Inter) or Domestic.
Region – Geographical region of origin (e.g., SEA = Southeast Asia, EA = East Asia).
Gender – Gender identity (e.g., Male, Female).
Academic – Academic level (e.g., Grad = Graduate).
Age – Student’s age.
Age_cate – Categorized age (numerical bin).
Stay – Duration of stay in the country (in years).
Stay_Cate – Categorical version of stay duration (e.g., Short, Long).
Japanese – Japanese language proficiency score.
Japanese_cate – Categorized Japanese proficiency (e.g., Low, Average, High).
English – English language proficiency score.
English_cate – Categorized English proficiency.
Intimate – Whether the student has an intimate partner.
Religion – Whether the student is religious.
Suicide – Whether the student has had suicidal thoughts.
Dep – Whether the student has experienced depression.
DepType – Type of depression experienced.
ToDep – Total number of depressive symptoms.
DepSev – Severity level of depression.
ToSC – Total social connection score.
APD – Depression related to academic performance.
AHome – Depression related to homesickness.
APH – Depression related to physical health.
Afear – Depression related to fear/anxiety.
ACS – Academic coping score.
AGuilt – Guilt-related depression.
AMiscell – Miscellaneous reasons for depression.
ToAS – Total academic stress.
Partner_bi – Binary (Yes/No) version of partner support.
Friends_bi – Binary version of friend support.
Parents_bi – Binary version of parent support.
Relative_bi – Binary version of relative support.
Professional_bi – Binary version of professional support.
Phone_bi – Binary version of phone support.
Doctor_bi – Binary version of doctor support.
religion_bi – Binary version of religion support.
Alone_bi – Binary version of coping alone.
Others_bi – Binary version of other support.
Internet_bi – Binary version of internet support.
```
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
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/MH%20.png)

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
```
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/MH2.png)
```sql
-- student with high English proficiency show a great depression tendencies
```


## 3. What are the most common causes of depression among students?
```sql
select AVG(apd) apd, AVG(AHome) AHome, AVG(APH)APH, AVG(ACS)ACS, AVG(AGuilt)AGuilt, 
AVG(AMiscell)AMiscell, AVG(ToAS)ToAS
from Mental_health
where Dep = 'yes' and Dep_type = 'major'
```
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/MH3.png)
```sql
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
```
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/MH5.png)
```sql
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
```
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/MH6.png)
```sql
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
```
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/MH%207.png)
```sql
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
```
![](https://github.com/XeNoX36/Students-Mental-Health/blob/main/Mental%20Health%209.png)
```sql
-- No, Higher academic stress is not associated to higher depression 
```
