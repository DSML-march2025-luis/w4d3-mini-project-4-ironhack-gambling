# Business Challenge: The IronHack Gambling Database Adventure

## Introduction

Welcome to the IronHack Gambling Database Adventure! This challenge is inspired by a real-life job interview scenario from IronHack Gambling. It's designed to test your SQL skills in a fun and engaging way. Imagine you're applying for a data analyst role at IronHack Gambling, and you've been given this challenge to showcase your expertise.

## Scenario

You have been given access to four critical database tables of the IronHack Gambling Data Warehouse: `Betting`, `Product`, `Customer`, and `Account`. These tables are central to 75% of all queries at IronHack Gambling. For this challenge, a snapshot of data involving 10 players and their transactions is provided. Your task is to demonstrate your SQL prowess by retrieving and manipulating this data to provide valuable insights.

## Challenge Format

- The challenge starts with simple queries and gradually increases in complexity.
- For each question, write your SQL query to find the answer.


## Questions

- **Question 01**:  Using the customer table or tab, please write an SQL query that shows Title, First Name and Last Name and Date of Birth for each of the customers.
    
    ```sql
    SELECT Title, FirstName, LastName, DateOfBirth FROM customer;
    ```

- **Question 02**:  Using customer table or tab, please write an SQL query that shows the number of customers in each customer group (Bronze, Silver & Gold). I can see by visual inspection that there are 4 Bronze, 3 Silver and 3 Gold but if there were a million customers how would I do this?
    
    ```sql
    SELECT CustomerGroup, COUNT(*) FROM customer GROUP BY CustomerGroup;
    ```

- **Question 03**: The CRM manager has asked me to provide a complete list of all data for those customers in the customer table but I need to add the currencycode of each player so she will be able to send the right offer in the right currency. Note that the currencycode does not exist in the customer table but in the account table.
    
    ```sql
    SELECT customer.*, account.CurrencyCode FROM customer LEFT JOIN account ON customer.CustId = account.CustId;
    ```

- **Question 04**: Now I need to provide a product manager with a summary report that shows, by product and by day how much money has been bet on a particular product. Please note that the transactions are stored in the betting table and there is a product code in that table  that is required to be looked up (classid & categortyid) to determine which product family this belongs to. Please write the SQL that would provide the report. 

    ```sql
    /* solution 1: adding together all amounts 
    * (warning: this is actually mixing amounts on different currencies)
    */
    SELECT p.product AS Product, b.BetDate AS Bet_Date, SUM(b.Bet_Amt) AS Total_Bet_Amount
    FROM betting b 
    JOIN product p
        ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
    WHERE b.Bet_Amt IS NOT NULL
    GROUP BY p.product, b.BetDate
    ORDER BY p.product, STR_TO_DATE(b.BetDate, '%d/%m/%Y') ASC;


    /* solution 2: grouping based on currency */
    SELECT p.product AS Product, b.BetDate AS Bet_Date, a.CurrencyCode, SUM(b.Bet_Amt) AS Total_Bet_Amount
    FROM betting b 
    JOIN product p
        ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
    JOIN account a
        ON b.AccountNo = a.AccountNo
    WHERE b.Bet_Amt IS NOT NULL
    GROUP BY p.product, b.BetDate, a.CurrencyCode
    ORDER BY p.product, STR_TO_DATE(b.BetDate, '%d/%m/%Y') ASC, a.CurrencyCode;
    ```

- **Question 05**: You’ve just provided the report from question 4 to the product manager, now he has emailed me and wants it changed. Can you please amend the summary report so that it only summarises transactions that occurred on or after 1st November and he only wants to see Sportsbook transactions.
    
    ```sql
    SELECT p.product AS Product, b.BetDate AS Bet_Date, a.CurrencyCode, SUM(b.Bet_Amt) AS Total_Bet_Amount
    FROM betting b 
    JOIN product p
        ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
    JOIN account a
        ON b.AccountNo = a.AccountNo
    WHERE b.Bet_Amt IS NOT NULL
        AND p.Product = "Sportsbook"
        AND STR_TO_DATE(b.BetDate, '%d/%m/%Y') >= STR_TO_DATE("01/11/2012", '%d/%m/%Y')
    GROUP BY p.product, b.BetDate, a.CurrencyCode
    ORDER BY p.product, STR_TO_DATE(b.BetDate, '%d/%m/%Y') ASC, a.CurrencyCode;
    ```
    
- **Question 06**: As often happens, the product manager has shown his new report to his director and now he also wants different version of this report. This time, he wants the all of the products but split by the currencycode and customergroup of the customer, rather than by day and product. He would also only like transactions that occurred after 1st December.
    
    ```sql
    SELECT c.CustomerGroup, a.CurrencyCode, SUM(b.Bet_Amt) AS Total_Bet_Amount
    FROM betting b 
    JOIN product p
        ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
    JOIN account a
        ON b.AccountNo = a.AccountNo
    JOIN customer c
        ON a.CustId = c.CustId
    WHERE b.Bet_Amt IS NOT NULL
        AND STR_TO_DATE(b.BetDate, '%d/%m/%Y') >= STR_TO_DATE("01/12/2012", '%d/%m/%Y')
    GROUP BY c.CustomerGroup, a.CurrencyCode
    ORDER BY c.CustomerGroup, a.CurrencyCode;
    ```

- **Question 07**: Our VIP team have asked to see a report of all players regardless of whether they have done anything in the complete timeframe or not. In our example, it is possible that not all of the players have been active. Please write an SQL query that shows all players Title, First Name and Last Name and a summary of their bet amount for the complete period of November.
    
    ```sql
    SELECT c.CustId, ANY_VALUE(c.Title) AS Title, ANY_VALUE(c.FirstName) AS FirstName, ANY_VALUE(c.LastName) AS LastName, SUM(b.Bet_Amt) AS Bet_Amount
    FROM customer c
    JOIN account a
        ON c.CustId = a.CustId
    LEFT JOIN betting b 
        ON a.AccountNo = b.AccountNo
    WHERE STR_TO_DATE(b.BetDate, '%d/%m/%Y') >= STR_TO_DATE("01/11/2012", '%d/%m/%Y')
        AND STR_TO_DATE(b.BetDate, '%d/%m/%Y') < STR_TO_DATE("01/12/2012", '%d/%m/%Y')
    GROUP BY c.CustId
    ORDER BY c.CustId ASC;
    ```

- **Question 08**: Our marketing and CRM teams want to measure the number of players who play more than one product. Can you please write 2 queries, one that shows the number of products per player and another that shows players who play both Sportsbook and Vegas.
    
    ```sql
    /*  number of products per player */
    SELECT b.AccountNo, COUNT(DISTINCT p.product) as Number_of_Products_Played
    FROM betting b
    JOIN product p
        ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
    GROUP BY b.AccountNo;

    /* players who play both Sportsbook and Vegas */
    SELECT * 
    FROM
        (SELECT AccountNo, COUNT(product)  as amount_of_products 
        FROM
            (SELECT b.AccountNo, p.product
            FROM betting b
            JOIN product p
                ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
            GROUP BY AccountNo, p.product
            ORDER BY AccountNo) AS products_per_customer
        WHERE product IN ('Sportsbook', 'Vegas')
        GROUP BY AccountNo) AS amount_of_products_per_customer
    WHERE amount_of_products > 1;
    ```

- **Question 09**: Now our CRM team want to look at players who only play one product, please write SQL code that shows the players who only play at sportsbook, use the bet_amt > 0 as the key. Show each player and the sum of their bets for both products. 
    
    ```sql
    /* players that have played only "Sportsbook" and sum of the amount for that product */
    SELECT b.AccountNo, SUM(b.Bet_Amt) AS total_bet_amount
    FROM betting b
    JOIN product p
        ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
    WHERE bet_amt > 0
    GROUP BY AccountNo
    HAVING COUNT(DISTINCT p.product) = 1
        AND MAX(p.product) = 'Sportsbook';
    ```

- **Question 10**: The last question requires us to calculate and determine a player’s favourite product. This can be determined by the most money staked. 
    
    ```sql
    /* favourite product for each player */
    WITH RankedBets AS (
        SELECT 
            b.AccountNo, 
            p.product, 
            SUM(b.Bet_Amt) AS total_staked,
            RANK() OVER (PARTITION BY b.AccountNo ORDER BY SUM(b.Bet_Amt) DESC) AS rnk
        FROM betting b
        JOIN product p
            ON b.ClassId = p.CLASSID AND b.CategoryId = p.CATEGORYID
        WHERE bet_amt > 0
        GROUP BY AccountNo, product
        ORDER BY b.AccountNo, total_staked DESC
    )
    SELECT AccountNo, Product, total_staked
    FROM RankedBets
    WHERE rnk = 1;
    ```


- **Free form analysis**: the extra mile in this project is doing an EDA to point out interesting avenues of exploration for this dataset. Where are the large pools of money? What are profitable periods? Can we segment our customers? Present a few insights that you were able to glean during your manipulation of the data.



## Tips

- Write or type your SQL answers for each question in a document.
- Feel free to use a separate editor to draft your queries before pasting them here.
- Ensure your queries are well-formatted and easy to understand.

- You are NOT required to do this, but if you find it easier to load the data into python, reshape it / clean it and write it back into SQL, feel free to do that. In that case, submit also your cleaning file and export your database as an .sql file dump. Make sure that your new database follows good database design (don't dump a denormalized table - your data warehouse team would not let you do that anyway).

## Submission

- Once you have completed the challenge, submit your answers.
- Present your findings to the class with a short presentation.

## Good luck!
