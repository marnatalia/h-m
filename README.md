# BI Analyst Data Challenge


## Part 1. Marketing Campaign Analysis

### Question 1. 

To link two tables together, we need a mapping table.
(1) Use your judgment based on the similarity of descriptions in campaign_name in
SocialChannelAdSpend and <campaign_month, campaign_group> in SocialChannelConversion
to come up with a mapping table. Your mapping table will have 3 columns, < campaign_name,
campaign_month, campaign_group >.
(2) Write the mapping table to the database.

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

With the help of the mapping table you saved to the database, calculate cost per applied,
cost per offered, cost per offer accepted, and cost per funded loan at the campaign level.
Please write down the SQL queries you used. The goal here is to generate a summary table
with campaign_name and all of our cpc metrics: cpa, cpo, cpoa, and cpfl.

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
    ) as s on m.campaign_name = s.campaign_name
    
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
    ) as cp on cp.campaignkey = concat(m.campaign_month,m.campaign_group)
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

Now that we have key CPC metrics like CPFL, we would like to use Tableau to visualize
data. At minimum, the dashboard we want consists of summary table of cpc metrics we just
generated, including cpa, cpo, cpoa, and cpfl by Campaign. Extra point: plots of the daily
cumulative spend by campaign (either faceted by campaign or sharing the same plot, where
each campaign has a trace)

#### Tableau Report

![image](https://user-images.githubusercontent.com/88731258/130311066-55c45b12-d2bf-4559-b20e-295e0d288433.png)


### Question 4.

By each application created date, how many of the applicants reach HappyPath 5?

```sql
 select 
    date_format(h.application_created ,'%y-%m-%d') as applicationcreated,
    sum(case when h.newhappypath = 5 then 1 else 0 end) as happypath_5
 from hm_application_status_history as h
 group by date_format(h.application_created,'%y-%m-%d')
 order by date_format(h.application_created,'%y-%m-%d')
```

|applicationcreated	| happypath_5 |
|---|---|
|2015-01-02	| 56 |
|2015-01-03	| 218 |
|2015-01-04	| 127 |

What is the conversion rate from HappyPath1 to HappyPath5 by each application created date?

```sql
 select ap.applicationcreated, count(1) as conversion_1_5
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

|applicationcreated	| conversion_1_5 |
|---|---|
|2015-01-02	| 56 |
|2015-01-03	| 218 |
|2015-01-04	| 127 |

