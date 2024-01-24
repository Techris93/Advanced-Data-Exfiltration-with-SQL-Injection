## Error-Based-DataExfiltration-SQL-Injection

### Overview 
- Use SQL queries to cause errors
- Use Burp Suite's Intruder to exfiltrate data with response codes
- Extract data from an SQL server with Error-based SQL Injection

### Scenario 

In MySQL, you can cause errors in various ways, and one of them is returning more than 1 row from a subquery. Here is an example of that:
```
SELECT IF((SELECT username from users LIMIT 1) LIKE 'a%',(SELECT table_name FROM information_schema.tables),'a')
```
This is how this query works:
- It selects the first username in the users table, and compares it to “a%", where “%" is a wildcard
- If the condition is True, the SELECT table_name FROM information_schema.tables query is run, which returns multiple rows, causing an error
- If the condition is False, the query simply returns 1 row containing “a".

This means that if the username selected is administrator, the server will throw an Internal Server Error as it matches “a%", and subsequently “ad%" etc.

**Step 1: Extracting Data from Response Codes**

Typically, when a database query is unsuccessful and exceptions are not properly managed, the server will generate an Internal Server Error, which is indicated by the HTTP status code 500.

a. Firstly, open Burp Suite Community, then head to the proxy tab, turn on Intercept, then open the browser and browse the vulnerable web application.

b. Then enter any username and password and press “Login", you should see a request on Burp Suite.
![image](https://i.imgur.com/YGw9mYG.png)

c. Right-click any area on the request and click “Send to Intruder".
![image](https://i.imgur.com/MAWm6O0.png)

d. Navigate to https://www.urlencoder.org/ and encode this payload: 
```
' UNION SELECT IF((SELECT password from users WHERE username='test user') LIKE 'a%',(SELECT table_name FROM information_schema.tables),'a'), "a", "b", "c" --
```

Note: This payload checks if test user's password starts with the character “a". If the condition is True, it executes SELECT table_name FROM information_schema.tables which returns more than 1 row, causing an error. If it is False, it simply returns the value “a". 

e. Paste the encoded payload into the “username" parameter in Burp Suite's Intruder.
![image](https://i.imgur.com/yp9rPMU.png)

f. Click `“Clear §"` to remove all payload positions. Then, select the area where the Intruder should place your payloads, which would be the character you are trying to brute force and click `“Add §"`.
![image](https://i.imgur.com/k2hTGa8.png)

g. Click on the “Payloads" tab. In the “Payload settings" / "Payload options" section, add all alphanumeric characters into the list, a-z and 0-9.
![image](https://i.imgur.com/ZSOkViI.png)
![image](https://i.imgur.com/6H4p0V1.png)

h. Go back to the “Positions" tab and click “Start attack".

Let the Intruder run until complete, or until you see a response status code such as 500. This means that the character matches test user’s password.
![image](https://i.imgur.com/xshgVz3.png)

i. Add the character to the next request and repeat the process until no more characters are found.

## References
https://app.stackup.dev
