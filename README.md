# data_cleaning_with_sql
Cleaning data using Structured Query Language

Attached below I have a raw dataset of global layoffs. As you can see within this datasets we can find duplicates, Null / missing values and unnecessary rows and columns. 

![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/e76d1bdd0bb9554ffbd85c92a51b16b392479ec6/Screenshot%20(2).png)

First and foremost before making any changes to the original dataset, we must make a duplicate of the table and fill the values just in case any mistakes were to happen, its best practice to do so. I also went ahead and altered the original dataname to layoffs_raw just so its clear. As you can see below the new table is made and filled with values, now we can move on to the next steps. 

![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%20(4).png?raw=true)

## Removing Duplicates

For the first step I like to remove any duplicates the data might have, since duplicates can skew analysis. In this dataset, because I don't have a unique row ID (which makes it alot easier to check for duplicates) I can create a new column called row_number to check for any duplicates. This column is going to have a window function (ROW_NUMBER()) that assigns a unique integer to each row, partitioned by all column names.

This way, true duplicates will receive multiple row numbers within the same partition, and I can keep only the first occurrence while removing the rest.
![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%202025-09-22%20234648.jpg?raw=true) 
Because we cannot update a CTE, there is another option to creating another table that has an extra row, and deleting the row that is greater than 1. ![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%20(5).png?raw=true) Finally I wrote a simple line of code to delete the duplicates:
DELETE
FROM layoffs2
WHERE unique_row_number>1;

## Standerdizing Data 

Standerdizing data refers to the process of ensuring that data within a database is consistent in its format, meaning, and structure. 
I’ll start by trimming columns and updating values to keep them neat and uniform. Next, I’ll check for duplicates or misspellings and make adjustments. For example, in the 'industry' column, 'crypto' appeared in three variations, so I standardized it to one value. As you see below I highlighed the changes in the action output affected. ![image alt](https://github.com/Hayat-Halabi/data_cleaning_with_sql/blob/main/Screenshot%202025-09-23%20011437.png?raw=true)
