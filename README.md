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
create table hm_MappingTable as
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



### Question 2.

With the help of the mapping table you saved to the database, calculate cost per applied,
cost per offered, cost per offer accepted, and cost per funded loan at the campaign level.
Please write down the SQL queries you used. The goal here is to generate a summary table
with campaign_name and all of our cpc metrics: cpa, cpo, cpoa, and cpfl.

#### SQL Code

```sql
    select 
        m.campaign_name, 
        s.totalspend as TotalSpend,
        round(s.totalspend/cp.applied,2) as CPA, 
        round(s.totalspend/cp.offered,2) as CPO, 
        round(s.totalspend/cp.offeraccepted,2) as CPOA, 
        round(s.totalspend/cp.funded,2) as CPF   
    from hm_MappingTable as m
    left join 
    (
        select 
            ca.campaign_name, 
            sum(ca.spend) as totalspend 
        from hm_SocialChannelAdSpend as ca
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
        from hm_SocialChannelConversions as cc
        group by concat(cc.campaign_month,cc.campaign_group)
    ) as cp on cp.campaignkey = concat(m.campaign_month,m.campaign_group)
```



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
    date_format(h.APPLICATION_CREATED ,'%Y-%m-%d') as ApplicationCreated,
    sum(case when h.NEWHAPPYPATH = 5 then 1 else 0 end) as CountStatus_5
 from hm_Application_Status_History as h
 group by date_format(h.APPLICATION_CREATED,'%Y-%m-%d')
 order by date_format(h.APPLICATION_CREATED,'%Y-%m-%d')
```

|ApplicationCreated	| CountStatus_5 |
|---|---|
|2015-01-02	| 56 |
|2015-01-03	| 218 |
|2015-01-04	| 127 |

What is the conversion rate from HappyPath1 to HappyPath5 by each application created date?

```sql
 select ap.ApplicationCreated, count(1) as Conversion_1_5
 from 
 (
     select 
         date_format(h.APPLICATION_CREATED,'%Y-%m-%d') as ApplicationCreated, 
         h.APPLICATION_ID,   
         count(1) as CountRecords
     from hm_Application_Status_History as h 
         where h.NEWHAPPYPATH<=5 and h.NEWHAPPYPATH>1
     group by date_format(h.APPLICATION_CREATED,'%Y-%m-%d'),h.APPLICATION_ID
 ) as ap
 where ap.CountRecords=4
 group by ap.ApplicationCreated
```

|ApplicationCreated	| Conversion_1_5 |
|---|---|
|2015-01-02	| 56 |
|2015-01-03	| 218 |
|2015-01-04	| 127 |

