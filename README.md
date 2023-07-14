# CASE STUDY: Maven Fuzzy Factory Analysis

##### Author: Michaen Ho


#

_The case study follows the six step data analysis process:_

###  ❓ [Ask](#1-ask)
### 💻 [Understand](#2-understand)
### 🛠  [Process](#3-process)
### 📊 [Analyze](#4-analyze)
### 🚲 [Act](#5-act)
### 📋 [Reflect](#6-reflect)

## Scenario
Maven Fuzzy Factory is ready to secure its next round of funding and we are required to tell a compelling story to investors. I, as a data analyst in the team, is tasked to pull the relevant data, and help my CEO craft a story about a data-driven company that has been producing rapid growth.

## 1. Ask
💡 **BUSINESS TASK: Extract and analyse traffic and website performance data to craft a growth story that we could sell to potential investors.**

Primary stakeholders: Potential investors of Maven Fuzzy Factory

## 2. Understand
Data Source: A custom built database named mavenfuzzyfactory that closely resembles the databases found in e-commerce companies

Tables: 
  - order_item_refunds
  - order_items
  - orders
  - products
  - website_pageviews
  - website_sessions
    

The database contains 6 tables containing information such as users' website sessions, their orders, and their refunds. It is a big dataset that contains records of about 400,000 users. Also, it is comprehensive as it includes the actual time when they enter every webpages, utm trackers to track the performance of campaigns and contents, devices that the users use to enter the websites, as well as all the products that they have bought.



⛔ The dataset has limitations:

Since it is a custom built database, the data used is synthetic (data privacy is a concern)



## 3. Process

The datasets have already been pre-processed; hence, there is no need for any data cleaning.

## 4. Analyze

1. Obtain overall session and order volumes for every quarter

```
SELECT
  YEAR(ws.created_at) AS yr,
  QUARTER(ws.created_at) AS qr,
  COUNT(DISTINCT ws.website_session_id) AS total_sessions,
  COUNT(DISTINCT o.order_id) AS total_orders
FROM website_sessions ws
	LEFT JOIN 
    orders o
      ON ws.website_session_id = o.website_session_id
GROUP BY 1,2;
```
<img width="1061" alt="Screenshot 2023-07-12 at 2 00 31 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/4cd30f3c-5264-4800-a34e-14a7a100e3de">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q1_16889764360750/Sheet1?:language=en-US&:display_count=n&:origin=viz_share_link)**





2. Obtain overall sessions to orders conversion rate, revenue per order and revenue per sessions for every quarter
```
SELECT
	YEAR(ws.created_at) AS yr,
	QUARTER(ws.created_at) AS qr,
    	COUNT(o.order_id) / COUNT(ws.website_session_id) AS conv_rate,
    	SUM(o.price_usd) / COUNT(o.order_id) AS revenue_per_order,
    	SUM(o.price_usd) / COUNT(ws.website_session_id) AS revenue_per_session
FROM website_sessions ws
	LEFT JOIN orders o
		ON ws.website_session_id = o.website_session_id
GROUP BY 1,2;							
```
<img width="1057" alt="Screenshot 2023-07-12 at 2 36 16 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/6cedb4aa-4b2b-4f2f-ac45-297b5f659c66">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q2_16892590097760/Sheet2?:language=en-US&:display_count=n&:origin=viz_share_link)**


3. Obtain the overall number of orders from different channels for every quarter

```
SELECT
    YEAR(ws.created_at) AS yr,
    QUARTER(ws.created_at) AS qr,
    COUNT(CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) AS g_nonbrand_orders,
    COUNT(CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) AS b_nonbrand_orders,
    COUNT(CASE WHEN ws.utm_campaign = 'brand' AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) AS brand_orders,
    COUNT(CASE WHEN ws.utm_source IS NULL AND ws.http_referer IS NOT NULL AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) AS o_search_orders,
    COUNT(CASE WHEN ws.utm_source IS NULL AND ws.http_referer IS NULL AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) AS direct_orders
FROM website_sessions ws
    LEFT JOIN orders o
        ON ws.website_session_id = o.website_session_id
GROUP BY 1,2;
```
<img width="1055" alt="Screenshot 2023-07-13 at 3 32 52 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/da2ac429-076a-4627-a07d-549e2d495945">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q3_1_16892590730100/Sheet3?:language=en-US&:display_count=n&:origin=viz_share_link)**

<img width="1056" alt="Screenshot 2023-07-13 at 3 31 41 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/57d1ea5f-4e02-46c5-a78a-f5457f9a63a1">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q3_2_16892591111130/Sheet4?:language=en-US&:display_count=n&:origin=viz_share_link)**



4. Obtain the overall session to order conversion rate for the different channels for every quarter

```
SELECT
    YEAR(ws.created_at) AS yr,
    QUARTER(ws.created_at) AS qr,
    COUNT(CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) / COUNT(CASE WHEN ws.utm_source = 'gsearch' AND ws.utm_campaign = 'nonbrand' THEN 1 ELSE NULL END) AS g_nonbrand_conv_rate,
    COUNT(CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) / COUNT(CASE WHEN ws.utm_source = 'bsearch' AND ws.utm_campaign = 'nonbrand' THEN 1 ELSE NULL END) AS b_nonbrand_conv_rate,
    COUNT(CASE WHEN ws.utm_campaign = 'brand' AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) / COUNT(CASE WHEN ws.utm_campaign = 'brand' THEN 1 ELSE NULL END) AS brand_orders_conv_rate,
    COUNT(CASE WHEN ws.utm_source IS NULL AND ws.http_referer IS NOT NULL AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) / COUNT(CASE WHEN ws.utm_source IS NULL AND ws.http_referer IS NOT NULL THEN 1 ELSE NULL END) AS o_search_conv_rate,
    COUNT(CASE WHEN ws.utm_source IS NULL AND ws.http_referer IS NULL AND o.order_id IS NOT NULL THEN 1 ELSE NULL END) / COUNT(CASE WHEN ws.utm_source IS NULL AND ws.http_referer IS NULL THEN 1 ELSE NULL END) AS direct_conv_rate
FROM website_sessions ws
    LEFT JOIN orders o
        ON ws.website_session_id = o.website_session_id
GROUP BY 1,2;
```
<img width="1058" alt="Screenshot 2023-07-13 at 3 23 09 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/76f18963-9815-4e0c-adcd-cee395fc0c52">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q4_1_16892591746640/Sheet5?:language=en-US&:display_count=n&:origin=viz_share_link)**

<img width="1058" alt="Screenshot 2023-07-13 at 3 28 15 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/5db16825-64ae-4b91-b438-f053764aa6a6">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q4_2_16892592172070/Sheet6?:language=en-US&:display_count=n&:origin=viz_share_link)**


5. Obtain total monthly revenues and profits, as well as monthly revenues and profits for different products

```
SELECT 
    YEAR(created_at) AS yr,
    MONTH(created_at) AS mo,
    SUM(CASE WHEN product_id = 1 THEN price_usd ELSE 0 END) AS revenue_1,
    SUM(CASE WHEN product_id = 1 THEN price_usd - cogs_usd ELSE 0 END) AS margin_1,
    SUM(CASE WHEN product_id = 2 THEN price_usd ELSE 0 END) AS revenue_2,
    SUM(CASE WHEN product_id = 2 THEN price_usd - cogs_usd ELSE 0 END) AS margin_2,
    SUM(CASE WHEN product_id = 3 THEN price_usd ELSE 0 END) AS revenue_3,
    SUM(CASE WHEN product_id = 3 THEN price_usd - cogs_usd ELSE 0 END) AS margin_3,
    SUM(CASE WHEN product_id = 4 THEN price_usd ELSE 0 END) AS revenue_4,
    SUM(CASE WHEN product_id = 4 THEN price_usd - cogs_usd ELSE 0 END) AS margin_4,
    COUNT(order_id) AS total_sales,
    SUM(price_usd) AS total_revenue,
    SUM(price_usd-cogs_usd) AS total_margin
FROM order_items
GROUP BY 1,2;
```


<img width="1054" alt="Screenshot 2023-07-13 at 4 15 26 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/c8d26f63-e2b3-46d2-b7f0-533882e46c75">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q5_2_16892593106170/Sheet9?:language=en-US&:display_count=n&:origin=viz_share_link)**


<img width="1057" alt="Screenshot 2023-07-13 at 4 05 28 PM" src="https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/087bc12e-b27b-4230-8e7c-5218240498af">

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q5_1_16892593892100/Sheet8?:language=en-US&:display_count=n&:origin=viz_share_link)**


6. Focusing on the products webpage, obtain its monthly overall sessions, its clickthrough rate and the conversion rate from visiting the page to placing an order 

```
CREATE TEMPORARY TABLE products_pageviews
SELECT 
    website_session_id,
    website_pageview_id,
    created_at AS saw_product_page_at
FROM website_pageviews
WHERE pageview_url = '/products';

SELECT 
    YEAR(saw_product_page_at) AS yr,
    MONTH(saw_product_page_at) AS mo,
    COUNT(DISTINCT pp.website_session_id) AS sessions_to_product_page,
    COUNT(DISTINCT wp.website_session_id) AS clicked_to_next_page,
    COUNT(DISTINCT wp.website_session_id) / COUNT(DISTINCT pp.website_session_id) AS clickthrough_rt,
    COUNT(DISTINCT o.order_id) AS orders,
    COUNT(DISTINCT o.order_id) / COUNT(DISTINCT pp.website_session_id) AS products_to_order_rt
FROM products_pageviews pp
    LEFT JOIN website_pageviews wp
	ON wp.website_session_id = pp.website_session_id
	    AND wp.website_pageview_id > pp.website_pageview_id
    LEFT JOIN orders o
	ON o.website_session_id = pp.website_session_id
GROUP BY 1,2;
```

![image](https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/ccebe89f-5234-4fc9-b4fb-6b7bdc4ce95a)

🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q6_16892594652410/Sheet1?:language=en-US&:display_count=n&:origin=viz_share_link)**

7. After launching the newest product, obtain sales data since then, and show how well each product cross sells from one another

```
CREATE TEMPORARY TABLE primary_products
SELECT
	order_id,
        primary_product_id,
        created_at AS ordered_at
FROM orders
WHERE created_at > '2014-12-05';


SELECT 
    primary_product_id,
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 1 THEN order_id ELSE NULL END) AS _xsold_p1,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 2 THEN order_id ELSE NULL END) AS _xsold_p2,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 3 THEN order_id ELSE NULL END) AS _xsold_p3,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 4 THEN order_id ELSE NULL END) AS _xsold_p4,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 1 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p1_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 2 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p2_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 3 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p3_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 4 THEN order_id ELSE NULL END) / COUNT(DISTINCT order_id) AS p4_xsell_rt

FROM 
(
SELECT
    pp.*,
    o.product_id AS cross_sell_product_id
FROM primary_products pp
    LEFT JOIN order_items o
	ON o.order_id = pp.order_id
	    AND o.is_primary_item = 0
) AS primary_w_cross_sell
GROUP BY 1;

```

![image](https://github.com/michaenho/mavenfuzzyfactory-analytics/assets/74249209/bd400456-7361-47f0-8f8e-c46160770710)


🎨 **[Click here for the full visualisation](https://public.tableau.com/views/q7_16892595127090/Sheet2?:language=en-US&:display_count=n&:origin=viz_share_link)**

 ⛔ For the complete SQL code for the project, [SQL code link](https://github.com/michaenho/mavenfuzzyfactory-analytics/blob/main/mavenfuzzyfactory-SQL)!
 



## 5. Act
Conclusion based on our analysis:
- Casual riders rides mostly during the weekends.
- Casual riders ride longer duration, but least total trips. 
- Casual riders rides longer on docked bike, but least total trips.
- Most popular station for casual riders are: Streeter Dr & Grand Ave, Lake Shore Dr & Monroe St, Millennium Park.
- Most active months for casual riders are from June to August.

Marketing recommendations to convert casual riders into members:

##### 🚩  Marketing effort on the top 5 most popular stations for the causal riders. It can be a booth, print media on the bike or the locking station area, or social media post on contest starting from the most popular stations. 

##### ⛱  Promotional short term membership offer during the summer months.

##### 🚴‍♂️ Promotional weekend term membership for the weekends.

##### 🎁 Point-award incentive system for riding more trips in a membership format to receive discount and partnership offers. 


## 6. Reflect

