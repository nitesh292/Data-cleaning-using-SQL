# Data-cleaning-using-SQL
```sql
-- Step 1: Create a working table
CREATE TABLE layoffs_work AS
SELECT * 
FROM layoffs;

-- Step 2: Assign row numbers to identify duplicates
CREATE TABLE layoffs_work2 AS
SELECT *, 
       ROW_NUMBER() OVER (
           PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions
           ORDER BY company
       ) AS rowno
FROM layoffs_work;

-- Step 3: Remove duplicate rows
DELETE 
FROM layoffs_work2
WHERE rowno > 1;

-- Step 4: Standardize company names by trimming spaces
UPDATE layoffs_work2 
SET company = TRIM(company);

-- Step 5: Standardize country names
UPDATE layoffs_work2 
SET country = 'United States'
WHERE country LIKE 'United States%';

-- Step 6: Remove rows with null values for critical columns
DELETE 
FROM layoffs_work2
WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;

-- Step 7: Fix null/empty `industry` values using non-null references
UPDATE layoffs_work2 t1
JOIN layoffs_work2 t2
  ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
  AND t2.industry IS NOT NULL;

-- Standardize industry column by removing placeholder empty strings
UPDATE layoffs_work2
SET industry = NULL
WHERE industry = '';

-- Step 8: Standardize and convert the date column format
UPDATE layoffs_work2
SET date = STR_TO_DATE(date, '%m/%d/%Y');

-- Step 9: Modify the date column to be of type DATE
ALTER TABLE layoffs_work2
MODIFY COLUMN date DATE;

-- Step 10: Drop the `rowno` column as it's no longer needed
ALTER TABLE layoffs_work2
DROP COLUMN rowno;

-- Final Step: Validate cleaned data
SELECT * 
FROM layoffs_work2;
```
