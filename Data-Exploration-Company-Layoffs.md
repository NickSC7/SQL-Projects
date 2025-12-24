## Project Overview

**Project Title**: Company Layoff Data Exploration

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore company layoff data. The project involves finding insights within the company layoff data that can be used for visualizations and foster better business intelligence.
## Objectives

1. **Explore the Data**

## Project Structure

### 1. Explore the Data

```sql
-- Looking for companies that are no longer in business (100 percent meaning no longer having employees)
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1;

-- Looking for companies that went under and which companies laid off the most employees
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;

-- Finding the date range for the data: 2020-03-11 to 2023-03-06
SELECT MIN(`date`), MAX(`date`)
FROM layoffs_staging2;

-- Finding which companies laid off the most employees over the entire time period: Amazon > Google > Meta
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY sum(total_laid_off) DESC;

-- Seeing which industry had the most layoffs over the date range: Concsumer > Retail > Other
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY sum(total_laid_off) DESC;

-- Looking for the countries with the most company layoffs: United States > India > Netherlands
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY sum(total_laid_off) DESC;

-- Finding which years had the most layoffs: 2023 > 2022 > 2020> 2021
SELECT YEAR(`date`) AS `Date`, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY year(`date`)
ORDER BY `Date` DESC;

-- Finding the total number of layoffs per month throughout the year 
SELECT SUBSTRING(`date`, 1, 7) as `Month`, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY `Month`
ORDER BY `Month`;

-- Using a CTE to create a rolling total of layoffs per month throughout the year
WITH Date_Rolling_Total AS
(
SELECT SUBSTRING(`date`, 1, 7) as `Month`, SUM(total_laid_off) AS total_laid_off
FROM layoffs_staging2
WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY `Month`
ORDER BY `Month`
)
SELECT `Month`, total_laid_off,
SUM(total_laid_off) OVER (ORDER BY `Month`) as rolling_total
FROM Rolling_Total;

-- Used 2 CTE's to find the top 5 most company layoffs per year
WITH Company_Year (company, `year`, total_laid_off) AS
(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
), 
Company_Ranking AS
(
SELECT *, DENSE_RANK() OVER (PARTITION BY `year` ORDER BY total_laid_off DESC) as yearly_ranking
FROM Company_Year
WHERE `year` IS NOT NULL
)
SELECT *
FROM Company_Ranking
WHERE yearly_ranking <= 5;

```

## Conclusion
Throughout this project, I have gained a better understanding of how to sift through a large dataset and find insights that will help create better business decisions. I utilized Common Table Expressions in my more advanced queries to dig deeper into the data and find how the data relates to each other. 
