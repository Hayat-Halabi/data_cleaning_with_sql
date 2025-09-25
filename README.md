# data_cleaning_with_sql
Cleaning data using Structured Query Language

Shown below I have depicted a raw dataset of global layoffs. This dataset contains duplicates, null/missing values, as well as unnecessary rows and columns. 

![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/e76d1bdd0bb9554ffbd85c92a51b16b392479ec6/Screenshot%20(2).png)

Before making any changes to the original dataset, we must make a duplicate of the table and fill the values in case any mistakes were to happen, as per best practices. I also altered the original dataname to layoffs_raw for increased clarity. The new table is created and filled with values, whereupon we can now move on to the next steps. 

![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%20(4).png?raw=true)

## Removing Duplicates

For the first step I like to remove any duplicates the data might have, since duplicates can skew analysis. In this dataset, I don't have a unique row ID, which makes it much easier to check for duplicates. I created a new column called row_number to check for any duplicates. This column is going to have a window function (ROW_NUMBER()) that assigns a unique integer to each row, partitioned by all column names.

This way, true duplicates will receive multiple row numbers within the same partition, and I can keep only the first occurrence while removing the rest.
![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%202025-09-22%20234648.jpg?raw=true) 
Because we cannot update a CTE, there is another option to creating another table that has an extra row, and deleting the row that is greater than 1. ![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%20(5).png?raw=true) Finally I wrote a simple line of code to delete the duplicates:
```sql
DELETE
FROM layoffs2
WHERE unique_row_number>1;
```
## Standerdizing Data 

Standerdizing data refers to the process of ensuring that data within a database is consistent in its format, meaning, and structure. 
I’ll start by trimming columns and updating values to keep them neat and uniform. Next, I’ll check for duplicates or misspellings and make adjustments. For example, in the 'industry' column, 'crypto' appeared in three variations, so I standardized it to one value. As you see below I highlighed the changes in the action output affected. ![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%202025-09-23%20011437.png?raw=true)

So far I've taken a look at the 'country' and 'industry' column, next up the locations column, I ran a simple line of code to view the output and found 1 discrepency. I then ran another line to check for any regular expressions I might of missed. This can happen when the original data contains special letters (like ö, ü, ó, ã), and databases or import processes don't interpret them correctly. I found that I missed 2 more! I went ahead and set an update statement to fix the errors. ![image](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/gg.png?raw=true)  I noticed that the 'date' column is in string format, which isn't useful if I want to work on any timeseries data, so I'm going to update it to a date column. I used a simple line of code to do so: 
```sql
SELECT `date`,
STR_TO_DATE ( `date` , '%m/%d/%Y') 
FROM layoffs2;

UPDATE layoffs2
SET `date` = STR_TO_DATE ( `date` , '%m/%d/%Y')

ALTER TABLE layoffs2
MODIFY COLUMN `date` DATE;
```
## NULL / Blank values

It’s safe to say that almost every dataset will contain some NULL values. The best way to handle them is to first understand why they are NULL. In some cases, NULLs are valid because the data doesn’t apply. In other cases, they should be replaced with a default value, such as 0 or 'unknown'. Sometimes, NULLs appear due to issues in the data pipeline, and in those situations they can often be corrected by retrieving the proper values from the source.

Previously, while working with the Industry column, there were a few blank and NULL values. I checked the companies associated with these blank values and was able to populate and update the NULL rows in the Industry column. I joined the table to itself on the company column to find matching entries where the industry information was available. I then populated the NULL values with the corresponding industry names using the following queries:

```sql
SELECT *
FROM layoffs2 t1
join layoffs2 t2
	ON t1.company = t2.company
WHERE( t1.industry is NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

UPDATE layoffs2
set industry = NULL 
where industry = '';

UPDATE layoffs2 t1
join layoffs2 t2
	ON t1.company = t2.company
    SET t1.industry = t2.industry 
WHERE( t1.industry is NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

```
Moving onto the total_laid_off and percentage_layoffs columns, I noticed that some rows have NULL values in both fields. Since these rows don’t provide any meaningful information, I decided to remove them from the dataset to ensure accurate calculations of totals and percentages.
Finally I decided to drop any unnecessary columns that add no value to the table, in the begining of this project I created a unique row number to check for any duplicates, now I no longer need that column so I'm going to drop it. ![image](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%20(7).png?raw=true)
