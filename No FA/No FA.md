<img width="909" height="575" alt="image" src="https://github.com/user-attachments/assets/4a9a5960-36d8-4be7-929f-496c9d6717be" />

\- Download two given files to see what's inside: [app.py](https://challenge-files.picoctf.net/c_foggy_cliff/9bf05fa33a4486d06b548daf776842ccbcf9914c5f65b32e7d7e8a36e3b64d32/app.py); [users.db](https://challenge-files.picoctf.net/c_foggy_cliff/9bf05fa33a4486d06b548daf776842ccbcf9914c5f65b32e7d7e8a36e3b64d32/users.db)

\- Access the website, we see the login form with no credential found in the page source

\- I turn into the leak data from `users.db`, I notice the password of `admin`, and from function `login()`, I found that the password is SHA256 hash (one-way). Then trying to [crack](https://crackstation.net/) that password:

<img width="1525" height="147" alt="image" src="https://github.com/user-attachments/assets/87185a0d-acef-419f-8c9d-37bba2a72999" />

\- Loging in with the cracked credential, we redirect to the `/two_fa`

\- Inspect the `app.py`, we found that the otp is generated randomly right after we loged in and is stored in the session cookie:
```
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = db.get_user_by_username(username)
        if user and hashlib.sha256(password.encode()).hexdigest() == user['password']:
            if user['two_fa']:
                # Generate OTP
                otp = str(random.randint(1000, 9999))
                session['otp_secret'] = otp
                session['otp_timestamp'] = time.time()
                session['username'] = username
                session['logged'] = 'false'
                # send OTP to mail ---
                return redirect(url_for('two_fa'))
            else:
                session['username'] = username
                session['logged'] = 'true'
                flash('Login successful!', 'green')
                return redirect(url_for('home'))
        else:
            flash('Invalid username or password', 'red')
    return render_template('login.html')
```
\- From the beginning, the application uses Flask's built-in `session` object, and stores entire session data on the client-side. And from `app.py`, sensitive data are directly assigned to the session

\- Use [online decoder](https://www.kirsle.net/wizards/flask-session.cgi), and send that OTP to login
<img width="1893" height="254" alt="image" src="https://github.com/user-attachments/assets/e95ff9be-a099-4a98-902d-ecfd0221b87a" />
<img width="692" height="222" alt="image" src="https://github.com/user-attachments/assets/5434e014-7c66-496a-a47e-e0877a26f0c6" />

> The Flask session cookie is cryptographically signed (to prevent tampering) but strictly not encrypted (to hide data). Because it is not encrypted, we can simply decode the Base64/zlib payload to extract the otp_secret in plaintext without needing the server's SECRET_KEY. However, because it is signed, we cannot blindly modify session['logged'] = 'true' to bypass the 2FA, as doing so would invalidate the signature. We are forced to read the OTP and legitimately submit it.



