# EjerciciosSQL-Advanced
22. Lección 301 - ADVANCED INTRO

23. Lección 302 - CTE vs. SUBQUERY
        -SQL Tutorial Lesson: Top-Selling Artists
          As the lead data analyst for a prominent music event management company, you have been entrusted with a dataset containing concert revenue and detailed information about 		various artists.
          Your mission is to unlock valuable insights by analyzing the concert revenue data and identifying the top revenue-generating artists within each music genre.
          Write a query to rank the artists within each genre based on their revenue per member and extract the top revenue-generating artist from each genre. Display the output of 		the artist name, genre, concert revenue, number of members, and revenue per band member, sorted by the highest revenue per member within each genre.

                  SELECT
                    artist_name,
                    concert_revenue,
                    genre,
                    number_of_members,
                    concert_revenue / number_of_members AS revenue_per_member,
                    RANK() OVER (
                    PARTITION BY genre
                    ORDER BY concert_revenue / number_of_members DESC) AS ranked_concerts
                  FROM concerts;


        -Supercloud Customer
          A Microsoft Azure Supercloud customer is defined as a customer who has purchased at least one product from every product category listed in the products table.
          Write a query that identifies the customer IDs of these Supercloud customers.

                    WITH supercloud_cust AS (
                      SELECT 
                      customers.customer_id, 
                      COUNT(DISTINCT products.product_category) AS product_count
                      FROM customer_contracts AS customers
                      INNER JOIN products 
                      ON customers.product_id = products.product_id
                      GROUP BY customers.customer_id
                    )
                    SELECT customer_id
                    FROM supercloud_cust
                    WHERE product_count = (
                    SELECT COUNT(DISTINCT product_category) FROM products
                    );


        -Swapped Food Delivery
          Zomato is a leading online food delivery service that connects users with various restaurants and cuisines, allowing them to browse menus, place orders, and get meals 			delivered to their doorsteps.
          Recently, Zomato encountered an issue with their delivery system. Due to an error in the delivery driver instructions, each item's order was swapped with the item in the 		subsequent row. As a data analyst, you're asked to correct this swapping error and return the proper pairing of order ID and item.
          If the last item has an odd order ID, it should remain as the last item in the corrected data. For example, if the last item is Order ID 7 Tandoori Chicken, then it should 		remain as Order ID 7 in the corrected data.
          In the results, return the correct pairs of order IDs and items.

                      WITH order_counts AS (
                        SELECT COUNT(order_id) AS total_orders 
                        FROM orders
                      )
                      SELECT *
                      FROM orders
                      CROSS JOIN order_counts;


24. Lección 303 - WINDOW FUNCTION
      -Card Launch Success
        Your team at JPMorgan Chase is soon launching a new credit card. You are asked to estimate how many cards you'll issue in the first month.
        Before you can answer this question, you want to first get some perspective on how well new credit card launches typically do in their first month.
        Write a query that outputs the name of the credit card, and how many cards were issued in its launch month. The launch month is the earliest record in the 				monthly_cards_issued table for a given card. Order the results starting from the biggest issued amount.

                        SELECT 
                        card_name,
                        issued_amount,
                        MAKE_DATE(issue_year, issue_month, 1) AS issue_date,
                        MIN(MAKE_DATE(issue_year, issue_month, 1)) OVER (
                        PARTITION BY card_name) AS launch_date
                        FROM monthly_cards_issued;


25. Lección 304 - SQL RANKING
      -Top 5 Artists
        Assume there are three Spotify tables: artists, songs, and global_song_rank, which contain information about the artists, songs, and music charts, respectively.
        Write a query to find the top 5 artists whose songs appear most frequently in the Top 10 of the global_song_rank table. Display the top 5 artist names in ascending order, 		along with their song appearance ranking.
        If two or more artists have the same number of song appearances, they should be assigned the same ranking, and the rank numbers should be continuous (i.e. 1, 2, 2, 3, 4, 		5). If you've never seen a rank order like this before, do the rank window function tutorial.

                          SELECT 
                          artists.artist_name
                          FROM artists
                          INNER JOIN songs
                          ON artists.artist_id = songs.artist_id
                          INNER JOIN global_song_rank AS ranking
                          ON songs.song_id = ranking.song_id
                          WHERE ranking.rank <= 10;


      -Histogram of Users and Purchases
        This is the same question as problem #13 in the SQL Chapter of Ace the Data Science Interview!
        Assume you're given a table on Walmart user transactions. Based on their most recent transaction date, write a query that retrieve the users along with the number of 			products they bought.
        Output the user's most recent transaction date, user ID, and the number of products, sorted in chronological order by the transaction date.

                            WITH latest_transactions_cte AS (
                            SELECT 
                            transaction_date, 
                            user_id, 
                            product_id, 
                            RANK() OVER (
                            PARTITION BY user_id 
                            ORDER BY transaction_date DESC) AS transaction_rank 
                            FROM user_transactions)                              
                            SELECT *
                            FROM latest_transactions_cte
                            WHERE transaction_rank = 1 


        -Odd and Even Measurements
          This is the same question as problem #28 in the SQL Chapter of Ace the Data Science Interview!
          Assume you're given a table with measurement values obtained from a Google sensor over multiple days with measurements taken multiple times within each day.
          Write a query to calculate the sum of odd-numbered and even-numbered measurements separately for a particular day and display the results in two different columns. Refer to 		the Example Output below for the desired format.
          Definition:
          Within a day, measurements taken at 1st, 3rd, and 5th times are considered odd-numbered measurements, and measurements taken at 2nd, 4th, and 6th times are considered even-		numbered measurements.
          Effective April 15th, 2023, the question and solution for this question have been revised

                              SELECT 
                              CAST(measurement_time AS DATE) AS measurement_day, 
                              measurement_value, 
                              ROW_NUMBER() OVER (
                              PARTITION BY CAST(measurement_time AS DATE) 
                              ORDER BY measurement_time) AS measurement_num 
                              FROM measurements;


26. Lección 305 - SQL LEAD LAG
        -SQL Tutorial Lesson: Stock Performance
          The Bloomberg terminal is the go-to resource for financial professionals, offering convenient access to a wide array of financial datasets. In this SQL interview query for 		Data Analyst at Bloomberg, you're given the historical data on Google's stock performance.
          Your task is to:
          Calculate the difference in closing prices between consecutive months.
          Calculate the difference between the closing price of the current month and the closing price from 3 months prior.
          This question serves as a platform for you to explore into the dataset and execute your queries. Please refrain from submitting your solution for this question.

                                WITH RankedStockPrices AS 
                                ( 
                                SELECT date, ticker, close, 
                                LAG(close) OVER (
                                PARTITION BY ticker
                                ORDER BY date) AS lag_1, 
                                LAG(close, 3) OVER (
                                PARTITION BY ticker 
                                ORDER BY date) AS lag_3
                                FROM stock_prices 
                                )
                                SELECT date, ticker, close, close - lag_1 AS diff_consecutive, close - lag_3 AS diff_three 
                                FROM RankedStockPrices 
                                WHERE lag_1 IS NOT NULL 
                                AND lag_3 IS NOT NULL 
                                ORDER BY date;


        -Y-on-Y Growth Rate
          This is the same question as problem #32 in the SQL Chapter of Ace the Data Science Interview!
          Assume you're given a table containing information about Wayfair user transactions for different products. Write a query to calculate the year-on-year growth rate for the 		total spend of each product, grouping the results by product ID.
          The output should include the year in ascending order, product ID, current year's spend, previous year's spend and year-on-year growth percentage, rounded to 2 decimal 		places.

                                SELECT EXTRACT(YEAR FROM transaction_date) AS year, product_id, spend AS curr_year_spend,
                                LAG(spend) OVER (
                                PARTITION BY product_id 
                                ORDER BY product_id, 
                                EXTRACT(YEAR FROM transaction_date)) AS prev_year_spend 
                                FROM user_transactions;


27.  Lección 306 - SQL SELF-JOINS
	
28.  Lección 307 - SQL UNION
        -Maximize Prime Item Inventory
          Effective April 3rd 2024, we have updated the problem statement to provide additional clarity.
          Amazon wants to maximize the storage capacity of its 500,000 square-foot warehouse by prioritizing a specific batch of prime items. The specific prime product batch 			detailed in the inventory table must be maintained.
          So, if the prime product batch specified in the item_category column included 1 laptop and 1 side table, that would be the base batch. We could not add another laptop 			without also adding a side table; they come all together as a batch set.
          After prioritizing the maximum number of prime batches, any remaining square footage will be utilized to stock non-prime batches, which also come in batch sets and cannot 		be separated into individual items.
          Write a query to find the maximum number of prime and non-prime batches that can be stored in the 500,000 square feet warehouse based on the following criteria:
          Prioritize stocking prime batches
          After accommodating prime items, allocate any remaining space to non-prime batches
          Output the item_type with prime_eligible first followed by not_prime, along with the maximum number of batches that can be stocked.
          Assumptions:
          Again, products must be stocked in batches, so we want to find the largest available quantity of prime batches, and then the largest available quantity of non-prime batches
          Non-prime items must always be available in stock to meet customer demand, so the non-prime item count should never be zero.
          Item count should be whole numbers (integers).

                                  SELECT item_type,
                                  SUM(square_footage) AS total_sqft,
                                  COUNT(*) AS item_count
                                  FROM inventory
                                  GROUP BY item_type;


        -Page With No Likes
          Assume you're given two tables containing data about Facebook Pages and their respective likes (as in "Like a Facebook Page").
          Write a query to return the IDs of the Facebook pages that have zero likes. The output should be sorted in ascending order based on the page IDs.

                                  SELECT page_id
                                  FROM pages
                                  EXCEPT
                                  SELECT page_id
                                  FROM page_likes;


29.  Lección 308 - WRITE CLEAN SQL

30.  Lección 309 - EXECUTION ORDER
	
31.  Lección 310 - SQL PIVOTING

32.  Lección 311 - STRING FUNCTIONS
        -SQL LOWER Practice Exercise
          Assume you're given the customer table containing all customer details.
          The branch manager is looking for a male customer whose name ends with "son" and he's 20 years old.
          Write a SQL query which uses LOWER and LIKE to find this customer's details.

                                    SELECT *
                                    FROM customers
                                    WHERE LOWER(customer_name) LIKE '%son'
                                    AND gender = 'Male'
                                    AND age = 20;


        -Pharmacy Analytics (Part 3)
          CVS Health wants to gain a clearer understanding of its pharmacy sales and the performance of various products.
          Write a query to calculate the total drug sales for each manufacturer. Round the answer to the nearest million and report your results in descending order of total sales. 		In case of any duplicates, sort them alphabetically by the manufacturer name.
          Since this data will be displayed on a dashboard viewed by business stakeholders, please format your results as follows: "$36 million".

                                      SELECT manufacturer, 
                                      ROUND(SUM(total_sales) / 1000000) AS sales_mil 
                                      FROM pharmacy_sales 
                                      GROUP BY manufacturer;


33. Lección 312 - INSTACART SQL CASE
        -Instacart Exploration – Case Study Checkpoint #1
          Explore the ic_products data. Here are some questions to investigate:
          How many items are there?
          How many different products are in the dataset?
          Table Schema
          Here are the schemas for all 5 tables in the Instacart market data.
          ic_order_products_curr, ic_order_products_prior
          These files specify which products were purchased in each Instacart order. ic_order_products_prior contains previous order contents for all customers, and 				ic_order_products_curr contains current orders; the table fields are the same.
          The 'reordered' field indicates that the customer has a previous order that contains the product. Other fields should be self-explanatory.


                                        SELECT prior.product_id, product_name, department, aisle
                                        FROM ic_products prod
                                        JOIN ic_order_products_prior prior
                                        ON prod.product_id = prior.product_id
                                        JOIN ic_departments dept
                                        ON prod.department_id = dept.department_id
                                        JOIN ic_aisles aisle
                                        ON prod.aisle_id = aisle.aisle_id
                                        GROUP BY 1,2,3,4
                                        ORDER BY count(1) DESC
                                        LIMIT 5;


        -Instacart Reorders – Case Study Checkpoint #2
          For this checkpoint, we want to investigate products that previously had low reorders, but currently have high reorders. Feel free to approach this by measuring counts, 		percentages, or both.
          Write a query to find products previously reordered fewer than 10 times, and currently reordered 10 or more times. Alternatively, write a query to find which products had 		the biggest and smallest percent changes in products ordered.
          Don't worry if your submitted answer is marked wrong – this is merely an open-ended exercise!
          Table Schema
          Here are the schemas for all 5 tables in the Instacart market data.
          ic_order_products_curr, ic_order_products_prior
          These files specify which products were purchased in each Instacart order. ic_order_products_prior contains previous order contents for all customers, and 				ic_order_products_curr contains current orders; the table fields are the same.
          The 'reordered' field indicates that the customer has a previous order that contains the product. Other fields should be self-explanatory.

                                          SELECT prod.product_id, prod.product_name, prod.aisle_id, prod.department_id,
                                          dept.department, aisles.aisle,
                                          SUM(op_prior.reordered) AS prior_reorders,
                                          SUM(op_curr.reordered) AS current_reorders
                                          FROM ic_products AS prod
                                          JOIN Ic_order_products_prior AS op_prior ON prod.product_id = op_prior.product_id
                                          JOIN ic_order_products_curr AS op_curr ON prod.product_id = op_curr.product_id
                                          JOIN ic_departments AS dept ON prod.department_id = dept.department_id
                                          JOIN ic_aisles AS aisles ON prod.aisle_id = aisles.aisle_id
                                          GROUP BY prod.product_id, prod.product_name, prod.aisle_id,
                                          prod.department_id, dept.department, aisles.aisle
                                          HAVING SUM(op_prior.reordered) < 10 AND SUM(op_curr.reordered) >= 10
                                          ORDER BY current_reorders DESC;