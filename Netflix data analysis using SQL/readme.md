# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id varchar(6),	
 type	varchar(10),
 title	varchar(150),
 director varchar(208),
 cast	varchar(1000),
 country varchar(150),	
 date_added	varchar(50),
 release_year	int,
 rating varchar(10),
 duration	varchar(15),
 listed_in	varchar(100),
 description varchar(250) 
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
 select type,rating from 
( select type,rating,count(*),
 Rank() over(partition by type order by count(*)desc) as ranking from netflix_db group by 1,2 ) as t1
where ranking = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT country_name AS country,
       COUNT(*) AS total_content
FROM (
    SELECT TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(country, ',', 1), ',', -1)) AS country_name
    FROM netflix_db
    UNION ALL
    SELECT TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(country, ',', 2), ',', -1))
    FROM netflix_db
    UNION ALL
    SELECT TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(country, ',', 3), ',', -1))
    FROM netflix_db
) AS all_countries
WHERE country_name <> ''
GROUP BY country_name
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
select * from netflix_db 
where type = 'movie'
AND duration = (select max(duration) from netflix_db);
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix_db
WHERE STR_TO_DATE(date_added, '%M %d, %Y') >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR);
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Toshiya Shinohara'

```sql
select * from netflix_db
where director like "%Toshiya Shinohara%";
```

**Objective:** List all content directed by 'Toshiya Shinohara'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix_db
WHERE type = 'TV Show'
  AND CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
  TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(listed_in, ',', n.n), ',', -1)) AS genre,
  COUNT(*) AS total_titles
FROM 
  netflix_db
JOIN (
  SELECT 1 AS n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
) AS n
  ON n.n <= 1 + LENGTH(listed_in) - LENGTH(REPLACE(listed_in, ',', ''))
GROUP BY genre
ORDER BY total_titles DESC;
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top  year 2021 with highest avg content release!

```sql
WITH yearly_counts AS (
  SELECT 
    YEAR(STR_TO_DATE(date_added, '%M %d, %Y')) AS release_year,
    COUNT(*) AS total_titles
  FROM netflix_db
  WHERE LOWER(country) LIKE '%india%'
    AND date_added IS NOT NULL
  GROUP BY YEAR(STR_TO_DATE(date_added, '%M %d, %Y'))
)
SELECT 
  release_year,
  total_titles,
  ROUND(AVG(total_titles) OVER (), 2) AS avg_content_per_year
FROM yearly_counts
ORDER BY release_year ASC;
```

**Objective:** Calculate and rank year by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
select * from netflix_db
where listed_in like '%documentaries%';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
FROM netflix_db
WHERE director IS NULL OR director = '';
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Jack fisher' Appeared in the Last 10 Years

```sql
SELECT 
  COUNT(*) AS total_movies
FROM netflix_db
WHERE 
  type = 'Movie'
  AND LOWER(cast) LIKE '%jack fisher%'
  AND STR_TO_DATE(date_added, '%M %d, %Y') >= (CURDATE() - INTERVAL 10 YEAR);
```

**Objective:** Count the number of movies featuring 'jack fisher' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT 
  actor AS actor_name,
  COUNT(*) AS total_movies
FROM (
  SELECT 
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(cast, ',', n.n), ',', -1)) AS actor,
    show_id
  FROM netflix_db
  JOIN (
    SELECT a.N + b.N * 10 + 1 AS n
    FROM 
      (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 
       UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) a,
      (SELECT 0 AS N UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 
       UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) b
  ) n
  ON n.n <= 1 + LENGTH(cast) - LENGTH(REPLACE(cast, ',', ''))
  WHERE 
    type = 'Movie'
    AND LOWER(country) LIKE '%india%'
    AND cast IS NOT NULL
) AS actors
GROUP BY actor_name
ORDER BY total_movies DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
SELECT 
  CASE
    WHEN LOWER(title) LIKE '%kill%' OR LOWER(description) LIKE '%kill%'
      OR LOWER(title) LIKE '%violence%' OR LOWER(description) LIKE '%violence%'
    THEN 'Bad'
    ELSE 'Good'
  END AS content_category,
  COUNT(*) AS total_content
FROM netflix_db
GROUP BY content_category;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

