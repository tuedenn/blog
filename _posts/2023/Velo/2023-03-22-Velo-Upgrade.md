---
title: Velociraptor upgrades
author: tuedenn
date: 2023-03-22 19:59:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

Trong bài viết trước, tôi đã trình bày về cách deploy server với phiên bản v0.6.7-4 và có viết 1 câu là:

> Tại thời điểm viết bài, đã có phiên bản v0.6.8 pre-release nhưng chưa stable, nên tôi vẫn sử dụng phiên bản v0.6.7-4

Nhưng hôm qua (21/03/2023), Mike đã release version v0.6.8-1 (https://github.com/Velocidex/velociraptor/commit/219a1c4facf4f7d1ed15260a89a3f30d98bf5e8f)

> Version 0.6.8 is now LIVE!
A big theme in this release was about performance improvement, making Velociraptor faster, more efficient and more scalable -- even more so than it currently is!
Learn about the new features and download v0.6.8 today.
https://github.com/Velocidex/velociraptor/releases/tag/v0.6.8-1


![v0.6.8-is-live.jpg](/assets/img/2023/Velo/05/v0.6.8-is-live.jpg)

Nhiều người sẽ có cùng câu hỏi đặt ra là: "Giờ tôi có nâng cấp version lên để fix bug, cải tiến hiệu năng,... được không? Nâng cấp lên có ảnh hưởng gì không? ..."

Tôi sẽ cố gắng giải đáp thắc mắc của mọi người trong bài viết này.

Velo cho phép người dùng **nhanh chóng** và **dễ dàng** nâng cấp cả server và client.

## Nâng cấp server

Download file `velociraptor-v0.6.8-windows-amd64.exe` và `velociraptor-v0.6.8-linux-amd64` ở đường dẫn https://github.com/Velocidex/velociraptor/releases/tag/v0.6.8-1

Khởi chạy command 
```
velociraptor-v0.6.8-windows-amd64.exe --config server.config.yaml debian server --binary velociraptor-v0.6.8-linux-amd64
```

Trong đó `server.config.yaml` là đường dẫn tới file config **cũ** <- lưu ý là file config phải giống nhau, tránh tình trạng reconfig lại dẫn đến xung đột. 

Kết quả thu được `Creating a package for velociraptor_0.6.8_server.deb` là thành công

![repack-server.png](/assets/img/2023/Velo/05/repack-server.png)

> Những dòng lỗi show trong ảnh trên bắt nguồn từ việc Velo xử lý logging. Bạn không cần quan tâm đến nó. Tôi đã trao đổi và góp ý với Mike về việc này. Anh ấy phản hồi rằng `i think for some commands it should disable logging completely. thanks for reporting it though! very useful`. Có thể Velo sẽ cải tiến trong các phiên bản tiếp theo :D
{: .prompt-info }

Tiến hành copy file `velociraptor_0.6.8_server.deb` lên server và khởi chạy

![installed-server.png](/assets/img/2023/Velo/05/installed-server.png)

Việc nâng cấp không ảnh hưởng tới dịch vụ

## Nâng cấp client

Cách đơn giản là sử dụng `Admin.Client.Upgrade` artifacts trên GUI server.


![remote_upgrade_msi.png](/assets/img/2023/Velo/05/remote_upgrade_msi.png)
![remote_upgrade_msi_2.png](/assets/img/2023/Velo/05/remote_upgrade_msi_2.png)

Lưu ý: Khi sử dụng cách này, artifacts có thể bị lỗi (do tiến trình gốc `nhận lệnh` đã bị thay thế) -> điều này là bình thường và không cần quan tâm đến nó

Để xác minh rằng client đã được nâng cấp, sử dụng `Generic.Client.Info` để thu thập thông tin client (Nên định kỳ khởi chạy artifact này)


# Tổng kết

Velo cho phép người dùng **nhanh chóng** và **dễ dàng** nâng cấp cả server và client. Nếu có bất kỳ thắc mắc trong quá trình nâng cấp, các bạn có thể trao đổi trực tiếp với tôi để được hỗ trợ