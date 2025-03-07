This is a repository to showcase skills, share projects and track my progress in Data Analytics related topics.
Let me know if you have any questions about any of the projects!








-- SQL Project: Data Cleaning
-- https://www.kaggle.com/datasets/swaptr/layoffs-2022



SELECT * FROM layoffs.layoffsraw;

-- First of all, create a staging table
CREATE TABLE staging1 SELECT * FROM layoffs.layoffsraw;
SELECT * FROM staging1;


-- steps to follow:
-- 1. clean duplicates
-- 2. standardize data and fix errors
-- 3. Populate blanks and manage NULLs
-- 4. consider column or row downsizing



-- 1. Clean duplicates
SELECT *, ROW_NUMBER() OVER (PARTITION BY company, location, industry,total_laid_off,percentage_laid_off, `date`,stage, country,funds_raised) AS row_num FROM staging1;

WITH duplicate_cte AS
(SELECT *, ROW_NUMBER() OVER (PARTITION BY company, location, industry,total_laid_off,percentage_laid_off, `date`,stage, country,funds_raised) AS row_num FROM staging1
)
SELECT * from duplicate_cte WHERE row_num > 1; -- shows empty = no duplicates



-- 2. Standardizing
SELECT `date` , convert (`date` , date) FROM staging1;


SELECT company FROM staging1 GROUP BY company ORDER BY company; -- no typos under company name
SELECT * FROM staging1 WHERE company LIKE 'Aurora%'; -- ...except two similarly named companies, but they check out ok

SELECT DISTINCT industry FROM staging1; -- no typos under industry name
SELECT DISTINCT country FROM staging1 GROUP BY country; -- no typos under country name


-- 3. Nulls and Blanks

SELECT * FROM staging1 WHERE industry = '' OR industry IS NULL ORDER BY industry; -- one company without industry, but:
SELECT * FROM staging1 WHERE company = 'Appsmith'; -- ...there are no other entries from where to populate the empty entry.

-- since no total_employees column, we cannot populate blanks where either total_laid_off or percentage_laid_off are missing

-- set the remaining blanks to nulls
UPDATE staging1 SET percentage_laid_off = NULL WHERE percentage_laid_off ='';
UPDATE staging1 SET total_laid_off = NULL WHERE total_laid_off =''; -- setting to NULL makes calculations easier during EDA phase


-- 4. remove useless rows/columns
 SELECT * FROM staging1
 WHERE total_laid_off IS NULL
 AND percentage_laid_off IS NULL;
 
 DELETE FROM staging1 
 WHERE total_laid_off IS NULL
 AND percentage_laid_off IS NULL; -- cleaning done!
 
 
 SELECT * FROM layoffs.staging1;
