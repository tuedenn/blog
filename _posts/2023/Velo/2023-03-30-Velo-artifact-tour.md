---
title: Quick tour about Velociraptor Artifact 
author: tuedenn
date: 2023-03-30 23:17:32 +0700
categories: [Blogging, Tools]
tags: [Velociraptor, Tool, DFIR, Hunting]
---

# Artifacts

> "Artifact is an object that is made by a person, especially something of historical or cultural interest" - [via Oxford](https://www.oxfordlearnersdictionaries.com/definition/english/artifact#:~:text=%E2%80%8Ban%20object%20that%20is,of%20historical%20or%20cultural%20interest)

Dịch nôm na ra tiếng Việt thì artifact là hiện vật, 1 thứ do con người tạo ra, có giá trị lịch sử hoặc văn hoá.

Trong lĩnh vực điều tra số, thú thật là tôi cũng chưa tìm thấy định nghĩa chính thức về forensics artifact. Nhưng để giải thích cho mọi người hiểu, có thể nói artifacts trong forensic là các object tồn tại trong máy tính, được sinh ra bởi các hoạt động của người dùng (bao gồm cả attacker), và được sử dụng làm bằng chứng để điều tra, xác định các hành vi trên máy tính. 

Nếu bạn là người đã quen với forensic thì khỏi nói, nhưng nếu bạn là một người chưa từng làm công việc liên quan đến forensic, thì tôi có thể đưa ra 2 ví dụ (giả định) cho bạn dễ hiểu:

Ví dụ 1 liên quan đến đời sống một chút: Điều tra tội phạm
1. Một vụ nổ súng tấn công vào một nhân vật VIP, cảnh sát được gọi đến để điều tra hung thủ
2. Cảnh sát sẽ thu thập tất cả các bằng chứng, chứng cứ, hiện vật còn sót lại tại hiện trường, như mảnh đạn, dữ liệu trích xuất hình ảnh camera, ... thậm chí là dữ liệu những người xuất nhập cảnh trong khoảng thời gian gần đấy
3. Dựa trên những "mảnh dữ liệu" còn sót lại, cảnh sát đưa ra một số nhận định về kịch bản có thể xảy ra (dựa trên nghiệp vụ cảnh sát, ai làm công an sẽ hiểu :D)
4. Sau đó, cảnh sát tiếp tục thu thập thêm các dữ kiện liên quan, để chứng minh/bác bỏ các kịch bản đó
5. Các manh mối được sâu chuỗi lại với nhau thành 1 bức tranh
6. Bức tranh càng sáng tỏ thì chân dung kẻ phạm tội càng rõ

Một ví dụ khác về điều tra tội phạm buôn bán mai thoé có tổ chức (Breaking Bad)
![BreakingBad_DEA.png](/assets/img/2023/Velo/07/BreakingBad_DEA.jpg)

Tổng kết lại: những hiện vật *còn sót lại tại hiện trường* được gọi là "Artifact".

Mapping lại với câu chuyện về điều tra tội phạm mạng:
1. Hacker tấn công vào hệ thống cơ sở dữ liệu của công ty Stark International, thực hiện dump dữ liệu bí mật để đem bán
2. Công ty phát hiện dữ liệu đang được rao bán trên chợ đen
3. Một nhóm chuyên gia điều tra số được thuê đến để điều tra lý do tại sao dữ liệu bí mật của họ bị lộ lọt
4. Những chuyên gia này sẽ đưa ra các giả thuyết dựa trên dấu hiệu để điều tra, chứng minh hoặc bác bỏ bằng cách thu thập các dữ liệu (còn sót lại) liên quan tới vụ tấn công. **Những dữ liệu này gọi là "artifact"**
5. Artifact có thể là file, memory, log, ...
6. Vì là "hiện vật", nên **không phải lúc nào artifact cũng tồn tại**, hoặc tồn tại nhưng **không còn nguyên vẹn**
7. Thách thức của người làm điều tra số là làm sao có thể vẽ lại được bức tranh toàn cảnh về cuộc tấn công (càng sáng tỏ càng tốt) từ artifact thu thập được
8. Mục tiêu là tìm ra nguyên nhân gốc rễ của vấn đề, đưa ra các biện pháp khắc phục, phòng ngừa trong tương lai.


# Velociraptor Artifact

> An Artifact is a way to package one or more VQL queries in a human readable YAML file, name it, and allow users to collect it. An Artifact file simply embodies the query required to collect or answer a specific question about the endpoint.
{: .prompt-info }

Trong thiết kế của Velo, 1 artifact là 1 hoặc nhiều câu truy vấn VQL được tổng hợp lại vào trong 1 file định dạng `.YAML`. Mỗi file artifact có 1 tên riêng, nhằm thu thập **những dữ liệu cụ thể**, phục vụ mục đích trả lời **1 câu hỏi nhất định** - nhằm chứng minh/bác bỏ giả thuyết (cách tiếp cận dựa trên giả thuyết - hypothesis driven). Bởi lẽ, mục tiêu chính của artifact là dùng để hunting (diện rộng).  -> Bạn có thể tìm hiểu thêm về **Hypothesis-driven Threat hunting**  tại [đây](https://socprime.com/blog/threat-hunting-hypothesis-examples/)

Có thể nhiều bạn thắc mắc tại sao lại chọn định dạng `.YAML`? 

Như các bạn thấy, `.YAML` là 1 định dạng dễ rất đọc & hiểu, tuỳ biến, rất thân thiện với con người. Rất hay được sử dụng trong việc viết các file config. Bạn có thể xem nội dung file client.config.yaml hoặc server.config.yaml đã tạo từ [bài viết trước](https://tuedenn.github.io/blog/posts/Velo-Deploy-Server/#kh%E1%BB%9Fi-t%E1%BA%A1o-file-c%E1%BA%A5u-h%C3%ACnh). Ngoài velo, rất nhiều công cụ khác sử dụng `.YAML` như Sigma, Ansible,... 

> Tôi thật lòng khuyên bạn nên làm quen với chúng từ sớm (cùng với json). **Thời điểm thích hợp nhất là ngay sau khi bạn đọc hết dòng này**!
{: .prompt-tip }

## Anatomy of an Artifact

Dưới đây là ví dụ về 1 artifact của Velociraptor

```yaml
  name: Custom.Artifact.Name
  description: |
    This is the human readable description of the artifact.

  type: CLIENT

  parameters:
    - name: FirstParameter
      default: Default Value of first parameter

  sources:
    - name: MySource
      precondition:
        SELECT OS From info() where OS = 'windows' OR OS = 'linux' OR OS = 'darwin'

      query: |
        SELECT * FROM info()
        LIMIT 10
```
{: file='Custom.Artifact.Name.yaml' }

Trong đó:

| Parameters    |      Mô tả      |  Ví dụ |
|:--------------|:----------------|:------|
| Name          | Tên của artifact, nhằm phân loại và tìm kiếm. Được đặt tên theo quy tắc phân cấp, và phân đoạn bằng dấu `"."` | `Windows.System.TaskScheduler` |
| Description   | Mô tả ngắn gọn về artifact, chứa các keywords để dễ dàng tìm kiếm, đọc hiểu. | `The Windows task scheduler is a common mechanism that malware uses for persistence. It can be used to run arbitrary programs at a later time. Commonly malware installs a scheduled task to run itself periodically to achieve persistence` |
| Type          | Kiểu artifact -> xác định ngữ cảnh của artifact, nơi VQL khởi chạy. | `CLIENT, SERVER, CLIENT_EVENT, SERVER_EVENT` |
| Parameters    | Những tham số truyền vào cho VQL. Khai báo (trên GUI) các trường như: `name`, `default`, `type` | `name: TasksPath`, `default: C:/Windows/System32/Tasks/**` |
| Sources       | Một artifact có thể tạo ra nhiều bảng kết quả khác nhau tuỳ theo `Source` được define ban đầu. Mỗi (uniqe) `Source` ứng với 1 bảng kết quả trong tab `Results` hoặc `Notebook` | `sources: -name: Analysis1, query: \| SELECT A FROM B, -name: Analysis2,  precondition: hixhix, query: \| SELECT X FROM Y` |
| Precondition  | Điều kiện tiên quyết cần phải thoả mãn trước khi truy vấn `Query`. Nhằm đảm bảo chỉ thu thập những dữ liệu phù hợp. Thường là truy vấn, filter theo hệ điều hành | `SELECT OS From info() where OS = 'windows'` |
| Query         | Một hoặc nhiều câu truy vấn VQL, nhằm thu thập chính xác dữ liệu mong muốn. Trả về kết quả là 1 bảng kết quả define trong `Source`. Bắt buộc phải chứa 1 câu lệnh `SELECT` | `SELECT * FROM info() LIMIT 10` |


Danh sách những built-in artifact có thể được xem ở [đây](https://docs.velociraptor.app/artifact_references/)

# Velociraptor Community Artifact 

như tôi có đề cập ở [bài viết trước](https://tuedenn.github.io/blog/posts/Velociraptor-fundamental/#t%C3%ADnh-n%C4%83ng) về tính năng của Velo: 

`Knowledge sharing: Queries shared within the community enable lower-skilled users to take advantage of queries written by more experienced DFIR experts.`

Velociraptor là công cụ điều tra số mã nguồn mở, **`by DFIR for DFIR`**. Việc chia sẻ, dùng lại các tri thức về điều tra số là rất quan trọng. Nó cho phép những người mới tham gia vào lĩnh vực, chưa có nhiều kiến thức chuyên sâu về DFIR, hoặc mới làm quen với Velo có thể dễ dàng sử dụng *lại* các tri thức, artifact từ những chuyên gia trong ngành. Đồng thời, đóng góp vào kho tri thức chung của cộng đồng `by community, for community`.

Bạn có thể xem những tri thức, artifact được chia sẻ, đóng góp bởi cộng đồng Velociraptor ở [link này](https://docs.velociraptor.app/exchange/)

## Import Artifact Exchange

Mặc định, bạn sẽ sử dụng được full các artifact có sẵn (built-in) của Velociraptor. Đây là những công cụ được viết ra bởi đội ngũ phát triển Velo. Nếu bạn muốn sử dụng những artifact đóng góp bởi cộng đồng, thì sẽ cần phải import vào server của mình.

> You can automatically import the entire content of the artifact exchange into your server by running the Server.Import.ArtifactExchange artifact.
Alternatively, download the artifact pack for Version 0.6.8 or for older versions, and manually upload them in the GUI (navigate to View Artifacts and click the Upload Artifact Pack button)
{: .prompt-info }


Có 2 cách để import artifact được chia sẻ từ cộng đồng:
- sử dụng artifact `Server.Import.ArtifactExchange` để tự động import tất cả các artifact có sẵn
- truy cập [link](https://docs.velociraptor.app/static/exchange/artifact_exchange_v2.zip) để download artifact pack for Version 0.6.9, sau đó upload thông qua GUI

Ở đây tôi lựa chọn phương án 1 - sử dụng artifact `Server.Import.ArtifactExchange`.

- Đầu tiên, bạn truy cập server GUI, vào phần `Collection`, chọn `New Collection: Select Artifacts to collect`. Sau đó điền `exchange` vào ô search như hình 1 dưới đây rồi chọn `Server.Import.ArtifactExchange`

![1-create-new-collection-import.png](/assets/img/2023/Velo/07/1-create-new-collection-import.png)

Một cách khác là truy cập GUI, vào phần `Notebook`, chọn  `Create a new notebook`

![1-create-notebook-import.png](/assets/img/2023/Velo/07/1-create-notebook-import.png)

sau đó sử dụng VQL để import artifact `Server.Import.ArtifactExchange`.

> Để dễ dàng, tôi khuyên bạn nên sử dụng cách đầu tiên. Để hiểu bản chất, tôi khuyên bạn nên test cả 2 cách.
{: .prompt-tip }

- Sau khi chọn artifact `Server.Import.ArtifactExchange`, bạn cần phải cấu hình parameters cho artifact

![2-select-prefix-import.png](/assets/img/2023/Velo/07/2-select-prefix-import.png)

Có 2 param là `ExchangeURL` (ứng với đường dẫn tới file zip được release trên github) và `Prefix` (quyết định việc hiển thị, phân loại các artifact trên server của bạn, mặc định là `Exchange.`). Tôi khuyên bạn nên đọc để hiểu thôi, còn lại các tham số này hãy để mặc định.

- Sau khi đã cấu hình parameters, cần chỉ định resource sẽ sử dụng cho việc khởi chạy. 

![3-specify-resources.png](/assets/img/2023/Velo/07/3-specify-resources.png)

Đây là tính năng control rất hữu ích của Velo. Nó cho phép set CPU limit, time limit, IOps/sec limit, row result limit,... Tránh tình trạng lỗi (thường là logic) gây cao tải trên máy tính người dùng. Hãy tưởng tượng tình trạng bạn khởi chạy 1 artifact cho 1 server cực kỳ quan trọng bên Mẽo, nhưng khi dev artifact, bạn bị lỗi logic dẫn tới việc thực thi loop, gây quá tải, chết dịch vụ trên server đấy, ... OMG

Nhưng ở đây artifact chạy trên `server` (Type) -> nó là limit resource của server. Tôi giải thích để mọi người hiểu cách hoạt động, còn ở đây bạn có thể để mặc định, không cần sửa gì, vì nó quá nhẹ, không ảnh hưởng.

- Sau khi chỉ định resource, bạn có thể review lại, sau đó bấm `Launch` để khởi chạy (như trong hình 1 bên trên)

- Artifact được khởi chạy, lưu log thực thi ở tab `Log` nhằm mục đích tracking, debug

![4-log-download.png](/assets/img/2023/Velo/07/4-log-download.png)

- Sau khi thực thi xong, bạn có thể xem kết quả ở tab `Result` hoặc chỉnh sửa thông qua `Notebook`

![5-results.png](/assets/img/2023/Velo/07/5-results.png)

![6-notebook.png](/assets/img/2023/Velo/07/6-notebook.png)


# Tổng kết

Hy vọng qua bài viết này, bạn đã có cái nhìn tổng quát về artifact trong DFIR cũng như trong Velo. Bạn giờ đây có thể khám phá và sử dụng kho artifact đầy phong phú của Velo. Tôi khuyên bạn nên chọn ra một vài artifact để test thử (trên môi trường test lab) để tìm hiểu thêm về cách hoạt động.

Trong bài viết tiếp theo, tôi dự định viết lại cho mọi người hiểu thêm về ngôn ngữ truy vấn VQL. Practive bằng cách thử viết 1 artifact riêng cho bản thân. Mong mọi người sẽ thích.

> Nếu bạn hứng thú với nội dung nào liên quan, có thể đề xuất hoặc nhắn tin riêng với tôi để trao đổi hoặc thảo luận. Cảm ơn!
{: .prompt-info }