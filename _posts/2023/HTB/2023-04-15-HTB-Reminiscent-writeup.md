---
title: HTB - Reminiscent challenge writeup
author: tuedenn
date: 2023-04-15 23:20:00 +0700
categories: [Blogging, HTB]
tags: [HTB, Volatility, DFIR, writeup]
---

# Reminiscent Challenge

Như đã đề cập [bài viết trước](../About-HTB-writeup-series/), HTB hiện đang có 13 challenge thuộc category Forensic.

![HTB-forensic-challenge.png](/assets/img/2023/HTB/Reminiscent/HTB-forensic-challenge.png)

**Reminiscent** là challenge đầu tiên trong list này, được rate 4,9*, 30 points và có khoảng 9k người đã giải được. Khi click vào sẽ hiển thị như sau:

![Reminiscent-Challenge.png](/assets/img/2023/HTB/Reminiscent/challenges.png)

- Mô tả: `Suspicious traffic was detected from a recruiter's virtual PC. A memory dump of the offending VM was captured before it was removed from the network for imaging and analysis. Our recruiter mentioned he received an email from someone regarding their resume. A copy of the email was recovered and is provided for reference. Find and decode the source of the malware to find the flag.`

**Dịch nôm na ra là**: Hệ thống ghi nhận 1 máy tính (VM) của nhân viên có traffic bất thường, và đã kịp thời dump mem của VM để phân tích trước khi hành vi đó bị xoá bỏ. Qua quá trình xác minh, nhân viên trả lời rằng anh ta nhận được 1 email giả dạng 1 ứng viên gửi CV. Nhiệm vụ của chúng ta là điều tra nguồn gốc của mã độc để tìm flag.

Tiến hành download và unzip file. Thu được 3 file như sau:

```bash
reminiscent $ file *            
    flounder-pc-memdump.elf: ELF 64-bit LSB core file, x86-64, version 1 (SYSV)
    imageinfo.txt:           ASCII text
    Resume.eml:              SMTP mail, ASCII text, with CRLF line terminators
```
Trong đó:
- `flounder-pc-memdump.elf`: file memdump từ máy VM
- `imageinfo.txt`: info của file dump (output từ volatility2)
- `Resume.eml`: smtp email header

Nội dung file `imageinfo.txt`:
![imageinfo.png](/assets/img/2023/HTB/Reminiscent/imageinfo.png)

Nội dung file `Resume.eml`:
![resume-email.png](/assets/img/2023/HTB/Reminiscent/resume-email.png)

## Key result 1

- Nhân sự tên là Frank (Lounder?) - `flounder@madlab.lcl` nhận được 1 email từ `bloodworm@madlab.lcl`
- Email lừa Frank click vào file `Resume.zip` được host ở địa chỉ: `http://10.10.99.55:8080/resume.zip`
- Máy Frank là Windows (Win7 sp1 64bit, maybe)

> **Nhận định cá nhân**: Chà, đây quả thật là 1 scenario nóng bỏng, được đa số các nhóm APT theo đuổi. Kỹ thuật này gọi là `Spearphishing Link`. Bạn có thể tìm hiểu kỹ hơn về technique procedure tại [đây](https://attack.mitre.org/techniques/T1566/002/).
{: .prompt-tip }

# WHATs NEXT?

Theo bạn, tiếp theo chúng ta cần làm gì?

Ở đây có 2 hướng tiếp cận mà tôi đã thực hiện (nếu bạn có cách làm khác có thể đóng góp), tôi sẽ đi lần lượt từng cách giải

## Cách 1 - follow the description

### Find the file

Sau khi đã biết được người dùng bị lừa tải xuống file có tên `Resume.zip`, cần xác định file info: đã được tải xuống chưa, tải xuống ở đâu, lúc nào, ... Sử dụng các plugin liên quan đến file của Volatility. (Vì như đã đề cập ở trên, file `imageinfo.txt` là info của file dump (output từ volatility2) -> tôi sử dụng Volatility, cụ thể là [Volatility3](https://github.com/volatilityfoundation/volatility3), bạn có thể thử sử dụng các công cụ khác)
![file-plugins.png](/assets/img/2023/HTB/Reminiscent/file-plugins.png)

Có thể thấy plugin `windows.filescan.FileScan` có thể sử dụng để scan các object file. Vì đã biết file là `resume.zip` nên grep theo pattern `resume`
![file-resume.png](/assets/img/2023/HTB/Reminiscent/file-resume.png)

Các cột lần lượt là `Offset`, `Name`, `Size`. (Bạn bỏ qua đoạn `100.0` vì đấy là cache terminal khi grep). 
Như trong [bài viết trước](../Velo-artifact-tour/) tôi có đề cập đến `Artifact` là gì: 
> Vì là “hiện vật”, nên không phải lúc nào artifact cũng tồn tại, hoặc **tồn tại nhưng không còn nguyên vẹn**. Thách thức của người làm điều tra số là làm sao có thể vẽ lại được bức tranh toàn cảnh về cuộc tấn công (càng sáng tỏ càng tốt) từ artifact thu thập được
{: .prompt-info }

### Dump the file

Ở trường hợp này, artifact chỉ còn là file shortcut (`.lnk`). Nhưng không sao, việc khó có người làm điều tra số lo. Sau khi biết được offet, tiến hành sử dụng `windows.dumpfiles.DumpFiles` để dump file `resume.pdf.lnk` (shortcut) về. Kết quả thu được 1 file `.dat` như hình dưới đây:
![dump-file.png](/assets/img/2023/HTB/Reminiscent/dump-file.png)

Tiến hành `strings` để tìm các keyword *có thể* liên quan. 
![string-dump-file.png](/assets/img/2023/HTB/Reminiscent/string-dump-file.png)

Nhận thấy 1 số keywords liên quan đến `windows`, `system32`, `powershell`, `base64 encoded string`. 

> Nhận định có thể mã độc nhúng vba object trong file `resume.pdf`. Khi chạy sẽ khởi tạo tiến trình powershell để thực thi câu lệnh **liên quan tới** đoạn string base64 encoded này.
{: .prompt-tip }

### Find the malicious

Tiến hành đem đoạn string thu được đi decode base64, có thể sử dụng cmd base64 trên linux hoặc cyberchef. Tôi thích dùng [CyberChef](https://gchq.github.io/CyberChef/) hơn!
![file-decode-base64-pwsh1.png](/assets/img/2023/HTB/Reminiscent/file-decode-base64-pwsh1.png)

Bức tranh bắt đầu rõ ràng hơn. Powershell spawn powershell. Tiếp tục decode, thu được kết quả sau:

![file-decode-base64-pwsh2.png](/assets/img/2023/HTB/Reminiscent/file-decode-base64-pwsh2.png)

Bạn thu được flag, có thể coi đó là xong, nếu bạn nghĩ đơn giản là đố, giải đố.

Nhưng nếu bạn nghĩ xa hơn thì sao? Case này có ứng dụng gì trong thực tế? 

Nếu bạn nghĩ được như thế thì rất tuyệt vời, mindset bạn rất tốt. Trong trường hợp tôi đang là người phỏng vấn bạn, bạn đã được 1 điểm cộng lớn!

> Đây là 1 scenario có thể gọi là khá kinh điển trong thực tế, tôi tạm dừng không phân tích chi tiết đoạn code powershell kia thực hiện hành vi gì. Thay vào đó, tôi coi đấy là 1 thử thách cho bạn tự phân tích. Tôi khuyên bạn nên tìm hiểu về powershell, và đào sâu phân tích xem cụ thể đoạn code này thực hiện những hành vi gì. Sau đó xâu chuỗi lại, vẽ ra 1 bức tranh về cuộc tấn công, gọi là attack map. Như thế chắc chắn bạn sẽ học được rất nhiều.
{: .prompt-tip }


Giờ cũng đã muộn, tôi nghĩ cách giải thứ 2 tôi sẽ viết vào 1 bài khác, rồi publish sau. Nếu có thời gian, có thể tôi sẽ vẽ attack map, rồi chúng ta cùng so sánh. Hẹn gặp lại trong bài viết sau. Chào thân ái!