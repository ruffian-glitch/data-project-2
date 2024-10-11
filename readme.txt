**Layoffs Data Cleaning**

This repository contains SQL scripts for cleaning and preparing layoffs data for analysis. The main objective is to ensure data integrity by removing duplicates, standardizing entries, dealing with null or blank values, and eliminating irrelevant rows.
SQL Scripts Overview
**1. Data Preparation**

    Creating a Staging Table
    A copy of the raw dataset is created to facilitate data cleaning.

    sql

    CREATE TABLE layoffs_staging LIKE layoffs;
    INSERT INTO layoffs_staging SELECT * FROM layoffs;

**2. Removing Duplicates**

    Identifying Duplicates
    The script identifies duplicate records using a Common Table Expression (CTE).

    sql

WITH duplicate_cte AS (
    SELECT *,
    ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
    FROM layoffs_staging
)
SELECT * FROM duplicate_cte WHERE row_num > 1;

Creating a New Table for Clean Data
A new table is created to store filtered records.

sql

CREATE TABLE layoffs_staging2 (
    company TEXT,
    location TEXT,
    industry TEXT,
    total_laid_off INT DEFAULT NULL,
    percentage_laid_off TEXT,
    date TEXT,
    stage TEXT,
    country TEXT,
    funds_raised_millions INT DEFAULT NULL,
    row_num INT
);

Inserting Filtered Data
Only unique records are inserted into the new table.

sql

    INSERT INTO layoffs_staging2
    SELECT *, ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) AS row_num
    FROM layoffs_staging;

    DELETE FROM layoffs_staging2 WHERE row_num > 1;

**3. Standardizing Data**

    Trimming Whitespace and Updating Values
    Data is standardized by trimming unnecessary spaces and updating entries.

    sql

UPDATE layoffs_staging2 SET company = TRIM(company);
UPDATE layoffs_staging2 SET industry = 'Crypto' WHERE industry LIKE 'Crypto%';

Date Format Correction
The date format is standardized to a proper SQL date format.

sql

    UPDATE layoffs_staging2 SET date = STR_TO_DATE(date, '%m/%d/%Y');
    ALTER TABLE layoffs_staging2 MODIFY COLUMN date DATE;

**4. Dealing with NULL Values**

    Identifying and Updating NULL Entries
    Null or blank values are examined and updated where necessary.

    sql

UPDATE layoffs_staging2 SET industry = NULL WHERE industry = '';

Using Join for Null Replacement
Industry values are filled in from existing records where applicable.

sql

    UPDATE layoffs_staging2 AS t1 
    JOIN layoffs_staging2 AS t2 ON t1.company = t2.company
    SET t1.industry = t2.industry
    WHERE t1.industry IS NULL AND t2.industry IS NOT NULL;

**5. Removing Irrelevant or Incomplete Data**

    Deleting Irrelevant Rows
    Records with null values in critical fields are removed from the dataset.

    sql

    DELETE FROM layoffs_staging2 WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

Final Steps

    Cleanup
    The temporary row number column is dropped from the cleaned table.

    sql

    ALTER TABLE layoffs_staging2 DROP COLUMN row_num;

Conclusion

This data cleaning process enhances the quality and usability of the layoffs dataset, making it ready for further analysis. Feel free to explore the SQL scripts and adapt them for your needs.
