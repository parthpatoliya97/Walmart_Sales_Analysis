## Walmart_Sales_Analysis

![flowchart](https://github.com/parthpatoliya97/Walmart_Sales_Analysis/blob/main/Workflow_image.png?raw=true)

#### Database Setup & Data Cleaning
- Create database
```sql
CREATE DATABASE walmart;
USE walmart;
```

- Check data types
```sql
DESCRIBE walmart;
```

- Remove $ sign from unit_price
```sql
SET SQL_SAFE_UPDATES = 0;
UPDATE walmart
SET unit_price = REPLACE(unit_price, '$', '');
SET SQL_SAFE_UPDATES = 1;
```

- Modify column data types
```sql
ALTER TABLE walmart 
MODIFY unit_price DECIMAL(10,2);

ALTER TABLE walmart 
MODIFY date DATE;

ALTER TABLE walmart 
MODIFY time TIME;
```

- Add total column (quantity * unit_price)
```sql
ALTER TABLE walmart
ADD COLUMN total DECIMAL(10,2);

SET SQL_SAFE_UPDATES = 0;
UPDATE walmart
SET total = unit_price * quantity;
SET SQL_SAFE_UPDATES = 1;
```

- Data cleaning - remove records with NULL values
```sql
SET SQL_SAFE_UPDATES = 0;
DELETE FROM walmart WHERE unit_price IS NULL;
DELETE FROM walmart WHERE quantity IS NULL;
SET SQL_SAFE_UPDATES = 1;
```

#### 1. Payment Methods Analysis
#### Understand customer preferences for payment methods to optimize payment strategies.
- Get unique payment methods
```sql
SELECT DISTINCT(payment_method) FROM walmart;
```

- Transactions by payment method
```sql
SELECT 
    payment_method,
    COUNT(*) AS total_transactions
FROM walmart
GROUP BY payment_method
ORDER BY COUNT(*) DESC;
```

#### 2. Highest-Rated Category in Each Branch
#### Recognize popular categories in specific branches to enhance customer satisfaction.
- Count unique branches
```sql
SELECT COUNT(DISTINCT(branch)) FROM walmart;
```

- Top rated categories by branch
```sql
WITH CTE AS (
    SELECT 
        Branch, 
        category, 
        rating,
        DENSE_RANK() OVER (PARTITION BY Branch ORDER BY rating DESC) AS rnk
    FROM walmart
)
SELECT Branch, category, rating
FROM CTE
WHERE rnk = 1;
```

#### 3. Busiest Day for Each Branch
#### Optimize staffing and inventory management by identifying peak days.
```sql
WITH cte AS (
    SELECT 
        Branch,
        DAYNAME(date) AS day_of_week,
        COUNT(*) AS total_transactions,
        DENSE_RANK() OVER(PARTITION BY Branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY Branch, day_of_week
)
SELECT Branch, day_of_week, total_transactions
FROM cte
WHERE rnk = 1;
```

#### 4. Quantity Sold by Payment Method
#### Track sales volume by payment type to understand purchasing habits.
```sql
SELECT 
    payment_method,
    SUM(quantity) AS total_units_sold
FROM walmart
GROUP BY payment_method
ORDER BY COUNT(*) DESC, SUM(quantity) DESC;
```

#### 5. Category Ratings by City
#### Guide city-level promotions by understanding regional preferences.
```sql
SELECT 
    City,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    ROUND(AVG(rating), 2) AS avg_rating
FROM walmart
GROUP BY City, category
ORDER BY City, category;
```
#### 6. Total Profit by Category
#### Identify high-profit categories to focus expansion efforts.
```sql
SELECT
    category,
    SUM(total) AS total_revenue,
    ROUND(SUM(total * profit_margin), 2) AS profit
FROM walmart
GROUP BY category
ORDER BY ROUND(SUM(total * profit_margin), 2) DESC;
```
#### 7. Most Common Payment Method per Branch
#### Understand branch-specific payment preferences to streamline payment processing.
```sql
WITH cte AS (
    SELECT 
        Branch,
        payment_method,
        COUNT(*) AS total_transactions,
        DENSE_RANK() OVER(PARTITION BY Branch ORDER BY COUNT(*) DESC) AS rnk
    FROM walmart
    GROUP BY Branch, payment_method
)
SELECT Branch, payment_method, total_transactions
FROM cte
WHERE rnk = 1;
```

#### 8. Sales Shifts Throughout the Day
#### Manage staff shifts and stock replenishment during high-sales periods.
```sql
SELECT 
    Branch,
    CASE 
        WHEN HOUR(time) < 12 THEN 'Morning'
        WHEN HOUR(time) > 12 AND HOUR(time) < 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS day_time,
    COUNT(*) AS total_transactions
FROM walmart
GROUP BY Branch, day_time
ORDER BY Branch, total_transactions DESC;
```

#### 9. Branches with Highest Revenue Decline Year-Over-Year
#### Detect branches with declining revenue to address local issues.
```sql
WITH yearly_revenue AS (
    SELECT 
        Branch,
        YEAR(date) AS year_,
        SUM(unit_price * quantity) AS revenue
    FROM walmart
    GROUP BY Branch, YEAR(date)
),
revenue_lag AS (
    SELECT 
        Branch,
        year_,
        revenue,
        LAG(revenue) OVER (PARTITION BY Branch ORDER BY year_) AS previous_revenue
    FROM yearly_revenue
),
yoy_growth AS (
    SELECT
        Branch,
        year_,
        revenue,
        previous_revenue,
        ROUND((revenue - previous_revenue)/previous_revenue, 2) AS YOY_growth
    FROM revenue_lag
)
SELECT *
FROM yoy_growth;
```


