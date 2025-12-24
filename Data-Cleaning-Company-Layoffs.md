## Project Overview

**Project Title**: Company Layoff Data Cleaning

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to clean company layoff data. The project involves setting up a layoff databse, importing the layoff data into mySQL, and cleaning the data.

## Objectives

1. **Set up a layoff database**
2. **Remove Duplicates (If any)**
3. **Standardize the Data**
4. **Analyze Null or Blank values**
5. **Remove Any Ambiguous or Meaningless Data**

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database called company_layoffs.
- **Table Creation**: After creating the database, a table named layoffs_raw_data table is created to store the data. The table structure includes columns for company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions. A staging table named layoffs_Staging is then created based off the previous raw data table. The raw data is inserted into the previously created staging table. The staging table will allow us to manipulate the data without worrying about messing up the original data set if a mistake is made. 

```sql

CREATE SCHEMA company_layoffs

CREATE TABLE `layoffs_raw_data` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL
)

-- Make sure table was created 
SELECT *
FROM layoffs_raw_data

-- Used mySQL data import Wizard to transfer CSV file to table

-- Create staging table to alter data without concern for raw data
CREATE TABLE layoffs_staging
LIKE layoffs_raw_data;

-- Make sure table was created
SELECT *
FROM layoffs_staging;

-- Insert data from raw data table into staging table
INSERT layoffs_staging
SELECT *
FROM layoffs_raw_data;
```

### 2. Removing Duplicates

- **Checking for duplicates**: First, a script was created to find any duplicate records within the data. That script was then used in a cte to filter out the duplicate records.
- **Creating a new staging table**: Then, a new staging table is created since the cte cannot be updated. The CTE data is inserted into the staging table to be altered.
- **Deleting duplicate records**: After the data has been inserted, the duplicate records are identifed within the table and deleled.
```sql

-- Duplicate Checking Script
SELECT *,
ROW_NUMBER() OVER(PARTITION BY company, location,
industry, total_laid_off, percentage_laid_off, `date`,
stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

-- CTE to find duplicated records
WITH duplicates_cte AS
(
SELECT *,
ROW_NUMBER() OVER(PARTITION BY company, location,
industry, total_laid_off, percentage_laid_off, `date`,
stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicates_cte
WHERE row_num > 1;

-- New staging table with row_num as extra column to delete duplicated records with row_num = 2
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) 

-- Make sure staging table was created correctly
SELECT *
FROM layoffs_staging2;

-- Insert cte data into new table
INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, 
country, funds_raised_millions) AS row_num
FROM layoffs_staging;

-- Check data was inserted correctly and new column was added
SELECT *
FROM layoffs_staging2

-- Identify duplicate records
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

-- Delect duplicate records
DELETE
FROM layoffs_staging2
WHERE row_num > 1;

-- Confirm the records were deleted
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;
```
### 3. Standardizing Data
- **Fixing Company Spacing**: After looking at the company column, the spacing of some names were sometimes 1 space ahead of the rest. I simply trimmed the column, so that all the spacing was uniform.
- **Standardizing industry names**: After looking at the industry column, there were two similar values for the crypto currency industry. Both cyrpto and crypto currency were being used. As they are the same industry, I updated the industry value to the same for every company in the cyrpto industry.
- **Fixing Location Names**: After looking at the location column, there were a few locations that had special characters replacing the unicode letters of their location's language. I updated the location names to be accurate to their proper name.
- **Standardizing Country Names**: After looking at the country column, there were 2 different spellings for United States. One of them had a trailing period at the end that needed to be removed. I removed the trailing period, so they shared a uniform name.
- **Reformatting and Changing Date**: After looking at the date column, the date format was incorrect. I reformatted the date to match the proper format, so mySQL could easily identify the dates when exploring the data. I also noticed the data type had not been changed when the data was implemented into the table and it still remained as text. I converted the column to the proper date data type.
```sql

-- Identified the spacing of some company names was off 
SELECT company, trim(company)
FROM layoffs_staging2

-- Updated the names to be uniform in spacing
UPDATE layoffs_staging2
SET company = trim(company);

-- Identified multiple names being used for crypto industry (Cyrpto and Crypto Currency)
SELECT DISTINCT(industry)
FROM layoffs_staging2
ORDER BY industry;

-- Identify the records using the same name
SELECT *
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';

-- Updated the Crypto industry name to be the same
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

-- Identified special characters in location names 
SELECT DISTINCT(location)
FROM layoffs_staging2
ORDER BY location;

-- Updated the names to their correct spelling
UPDATE layoffs_staging2
SET location = 'Düsseldorf'
WHERE location = 'DÃ¼sseldorf';

UPDATE layoffs_staging2
SET location = 'Florianópolis'
WHERE location = 'FlorianÃ³polis'

UPDATE layoffs_staging2
SET location = 'Malmö'
WHERE location = 'MalmÃ¶';

-- Identified 2 different United States entities; one with a trailing period
SELECT DISTINCT country
FROM layoffs_staging2;

-- Updates table and removes trim off the United States. entity
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country = 'United States.';

-- Convert the date column into the proper format
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

-- Convert the date column into the proper data type
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

### 4. Analyze Null or Blank values
- **Finding NULL and Blank Values**: First, I noticed that the industry table had some blank and null values. I made a query to find all the NULL and blank values throughout the column.
- **Setting Blank Values to Null**: After finding the NULL or blank values within the industry column, I decided it would be easier to handle the blank values as null.
- **Fixing NULL and Blank Values**: After setting all industy values to null, I wanted to see if there were other records from the same company that had an industry value. I used a SELF JOIN to match the companies with their other records with an industry value. Using an update statement, I changed all the null values to their respected industry.

```sql

-- Identified null or blank values in the industry column
SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry  = '';

-- Set blank values to NULL
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';


-- Analyzes companies with a null industry value and an actual industry value in another record with the same company name
SELECT *
FROM layoffs_staging2 t1
JOIN layoffs_staging t2 
	ON t1.company = t2.company
    AND t1.location = t2.location
WHERE t1.industry IS NULL
AND t2.industry is NOT NULL;

-- Populates the company's missing industry value with its matching industry value
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry is NOT NULL;

```

### 5. Remove Any Ambiguous or Meaningless Data

- **Removing Ambiguous Data**: The data is about company layoffs, but there are still remaining records that have no layoff data that may get in the way of exploratory analysis. After analyzing which data had no layoff data like total_laid_off or percentage_laid_off, I deleted those records since they provided no value.
- **Removing Meaningless Data**: Earlier in the project, a row_num column was made to help identify duplicate records in the dataset. After cleaning up the duplciates, that column is no longer necessary in the dataset. I deleted the column, and finalized cleaning the layoff dataset.

```sql

-- Analyzes which companies have no layoff data
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Deletes entities with no layoff data
DELETE 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Deletes the row number column from previous exploration as its no longer needed
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

-- Layoff table is cleaned and ready for explorative analysis

```
## Conclusion

Throughout this project, I have gained a better understanding of database setup and data cleaning. The real-world scenario with the company layoff data helps me feel better prepared for my future career. Making sure the data is clean and ready for exploratory analysis is often one of the first steps taken before diving into the data to drive better business decicions. Next, I will be performing explorative data analysis on this same data set.

