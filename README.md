# CSDS397Assignment2
In this repo, you will find all of the necessary instructions to replicate an ELT ingestion of data. I used Microsoft SQL Server, and some of the syntax may be specific to that program.

Firstly, after downloading the CSV file in the Data folder, you will want to ingest the data. Scripts for this are in the Ingestion folder. Create the necessary tables by running the scripts in Create Tables, then take in the raw data by running the script in Read From CSV after replacing the file pathway with one to your copy of the file.

Then, you should clean the data. The scripts in each of the files in the Cleaning folder contain SQL procedures that make it a bit easier. After creating the procedures, run the deduplication procedure with the following statement:
EXEC deduplicate;

Then, use the moveRows procedure to flag any rows that may need to be manually checked. This uses dynamic SQL to move any rows to the flagged table that meet a certain condition. As an example:
EXEC moveRows 'name IS NULL';
This will flag any rows where the name is null. You can use this to flag any missing/null/bad values in any column by replacing the condition.

Then, fix any typos or inconsistencies in values with the fixTypos procedure. You can use something like the following to see all unique values in a given column:
SELECT department FROM employee_data_source GROUP BY department;
The fixTypos procedure can be used to fix any typos by replacing all instances of a value in a certain column with another value. For example:
EXEC fixTypos 'department', 'Slaes', 'Sales';
will replace any instances of 'Slaes' with 'Sales'. You can change the values to whatever other typos exist. Notably, the checks don't care about capitalization, so something like
EXEC fixTypos 'department', 'IT', 'IT';
will automatically replace any 'it' or 'It' or 'iT' with 'IT', which helps the formatting consistency. Run that command on each value after fixing the typos to ensure capitalization consistency.

SQL Server automatically handles the date formatting consistency with its DATE variable type during ingestion. To ensure that all hiring dates are in the past, you can use something like:
EXEC moveRows ‘date_of_joining > CAST(GETDATE() AS DATE)’

Finally, since some of the salaries may look slightly suspiciously high. The following statement will help you get a better overview on some of the properties of the table:
SELECT 
    AVG(salary) AS avgSal,
    MIN(salary) AS minSal,
    MAX(salary) AS maxSal,
    STDEV(salary) AS stdevSAL
FROM employee_data_source;

If you add 2 or 3 standard deviations to the average salary, any salaries higher than that may want to get flagged for manual review:
EXEC moveRows 'salary > 1000000';

Finally, when all the data is nice and cleaned, use the following statement to move it to the permanent table:
INSERT INTO employee_data SELECT * FROM employee_data_source;
