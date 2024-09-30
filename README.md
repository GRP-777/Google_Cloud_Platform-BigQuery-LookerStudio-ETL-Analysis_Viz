
# Google Ads USD Spending & ROI in 2024 Presidential Campaigns

![USA_Federal_Political_Google_Ads_2024_page-0001](https://github.com/user-attachments/assets/6cc5a737-7ddf-483b-98a6-73d5c06f423f)

## Tech stack

- BigQuery
- Looker Studio


## Disclaimer
This analysis is objective and does not intend to reflect the political biases of the data analyst. The findings presented here are unbiased and can be interpreted to show positive aspects for either candidate depending on the perspective.

## Project Introduction
An explanation of the fundamental elements and procedures involved in the analysis. A diagram illustrating the technologies used, data flow, and expected outcomes would be beneficial.

## Research Questions
The primary focus will be the relationship between investment and gross results.

    1. How much money has been spent on Google Ads for each candidate?
    2. What gross results (in terms of impressions) have they achieved based on this expenditure?

## Dataset Location and Availability
- The primary dataset is located in bigquery-public-data.google_political_ads.creative_stats.
- Key numerical columns include spend_range_max_usd and impressions.
- The categorical column for candidate identification is advertiser_name.
- The analysis will focus on ads starting from date_range_start.

## ETL in BigQuery

![image](https://github.com/user-attachments/assets/00611fa1-ad14-49b5-876b-86be99c87e6c)

### Data Cleaning

#### Advertiser Identification
Using DISTINCT and WHEN clauses, advertiser_name is identified to filter for Democratic and Republican candidates based on keywords and expressions.

#### Amount spent
While spend_range_max_usd is not the sole indicator of campaign spending, it provides the closest approximation of the total expenditures. Only the highest range is considered after a trim procedure.

#### Date Filtering
Ads from Januay 22, 2024, to September 30, 2024 are considered.

#### Order by Expenditures
ORDER BY function helps us to be able to see the max spendings on top to have a quick view of estimates.

#### Cleaning view schema
     fine-volt-436819-u5.dm1.khdtclean2


#### Query: selecting, filtering and cleaning columns 
        
        CREATE VIEW IF NOT EXISTS fine-volt-436819-u5.dm1.khdtclean2 AS
        SELECT
        CASE
            WHEN advertiser_name LIKE 'HARRIS%' THEN 'KAMALA HARRIS'
            WHEN advertiser_name LIKE 'KAMALA%' THEN 'KAMALA HARRIS'
            WHEN advertiser_name LIKE 'TRUMP%' THEN 'DONALD TRUMP'
            WHEN advertiser_name LIKE 'DONALD%' THEN 'DONALD TRUMP'
            WHEN advertiser_name LIKE 'GREAT%' THEN 'DONALD TRUMP'
            ELSE advertiser_name
        END AS candidate,
        DATE(date_range_start) > DATE('2024-01-22') AS current_elections,
        CASE
            WHEN impressions NOT IN ('-') THEN CAST(REPLACE(REGEXP_EXTRACT(impressions, r'-\d+'),'-', '') AS INTEGER)
            ELSE CAST(impressions AS INTEGER)
        END AS impressions,
        spend_range_max_usd
        FROM bigquery-public-data.google_political_ads.creative_stats
        WHERE DATE(date_range_start) > DATE('2024-01-22') 
        AND (advertiser_name LIKE 'HARRIS%' OR advertiser_name LIKE 'KAMALA%' OR advertiser_name LIKE 'TRUMP%' OR advertiser_name LIKE 'DONALD%' OR advertiser_name LIKE 'GREAT%')
        ORDER BY spend_range_max_usd DESC;



### Data Modeling
#### SUM
Sum of the USD amount spent per candidate, this gives a total for each one. 
#### Average
To assess campaign performance more accurately, calculate the average impressions per candidate. Summing total impressions alone may not provide a clear picture. By using the average, we can get a better approximation of a form of ROI.

#### Aggregation view schema
     fine-volt-436819-u5.dm1.khdtgrouped2
     
#### Query: aggregating columns
        CREATE VIEW IF NOT EXISTS fine-volt-436819-u5.dm1.khdtgrouped2 AS
        SELECT candidate, 
        SUM(spend_range_max_usd) total_max_spend, 
        CAST(AVG(impressions) as INT64) average_impressions
        FROM fine-volt-436819-u5.dm1.khdtclean2
        GROUP BY candidate;

## Data Visualization in Looker Studio

![image](https://github.com/user-attachments/assets/9f421901-2d8b-427f-afe7-a9ec1a2ca749)

- *Data Connection*: The BigQuery table is connected to Looker Studio. Google Cloud Platform offers an interconnected environment, so this connection was easily performed.
- *Viz explanation*: Simple charts (e.g., bar charts, scorecards) are used to compare investment and performance, making the results easy to understand.

## Results

![image](https://github.com/user-attachments/assets/c480f46e-ce24-478f-897c-62b17955d8e4)

    1. Spending on Kamala Harris is three times higher than on Donald Trump.
    2. Donald Trump has five times more impressions than Kamala Harris.


## Limitations of the Analysis:
- *Advertiser Identification*: The analysis relies on keyword matching for advertiser identification, which may not capture all instances, even though a detailed ETL was done.
- *Platform Limitations*: The analysis is limited to Google Ads and does not consider other platforms like social media, newspapers, and others.
- *Time Period*: The analysis covers July 22, 2024 to September 30, 2024, and results may vary for different time periods.

## Professional Contact:
- LinkedIn: https://www.linkedin.com/in/german-robles-perez/
- GitHub: https://github.com/GRP-777
- Email: groblesperez0@gmail.com
