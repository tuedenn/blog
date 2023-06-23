---
title: Velociraptor fundamentals
author: tuedenn
date: 2023-03-05 11:33:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

# Tổng quan Velociraptor

Velociraptor là một công cụ mã nguồn mở tiên tiến, độc đáo về endpoint monitoring, digital forensics và cyber response trên đa nền tảng. Velo được phát triển bởi đội ngũ chuyên gia trong lĩnh vực Digital Forensic and Incident Respons (DFIR). Khả năng tuỳ biến của Velo nằm ở việc sử dụng ngôn ngữ truy vấn Velociraptor Query Language (VQL) để thu thập, truy vấn và giám sát hầu hết mọi khía cạnh của endpoint. Velo có nhiều ưu điểm hơn so với GRR.

# What is velo?

## History

Velo được Rapid7 mua lại & dẫn dắt (giống như cách Rapid7 mua lại metasploit khoảng 10 năm trước). Velo kỳ vọng sẽ tiếp tục phát triển mạnh và bền vững hơn, tương tự cách mà Metasploit đã đạt được.

- cùng trong hệ sinh thái sản phầm của Rapid7, nghiễm nhiên Velo được thừa hưởng tri thức từ các sản phẩm của Rapid7 như Metasploit, AttackerKB,…
- new vuln → attackerKB (knowledge) → metasploit (simu & attack) → velo (hunt & forensics)
- Tương lai: Metasploit as **RED** <> **BLUE** as Velociraptor

> **Velociraptor is a unique Free and Open Source DFIR tool, giving *you* power and flexibility through the Velociraptor Query Language**
> 

## What is velo?

> Velociraptor is a unique, advanced open-source endpoint monitoring, digital forensic and cyber response platform
> 

Velo là 1 công cụ open-source tiên tiến, độc đáo về endpoint monitoring, digital forensics và cyber response.

Velo được phát triển bởi đội ngũ *chuyên gia* trong lĩnh vực Digital Forensic and Incident Respons (DFIR) - những người cần 1 công cụ mạnh mẽ, hiệu quả để thực hiện việc hunting các dấu hiệu (artifacts) (cũng như giám sát hành vi) trên 1 nhóm các endpoints.

![Velo-expert-knowledge.png](/assets/img/2023/Velo/01/Velo-expert-knowledge.png)

Velo cung cấp khả năng điều tra & xử lý hiệu quả hơn trong các trường hợp như:

- Tái hiện lại các hành vi của kẻ tấn công thông qua việc điều tra số
- Săn lùng, tìm kiếm các dấu hiệu độc hại của những kẻ tấn công tinh vi
- Điều tra các hành vi tấn công mã độc & xâm nhập mạng, …
- Giám sát các hành vi đáng ngờ của người dùng (như copy file qua USB)
- Thu thập thông tin về endpoint theo thời gian thực để phục vụ threat hunting cũng như điều tra số trong tương lai
- …

Velo là 1 công cụ mạnh mẽ nhưng lại rất linh hoạt nhờ vào ngôn ngữ truy vấn **V**elociraptor **Q**uery **L**anguage (VQL). VQL được coi là 1 framework để tạo ra các artifacts có tính tuỳ biến cao, cho phép thu thập, truy vấn, và giám sát hầu hết mọi khía cạnh của endpoint, trên từng máy hoặc diện rộng trên toàn mạng. VQL có thể được sử dụng trên cả server lẫn endpoint để thực hiện các hành vi mà người quản trị mong muốn.

# Why velo?

## Design

Ban đầu, Velo được thiết kế để những tổ chức có quy mô nhỏ (và vừa) có thể triển khai dễ dàng & tiết kiệm chi phí

- Mục tiêu ban đầu 10-15k endpoints
    - **single serve**r limits reacehed at about 20k endpoints
- không phụ thuộc bởi các 3rd party
- sử dụng local file storage → đơn giản & tiết kiệm
- chủ yếu phục vụ cho việc triển khai theo kiểu tư vấn tạm thời (ephemeral consultancy)

Giờ đây, Velo đã phát triển đến mức trở thành 1 ứng dụng mức doanh nghiệp:

- large number of endpoints
    - từ bản 0.5.9 (cuối 2021), velo cho phép thiết kế kiến trúc dạng **multi-frontend**
    - retain the **EVERYTHING IS A FILE** philosophy: uses distributed filesystem (EFS)
    - base gRPC
    - mô mình **Master - Minion**
        
        ![Velo-master-minion.png](/assets/img/2023/Velo/01/Velo-master-minion.png)
        
        - phần lớn các hoạt động giao tiếp và thu thập thông tin từ phía client(endpoint) được thực hiện bởi minion
        - minion nhận & gửi event cho master
        - master control & làm trung gian cho các minion
        - 100k+ endponts **easy**
- khả năng giám sát liên tục (permanent monitoring)
- khả năng response

## Tính năng

- **Access to a free open-source solution**. Lower cost of ownership has been a primary reason for organizations to consider open-source software.
- **Forensic analysis at the endpoint**. VQL enables automated and surgical analysis to be done on the endpoint, hunting threats at scale in minutes without affecting endpoint performance.
- **Knowledge sharing**. Queries shared within the community enable lower-skilled users to take advantage of queries written by more experienced DFIR experts.
- **Surgical collection at speed and scale**. Efficiently triage and rapidly analyze forensic evidence to determine root cause quickly.
- **Accelerated mean time to detect (MTTD) and mean time to respond (MTTR).** Large-scale proactive hunting within minutes allows users to quickly action tactical interventions and remediation, limiting potential damage.
- **Support for Linux, Windows, and macOS.**
- **Powerful VQL query language.** Investigators define artifacts to collect and hunt endpoints without needing to modify any of the source code or deploy additional software, adapting queries quickly in response to shifting threats and new information gained through the investigation.

## So sánh với GRR

![GRR-issues.png](/assets/img/2023/Velo/01/GRR-issues.png)

![GRR-architect.png](/assets/img/2023/Velo/01/GRR-architect.png)

![Velo-features.png](/assets/img/2023/Velo/01/Velo-features.png)