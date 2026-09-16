# Netflix Movies and TV Shows Data Analysis using SQL

<p align="center">
  <img width="900" height="400" src="https://github.com/user-attachments/assets/90dd70f1-35b4-48cb-959a-85bb5ac461c4" alt="Netflix Data Analysis">
</p>

## 📖 Overview

This project focuses on performing a comprehensive analysis of **Netflix Movies and TV Shows data using SQL**.

The objective is to explore the Netflix content catalog, answer practical business questions, identify meaningful patterns, and generate insights related to content types, ratings, release years, countries, genres, directors, actors, and content descriptions.

The analysis demonstrates the use of SQL techniques such as:

* Aggregation and grouping
* Common Table Expressions (CTEs)
* Window functions
* String manipulation
* Filtering and conditional logic
* Date functions
* Subqueries
* Data categorization

---

## 🎯 Objectives

* Analyze the distribution of content types (Movies vs TV Shows).
* Identify the most common ratings for Movies and TV Shows.
* Analyze content based on release years, countries, and durations.
* Explore genres and content categories.
* Identify directors and actors associated with Netflix content.
* Analyze Indian content and release trends.
* Categorize content based on keywords found in descriptions.
* Extract business-oriented insights from the dataset.

---

## 📊 Dataset

The analysis uses the **Netflix Movies and TV Shows** dataset available on Kaggle.

**Dataset:** Netflix Movies and TV Shows

**Source:** Kaggle — Netflix Shows Dataset

---

## 🗃️ Database Schema

```sql
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

---

# 🔍 Business Problems and SQL Solutions

## 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:**
Determine the distribution of Movies and TV Shows available in the dataset.

---

## 2. Find the Most Common Rating for Movies and TV Shows

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),

RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (
            PARTITION BY type 
            ORDER BY rating_count DESC
        ) AS rank
    FROM RatingCounts
)

SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

**Objective:**
Identify the most frequently occurring rating for each type of content.

---

## 3. List All Movies Released in a Specific Year

Example: 2020

```sql
SELECT *
FROM netflix
WHERE release_year = 2020;
```

**Objective:**
Retrieve all content released in a specific year.

---

## 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT *
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1

WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:**
Identify the top five countries with the highest number of content items.

---

## 5. Identify the Longest Movie

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:**
Find movies based on their duration and identify the longest movie in the dataset.

---

## 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY')
      >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:**
Retrieve content that was added to Netflix during the last five years.

---

## 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t

WHERE director_name = 'Rajiv Chilaka';
```

**Objective:**
Identify all content associated with director **Rajiv Chilaka**.

---

## 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:**
Identify TV Shows that have more than five seasons.

---

## 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Objective:**
Analyze the distribution of Netflix content across different genres.

---

## 10. Find the Years with the Highest Percentage of Indian Content Releases

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id)
         FROM netflix
         WHERE country = 'India')::numeric * 100,
        2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:**
Calculate and rank the years based on the percentage of content releases associated with India and identify the top five years.

---

## 11. List All Movies that are Documentaries

```sql
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Objective:**
Retrieve content classified as documentaries.

---

## 12. Find All Content Without a Director

```sql
SELECT *
FROM netflix
WHERE director IS NULL;
```

**Objective:**
Identify Netflix content where director information is unavailable.

---

## 13. Find Movies Featuring Actor 'Salman Khan' in the Last 10 Years

```sql
SELECT *
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Objective:**
Identify content featuring Salman Khan released within the last ten years.

---

## 14. Find the Top 10 Actors with the Highest Number of Appearances in Indian Content

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Objective:**
Identify the top ten actors with the highest number of appearances in Indian-produced content.

---

## 15. Categorize Content Based on 'Kill' and 'Violence' Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%'
              OR description ILIKE '%violence%'
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

**Objective:**
Categorize content based on whether the description contains keywords related to **"kill"** or **"violence"**, and calculate the number of items in each category.

---

# 🧠 SQL Concepts Used

This project applies several important SQL concepts:

| SQL Concept      | Application                                         |
| ---------------- | --------------------------------------------------- |
| `SELECT`         | Retrieving data                                     |
| `WHERE`          | Filtering records                                   |
| `GROUP BY`       | Aggregating data                                    |
| `ORDER BY`       | Ranking and sorting                                 |
| `COUNT()`        | Counting records                                    |
| `RANK()`         | Ranking ratings                                     |
| `CASE`           | Conditional categorization                          |
| `CTE`            | Structuring complex queries                         |
| Window Functions | Analytical calculations                             |
| Subqueries       | Nested calculations                                 |
| String Functions | Processing countries, genres, actors, and directors |
| Date Functions   | Time-based analysis                                 |
| `UNNEST()`       | Splitting multi-value fields                        |

---

# 📌 Key Findings

### Content Distribution

The dataset contains a diverse collection of **Movies and TV Shows**, covering different ratings, genres, countries, and release periods.

### Common Ratings

Analyzing ratings provides an understanding of the types of content available and their potential target audiences.

### Geographical Insights

Country-level analysis highlights the geographical distribution of Netflix content and provides insights into regional content availability.

### Indian Content

The analysis of Indian releases helps identify years with relatively higher content contributions from India.

### Genre Distribution

Genre analysis provides an overview of the different types of entertainment content available on Netflix.

### Content Categorization

Keyword-based categorization demonstrates how textual descriptions can be analyzed using SQL conditional logic.

---

# 📈 Analysis Summary

This project demonstrates how SQL can be used to transform a raw entertainment dataset into meaningful analytical insights.

The analysis covers:

* Content-type distribution
* Ratings analysis
* Release-year analysis
* Country-level analysis
* Movie-duration analysis
* Recent content analysis
* Director analysis
* TV-show season analysis
* Genre analysis
* Indian content analysis
* Documentary identification
* Missing-data analysis
* Actor analysis
* Keyword-based content categorization

---

# 🛠️ Tools & Technologies

| Category       | Technology                          |
| -------------- | ----------------------------------- |
| Database       | PostgreSQL                          |
| Query Language | SQL                                 |
| Dataset        | Netflix Movies and TV Shows         |
| Data Source    | Kaggle                              |
| Analysis       | SQL-based Exploratory Data Analysis |

---

# 📂 Project Structure

```text
Netflix-SQL-Data-Analysis/
│
├── netflix_titles.csv
├── netflix_data_analysis.sql
└── README.md
```

> The project structure may vary depending on the files included in the implementation.

---

# 🚀 Possible Future Enhancements

* Build an interactive dashboard using Power BI or Tableau.
* Perform advanced time-series analysis of content releases.
* Analyze country and genre combinations.
* Perform actor and director network analysis.
* Add additional data-quality checks.
* Create automated reporting from the SQL analysis.
* Extend the analysis with Netflix content trends over time.

---

# 🎓 Learning Outcomes

Through this project, the following skills are demonstrated:

* Writing analytical SQL queries
* Working with relational datasets
* Data aggregation and filtering
* Using CTEs and subqueries
* Applying window functions
* Performing string manipulation
* Handling date-based analysis
* Extracting business insights from data
* Structuring SQL-based exploratory data analysis
* Translating business questions into SQL solutions

---

# 📌 Conclusion

This Netflix data analysis project demonstrates the practical use of **SQL for exploratory data analysis and business intelligence**.

By answering a series of business-oriented questions, the project explores Netflix's content catalog from multiple perspectives, including content type, ratings, geography, genres, release trends, directors, actors, and descriptions.

The analysis shows how SQL can be used not only to retrieve data, but also to **identify patterns, answer business questions, and generate actionable insights from real-world datasets**.
