Lab 5 — SQL Queries
1. List all accounts of Perryridge branch with balance < 1000.
SELECT *
FROM account
WHERE branch_name = 'Perryridge'
AND balance < 1000;
2. Find the customer names, loan numbers, and loan amounts, for all loans at the Perryridge branch.
SELECT customer_name, loan_number, amount
FROM borrower, loan
WHERE borrower.loan_number = loan.loan_number
AND branch_name = 'Perryridge';
3. Find the names of all branches that have assets greater than at least one branch located in Brooklyn.
SELECT branch_name
FROM branch
WHERE assets > ANY
(SELECT assets
 FROM branch
 WHERE branch_city = 'Brooklyn');
4. Find all customers who have an account but no loan at the bank.
SELECT customer_name
FROM customer c
WHERE EXISTS
(SELECT *
 FROM depositor d
 WHERE d.customer_name = c.customer_name)
AND NOT EXISTS
(SELECT *
 FROM borrower b
 WHERE b.customer_name = c.customer_name);
5. Find the names of all branches that have assets greater than those of at least one branch located in Brooklyn.
SELECT branch_name
FROM branch
WHERE assets > ANY
(SELECT assets
 FROM branch
 WHERE branch_city = 'Brooklyn');
6. Find all customers who have an account at all the branches located in Brooklyn.
SELECT customer_name
FROM customer c
WHERE NOT EXISTS
(SELECT *
 FROM branch b
 WHERE branch_city = 'Brooklyn'
 AND NOT EXISTS
 (SELECT *
  FROM depositor d, account a
  WHERE d.account_number = a.account_number
  AND d.customer_name = c.customer_name
  AND a.branch_name = b.branch_name));
7. Find all customers who have at most one account at the Perryridge branch.
SELECT customer_name
FROM customer c
LEFT JOIN depositor d
ON c.customer_name = d.customer_name
LEFT JOIN account a
ON d.account_number = a.account_number
AND a.branch_name = 'Perryridge'
GROUP BY customer_name
HAVING COUNT(account_number) <= 1;
8. Find the average account balance at each branch.
SELECT branch_name, AVG(balance)
FROM account
GROUP BY branch_name;
9. Find the average balance for each customer who lives in Harrison and has at least three accounts.
SELECT customer_name, AVG(balance)
FROM customer c, depositor d, account a
WHERE c.customer_name = d.customer_name
AND d.account_number = a.account_number
AND customer_city = 'Harrison'
GROUP BY customer_name
HAVING COUNT(account_number) >= 3;
10. List loan data, ordered by decreasing amounts, then increasing loan numbers.
SELECT *
FROM loan
ORDER BY amount DESC, loan_number ASC;
11. Find the average account balance of those branches where the account balance is greater than Rs. 1200.
SELECT AVG(balance)
FROM account
WHERE balance > 1200;
12. Find the maximum across all branches of the total balance at each branch.
SELECT MAX(total)
FROM
(SELECT branch_name, SUM(balance) total
 FROM account
 GROUP BY branch_name);
13. Select the accounts with maximum balance.
SELECT *
FROM account
WHERE balance =
(SELECT MAX(balance)
 FROM account);
14. Find all branches where the total account deposit is greater than the average of the total account deposits at all branches.
SELECT branch_name, SUM(balance)
FROM account
GROUP BY branch_name
HAVING SUM(balance) >
(SELECT AVG(total)
 FROM
 (SELECT branch_name, SUM(balance) total
  FROM account
  GROUP BY branch_name));
