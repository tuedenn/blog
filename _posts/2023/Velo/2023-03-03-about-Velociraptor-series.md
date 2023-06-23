---
title: About my Velociraptor series
author: tuedenn
date: 2023-03-03 19:33:00 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

# Q: Tại sao lại viết về Velociraptor, mà không phải là công cụ nào khác?

Vài năm trước, tôi đã tìm hiểu về 1 số công cụ dạng agent-less (powershell remote) (https://mgreen27.github.io/posts/2017/01/12/PowerShell_Remoting_IR.html), và thấy thích thú với 1 công cụ tên là **Kansa**. Thật sự tôi học hỏi được nhiều điều từ Kansa và đã áp dụng nó vào trong công việc của mình. Tuy nhiên, enable power-remote (WinRM) cũng tiềm ẩn nhiều nguy cơ, trong khi đó powershell là 1 trong số những techniques được attacker sử dụng phổ biến nhất.

Tôi cũng đã từng tìm hiểu qua về GRR, nhưng thoạt đầu nó có giao diện hơi xấu (hoặc không vừa mắt cá nhân tôi), và cũng chưa có nhu cầu triển khai & đào sâu về nó, nên chỉ đọc qua loa 1 số paper, blog. Tản mạn 1 chút, tôi có quen và từng trao đổi với 1 người anh hiện đảm nhiệm chức vụ quan trọng trong 1 tổ chức tương đối có tiếng trong ngành. Anh ấy hứng thú với GRR. Tại thời điểm 2 anh em trao đổi, anh ấy có ý định triển khai GRR tại tổ chức (và các đơn vị phụ thuộc) mình. Tại thời điểm hiện tại, tôi biết được quá trình triển khai vẫn đang tiếp tục. **Có thể**, rất nhiều tổ chức tại Việt Nam đã/sẽ sử dụng GRR như 1 công cụ để ứng cứu sự cố từ xa.

Trong quá trình tìm tòi lang thang trên mạng, tôi tình cờ đọc được 1 bài viết trên medium của Mike Cohen nói về kỹ thuật forensics. Tôi không nhớ chính xác là bài viết nào, nhưng viết khá tỉ mỉ. Theo thói quen, tôi đọc xong thì bấm truy cập vào trang cá nhân để xem có gì hay ho nữa không, thì tình cờ nhìn thấy Velo, nhưng đọc lướt qua thấy cũng không có nhiều dấu ấn. 

Một thời gian sau, tôi được công ty đầu tư cho học khoá SANS FOR508 của SANS. Cũng trong khoá học này, tôi lại được tiếp cận với Velociraptor. Khá có duyên, nhưng khổ nỗi là Velociraptor sử dụng 1 ngôn ngữ truy vấn riêng (mới) tên là VQL. Thoạt đầu nhìn vào thấy muốn tìm hiểu thì phải học thêm 1 ngôn ngữ mới, trong khi đặc thù công việc của tôi lúc đó chưa cần đến Velociraptor (chưa có đất dụng võ) và tôi cũng hơi lười :)). Nên tôi cũng chưa bắt tay vào tìm hiểu sâu, mà chỉ đọc loáng thoáng qua. Nhưng cũng lưu tâm và bookmark lại 1 số blog, paper để sau này nghiên cứu thêm.

Thời gian gần đây, tôi có ý định chuyển hướng công việc nên tìm hiểu kỹ hơn về các công cụ hunting/response, cố gắng tìm ra 1 công cụ phù hợp với môi trường doanh nghiệp (bên A). Sau 1 thời gian, tôi tình cờ phát hiện ra người thiết kế và tạo ra Velo - Mike Cohen cũng chính là người tham gia dev (chính?) GRR (và Rekall). Ông từng làm hơn 7 năm trong Google (Greater Seattle Area) từ 09/2010 đến 02/2018 

> "Working with Google I developed a number of projects including: GRR - an advanced incident response and remote forensics tool, Rekall a memory analysis and forensic framework. I also worked in Google cloud IAM."

Vậy Velo với GRR có gì khác nhau? :D Đó là suy nghĩ của tôi ngay lúc đó. Với bản tính tò mò, tôi lang thang và tìm được 1 số bài viết (và 1 số bài viết khác tôi không bookmark lại):
- https://velociraptor.velocidex.com/velociraptor-e48a47e0317d
- https://docs.velociraptor.app/docs/overview/history/
- https://docs.velociraptor.app/blog/html/2018/08/10/design_differences_between_velociraptor_and_grr/
- https://www.libhunt.com/compare-velociraptor-vs-grr

Tôi nghĩ đến đây mọi người cũng đã có câu trả lời tại sao tôi lại chọn Velo, chứ không phải GRR hay công cụ nào khác. :D 


# Series về Velociraptor

Tôi thấy Velo là 1 công cụ nhanh, mạnh, linh hoạt. Một công cụ phù hợp để thực hiện các việc liên quan đến Threat Hunting & DFIR. Velo có 1 đội ngũ phát triển mạnh & nhiệt tình trong việc support, cải thiện tính năng. Cộng thêm 1 cộng đồng hoạt động tương đối sôi nổi (thời điểm tôi viết bài này, cộng đồng còn tương đối ít, nhưng tôi đánh giá là tương đối sôi nổi, có nhiều gương mặt uy tín trong làng DFIR mà tôi sinh hoạt cùng ở một số cộng đồng khác, họ cũng hoạt động khá tích cực trong cộng đồng Velociraptor)... 

> Nhìn chung, Velo có đầy đủ tiềm lực và điều kiện để phát triển mạnh mẽ & bùng nổ. :D 

Tôi đã đề xuất triển khai Velociraptor tại đơn vị tôi đang làm việc, thật vui là mọi người đã tin tưởng và đồng ý. Tôi nghĩ đơn vị tôi là đơn vị đầu tiên tại Việt Nam triển khai công cụ này (nếu đơn vị bạn đã triển khai prod từ trước 2023 thì có thể nhắn tin riêng cho tôi để tôi đính chính, chỉnh sửa lại nội dung xD). 

Tôi rất mong muốn mình có cơ hội tham gia một số hội thảo (với vai trò đại diện đơn vị làm việc, hoặc cá nhân) để giới thiệu cách thức tôi áp dụng Velo vào giải quyết bài toán Threat Hunting & Incident Response trong doanh nghiệp. Đó là 1 trong số nguyện vọng của tôi nhằm thúc đẩy cộng đồng DFIR tại Việt Nam.

Tại thời điểm viết bài, tôi chưa thấy 1 bài viết nào "bằng tiếng Việt" nói về công cụ này. Qua quá trình hỏi han một số anh em trong mảng DFIR, cũng chưa ai nghe nói tới công cụ Velociraptor. Tôi nghĩ mình nên chia sẻ lại kinh nghiệm & kiến thức thu thập được của bản thân cho cộng đồng, (góp phần) thúc đẩy nhiều người tham gia vào mảng Threat Hunting & Digital Forensics and Incident Response tại Việt Nam.

Tôi mong các bạn cũng (sẽ) có 1 cái nhìn tích cực về Velociraptor. Give it a chance!

> Tin tôi đi, trong 1 tương lai không xa, có thể Velociraptor sẽ là công cụ số 1 về Threat Hunting & DFIR. :D
> {: .prompt-tip }


![Rocket-Velociraptor.png](/assets/img/2023/Velo/Rocket-Velociraptor.png)

<br>
<br>

> Fact: Velociraptor (khủng long móng vuốt) mới là loài khủng long "săn mồi vô địch" của thế giới tiền sử, chứ không phải Tyrannosaurus-rex (khủng long bạo chúa aka T-rex) :D 
{: .prompt-tip }

Nếu muốn tìm hiểu thêm về khủng long thì các bạn có thể xem "Công viên kỷ Jura" (để giải trí) hoặc ghé [bảo tàng khủng long Phillip J. Currie](https://dinomuseum.ca/)

<br>

#HappyHunting