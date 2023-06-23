---
title: DFIR approach in Velociraptor
author: tuedenn
date: 2023-03-08 21:39:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

# Khác biệt giữa cách tiệp cận của Velo với DFIR truyền thống

Bài viết so sánh cách tiếp cận của Velociraptor (Velo) với cách tiếp cận DFIR truyền thống. Velo chủ yếu phân tích ở phía Client, giúp giảm tải cho server và tăng tốc độ xử lý. Ngoài ra, Velo tập trung vào các dữ liệu có giá trị cho việc DFIR cụ thể, có tính năng collect cho phép thu thập dữ liệu có chủ đích rồi upload lên server để phục vụ mục đích điều tra. Velo cũng có tính năng tích hợp các parser framework như Grok và Sqlite. VQL của Velo được thiết kế để tự động hóa nhiều công việc DFIR thường gặp.

# Traditional approach

**Quá trình điều tra số thông thường:**

1. Trích xuất thông tin
    1. thu thập raw data từ endpoint
        1. MFT, Evtx,…
        2. các data có giá trị phục vụ cho việc Forensics
    2. nén
        1. tar, zip, vhdx,…
        2. giảm kích thước
2. Upload/copy đến server phân tích
    1. cloud upload
    2. external drive → copy
3. Phân tích data trên server (timesketch, elastic, opensearch, etc.)
    1. standalone tools
    2. timeline
    3. etc

# Velociraptor approach

1. **parse** và **analyze data** ở trên chính máy endpoint
    
    ![endpoint-log-message.png](/assets/img/2023/Velo/02/endpoint-log-message.png)
    
    *As the query is running on the endpoint any log messages are sent to the server. Click the log tab to see if there were any errors and how many rows are expected.*
    
2. collect **có chủ đích** các artifact
    
    ![Artifact-produce-rows.png](/assets/img/2023/Velo/02/Artifact-produce-rows.png)
    
    *All artifacts produce rows since they are just queries. Some artifacts also upload files. You can create a download zip to export all the uploaded files.*
    
3. có thể **link/scale** các collection
    
    ![link-scale-collection.png](/assets/img/2023/Velo/02/link-scale-collection.png)
    
4. vì **sử dụng ngôn ngữ query** nên **dễ dàng, linh hoạt** trong việc tạo các analysis khác nhau
    1. VQL 
    2. .yaml file với các trường metadata xung quanh truy vấn
    3. dễ dàng chia sẻ trong cộng đồng
    4. dễ dàng hiểu mà thậm chí k cần biết VQL
    

---

Visualize:

## Ví dụ 1 case thực tế

![IR-on-linux.png](/assets/img/2023/Velo/02/IR-on-linux.png)

## Difference

![Velo-approach.png](/assets/img/2023/Velo/02/Velo-approach.png)

---

1 thế mạnh khác của VQL là có thể **tích hợp các parser framework**

- **Grok**: is a way of applying regular expressions to extract structured information from log files.
    - Used by many log forwarding platforms such as Elastic
    - Grok expressions are well published
    - Can be incorporated into VQL.
- **Sqlite**: parse database SQL
- etc

**Ví dụ** trong việc parse SSH log sử dụng grok

![parse-ssh-log.png](/assets/img/2023/Velo/02/parse-ssh-log.png)

![grok-parse-syslog.png](/assets/img/2023/Velo/02/grok-parse-syslog.png)

![Grok-expressions.png](/assets/img/2023/Velo/02/Grok-expressions.png)

---

## Sumary

- **Design**: Thay vì thực hiện phân tích chủ yếu ở server như cách DFIR truyền thống, Velo chủ yếu phân tích ở phía Client (ở server chỉ là post-process)
    - Lợi: share tải cho clients tự xử lý thay vì bắt 1 server đảm nhiệm chức năng xử lý 1 lượng lớn dữ liệu
        - server phải đủ mạnh → cost phần cứng cao
        - trong khi client đã có sẵn phần cứng, chỉ cần xử lý 1 lượng thông tin nhỏ (hơn rất nhiều so với khối lượng trên server - nếu tiếp cận theo cách truyền thống)
        - thời gian xử lý trên server sẽ lâu hơn (rất) nhiều so với chạy đồng thời trên các endpoint rồi gửi về server
    - Hại: có thể gây cao tải cho client khi trực tiếp xử lý, phân tích artifact
        - Velo đã tính đến việc này ngay từ khi thiết kế ban đầu, việc của threat hunter là limit CPU usage/opsec/time/… cho mỗi lần hunting
            - **`Client memory and CPU usage is controlled via throttling and active cancellations.`**
- **Approach**: Thay vì Thu thập khối lượng lớn dữ liệu như cách tiếp cận DFIR truyền thống, Velo tập trung vào các dữ liệu có giá trị cho việc DFIR **cụ thể**
    - Velo cũng có tính năng collect cho phép thu thập dữ liệu (file) có chủ đích rồi upload lên server để phục vụ mục đích điều tra
        
        ![upload-param-option.png](/assets/img/2023/Velo/02/upload-param-option.png)
        
        *Khi hunt/run artifact → option `parameters` cài đặt upload file*
        
    - Server cũng được handle, ****optimized**** để xử lý vấn đề tốc độ và khả năng mở rộng
        - Concurrency để đảm bảo tính ổn định
        - Bandwidth limit để đảm bảo băng thông mạng ổn định

# DFIR in VELO

**The goal of VQL** is to **automate** *as much* ****of the **routine DFIR work** *as possible*

refer: [SANS 2022 summit](https://docs.velociraptor.app/presentations/2022_sans_summit/)