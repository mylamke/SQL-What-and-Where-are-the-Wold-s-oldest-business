# SQL-What-and-Where-are-the-Wold-s-oldest-business
# I. INTRODUCTION 
 
1. Project Description

An important aspect of business involves planning for the future and ensuring resilience against changing market conditions. Some businesses excel in this area, enduring for centuries. This project aims to analyze data from BusinessFinancing.co.uk on the world's oldest businesses, focusing on their founding dates and industries.

The data is dispersed across multiple datasets, necessitating the use of joining techniques to consolidate information. By merging these datasets, we can employ manipulation tools like grouping and filtering to gain insights into these enduring businesses. This analysis will reveal when these businesses were established and the diverse industries they represent, shedding light on their remarkable longevity and adaptive strategies in a dynamic market environment.

2. Dataset 

The database contains three tables.

### **`categories`**

| column | type | meaning |
| --- | --- | --- |
| category_code | varchar | Code for the category of the business. |
| category | varchar | Description of the business category. |

### **`countries`**

| column | type | meaning |
| --- | --- | --- |
| country_code | varchar | ISO 3166-1 3-letter country code. |
| country | varchar | Name of the country. |
| continent | varchar | Name of the continent that the country exists in. |

### **`businesses`**

| column | type | meaning |
| --- | --- | --- |
| business | varchar | Name of the business. |
| year_founded | int | Year the business was founded. |
| category_code | varchar | Code for the category of the business. |
| country_code | char | ISO 3166-1 3-letter country code. |

# II. Main Process

## **1. How many businesses were founded before 1000?**

Wow! That's a lot of variation between countries. In one country, the oldest business was only founded in 1999. By contrast, the oldest business in the world was founded back in 578. That's pretty incredible that a business has survived for more than a millennium.

I wonder how many other businesses there are like that.

```
select count(business) as business_before_1000
from glass-standard-378403.SQL.businesses
where year_founded < 1000 
```

| business_before_1000 |
|----------------------|
| 6                    |


## **2. Which businesses were founded before 1000?**

Having a count is all very well, but I'd like more detail. Which businesses have been around for more than a millennium?

```
select business
from glass-standard-378403.SQL.businesses
where year_founded < 1000 
order by business
```
| business                    |
|-----------------------------|
| Kongō Gumi                  |
| Monnaie de Paris            |
| Sean's Bar                  |
| St. Peter Stifts Kulinarium |
| Staffelter Hof Winery       |
| The Royal Mint              |



## **3. Exploring the categories**

Now we know that the oldest, continuously operating company in the world is called Kongō Gumi. But was does that company do? The category codes in the `businesses` table aren't very helpful: the descriptions of the categories are stored in the `categories` table.

This is a common problem: for data storage, it's better to keep different types of data in different tables, but for analysis, you want all the data in one place. To solve this, you'll have to join the two tables together.

```
Select business, category_code, string_field_1 as categories
from (
       select business, category_code
       from glass-standard-378403.SQL.businesses
       where year_founded < 1000 
       order by business) 
       as table_1
left join  glass-standard-378403.SQL.categories as category_table
on table_1.category_code = category_table.string_field_0
```


| business                    | category_code | categories                        |
|-----------------------------|---------------|-----------------------------------|
| Kongō Gumi                  | CAT6          | Construction                      |
| St. Peter Stifts Kulinarium | CAT4          | Cafés, Restaurants & Bars         |
| The Royal Mint              | CAT12         | Manufacturing & Production        |
| Monnaie de Paris            | CAT12         | Manufacturing & Production        |
| Staffelter Hof Winery       | CAT9          | Distillers, Vintners, & Breweries |
| Sean's Bar                  | CAT4          | Cafés, Restaurants & Bars         |

## **4. Counting the categories[¶](https://sessions.datacamp.com/proxy/absolute/c56ac02c-dced-4605-ae8b-5eab7f904f0e/notebooks/production/users/12209077/dsbv0gi66a/notebooks/notebook.ipynb#5.-Counting-the-categories)**

With that extra detail about the oldest businesses, we can see that Kongō Gumi is a construction company. In that list of six businesses, we also see a café, a winery, and a bar. The two companies recorded as "Manufacturing and Production" are both mints. That is, they produce currency.

I'm curious as to what other industries constitute the oldest companies around the world, and which industries are most common.

```
Select count (string_field_1) as categorie_count, string_field_1 as categories 
from (
       select business, category_code
       from glass-standard-378403.SQL.businesses
       #where year_founded < 1000 
       order by business) 
       as table_1
left join  glass-standard-378403.SQL.categories as category_table
on table_1.category_code = category_table.string_field_0
group by  categories 
order by categorie_count desc
```

| categorie_count | categories                        |
|-----------------|-----------------------------------|
| 37              | Banking & Finance                 |
| 22              | Distillers, Vintners, & Breweries |
| 19              | Aviation & Transport              |
| 16              | Postal Service                    |
| 15              | Manufacturing & Production        |
| 7               | Media                             |
| 6               | Food & Beverages                  |
| 6               | Agriculture                       |
| 6               | Cafés, Restaurants & Bars         |
| 4               | Energy                            |
| 4               | Retail                            |
| 4               | Tourism & Hotels                  |
| 3               | Mining                            |
| 3               | Defense                           |
| 3               | Conglomerate                      |
| 3               | Consumer Goods                    |
| 2               | Construction                      |
| 2               | Telecommunications                |
| 1               | Medical                           |


## **5. Oldest business by continent**

It looks like "Banking & Finance" is the most popular category. Maybe that's where you should aim if you want to start a thousand-year business.

One thing we haven't looked at yet is where in the world these really old businesses are. To answer these questions, we'll need to join the `businesses` table to the `countries` table. Let's start by asking how old the oldest business is on each continent.

```
SELECT business, year_founded, continent
FROM
       (
       SELECT business, year_founded, string_field_2 AS continent,
              ROW_NUMBER() OVER( PARTITION BY string_field_2 ORDER BY year_founded asc) AS rn 
       FROM (
              SELECT *
              FROM glass-standard-378403.SQL.businesses AS table_1
              INNER JOIN glass-standard-378403.SQL.country AS country_table
                     ON table_1.country_code = country_table.string_field_0
              ) AS inner_table
       ) as table_3
WHERE rn = 1
ORDER BY year_founded
```

| business                    | year_founded | continent     |
|-----------------------------|--------------|---------------|
| Kongō Gumi                  | 578          | Asia          |
| St. Peter Stifts Kulinarium | 803          | Europe        |
| La Casa de Moneda de México | 1534         | North America |
| Casa Nacional de Moneda     | 1565         | South America |
| Mauritius Post              | 1772         | Africa        |
| Australia Post              | 1809         | Oceania       |


## **6. Joining everything for further analysis**

Interesting. There's a jump in time from the older businesses in Asia and Europe to the 16th Century oldest businesses in North and South America, then to the 18th and 19th Century oldest businesses in Africa and Oceania.

As mentioned earlier, when analyzing data it's often really helpful to have all the tables you want access to joined together into a single set of results that can be analyzed further. Here, that means we need to join all three tables.

```
SELECT *
              FROM glass-standard-378403.SQL.businesses AS business_table
              INNER JOIN glass-standard-378403.SQL.country AS country_table
                     ON business_table.country_code = country_table.string_field_0
              INNER JOIN glass-standard-378403.SQL.categories AS category_table
                    ON business_table.category_code = category_table.string_field_0
```

## **7. Counting categories by continent**
