---
title: About HTB writeup series
author: tuedenn
date: 2023-04-15 22:08:00 +0700
categories: [Blogging, HTB]
tags: [HTB, Hackthebox, Volatility, DFIR, writeup]
---

# Introduction

Đây là bài đầu tiên trong loạt bài tôi dự định viết về series HTB challenges writeup. Vì là bài đầu tiên nên tôi có thể sẽ hơi lan man một chút, bởi vì tôi thích dẫn dắt mọi người vào câu chuyện tôi muốn nói. Bạn có thể bỏ qua phần kể lể về my story, bằng cách jump to [#HTB](./#htb-forensic-challenges) hoặc bỏ qua bài viết này và vào thẳng từng challenge, ví dụ như  [Challenge 1 writeup](../HTB-Reminiscent-writeup).

Nếu không, mời bạn đọc câu chuyện của tôi. Tôi rất vui nếu các bạn sinh viên có thể áp dụng phần nào đó câu chuyện của tôi vào việc sắp xếp thời gian của bản thân mình!

## My story

Những lúc rảnh rỗi, đa số mọi người thường lướt phaybuk, insta, tocktock,... Nhưng có bao giờ bạn giật mình nhận ra rằng mình đã nằm lười xem mấy thứ này cả vài tiếng đồng hồ rồi hay không? Tôi nghĩ là có. Và tôi cũng không phải ngoại lệ. Thời sinh viên, nhiều lúc tôi đi học đi làm về, chỉ định bỏ điện thoại ra xem fb có thông báo hay tin nhắn gì mới hay không, chừng vài phút rồi đi tắm rửa, abcxyz (đa số mọi người đều nghĩ vậy :)). Nhưng khi nhìn đồng hồ thì giật mình vì đã 10, 11 giờ đêm. Tức là tôi đã từng dành 1/6 ngày để nằm lướt fb. Thật sự rất lãng phí vì nó chẳng đem lại nhiều giá trị gì cho tôi. Thế là tôi quyết định phải thay đổi. 

Thay vì lướt fb, tôi chuyển sang đọc tin trên twitter. Nếu bạn thực sự muốn học hỏi tiếp thu kiến thức mới về ngành ATTT, tôi khuyên bạn nên *dành thời gian vào 
twitter thường xuyên*.

Tôi thường đọc tin trên twitter đến khoảng 9h tối rồi làm những việc cá nhân như tắm rửa, vệ sinh, ... Chừng 10-11h tôi quay lại đọc & nghiên cứu kỹ hơn những gì tôi đã bookmark trước đó. Khoảng 12h (muộn nhất là 1h sáng) thì đi ngủ. Thời sinh viên, tôi thường thức đến 3h sáng, nhưng khi đi làm, tôi cần phải ngủ sớm hơn để đủ tỉnh táo vào ngày hôm sau. Những ngày tôi thức quá độ đến tầm 2h sáng, hôm sau uống 2 cốc cafe vẫn lim dim lờ đờ.

Cuối tuần, tôi dành phần lớn thời gian để tổng dọn dẹp nhà cửa, và sắp xếp thời gian rảnh để đi chơi/chơi game với bạn bè người thân. Đến tối, tôi tổng hợp & review lại những gì mình làm được trong tuần, lên plan cho tuần sau. Tôi cũng tranh thủ thời gian để thực hiện 1 số thử thách/lab mới publish trong tuần trên tryhackme/HTB.

Việc viết blog cũng dựa trên những gì tôi đã làm & những gì tôi có thể chia sẻ.

## HTB Forensic challenges

Giống với Tryhackme, Hackthebox (HTB) cũng là 1 nền tảng cung cấp những thử thách dựng sẵn để người chơi có thể tham gia giải đố, tìm flag. Cả 2 đều hỗ trợ free tier, nhưng vẫn tương đối hạn chế. Nếu muốn access vào các challenge/room chuyên biệt, bạn cần trả 1 lượng phí nhất định tuỳ theo nền tảng. Tôi khuyên bạn khi có điều kiện nên đầu tư thử để trải nghiệm. **Đầu tư vào học hành là 1 khoản đầu tư có lãi**!

Quay lại vấn đề chính, HTB đa phần là các lab/challenge cho việc attack web, những lab/challenge cho việc forensic vẫn còn rất ít. Hiện có 13 challenge thuộc category Forensic trên HTB

![HTB-forensic-challenge.png](/assets/img/2023/HTB/Reminiscent/HTB-forensic-challenge.png)

Tôi sẽ cố gắng giải lần lượt từng bài rồi viết writeup lại cho mọi người tham khảo.

## Target

Có thể sẽ có người hỏi tôi viết writeup nhằm mục đích gì?

Có rất nhiều bài writeup ngoài kia, bạn có thể search và đọc chúng. Nhưng nếu bạn đã mất công theo dõi, đón đọc loạt bài viết của tôi, tôi cũng sẽ cố gắng làm sao để bạn thu nhặt được cho mình ít nhất một thứ gì đấy để không lãng phí thời gian của nhau.

Đa phần, các bài viết writeup sẽ tập trung vào việc tìm flag, đưa ra lời giải, done! Còn tôi lại nghĩ khác, tôi muốn link các challenge này lại với công việc thực tế, xem nó có giải quyết vấn đề gì hoặc ứng dụng vào tình huống nào khi thực hiện điều tra sự cố thực tế hay không. 

> Tất nhiên, tôi/em cũng mới chỉ có vài ba năm kinh nghiệm nên nhiều thứ vẫn chưa thể nào bao quát hết được. Rất mong những anh, chị, hoặc các bạn khác đã có nhiều kinh nghiệm hơn, có thể góp ý để loạt bài viết này được cải thiện, cùng chung tay xây dựng nền DFIR nước nhà.
{: .prompt-tip }

#DFIR-VN #DFIR-land