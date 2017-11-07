+++
author = ""
comments = true
date = "2017-11-06T16:50:29+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "date-la-gi"
tags = ["swift"]
title = "Date là gì, có ăn được không?"

+++

## Date

Date là một đối tượng gần như không thể thiếu trong các app hiện nay. Nó xuất hiện trong lịch, trong thương mại điện tử, trong các app nhắc việc.
Làm việc với date tưởng chừng như đơn giản nhưng lại là vấn đề hết sức đau đầu vì nó mang tính tương đối :D. Tỉ như ở Việt Nam, người dân đang vội vã đón con cuối chiều, nhưng ở nước khác, họ lại đang cafe buổi sáng. Phức tạp hơn, có những nước lớn, thời gian lại phân theo nhiều timezone khác nhau (Nga, Mỹ), đầu quốc gia một giờ, cuối quốc gia lại là một giờ khác. Oh my god!

IOS hỗ trợ làm việc với time tương đối tốt, thậm chí cũng đã có nhiều thư viện đơn giản hóa việc này. Tuy nhiên việc hiểu bản chất của *Date* cũng giúp bạn suy ra cách thức vận hành của thư viện hoặc có thể tự tạo cho mình một thư viện riêng phù hợp với yêu cầu. Chúng ta gọi dễ thương đấy là *lightweight* :D

Hôm nay mình sẽ trình bày những vấn đề cơ bản nhất của Date để các bạn có được cái nhìn overview

### Now

Lấy ra thời gian hiện tại đơn giản chỉ cần 1 dòng (local time)

> let now = Date()

Tuy nhiên trong thực tế, không ai lấy ra local time của máy cả, vì user có thể cheat cái này :D. Chúng ta sẽ lấy current time của server trả về và convert date theo timezone của server.

Trong đối tượng *date* có 3 thuộc tính (tính bằng giây)

* timeIntervalSinceReferenceDate
* timeIntervalSince1970
* timeIntervalSinceNow

Đi lần lượt theo thứ tự thì cái đầu tiên lấy time theo mốc 01-01-2001 00:00, cái thứ hai lấy theo mốc 01-01-1970, cái thứ ba lấy theo mốc hiện tại :D. Thuộc tính thời gian hay được sử dụng là *timeIntervalSince1970*, còn có tên gọi khác *epoch time* [tham khảo](https://vi.wikipedia.org/wiki/Th%E1%BB%9Di_gian_Unix). 

### Convert time

Để convert đối tượng *Date* ta dùng *DateFormatter*

{{< highlight objc "style=monokai" >}}
    let dateFormatter = DateFormatter()
    dateFormatter.timeZone = TimeZone(abbreviation: TimeZone.current.abbreviation() ?? "GMT+7")
    dateFormatter.dateFormat = "dd/MM/yyyy"
{{< /highlight >}}

*DateFormatter* giúp ta convert *Date* sang *String* và ngược lại. Để hiển thị date lên app ta sẽ gán *dateFormat* cho đối tượng *DateFormatter* được tạo ra như ở ví dụ trên. Định dạng date format là một string, ơn giời là nó đi theo chuẩn, tham khảo ở [http://nsdateformatter.com/](http://nsdateformatter.com/). Thường thì server sẽ trả về cho client string date theo định dạng nào đó, ta cần phải xác định đúng định dạng để đưa về kết quả mong muốn. Ta cũng không được quên gán timezone để đưa về mốc thời gian ta qui định. Túm váy lại có hai bước cần làm khi tạo *DateFormater*

* Gắn timezone
* Gắn dateFormat

### What is the date today?
Hôm nay là thứ mấy? Với *DateFormater* chúng ta có 1 trick để lấy ra thứ theo tiếng Anh.

{{< highlight objc "style=monokai" >}}
let dateFormatter = DateFormatter()
dateFormatter.dateFormat = "E" // or "EEEE"
print(dateFormatter.string(from: now)) // Tue
{{< /highlight >}}

Kết quả trả về là thứ viết tắt theo tiếng Anh như mon, tue, wed...

Còn nếu muốn trả về thứ theo tiếng mẹ đẻ ta có thể làm như sau

{{< highlight objc "style=monokai" >}}
Calendar.current.component(.weekday, from: self)
{{< /highlight >}}

Kết quả trả về là Int bắt đầu từ 1 tương ứng với chủ nhật (từ 1 chứ không phải từ 0 nhá)

### Show me the date

Giả sử ta muốn lấy *Date* sau một giờ nữa chẳng hạn thì làm thế nào? Cách đơn giản nhất là

{{< highlight objc "style=monokai" >}}
let now = Date()
let oneHourLater = now.addingTimeInterval(60*60) // time unit: second
{{< /highlight >}}

Thêm cách nữa sử dụng *DateComponents*

{{< highlight objc "style=monokai" >}}
let calendar = Calendar.current
let oneHourLater2 = calendar.date(byAdding: .hour, value: 1, to: now)
{{< /highlight >}}

Để giản tiện, chúng ta có thể tạo *extension* như cách các thư viện thường dùng [tham khảo](https://github.com/davedelong/Chronology/blob/master/Inspiration.md)

Vậy cuối cùng *date* có ăn được không? mời mọi người thưởng thức [source code](https://gist.github.com/gg4acrossover/ed80aefc44700d03661aee09eaa1a156) ví dụ trong bài để tự tìm ra câu trả lời :D.




