---
title: HTB - Obscure challenge writeup
author: tuedenn
date: 2023-04-30 21:14:02 +0700
categories: [Blogging, HTB]
tags: [HTB, pcap, webshell, DFIR, writeup]
---

## Description
An attacker has found a vulnerability in our web server that allows arbitrary PHP file upload in our Apache server. Suchlike, the hacker has uploaded a what seems to be like an obfuscated shell (support.php). We monitor our network 24/7 and generate logs from tcpdump (we provided the log file for the period of two minutes before we terminated the HTTP service for investigation), however, we need your help in analyzing and identifying commands the attacker wrote to understand what was compromised.

## Analysis web shell support.php
Sử dụng php online [onlinegdb](https://www.onlinegdb.com/online_php_interpreter)  để phân tích và run file `support.php` (do lười setup môi trường, bạn có thể dùng phpstorm hoặc vscode with debugger)

![deobfuscate-1.png](/assets/img/2023/HTB/Obscure/deobfuscate-1.png)
Nhận thấy, dòng 6 ->  `$N=str_replace('FD','','FDcreFDateFD_fFDuncFDFDtion');` đơn giản là thay thế chuỗi `'FD'` từ `'FDcreFDateFD_fFDuncFDFDtion'` -> kết quả sẽ thu được chuỗi `create_function`  (biến `$N`)

Tiếp đó đến dòng 10, `$x=$N('',$u);$x();` -> gọi `$N` để tạo function với code là `$u` sau đó thực thi function này (xem thêm về hàm `create_function` ở [Link](https://www.php.net/manual/en/function.create-function.php) 

Thực hiện `var_dump($u)` để xem giá trị được store tại biến `$u`
Thu được kết quả:

![deobfuscate-2.png](/assets/img/2023/HTB/Obscure/deobfuscate-2.png)

Sau khi Beautify:

![deobfuscate-3.png](/assets/img/2023/HTB/Obscure/deobfuscate-3.png)


Ở đây chú ý đến 2 block code là `function x($t, $k)` từ dòng 6 đến dòng 16; và block `if(@preg_match...)` từ dòng 17 đến dòng 24

### Block If
- sử dụng hàm `preg-match` làm điều kiện kiểm tra -> chỉ tiếp tục thực hiện nếu thoả mãn điều kiện
	- xem thêm về hàm preg-match ở [Link](https://www.php.net/manual/en/function.preg-match)
- input để kiểm tra là `@file_get_contents("php://input")` 
	- xem thêm ở [Link](https://stackoverflow.com/questions/2731297/file-get-contentsphp-input-or-http-raw-post-data-which-one-is-better-to) 
	- Hiểu đơn giản, input để kiểm tra chính là dữ liệu thô trả về từ webshell (raw response data) 
- Tiếp đó, search nội dung có khớp với pattern `/$kh(.+)$kf/` -> `/6f8af44abea0(.+)351039f4a7b5/` hay không
- Kết quả trả về (`matches`) được lưu vào array `$m`
- Khi matching, sử dụng các hàm [`ob-start()`](https://www.php.net/manual/en/function.ob-start), [`ob-get-contents()`](https://www.php.net/manual/en/function.ob-get-contents.php), [`ob-end-clean`](https://www.php.net/manual/en/function.ob-end-clean.php) , [`gzuncompress`](https://www.php.net/manual/en/function.gzuncompress), [`base64`](https://www.php.net/manual/en/function.base64-decode), ... để xử lý chuỗi kết quả
- Output được print ra ở định dạng `0UlYyJHG87EJqEz66f8af44abea0 <$r> 351039f4a7b5` trong đó `$r` là output của dòng 22 (call function X)

Vậy câu hỏi ở đây là gì? Ta cần biết được input (tức `@file_get_contents("php://input")`) để kiểm tra, decrypt ngược lại

## Analysis file 19-05-21_22532255.pcap 

Follow TCP stream được kết quả sau (stream 1)

![follow-http-stream1.png](/assets/img/2023/HTB/Obscure/follow-http-stream1.png)

Nhận thấy Input là: `6f8af44abea0QKwu/Xr7GuFo50p4HuAZHBfnqhv7/+ccFfisfH4bYOSMRi0eGPgZuRd6SPsdGP//c+dVM7gnYSWvlINZmlWQGyDpzCowpzczRely/Q351039f4a7b5`
output là: `0UlYyJHG87EJqEz66f8af44abea0QKxO/n6DAwXuGEoc5X9/H3HkMXv1Ih75Fx1NdSPRNDPUmHTy351039f4a7b5`  
Verify lại bằng test input

![test-output-msg.png](/assets/img/2023/HTB/Obscure/test-output-msg.png)

Trông có vẻ output không đúng lắm -> **tôi nghĩ** nguyên nhân là do các hàm `ob_` xử lý buffer từ `php:input` (post data), nên khi test với input này có thể `@ob_get_contents()` không sinh ra kết quả giống như hình trên.
Vì thế, cần tiến hành Decrypt lại đoạn code từ Funtion X

### Decrypt

Funtion X bản chất là 1 phép xor đơn giản -> đem đi xor ngược lại là xong.
Kiểm tra bằng cách testing với input từ Stream 1

![decrypt-output.png](/assets/img/2023/HTB/Obscure/decrypt-output.png)

Xâu chuỗi lại ta được kết quả bản rõ của Stream 1 như sau:

- Input: `chdir('/var/www/html/uploads');@error_reporting(0);@system('id 2>&1');`
- Output: `uid=33(www-data) gid=33(www-data) groups=33(www-data)`

### Final

stream1  -> get UID via `id` command

![stream1.png](/assets/img/2023/HTB/Obscure/stream1.png)

Stream 23  -> Discovery `/home/` folder using `ls -lah /home/**`

![stream23.png](/assets/img/2023/HTB/Obscure/stream23.png)

stream 24  -> change working directory to `/home/developer`

![stream24.png](/assets/img/2023/HTB/Obscure/stream24.png)

stream 25 -> `base64` file `/home/developer/pwdb.kdbx` to `stdout`

![stream25.png](/assets/img/2023/HTB/Obscure/stream25.png)

Đến đây, ta đã có thể hiểu được timeline của attacker, từ lúc upload shell thành công cho đến lúc dump thông tin file `pwdb.kdbx` (keepass local database)

Tiến hành decode base64 đoạn output trên, rồi save lại dưới tên `pwdb.kdbx`.
```
❯ file pwdb.kdbx
pwdb.kdbx: Keepass password database 2.x KDBX
```

# Analysis file pwdb.kdbx 

Muốn mở file kdbx, cần có keepass. Tải xuống file keepass portable ở `https://sourceforge.net/projects/keepass/files/KeePass%202.x/2.54/KeePass-2.54.zip/download`

Khó khăn ở đây là cần phải có password mới có thể xem nội dung file này.
Follow link: https://www.thedutchhacker.com/how-to-crack-a-keepass-database-file/ để crack password với hy vọng là có thể thành công

## Keepass2john
Sử dụng `keepass2john` để lấy hash pass của file keepass

![keepass2john.png](/assets/img/2023/HTB/Obscure/keepass2john.png)

## Hashcat
Sau khi đã có hash, cần crack để tìm ra password dạng plain text.  Sử dụng công cụ `hashcat`

![cracked-passwd.png](/assets/img/2023/HTB/Obscure/cracked-passwd.png)

## Result
Sử dụng password sau khi crack để mở file, thấy có 1 entry là `Passwords` có title `Flag` -> Nội dung chính là flag cần tìm

![got the flag.png](/assets/img/2023/HTB/Obscure/got the flag.png)

Nhập flag và get points!

![pwned.png](/assets/img/2023/HTB/Obscure/pwned.png)

## Tổng kết
Thử thách mô tả lại hành vi của attacker sau khi khai thác lỗ hổng web, tiến hành upload shell, từ đó thu thập các thông tin nhạy cảm của web server. Đây có thể coi là 1 case rất hay gặp trong nghề Điều tra số.
Nhưng bạn hãy thử đặt câu hỏi, nếu không có captured pcap, liệu chúng ta có thể biết được attacker đã làm gì hay không? Câu trả lời là có, nếu bạn config logging full access log (include response data!?)