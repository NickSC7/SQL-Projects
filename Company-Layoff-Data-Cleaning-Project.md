## Project Overview

**Project Title**: Company Layoff Data Cleaning

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to clean company layoff data. The project involves setting up a layoff databse, importing the layoff data into mySQL, and cleaning the data.

## Objectives

1. **Set up a layoff database**: Create and populate a company layoff database with layoff data.
2. **Remove Duplicates (If any)**: Identify and remove any duplicates
3. **Standardize the Data**:
4. **Analyze Null or Blank values**:
5. **Remove Any Columns**:

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
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

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
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

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


