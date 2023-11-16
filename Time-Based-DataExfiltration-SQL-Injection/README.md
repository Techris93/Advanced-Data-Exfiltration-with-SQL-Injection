# Time-Based-DataExfiltration-SQL-Injection

## Objectives:
- Use SQL queries to cause time delays
- Use Burp Suite's intruder
- Extract data from an SQL server with Time-based SQL Injection

## Scenario:
In some applications, input fields are vulnerable to SQL injection but do not display the results. However, some applications block outgoing traffic or functions that allow out-of-band exfiltration. In such cases, you can get around such limitations by executing time-based SQL injection attacks.

Time-based SQL injection attacks involve causing a time delay in the SQL query to exfiltrate data.

## Required Resources:
- Burp Suite
- A PC with Internet access.
- A vulnerable web application.

**Step 1: Step 2: Using Burp Suite's Intruder**

a. Open Burp Suite Community. Then head to the `proxy` tab, turn on `Intercept`, and open the browser to browse the vulnerable website.
![image](https://i.imgur.com/YWjDe14.png)

b. Once the webpage has opened on the browser, enter any `username` and `password` and press `“Login"`. You should see a request on Burp Suite like the one below.
![image](https://i.imgur.com/zv5U7zE.png)

c. Right-click any area on the request and click `“Send to Intruder"`. Then switch to the `“Intruder"` tab.
![image](https://i.imgur.com/eWgEmDF.png)
Here, you can see the username and password fields highlighted, this means that the payloads will be loaded into those parameters.

d. Click on the `“Payloads"` tab and you will see a page like the one below.
![image](https://i.imgur.com/CVLkAuw.png)

e. In the `“Payload settings" or "Payload options" section`, add all alphanumeric characters into the list,` a-z, 0-9`. 
![image](https://i.imgur.com/iBxIVDe.png)

f. Head back to the `“Positions"` tab to choose an attack type. Select the `“Sniper"` attack type. Click `“Start attack"`. 
![image](https://i.imgur.com/06d1O7S.png)
To view the response time, click on `“Columns"` in the main menu and `“Response received"`

g. Now, let's test this with SQL injection! Close the popup window and edit the `“Payload Positions"` section in Burp Suite. 

**NOTE**: First, navigate to https://www.urlencoder.org/ and encode the following payload with the default settings to make it a valid string in the request body.
```
' UNION SELECT IF((select password from users WHERE username="test user") LIKE "a%" ,SLEEP(10),'a'), "a", "b", "c" -- 
```
h. Copy and paste the encoded payload into the “username" parameter in Burp Suite's Intruder.
![image](https://i.imgur.com/Vvgyg5Y.png)

i. Click “Clear §" to remove all payload positions. Then, highlight the area where the Intruder should place the payloads, and click `“Add §"`. After clicking `“Add §”`, the text on your screen should look like the one below.
![image](https://i.imgur.com/VbsRnOb.png)

j. Click `“Start attack"`.

Since the test user's password begins with `“3"`, you will see a delay of 10 seconds from the request that tested the condition LIKE '3%'.
![image](https://i.imgur.com/WRKgozu.png)

k. Then, add `3` into the “LIKE" condition in the Burp Suite Intruder request as follows:
```
' UNION SELECT IF((select password from users WHERE username="test user") LIKE "3%" ,SLEEP(10),'a'), "a", "b", "c" --
```

l. Define the next payload position. First, click `“Clear §”` to remove the previous position. Then, highlight the new position as follows:
```
' UNION SELECT IF((select password from users WHERE username="test user") LIKE "3[START]a[END]%" ,SLEEP(10),'a'), "a", "b", "c" --
```
where [START] and [END] are the starting and ending points to be highlighted.

m. Start the second round of attack to find the second character in the password string.

n. Repeat this process until no characters are found to have caused a delay in response time. This means that it is the end of the string, or the string includes a special character.

## References
https://app.stackup.dev/

