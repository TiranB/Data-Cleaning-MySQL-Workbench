# Data-Cleaning-MySQL-Workbench
This project demonstrates how to clean and prepare raw sales data using SQL queries in MySQL Workbench. 
The goal is to transform inconsistent and incomplete data into a structured, analysis-ready format.

## Data
This data tracks layoff events by companies, and includes contextual info like how much funding the company has raised, what industry they’re in, and what stage they're at
- Source:-
- Content:-

| Column Name              | Description                                                                       |
|--------------------------|-----------------------------------------------------------------------------------|
| `company`                | The name of the company where layoffs occurred (e.g., Atlassian, SiriusXM)        |
| `location`               | The city where the company or layoffs were primarily located                      |
| `industry`               | The sector or domain the company operates in (e.g., Media, Other)                 |
| `total_laid_off`         | The number of employees laid off                                                  |
| `percentage_laid_off`    | Proportion of workforce laid off, expressed as a decimal (e.g., 0.05 = 5%)        |
| `date`                   | The date when the layoffs were reported or took place                             |
| `stage`                  | The funding stage of the company (e.g., Post-IPO = after initial public offering) |
| `country`                | The country where the company is based                                            |
| `funds_raised_millions`  | Total funds raised by the company in millions of dollars                          |

## Creating New Database
Created a new database called *world_layoffs* and directly imported the CSV file into the database with default data types.

## Creating a staging table
A stage table is created for the purpose of keeping the raw dataset untouched. That's because we are going to do several changes and clean the raw dataset. That has to be done in the *layoff_staging* table

```sql
CREATE TABLE layoffs_staging
LIKE layoffs;
INSERT layoffs_staging
SELECT * FROM layoffs;
SELECT * 
FROM layoffs_staging;
```

## Removing the Duplicates
- Group each row into a category based on every column and then numbered them.

```sql
SELECT *, 
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```

- Filter the rows which are having row_num as 2 to check the duplicate values.
```sql
  WITH duplicate_cte AS
(
SELECT *, 
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT * 
FROM duplicate_cte
WHERE row_num > 1;
```

- since we cannot delete columns from this temporary table, we have to create a new table with included row_num column. so then we can alter that table.
- Right click the *layoff_staging* table and select *copy to clipboard* and then *create statement*
- Edit the newly created table as *layoff_staging2* and add a new column as *row_num*

```sql
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
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
- Insert  the data to the new table (including *raw_num*)
```sql
INSERT INTO layoffs_staging2
SELECT *, 
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 'date', stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```

- Delete the data which have *raw_num* as 2

```sql
DELETE  
FROM layoffs_staging2
WHERE row_num > 1;
```

## Standardizing Data

- Remove the white spaces in *company* column
```sql
UPDATE layoffs_staging2
SET company = trim(company);
```

- Set separate *industry* values *Crypto*, *Crypto Currency*, *CryptoCurrency* into one value *Crypto*
```sql
UPDATE 	layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

- Set separate *country* values *United States*, *United States.* intoone value *United States*
```sql
UPDATE layoffs_staging2
SET country = 'United States'
WHERE country like 'United States%';
```

- Change the string data in *date* column into a date and then change column datatype as *date*
```sql
UPDATE layoffs_staging2
SET `date` = str_to_date(`date`,'%m/%d/%Y')
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

## Manage Null and Missing values

- We can see there are missing or null values in the industry column. But there are other rows which show exact same *company* name and *location*. So we can pick the *industry* from that rows.
  First the missing values have to be replaced as Null values for simplicity of the query.
```sql
UPDATE layoffs_staging2
SET industry = NULL 
WHERE industry = '';
```

- Then we can join
```sql
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

- Delete the unusable data which have null values in both *total_laid_off* and *percentage_laid_off*
```sql
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

## Remove unnecessary column
```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```


## Notice
If you cannot update a table, check *SQL Editor* in *Preferences*.
Uncheck the *safe updates*

![uncheck the safe updates]()



  
  


