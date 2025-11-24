# Netflix Movies and TV Shows Data Analysis using SQL

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives
Analyze the distribution of content types (movies vs TV shows).
Identify the most common ratings for movies and TV shows.
List and analyze content based on release years, countries, and durations.
Explore and categorize content based on specific criteria and keywords.

## Dataset
The data for this project is sourced from the Kaggle dataset:

Dataset Link: Movies Dataset *(Add real link)*

## Schema
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix (
    show_id      VARCHAR(6),
    type         VARCHAR(10),
    title        VARCHAR(150),
    director     VARCHAR(208),
    casts        VARCHAR(1000),
    country      VARCHAR(150),
    date_added   VARCHAR(50),
    release_year INT,
    rating       VARCHAR(10),
    duration     VARCHAR(15),
    listed_in    VARCHAR(100),
    description  VARCHAR(250)
);

## Business Problems and SQL Solutions

1. Count the Number of Movies vs TV Shows
SELECT 
    type AS content_type,
    COUNT(*) AS total_titles
FROM netflix
GROUP BY type
ORDER BY total_titles DESC;

2. Most Common Rating for Movies and TV Shows
WITH RatingCounts AS (
    SELECT type, rating, COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT type, rating, rating_count,
           RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating,
    rating_count
FROM RankedRatings
WHERE rank = 1;

3. List Movies Released in 2020
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND release_year = 2020
ORDER BY title;

4. Top 5 Countries with the Most Netflix Content
SELECT 
    country_trim AS country,
    COUNT(*) AS total_content
FROM (
    SELECT 
        TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) AS country_trim
    FROM netflix
    WHERE country IS NOT NULL
) AS t
WHERE country_trim <> ''
GROUP BY country_trim
ORDER BY total_content DESC
LIMIT 5;

5. Longest Movie on Netflix
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;

6. Content Added in Last 5 Years
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';

7. Movies/Shows by Rajiv Chilaka
SELECT *
FROM (
    SELECT n.*,
           TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) AS director_name
    FROM netflix AS n
) AS t
WHERE director_name = 'Rajiv Chilaka';

8. TV Shows with More than 5 Seasons
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;

9. Count of Content in Each Genre
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre
ORDER BY total_content DESC;

10. Top 5 Years with Highest % of Indian Content
SELECT 
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country LIKE '%India%')::numeric * 100,
        2
    ) AS percentage_of_indian_titles
FROM netflix
WHERE country LIKE '%India%'
GROUP BY release_year
ORDER BY percentage_of_indian_titles DESC
LIMIT 5;

11. All Documentary Movies
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND listed_in ILIKE '%Documentaries%';

12. Content With No Director Listed
SELECT *
FROM netflix
WHERE director IS NULL OR director = '';

13. Salman Khan Movies in Last 10 Years
SELECT *
FROM netflix
WHERE casts ILIKE '%Salman Khan%'
  AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE) - 10
  AND type = 'Movie';

14. Top 10 Actors in Indian Movies
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(casts, ','))) AS actor,
    COUNT(*) AS total_titles
FROM netflix
WHERE country LIKE '%India%'
  AND type = 'Movie'
GROUP BY actor
ORDER BY total_titles DESC
LIMIT 10;

15. Categorize Content as Good or Bad (Violence/Kill Related)
SELECT 
    category,
    type,
    COUNT(*) AS content_count
FROM (
    SELECT *,
        CASE 
            WHEN description ILIKE '%kill%' 
              OR description ILIKE '%violence%' 
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category, type
ORDER BY type, category;

## Findings and Conclusion

Content Distribution: The dataset contains a diverse range of movies and TV shows with varying ratings and genres.  
Common Ratings: Insights into the most common ratings provide an understanding of the content's target audience.  
Geographical Insights: The top countries and the average content releases by India highlight regional content distribution.  
Content Categorization: Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.  

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

## Connect with Me
LinkedIn: https://in.linkedin.com/in/marcus-manoj-manuel-a489642a9
