---
title: Velociraptor Deployment Client
author: tuedenn
date: 2023-03-22 23:39:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

Velo là 1 công cụ mạnh mẽ, linh hoạt. Nó được viết bằng golang nên có thể hoạt động trên đa nền tảng từ Windows, Linux đến MacOS,...

Bài viết này sẽ tập trung hướng dẫn bạn cách deploy agent trên nền tảng Windows thông qua GPO.

## Chuẩn bị

Download file `velociraptor-v0.6.8-windows-386.msi` từ đường dẫn https://github.com/Velocidex/velociraptor/releases/tag/v0.6.8-1
Lưu cùng thư mục với file `client.config.yaml` đã tạo trước đó (xem lại bài viết **Velociraptor Deployment Server**)

![require](/assets/img/2023/Velo/06/require.png)

### Tại sao lại là MSI

Velo hỗ trợ chạy trực tiếp từ file .exe thông qua câu lệnh

```
velociraptor --config client.config.yaml client -v
```
Nhưng sẽ có khó khăn trong việc deploy diện rộng. Bạn hoàn toàn có thể (*Và tôi khuyên bạn nên*) sử dụng phương pháp này **khi chạy trên môi trường TEST local**. 
Nếu bạn muốn triển khai diện rộng, hoặc tìm hiểu cách triển khai trong môi trường doanh nghiệp (enterprise), tôi khuyên bạn sử dụng MSI. 

MSI là 1 gói cài đặt Windows tiêu chuẩn (standard Windows installer package). Hầu hết các công cụ quản trị trong môi trường doanh nghiệp (enterprise) đều triển khai phần mềm thông qua `.msi`

> Nếu bạn vẫn một mực không thích sử dụng file msi ( XD ), tôi vẫn có thể chiều bạn. Hãy nhắn tin riêng cho tôi nếu gặp khó khăn trong quá trình triển khai với file `.exe`
{: .prompt-tip }

### Repack MSI

Tôi có trao đổi với Mike và nhận được phản hồi rằng:
> since 0.6.8 the config repack command is just a wrapper around some VQL code. And VQL does not use the current directory usually

Từ bản v0.6.8, việc repack đơn thuần là thực hiện 1 đoạn mã VQL, vì thế, việc truyền thông tin đầu vào là rất quan trọng.

Sử dụng câu lệnh sau:

```sql
velociraptor-v0.6.8-windows-amd64.exe config repack --exe path/to/velociraptor-v0.6.8-windows-amd64.msi client.config.yaml velociraptor-v0.6.8-repack.msi
```

**Lưu ý:**

- Sử dụng `--exe` thay vì `--msi` như trên tài liệu trang chủ (vì --msi không còn sử dụng, tôi đã trao đổi lại với Mike)
- Sử dụng fullpath thay vì path ("VQL does not use the current directory usually")

Kết quả:

![repack-msi.png](/assets/img/2023/Velo/06/repack-msi.png)

Sau khi repack, thu được file `velociraptor-v0.6.8-repack.msi`

### Deploy

Có nhiều hình thức để triển khai 1 package .msi như SSCM, GPO. 
Tham khảo thêm: https://learn.microsoft.com/en-US/troubleshoot/windows-server/group-policy/use-group-policy-to-install-software
Tôi sẽ hướng dẫn sử dụng GPO để deploy .msi, cụ thể như sau:


1. Khởi tạo thư mục network share
    1. Khởi tạo thư mục network share → setup quyền **readonly**
    2. copy file `velociraptor-v0.6.8-repack.msi` lên đó
    
    Ví dụ: khởi tạo thư mục `\\10.xx.xx.120\tool-soft\Velociraptor` , set Everyone can Read only
    
    ![create-folder.png](/assets/img/2023/Velo/06/create-folder.png)
    
2. **Khởi tạo Group Policy object (GPO)**
    
    Lựa chọn OU cần deploy, tạo mới GPO sau đó tiến hành edit
    
    ![create-GPO.png](/assets/img/2023/Velo/06/create-GPO.png)
    
    ![create-GPO-2.png](/assets/img/2023/Velo/06/create-GPO-2.png)
    
3. **Khởi tạo Scheduled Task**
    1. Create `Immediate task`
        - Lập tức khởi chạy ngay khi máy tính nhận/update GPO mới (có thể vài phút, hoặc vài tiếng)
        - Nếu chọn `Scheduled Task` sẽ phải chờ đến lần reboot tiếp theo (có thể vài ngày, hoặc không bao giờ :D)
        
        ![create-scheduled-task.png](/assets/img/2023/Velo/06/create-scheduled-task.png)
        
        *immediate task*
        
    2. Kết quả:
        
        ![edit-scheduled-task](/assets/img/2023/Velo/06/edit-scheduled-task.png)
        
4. **Cấu hình Scheduled task vừa tạo**
    1. General
        
        Khởi tạo Tên, mô tả, quyền thực thi và một số tuỳ chọn khác như `Run whether logged on or not`, `Hidden`
        
        ![edit-scheduled-task-1.png](/assets/img/2023/Velo/06/edit-scheduled-task-1.png)
        
    2. Actions
        
        Chỉ định thực thi `C:\windows\system32\msiexec.exe` để cài đặt Velo dưới dạng service bằng cách truyền đối số (arguments) `/i \\path\to\Velociraptor\velociraptor-v0.6.8-repack.msi`
        
        ![edit-scheduled-task-2.png](/assets/img/2023/Velo/06/edit-scheduled-task-2.png)
        
        *Using msiexec to install as a service*
        
    3. Settings
        
        Mục đích để kiểm soát thời gian
        
        → `Stop the task if it runs longer than 3days`: dừng task sau 3 ngày
        
        ![edit-scheduled-task-3.png](/assets/img/2023/Velo/06/edit-scheduled-task-3.png)
        

Ngay khi update GPO, Velo sẽ chạy trên máy client dưới dạng 1 service tên là `Velociraptor` và kết nối đến server (*thông qua port 8000)*.

Có thể vào admin GUI để kiểm tra:

![result-client](/assets/img/2023/Velo/06/result-client.png)

**List all file & thư mục trong C drive (deep = 10) tốn 374s**


<br>
<br>
<hr>

<br>
Bài viết đã hướng dẫn bạn deploy .msi package thông qua GPO, hy vọng bạn có thể tự triển khai một cách dễ dàng, không chỉ với Velo mà bạn hoàn toàn có thể áp dụng để triển khai các công cụ, phần mềm khác.

<br>

<hr>
> Thú thật là tôi đang viết dở bài viết này thì Velo release version 0.6.8-1, nên tôi phải chỉnh sửa lại nội dung 1 chút, nhưng không sao. Version 0.6.8-1 có nhiều cải tiến mạnh mẽ, bạn có thể tham khảo thêm ở link: https://docs.velociraptor.app/blog/2023/2023-02-13-release-notes-0.6.8/
