## This is a repository to showcase skills, share projects and track my progress in Data Analytics related topics. Let me know if you have any questions about any of the projects!  








# SQL Project: Data Cleaning  
Source DB:   
https://www.kaggle.com/datasets/swaptr/layoffs-2022  



SELECT * FROM layoffsraw1;  

** First of all, create a staging table  
CREATE TABLE staging2 SELECT * FROM layoffsraw1;  
SELECT * FROM staging2;  


## steps to follow:  
-- 1. clean duplicates  
-- 2. standardize data and fix errors  
-- 3. Populate blanks and manage NULLs  
-- 4. consider column or row downsizing 



## 1. Clean duplicates  
  -- first check if any duplicates are present  
WITH duplicate_cte AS  
(SELECT *, ROW_NUMBER() OVER (PARTITION BY company, location, industry,total_laid_off,percentage_laid_off, `date`,stage, country,funds_raised) AS row_num FROM staging2
)  
SELECT * FROM duplicate_cte WHERE row_num > 1; -- two rows remain, so lets check them and delete if necessary:  

SELECT * FROM staging2 WHERE company = 'Beyond Meat' OR company = 'Cazoo'; -- there are indeed two duplicates. Since UPDATE is not supported via CTE on mysql, so:  
-- lets create a staging3 to delete the rows from there:  

CREATE TABLE `staging3` (  
  `company` text,  
  `location` text,  
  `industry` text,  
  `total_laid_off` text,  
  `percentage_laid_off` text,  
  `date` text,  
  `stage` text,  
  `country` text,  
  `funds_raised` text,  
  row_num INT  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;  

INSERT INTO `staging3`
(`company`,
`location`,
`industry`,
`total_laid_off`,
`percentage_laid_off`,
`date`,
`stage`,
`country`,
`funds_raised`,
`row_num`)
SELECT `company`,
`location`,
`industry`,
`total_laid_off`,
`percentage_laid_off`,
`date`,
`stage`,
`country`,
`funds_raised`, 
ROW_NUMBER() OVER (  
	PARTITION BY company, location, industry, total_laid_off , percentage_laid_off,`date`, stage, country, funds_raised) AS row_num FROM staging2;
  
DELETE FROM staging3 WHERE row_num > 1;  
SELECT * FROM staging3; -- duplicates eliminated - now row_num column can be eliminated

ALTER TABLE staging3  
DROP COLUMN row_num;


## 2. Standardizing  

  -- first the date: we need only yyyymmdd  
ALTER TABLE staging3 ADD COLUMN formatted_date DATE;  
UPDATE staging3 SET formatted_date = STR_TO_DATE(date, '%Y-%m-%dT%H:%i:%s.000Z');  
ALTER TABLE staging3 DROP COLUMN date;  
ALTER TABLE staging3 RENAME COLUMN formatted_date TO date;  
SELECT * FROM staging3;  

 -- location needs to be standardized from [''] and 'nonUS' 
 UPDATE staging3  
 SET location = TRIM(  
 BOTH ' ' FROM SUBSTRING_INDEX(SUBSTRING_INDEX(location, ',', 1), '[', -1))  
 WHERE location LIKE "'%']";  
 
  UPDATE staging3  
  SET location = REPLACE(location, "']", "")  
  WHERE location LIKE '%]';
 
 
 -- Check for typos in company name:  
SELECT company, COUNT(*) AS count FROM staging3  
GROUP BY company HAVING count > 1;  

SELECT DISTINCT company FROM staging3  
GROUP BY company ORDER BY company; -- no typos under company name  
SELECT * FROM staging3 WHERE company LIKE 'Aurora%'; -- ...except two similarly named companies, but they check out ok  

SELECT * FROM staging3 WHERE company LIKE '&Paid' OR company LIKE '#Open'; -- these looked like typos but check out as company names  


SELECT DISTINCT country FROM staging1 GROUP BY country ORDER BY country; -- no typos under country name  


## 3. Nulls and Blanks  

SELECT * FROM staging1 WHERE industry = '' OR industry IS NULL ORDER BY industry; -- one company without industry, but:  
SELECT * FROM staging1 WHERE company = 'Appsmith'; -- ...there are no other entries from where to populate the empty entry.  

-- since no total_employees column, we cannot populate blanks where either total_laid_off or percentage_laid_off are missing  

-- set the remaining blanks to NULL (makes calculations easier during EDA phase)  
UPDATE staging1 SET percentage_laid_off = NULL WHERE percentage_laid_off ='';  
UPDATE staging1 SET total_laid_off = NULL WHERE total_laid_off ='';  


## 4. remove useless rows/columns    
 SELECT * FROM staging1  
 WHERE total_laid_off IS NULL  
 AND percentage_laid_off IS NULL;  
 
 DELETE FROM staging1   
 WHERE total_laid_off IS NULL  
 AND percentage_laid_off IS NULL; -- cleaning done!  
 
 
 SELECT * FROM layoffs.staging1;  
