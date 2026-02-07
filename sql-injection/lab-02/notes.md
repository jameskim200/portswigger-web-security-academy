Lab: SQL injection vulnerability allowing login bypass 
https://portswigger.net/web-security/sql-injection/lab-login-bypass

This lab contains a SQL injection vulnerability in the login function.

To solve the lab, perform a SQL injection attack that logs in to the application as the administrator user.

---

### Enumeration

- Navigating to `My account` leads us to the login page at `/login`
- In the `/login` page, we see two fields we can enter for entering our username and password
- Inspecting the page, it submits a form with method POST
  - in addition to the username and password value, there is a hidden csrf value
  - it also seens to have a session Cookie attached to the POST request (intercepted with Burp)
  - `csrf=l89q3lEcsWIhkXAKussCas3IBo71TdAh&username=test_u&password=test_p`
- admin / admin displays the error message "Invalid username or password."

### Vulnerability Detection

- inserting a single quote to the username field leads to an `Internal Server Error` 
- inserting a single quote to the password field leads to the same `Internal Server Error`
- this leads to believe the backend is taking the user input and concatenating the user input to the query, leaving the login function vulnerable to SQL injection

### Exploitation

- let's derive a SQL query that the login function may be using in the background
  - something like `SELECT * FROM users WHERE username = '<username>' AND password = '<password>'`
  - how can we ensure that the WHERE clause returns TRUE?
  - admin'-- -
  - administrator'-- -
  - ' OR 1=1-- -
  - but this is not the desired goal! because our objective is to login as user `adminstrator`
  - `administrator' OR 1=1-- -`

