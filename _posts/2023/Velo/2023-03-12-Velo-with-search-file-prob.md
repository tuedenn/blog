---
title: Velo with search file problem
author: tuedenn
date: 2023-03-12 23:39:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

# Nghiên cứu Hiệu năng trong vấn đề search file

Bài viết này nghiên cứu về hiệu năng trong vấn đề search file. Việc tìm kiếm file là yêu cầu bắt buộc trong DFIR & Threat Hunting, nhưng nếu không kiểm soát được công cụ/script truy vấn thì có thể ảnh hưởng đến tải hệ thống và business. Velociraptor đã giải quyết vấn đề này bằng cách sử dụng plugin glob, timeouts, và các cơ chế giới hạn số lượng kết quả trả về và độ sâu của thư mục được duyệt.

# Hiệu năng trong vấn đề search file

- refer: [https://docs.velociraptor.app/blog/2022/2022-01-05-searching-for-files-on-linux/](https://docs.velociraptor.app/blog/2022/2022-01-05-searching-for-files-on-linux/)

## Use case

- Log4j
- Tìm tất cả các file vulnerable (.jar)
- trên 1 máy
- trên 1000.. máy?

## Prob

- search file là 1 yêu cầu bắt buộc trong DFIR & Threat Hunting
- nếu không kiểm soát được công cụ/script truy vấn thì sao?
    - trên 1 máy
    - trên 1000.. máy?
- Đặc biệt nguy hiểm nếu k control được thời gian, CPU usage, …
    - tăng tải, gián đoạn dịch vụ
    - ảnh hưởng đến business
- Trên hệ thống Linux rất hay xảy ra vấn đề này
    - symlink
    - unusual configuration

→ đây là 1 big prob trong quá trình làm việc (cung cấp dịch vụ) trước đây

## Velo solved probs

- ****The Glob Plugin****
    
    Since 0.6.3 the `glob()` plugin can accept a `recursion_callback` argument.
    
    This is a VQL lambda function that receives the full row and **return a boolean** to decide if the directory should be recursed into. If the lambda returns **FALSE**, the glob plugin does not bother to enter the directory at all, therefore saving a lot of effort.
    
    ```sql
    SELECT FullPath
    FROM glob(globs="/**/*.pem",
    					x => NOT x.FullPath =~ '^/(shared|proc|snap)')
    ```
    -> các đường dẫn không bắt đầu bằng `shared`, `proc` hoặc `snap` -> tránh symlink

    ![Glob-plugin.png](/assets/img/2023/Velo/03/Glob-plugin.png)

    *Controlling recursion in the glob plugin*

    - PROB: nếu không biết path thì sao?
        - Solve:
            - The `recursion_callback` mechanism is very flexible and allows us to choose arbitrary conditions to control the recursion
            
            ![Major numbers larger than 7 are considered “local”](/assets/img/2023/Velo/03/Device-major-n-minor-numbers.png)
            
            *Major numbers larger than 7 are considered “local”*
            
            ```sql
            SELECT FullPath
            FROM glob(globs='/**/*.pem',
                      recursion_callback='x=>x.IsLink OR x.Data.DevMajor > 7')
            ```
            
            - This query is very efficient, **following links** but skipping /proc, **remote filesystems** but covering additional attached storage.
            - We do not need to rely on guessing ****where remote filesystems are mounted, and ****excluding only those directorie**s**, instead **limiting recursion to the type of device hosting the filesystem**.
- **Timelimit (timeouts)**
    
    default 600s timeout -> cancel the search
    
- **Limit row returned**
    
    VQL: `LIMIT 1000` -> limit số dòng kết quả trả về (row returned)
    
- **Limit directory traversal depth**
    
    Velociraptor’s glob expressions can specify a recursion depth -> chỉ định recursion depth 
    

## Conclusions

Velociraptor’s `glob()` plugin has fine grained controls allowing coverage of only a small set of filesystem types, or mount points. This allows Velociraptor to safely search the entire system for files balancing coverage with the risk following symlinks


> Velociraptor đã giải quyết vấn đề hiệu năng trong việc tìm kiếm file diện rộng, tránh ảnh hưởng đến tải hệ thống và nghiệp vụ business.
{: .prompt-tip }