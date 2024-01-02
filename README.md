# Portfolio-5/ MySQL Data Cleaning
The task is to clean the dataset below (the same as in Portfilio Project 4) in SQL

![image](https://github.com/A-Olson8/Portfolio-5/assets/95314634/3749361d-b618-4fc6-86c7-eefdae94b97c)
![image](https://github.com/A-Olson8/Portfolio-5/assets/95314634/a8e81eae-39cd-40f6-aa54-075b3f31a1cb)
![image](https://github.com/A-Olson8/Portfolio-5/assets/95314634/fc5232c1-045f-43d2-9d50-58dc261586df)


-- Since the table was imported from a CSV file, a primary key (id) should be created


```
ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
ADD ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY;

ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` AUTO_INCREMENT = 1;


```

-- The first step of data cleaning is to drop any duplicate rows
-- The code directly below shows the duplicate rows using the ROW_NUMBER() function


```
SELECT `First Name`, `Phone Number`, ROW_NUMBER()   
OVER (PARTITION BY `First Name` ORDER BY `First Name`) AS row_num   
FROM `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`;  

-- This actually deletes the duplicate rows
DELETE FROM `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` WHERE `ID` IN(  
    SELECT `ID` FROM (SELECT `ID`, ROW_NUMBER()   
       OVER (PARTITION BY `First Name` ORDER BY `First Name`) AS row_num   
    FROM `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`) AS temp_table WHERE row_num>1  
);


```

-- Going column by column, the second step is to trim all of the whitespace in the columns (`First Name`, `Last Name`).

-- To do this, safe updates will need to be toggled off in Edit -> Preferences -> SQL Editor -> Safe Updates
-- Safe Updates should be toggled on after the updates


```

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `First Name` = TRIM(`First Name`);

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Last Name` = TRIM(`Last Name`);


```

-- The third step is to make sure that the Gender column has values


```

Update `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Gender` = CASE When `Gender` = 'male' THEN 'M'
	   When `Gender` = 'Male' THEN 'M'
       When `Gender` = 'Female' THEN 'F'
       When `Gender` = 'female' THEN 'F'
	   ELSE `Gender`
	   END

```


-- The Fourth Next step is to make sure that the Phone Numbers column has uniform values (xxx-xxx-xxxx)

-- The 4 lines of code below strips the '-' and '|' characters so that we're left with (xxxxxxxxxx)

-- Lastly, we need to interject the '-' character in all of the phone number values
-- so that they all follow a uniform standard (xxx-xxx-xxxx)


```
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Phone Number` = REPLACE(`Phone Number`, '-', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Phone Number` = REPLACE(`Phone Number`, '|', '')



UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET 
    `Phone Number` = 
        LEFT(`Phone Number`, 3) + "-" + 
        SUBSTRING(`Phone Number`, 3,3) + "-" +
        RIGHT(`Phone Number`, 4)



```

-- Fifth, we will split the address into a street, city, state and zip code column
-- We will then trim any excess spaces/characters 

-- The lines below will create the four new columns


```

ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
Add `Split Street Address` Nvarchar(255);

ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
Add `Split City Address` Nvarchar(255);

ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
Add `Split State Address` Nvarchar(255);

ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
Add `Split Zip Code` Nvarchar(255);




-- We then populate and clean the `Split Street Address` Column

--Split
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split Street Address` = SUBSTRING_INDEX(Address, ',', 1)

-- Clean up
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Phone Number` = REPLACE(`Split Street Address`, '833 Waterbury Drive.', '833 Waterbury Drive')



-- Populate and clean the `Split City Address` Column

-- Split
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split City Address` = SUBSTRING_INDEX(Address, ',', 2)

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split City Address` = SUBSTRING_INDEX(`Split City Address`, ',', -1)

--Clean Up
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split City Address` = TRIM(`Split City Address`);

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split City Address` = REPLACE(`Split City Address`, '!', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split City Address` = REPLACE(`Split City Address`, ':', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split City Address` = REPLACE(`Split City Address`, '"Charlotte""', 'Charlotte')



-- Populate and clean the `Split State Address` Column

-- Split
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split State Address` = SUBSTRING_INDEX(Address, ',', 3)

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split State Address` = SUBSTRING_INDEX(`Split State Address`, ',', -1)

-- Clean up
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split State Address` = TRIM(`Split State Address`);

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split State Address` = REPLACE(`Split State Address`, 'N.C.', 'North Carolina')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split State Address` = REPLACE(`Split State Address`, 'N.C', 'North Carolina')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split State Address` = REPLACE(`Split State Address`, 'S.C', 'South Carolina')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split State Address` = REPLACE(`Split State Address`, 'North Carolina.', 'North Carolina')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split State Address` = REPLACE(`Split State Address`, 'South Carolina.', 'South Carolina')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split State Address` = REPLACE(`Split State Address`, '"Charlotte""', 'North Carolina')



-- Populate and clean the `Split Zip Code` Column

-- Split
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split Zip Code` = SUBSTRING_INDEX(Address, ',', -1)

-- Clean up
UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Split Zip Code` = TRIM(`Split Zip Code`);

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Split Zip Code` = REPLACE(`Split Zip Code`, '"Charlotte""', '28212')



-- Lastly, delete the address column that is no longer needed

ALTER TABLE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
DROP COLUMN Address;


```

-- The Sixth step is to clean the Position column


```

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1` SET `Position` = TRIM(`Position`);

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, ':', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '}', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '[', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, ']', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '/', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '!', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '?', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '<', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '(', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Position` = REPLACE(`Position`, '^', '')



```

-- Step Seventh is to clean the Last Years Salary column


```

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Salary` = REPLACE(`Last Years Salary`, 'NaN', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Salary` = REPLACE(`Last Years Salary`, 'N/a', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Salary` = REPLACE(`Last Years Salary`, '$', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Salary` = REPLACE(`Last Years Salary`, ',', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Salary` = REPLACE(`Last Years Salary`, '"', '')


```


-- Step Eight is to clean the Last Years Commission column


```

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Commission` = REPLACE(`Last Years Commission`, 'NaN', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Commission` = REPLACE(`Last Years Commission`, 'N/a', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Commission` = REPLACE(`Last Years Commission`, '$', '')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Last Years Commission` = REPLACE(`Last Years Commission`, ',', '')


```

-- Lastly, step nine is to standardize the date format within the Start Date and End Date columns


```

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `Start Date` = REPLACE(`Start Date`, '/', '-')

UPDATE `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
SET `End Date` = REPLACE(`End Date`, '/', '-')


```


-- Display all employees who had income last year


```

Select * from `SQL Data Cleaning Demo`.`DataCleaningPortfolioFile - Sheet1`
Where `Last Years Salary` != '';


```
The FInal Output

![image](https://github.com/A-Olson8/Portfolio-5/assets/95314634/a4770d0c-3436-45fa-8c2a-acb00fc14114)

