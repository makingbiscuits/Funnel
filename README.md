# Funnel Analysis


✨ ✨ Solution for Funnel Analysis task for Turing College✨ ✨ 

<a href = 'https://docs.google.com/spreadsheets/d/1Mp0oRW0kbsnRRM18v1WYsB8SQi72Mt0iFMcuzpMdUek/edit?usp=sharing
'> The dashboard with visualisations.</a> It explains the main insights of the funnel analysis and provides a solution of the eCommerce Conversion and Buying funnels.

:rocket: SQL (BigQuery), Excel, Google Sheets, Funnel Analysis

### The task:

Your task is to build useful funnel chart from raw_events table data.
1. Analyze the data in raw_events table. Spend time querying the table, getting more familiar with data. Identify events captured by users visiting the website.
* The data in raw_events table captures a lot of events from users based on their timestamps. This can be useful for a number of analysis. However sometimes more data does not help as it can inflate the data. I.e. if we want to see how many users have gone to the checkout the one user who may have gone back and forth to checkout for 8 times can be dangerous when building a funnel chart as it can overpresent how many users in total get to the checkout. Always be mindful of duplicate data and how it can affect your analysis. 
* First Order of the business is to write a query that eliminates the duplicates for our funnel analysis.
The query should contain all of the columns from raw_events table. It should however only have 1 unique event per user_pseudo_id.
* In case of duplicate events the chosen event should be the first one (based on event_timestamp)

2. Now that you have your unique events table create a sales funnel chart from events in it. Not all events are relevant, productive to be used in this chart. Identify & collect data that you think could be used
* Use between 4 to 6 types of events in this analysis.
* Create a funnel chart with a category split. Business is interested in the differences between top 3 countries in the funnel chart.
* Top countries are decided by their overall number of events.
* Provide insights if any found.
* See if you can come up with any other ideas/slices for funnel analysis that could be worth a look.

### Main SQL Query:

``` SQL 
WITH unique_table AS 
(
  WITH unique_ev AS 
(
  SELECT user_pseudo_id, 
    event_name,
    MIN(event_timestamp) AS event_timestamp
  FROM `tc-da-1.turing_data_analytics.raw_events` 
  GROUP BY 1,2
  ORDER BY 1,2
)

SELECT unique_ev.*, 
  re.event_date, 
  re.event_value_in_usd, 
  re.user_id, 
  re.category, 
  re.total_item_quantity,
  re.purchase_revenue_in_usd,
  re.campaign
FROM unique_ev
JOIN `tc-da-1.turing_data_analytics.raw_events` AS re
ON unique_ev.user_pseudo_id = re.user_pseudo_id
AND unique_ev.event_name = re.event_name
AND unique_ev.event_timestamp = re.event_timestamp
ORDER BY 1,2
)

SELECT country as top_country,
  COUNT (event_name) AS total_events,
  SUM(CASE WHEN event_name = 'page_view' THEN 1 ELSE 0 END) AS page_view,
  SUM(CASE WHEN event_name = 'scroll' THEN 1 ELSE 0 END) AS scroll,
  SUM(CASE WHEN event_name = 'add_to_cart' THEN 1 ELSE 0 END) AS add_to_cart,
  SUM(CASE WHEN event_name = 'purchase' THEN 1 ELSE 0 END) AS purchase
FROM unique_table
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3
```
