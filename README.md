
# BI Analyst Data Challenge


## Part 1. Marketing Campaign Analysis

### Question 1. 

To link two tables together, we need a mapping table.
(1) Use your judgment based on the similarity of descriptions in campaign_name in SocialChannelAdSpend and <campaign_month, campaign_group> in SocialChannelConversion to come up with a mapping table. Your mapping table will have 3 columns, < campaign_name, campaign_month, campaign_group >.
(2) Write the mapping table to the database.

MySQL database was used for this assignment. 

**Assumption: Campaigns with the word "Test" -  are separate campaigns. 
For example `1809 Social Channel - Group 2 - Test 1` is not equivalent to `<201809, group_2_1>`**

#### SQL Code: 

```sql
create table hm_mappingyable as
select distinct 
    ca.campaign_name, 
    cc.campaign_month, 
    cc.campaign_group
from hm_socialchanneladspend as ca
inner join hm_socialchannelconversions as cc 
on 
concat(right(cc.campaign_month,4),substring(cc.campaign_group,7)) 
= 
replace(replace(concat(left(ca.campaign_name,4),substring(ca.campaign_name,29 )),'-','_'),' ','')
order by ca.campaign_name 
```

#### Mapping Table:

|campaign_name|campaign_month|campaign_group|
|---|---|---|
|1808 Social Channel - Group 1a|201808|group_1a|
|1808 Social Channel - Group 1b|201808|group_1b|
|1808 Social Channel - Group 2|201808|group_2|
|1808 Social Channel - Group 3a|201808|group_3a|
|1808 Social Channel - Group 3b|201808|group_3b|
|1808 Social Channel - Group 3c|201808|group_3c|
|1808 Social Channel - Group 3d|201808|group_3d|
|1809 Social Channel - Group 3a-1|201809|group_3a_1|
|1809 Social Channel - Group 3b-1|201809|group_3b_1|
|1809 Social Channel - Group 3c-1|201809|group_3c_1|
|1809 Social Channel - Group 3d-1|201809|group_3d_1|
|1809 Social Channel - Group 3s -2|201809|group_3s_2|
|1809 Social Channel - Group 4|201809|group_4|



### Question 2.

With the help of the mapping table you saved to the database, calculate cost per applied, cost per offered, cost per offer accepted, and cost per funded loan at the campaign level. Please write down the SQL queries you used. The goal here is to generate a summary table with campaign_name and all of our cpc metrics: cpa, cpo, cpoa, and cpfl.

#### SQL Code

```sql
  select 
    m.campaign_name, 
    s.totalspend as totalspend,
    round(s.totalspend/cp.applied,2) as cpa, 
    round(s.totalspend/cp.offered,2) as cpo, 
    round(s.totalspend/cp.offeraccepted,2) as cpoa, 
    round(s.totalspend/cp.funded,2) as cpf   
  from hm_mappingtable as m
  left join 
    (
    select 
        ca.campaign_name, 
        sum(ca.spend) as totalspend 
    from hm_socialchanneladspend as ca
    group by ca.campaign_name
    ) as s 
  on m.campaign_name = s.campaign_name
    
  left join 
    (
    select 
        concat(cc.campaign_month,cc.campaign_group ) as campaignkey, 
        sum(cc.applied) as applied, 
        sum(cc.offered) as offered, 
        sum(cc.offer_accepted) as offeraccepted, 
        sum(cc.funded) as funded 
    from hm_socialchannelconversions as cc
    group by concat(cc.campaign_month,cc.campaign_group)
    ) as cp 
  on cp.campaignkey = concat(m.campaign_month,m.campaign_group)
```

### Table With Performance Metrics and Tptal Spend

|campaign_name|totalspend|cpa|cpo|cpoa|cpf|
|---|---|---|---|---|---|
|1808 Social Channel - Group 1a|7817|38.51|59.22|139.59|260.57|
|1808 Social Channel - Group 1b|4363|38.96|58.17|189.7|872.6|
|1808 Social Channel - Group 2|8508|61.65|102.51|243.09|500.47|
|1808 Social Channel - Group 3a|5057|3.09|5.69|14.21|28.41|
|1808 Social Channel - Group 3b|11063|26.53|39.09|124.3|257.28|
|1808 Social Channel - Group 3c|6955|44.87|67.52|204.56|409.12|
|1808 Social Channel - Group 3d|6000|142.86|193.55|461.54|750|
|1809 Social Channel - Group 3a-1|4339|10.28|14.91|42.96|83.44|
|1809 Social Channel - Group 3b-1|3220|31.57|48.06|153.33|402.5|
|1809 Social Channel - Group 3c-1|26783|393.87|569.85|1071.32|2678.3|
|1809 Social Channel - Group 3d-1|8190|431.05|630|1638|4095|
|1809 Social Channel - Group 3s -2|7116|11.84|17.88|53.91|124.84|
|1809 Social Channel - Group 4|4428|8.86|18.45|45.18|116.53|




### Question 3. 

Now that we have key CPC metrics like CPFL, we would like to use Tableau to visualize data. At minimum, the dashboard we want consists of summary table of cpc metrics we just generated, including cpa, cpo, cpoa, and cpfl by Campaign. Extra point: plots of the daily cumulative spend by campaign (either faceted by campaign or sharing the same plot, where each campaign has a trace)

#### Tableau Report

![image](https://user-images.githubusercontent.com/88731258/130311066-55c45b12-d2bf-4559-b20e-295e0d288433.png)


## Part 2: Application Funnel Analysis

### Question 4.

By each application created date, how many of the applicants reach HappyPath 5?

*Assumption: All the stages from 1 to 5 must be passed within the same application created date*

#### SQL Code: 

```sql
 select ap.applicationcreated, count(1) as countapplicants_1_5
 from 
 (
     select 
         date_format(h.application_created,'%y-%m-%d') as applicationcreated, 
         h.application_id,   
         count(1) as countrecords
     from hm_application_status_history as h 
         where h.newhappypath<=5 and h.newhappypath>1
     group by date_format(h.application_created,'%y-%m-%d'),h.application_id
 ) as ap
 where ap.countrecords=4
 group by ap.applicationcreated 
```


|applicationcreated	| countapplicants_1_5 |
|---|---|
|2015-01-02	| 56 |
|2015-01-03	| 218 |
|2015-01-04	| 127 |



What is the conversion rate from HappyPath1 to HappyPath5 by each application created date?


#### SQL Code: 

```sql
 select 
    f1.dateappcreated,
    f1.path_5_appcount,
    f1.path_1_appcount,
    f1.conversion_percent_1_5
 from 
     (
     select 
        f.last_newhappypath, 
        f.dateappcreated,
        f.newhappypathfunnel as path_5_appcount,
        lead(f.newhappypathfunnel,1) over (partition by f.dateappcreated order by f.last_newhappypath desc) as path_1_appcount,
        round((f.newhappypathfunnel / lead(f.newhappypathfunnel,1) over (partition by f.dateappcreated order by f.last_newhappypath desc))*100,2) 
        as conversion_percent_1_5
    from 
    (
    select 
          cp.last_newhappypath, 
          cp.count_last_newhappypath,
          cp.dateappcreated,
          sum( cp.count_last_newhappypath) over( partition by cp.dateappcreated order by cp.last_newhappypath desc  ) as newhappypathfunnel
    from 
          (
          select 
              m.last_newhappypath, count(1) as count_last_newhappypath, m.dateappcreated
          from 
              (
              select
                  h.APPLICATION_ID,
                  max(h.NEWHAPPYPATH) as last_newhappypath,
                  date_format(h.application_created,'%Y-%m-%d') as dateappcreated
               from hm_application_status_history as h
               group by h.APPLICATION_ID, date_format(h.application_created,'%y-%m-%d')
               ) as m
          group by m.last_newhappypath, m.dateappcreated
          ) as cp  
          order by cp.last_newhappypath 
    ) as f 
    where f.last_newhappypath in (1,5) 
    order by  f.dateappcreated, f.last_newhappypath
) as f1 
where f1.last_newhappypath = 5
```


|dateappcreated	|path_5_appcount	|path_1_appcount	|conversion_percent_1_5|
|---|---|---|---|
|2015-01-02	|56	|784	|7.14|
|2015-01-03	|218	|2502	|8.71|
|2015-01-04	|127	|1549	|8.20|



### Question 5

In which HappyPath do the applicants drop the most? Please write SQL queries below.

Answer: At 70.75% Happy Path 1 has the the highest applicants drop rate. 

#### SQL Code: 

```sql
select 
    f.last_newhappypath as status, 
    -- f.count_last_newhappypath as count_current_newhappypath, 
    f.newhappypathfunnel as count_apps, 
    lag(f.newhappypathfunnel,1) over (order by f.last_newhappypath desc) as reached_next_status,
    cast(ifnull((1- lag(f.newhappypathfunnel,1) over (order by f.last_newhappypath desc)/ f.newhappypathfunnel)*100,'') as decimal(10,2)) as drop_percent
from 
    (
    select 
          cp.last_newhappypath, 
          cp.count_last_newhappypath,
          sum( cp.count_last_newhappypath) over( order by cp.last_newhappypath desc) as newhappypathfunnel
      from 
          (
            select 
                  m.last_newhappypath, count(1) as count_last_newhappypath 
             from 
                (
                 select 
                        h.APPLICATION_ID,
                        max(h.NEWHAPPYPATH) as last_newhappypath
                   from hm_application_status_history as h
                 group by h.APPLICATION_ID
                ) as m 
            group by m.last_newhappypath
          ) as cp 
    order by cp.last_newhappypath
    ) as f order by f.last_newhappypath
```


|status|count_apps|reached_next_status|drop_percent|
|---|---|---|---|
|1|4835|1414|70.75|
|2|1414|605|57.21|
|3|605|491|18.84|
|4|491|401|18.33|
|5|401|273|31.92|
|6|273|244|10.62|
|7|244|233|4.51|
|8|233|218|6.44|
|9|218|213|2.29|
|10|213||0|


### Question 6.
If we define 90% applications from old status to new status (the next status) as mature, how long does each status take to get matured (in days)? For example, it takes X days for 90% of applicants to reach HappyPath N from HappyPath N-1.

#### SQL Code: 

```sql
select c.newhappypath, round(avg(c.date_difference)/1440,4) as average_conversion_in_days
from 
(
    select  
    a.application_id, a.newhappypath, a.date_difference,
    
    -- calculating row number for each applicants for each group of happy path 
    -- sorted by date difference between current and previous statuses
    
    row_number() over (partition by a.NEWHAPPYPATH order by date_difference, a.CREATEDDATE ) as row_num,
    
    -- calculating total number of applicants for each happy path 
    
    count(1) over (partition by a.NEWHAPPYPATH) as total_apps_per_status,
    
    -- determine applicant position in set of applicants within each happy path for further elimination of the "slowest" 10%
    
    row_number() over (partition by a.NEWHAPPYPATH order by date_difference, a.CREATEDDATE ) / 
    count(1) over (partition by a.NEWHAPPYPATH)*100 as percent_of_total_per_path   
    from 
        (   
        -- calculating date difference between current and previous statuses        
        select 
             h.application_id, h.newhappypath, 
             h.createddate,
             lag(h.createddate,1) over (partition by h.application_id order by h.newhappypath ) as prev_status_created,
             timestampdiff(minute,lag(h.createddate,1) over (partition by h.application_id order by h.newhappypath ),h.createddate) 
             as date_difference            
        from hm_application_status_history as h 
        ) as a
        where a.date_difference is not null 
) as c 
where c.percent_of_total_per_path<=90     -- selecting "fastest" 90% of aplicants for each happy path value
group by c.newhappypath
```

|newhappypath|average_conversion_in_days|
|---|---|
|2|0|
|3|0.203|
|4|0.0016|
|5|0.0352|
|6|2.5899|
|7|0.0129|
|8|0.0003|
|9|0.3439|
|10|0.0002|


### Question 7. 

Build visualizations of Q1 and Q2 using R or Python. Transform the table if needed.

#### Metrics Table from Q2. Built in R

![image](https://user-images.githubusercontent.com/88731258/130383689-29c4b502-327b-431d-9bad-c3d49af39039.png)


#### Visual of Total Spend. Built in R

![image](https://user-images.githubusercontent.com/88731258/130383155-4f96800c-578e-4442-a5fc-b7ffc256bdb1.png)


#### Visual for Conversion in Days for each Happy Path. Built in R

![image](https://user-images.githubusercontent.com/88731258/130382911-f4af0623-1ff4-40c0-9fe6-cdf5be894193.png)

