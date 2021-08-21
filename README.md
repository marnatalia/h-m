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
CREATE TABLE HM_MAPPINGTABLE AS
SELECT DISTINCT 
    CA.CAMPAIGN_NAME, 
    CC.CAMPAIGN_MONTH, 
    CC.CAMPAIGN_GROUP
FROM HM_SOCIALCHANNELADSPEND AS CA
INNER JOIN HM_SOCIALCHANNELCONVERSIONS AS CC 
    ON 
    CONCAT(RIGHT(CC.CAMPAIGN_MONTH,4),SUBSTRING(CC.CAMPAIGN_GROUP,7)) 
    = 
    REPLACE(REPLACE(CONCAT(LEFT(CA.CAMPAIGN_NAME,4),SUBSTRING(CA.CAMPAIGN_NAME,29 )),'-','_'),' ','')
    ORDER BY CA.CAMPAIGN_NAME 
```



### Question 2.

With the help of the mapping table you saved to the database, calculate cost per applied,
cost per offered, cost per offer accepted, and cost per funded loan at the campaign level.
Please write down the SQL queries you used. The goal here is to generate a summary table
with campaign_name and all of our cpc metrics: cpa, cpo, cpoa, and cpfl.

#### SQL Code

```sql
    SELECT M.CAMPAIGN_NAME, 
    S.TOTALSPEND,
    ROUND(S.TOTALSPEND/CP.APPLIED,2) AS CPA, 
    ROUND(S.TOTALSPEND/CP.OFFERED,2) AS CPO, 
    ROUND(S.TOTALSPEND/CP.OFFERACCEPTED,2) AS CPOA, 
    ROUND(S.TOTALSPEND/CP.FUNDED,2) AS CPF
    
    FROM HM_MAPPINGTABLE AS M
    LEFT JOIN 
    (
    SELECT CA.CAMPAIGN_NAME, SUM(CA.SPEND) AS TOTALSPEND FROM HM_SOCIALCHANNELADSPEND AS CA
    GROUP BY CA.CAMPAIGN_NAME
    ) AS S ON M.CAMPAIGN_NAME = S.CAMPAIGN_NAME
    
    LEFT JOIN 
    (
    SELECT CONCAT(CC.CAMPAIGN_MONTH,CC.CAMPAIGN_GROUP ) AS CAMPAIGNKEY, SUM(CC.APPLIED) AS APPLIED, SUM(CC.OFFERED) AS OFFERED, SUM(CC.OFFER_ACCEPTED) AS OFFERACCEPTED, SUM(CC.FUNDED) AS FUNDED FROM HM_SOCIALCHANNELCONVERSIONS AS CC
    GROUP BY CONCAT(CC.CAMPAIGN_MONTH,CC.CAMPAIGN_GROUP)
    ) AS CP ON CP.CAMPAIGNKEY = CONCAT(M.CAMPAIGN_MONTH,M.CAMPAIGN_GROUP)SELECT 
        M.CAMPAIGN_NAME, 
        S.TOTALSPEND,
        ROUND(S.TOTALSPEND/CP.APPLIED,2) AS CPA, 
        ROUND(S.TOTALSPEND/CP.OFFERED,2) AS CPO, 
        ROUND(S.TOTALSPEND/CP.OFFERACCEPTED,2) AS CPOA, 
        ROUND(S.TOTALSPEND/CP.FUNDED,2) AS CPF
    FROM HM_MAPPINGTABLE AS M
    LEFT JOIN 
    (
        SELECT CA.CAMPAIGN_NAME, SUM(CA.SPEND) AS TOTALSPEND 
            FROM HM_SOCIALCHANNELADSPEND AS CA
        GROUP BY CA.CAMPAIGN_NAME
    ) AS S ON M.CAMPAIGN_NAME = S.CAMPAIGN_NAME
    
    LEFT JOIN 
    (
        SELECT 
            CONCAT(CC.CAMPAIGN_MONTH,CC.CAMPAIGN_GROUP ) AS CAMPAIGNKEY, 
            SUM(CC.APPLIED) AS APPLIED, 
            SUM(CC.OFFERED) AS OFFERED, 
            SUM(CC.OFFER_ACCEPTED) AS OFFERACCEPTED, 
            SUM(CC.FUNDED) AS FUNDED 
        FROM HM_SOCIALCHANNELCONVERSIONS AS CC
        GROUP BY CONCAT(CC.CAMPAIGN_MONTH,CC.CAMPAIGN_GROUP)
    ) AS CP ON CP.CAMPAIGNKEY = CONCAT(M.CAMPAIGN_MONTH,M.CAMPAIGN_GROUP)
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

