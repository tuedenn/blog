---
title: Velociraptor Deployment Server
author: tuedenn
date: 2023-03-18 22:39:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

Mô hình triển khai Velo thông thường được mô tả như hình dưới:

![deploy-overview.png](/assets/img/2023/Velo/04/deploy-overview.png)

Trình bày ngắn gọn: Velo server và agent kết nối với nhau thông qua kết nối liên tục dạng C&C. Quản trị viên sẽ thực hiện truy cập vào Velo server (để hunting/forensic) thông qua 1 webbase quản trị (GUI).

Tuỳ vào mục đích của bạn là gì mà có thể chọn phương thức triển khai server khác nhau:
- SSL-Self Signed: Phù hợp cho nhu cầu triển khai on-prime server, cần máy chủ vật lý/ảo hoá
- Cloud Deployment: Phù hợp cho nhu cầu triển khai on-cloud server, sử dụng server thuê của các nhà cung cấp như GCP, AWS, Azure,..
- Instant deployment: Phù hợp cho nhu cầu triển khai local phục vụ mục đích testing

Bài viết này tập trung vào cách triển khai (deploy) Velociraptor server theo hình thức **SSL-Self Signed** (nếu các bạn có nhu cầu triển khai theo cách khác, có thể tham khảo hướng dẫn trên trang chủ)

## Self-cert

> Self-cert -> self-signed certificates
{: .prompt-info }

Velo client và server được xác thực với nhau thông qua self-signed Certificate Authority (CA). CA này được tạo ra trong khi khởi tạo cấu hình. 

Velo không chấp nhận các self-cert khác, vì thế chúng ta không nên thử tạo mới rồi upload self-cert lên velo (config)

## Chuẩn bị

Đầu tiên, bạn cần chuẩn bị 1 số file sau (Tiến hành tải xuống ở đường dẫn: https://github.com/Velocidex/velociraptor/releases/tag/v0.6.7-5) Lựa chọn phiên bản tuỳ thuộc vào nhu cầu và hệ điều hành mà bạn đang sử dụng. 
- `velociraptor-v0.6.7-4-windows-amd64.exe` để generate file config & server package
- `velociraptor-v0.6.7-4-linux-amd64` để tạo server package (apply server config)
- `velociraptor-v0.6.7-4-windows-amd64.msi` để cài đặt agent service

> Tại thời điểm viết bài, đã có phiên bản v0.6.8 pre-release nhưng chưa stable, nên tôi vẫn sử dụng phiên bản v0.6.7-4
{: .prompt-info }

## Khởi tạo file cấu hình

Trong tài liệu hướng dẫn, Velo cho phép 2 cách để khởi tạo file config. Bài viết này sẽ trình bày cách sử dụng `configuration wizard` để generate config file.

Sau khi đã có các file yêu cầu, tiến hành generate config với câu lệnh:
```
velociraptor-v0.6.7-4-windows-amd64.exe config generate -i
```

![gen-Self-Signed-Deployment.png](/assets/img/2023/Velo/04/gen-Self-Signed-Deployment.png)

*Generating Self Signed Deployment*

`configuration wizard` sẽ yêu cầu bạn cung cấp 1 số thông tin cần thiết như sau:

1. What OS will the server be deployed on?
   
   Tuỳ thuộc vào hệ điều hành mà bạn sử dụng để deploy Velociraptor server. Sử dụng mũi tên lên/xuống để lựa chọn.
   Tài liệu khuyến nghị bạn nên deploy trên debian/ubuntu server.
   Ở đây tôi lựa chọn linux

2. Path to the datastore directory
   
   Tuỳ thuộc vào nhu cầu bạn muốn lưu trữ dữ liệu ở đâu. Có thể là đường dẫn local trên server hoặc đường dẫn network share.
   Ở đây tôi lựa chọn lưu trữ tại `/opt/velociraptor`

3. The public DNS name of the Frontend
    
    Tuỳ thuộc nhu cầu của bạn & cách thức deploy. 
    Vì sử dụng Self-cert nên tại bước này, tôi điền địa chỉ IP server của mình.

4. The frontend port to listen on
    
    Chỉ định 1 port để giao tiếp giữa client và server. Server (front end) sẽ listen ở port này để lắng nghe client kết nối đến. Mặc định 8000
    -> Khuyến nghị bạn nên mở inbound access from anywhere (0.0.0.0)

5. The port for the Admin GUI to listen on
    
    Chỉ định 1 port để truy cập vào portal quản trị (admin GUI). Mặc định là 8889
    -> khuyến nghị sử dụng tunnel/port forward 

6. GUI Username or email address to authorize

    Tạo username và password để đăng nhập vào admin GUI.
    Tài khoản được tạo sẽ có quyền quản trị viên (administrators) và được lưu password hash vào datastore (vì sử dụng self-cert SSL nên phương thức xác thực vẫn là basic authen)

7. Where should I write the server/client config file?
    
    Tuỳ chỉnh đường dẫn để lưu file config cho server và client. 
    Mặc định lưu ở thư mục hiện tại.

Kết quả như hình dưới dây:

![gen-Self-Signed-Deployment-full.png](/assets/img/2023/Velo/04/gen-Self-Signed-Deployment-full.png)

Kiểm tra 2 file cấu hình trong thư mục hiện tại:

- Client config
  
![Client-config.png](/assets/img/2023/Velo/04/Client-config.png)

client kết nối đến port 8000 phía server, xác thực thông qua CA.
Trong file config cũng xác định 1 số thông tin như `service_name`, `install path`, `service_description`,...

- Server config
  
![Server-config-1.png](/assets/img/2023/Velo/04/Server-config-1.png)
![Server-config-2.png](/assets/img/2023/Velo/04/Server-config-2.png)

Tương tự như client, nhưng ở đây chúng ta chú ý vào các trường `bind_address` và `bind_port`
Server mở các cổng:
- 0.0.0.0:8000 -> front end (có thể gây hiểu lầm, đại ý là port giao tiếp chính giữa server-client) 
- 127.0.0.1:8889 -> web-base admin GUI
- 127.0.0.1:8001 -> API service (ví dụ gRPC API clients)
- 127.0.0.1:8003 -> monitor service (ví dụ như Grafana/Prometheus hay Data Dog)

Ở đây chúng ta chỉ cần chú ý đến các port 8000 và 8889.
Ngoài ra, nếu muốn tìm hiểu chi tiết về các trường trong file config, có thể tham khảo thêm ở link: https://docs.velociraptor.app/docs/deployment/references/

> Đặc biệt lưu ý, trong file config có chứa thông tin về private key, nếu bị lộ ra ngoài có thể bị khai thác tấn công Man-in-the-middle. Vì thế cần đảm bảo sự bí mật của file config nói chung hay private key nói riêng
{: .prompt-danger }


## Khởi tạo server package

Hiểu đơn giản, đây là bước áp dụng config vừa tạo cho file thực thi trên server, kết quả sẽ sinh ra 1 Debian package. File này sẽ được delivery lên server và cài đặt.

Sử dụng câu lệnh sau để khởi tạo debian package

```
velociraptor-v0.6.7-4-windows-amd64.exe --config server.config.yaml debian server --binary velociraptor-v0.6.7-4-linux-amd64
```

- `server.config.yaml` là file server config vừa tạo
- `velociraptor-v0.6.7-4-linux-amd64` là file thực thi trên linux (đã download từ bước **Chuẩn bị**)

![create-debian-package.png](/assets/img/2023/Velo/04/create-debian-package.png)

kết quả sẽ tạo ra file `velociraptor_0.6.7-4_server.deb`. Tiến hành upload lên server và install

![upload-deb-package.png](/assets/img/2023/Velo/04/upload-deb-package.png)

Chạy câu lệnh `sudo dpkg -i velociraptor_0.6.7-4_server.deb` để install package là hoàn tất quá trình. Có thể truy cập admin GUI để kiểm tra kết quả

![succcess-setup.png](/assets/img/2023/Velo/04/succcess-setup.png)