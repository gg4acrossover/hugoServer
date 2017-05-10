+++
date = "2016-09-01T17:22:59+07:00"
draft = false
title = "Các thuộc tính của property trong objC"
tags = ["objc"]
image = "images/cover.jpg"
comments = false

+++

Thời gian đầu làm việc với obj-c mình khá băn khoăn trong việc sử dụng các thuộc tính trong property như strong, weak, copy, assign. Nhân lúc rảnh rỗi sinh nông nổi, mình giới thiệu qua về vấn đề này để những ai mới làm quen với obj-c sẽ tiếp cận nhanh hơn.

Vì những cái sắp trình bày có liên quan đến bộ nhớ, mình sẽ nói qua về stack và heap trước khi đi vào vấn đề chính.

# Stack
Stack là vùng bộ nhớ để chứa biến local, biến tạm. Khi bạn tạo ra đối tượng và được lưu trên stack thì việc quản lý bộ nhớ diễn ra một cách tự động. Thế nên bạn không phải bận tâm đến vấn đề memory leak.

# Heap
Vùng bộ nhớ mà bạn muốn sử dụng tự phải cấp phát và giải phóng khi cần thiết. Bạn trực tiếp quản lý nó, thế nên khả năng rò rỉ bộ nhớ có thể xảy ra. Như C, ta đã malloc thì phải free. Vấn đề này dễ thở hơn khi sử dụng JAVA hay Obj-C (cơ chế garbage collection).

**============**

Lợi thế trong việc sử dụng stack là tốc độ nhanh, sự đơn giản (tự động hủy khi hàm return hoặc khi ra khỏi scope). Tuy nhiên, khi lập trình chúng ta có xu hướng muốn quản lý lifetime của đối tượng, thế nên ta sử dụng heap khá thường xuyên.

Đối tượng trong obj-c thì hầu hết được lưu ở heap. Apple dịu dàng nghĩ ra strong, weak, copy, assign để coder quản lý bộ nhớ thuận tiện hơn (nhờ ARC). Nhưng nhiều thuộc tính như thế dễ làm coder băn khoăn cái nào nên dùng, cái nào không.

# Strong
Cơ chế quản lý bộ nhớ của ARC là nó sẽ đếm số reference đến một đối tượng cụ thể nào đó. Khi con số reference này bằng 0 thì đối tượng đó tự động giải phóng khỏi bộ nhớ. Ta không phải hủy bằng tay nữa. Strong sẽ làm cho reference tới đối tượng tăng lên 1 đơn vị, khi đó ta nói đối tượng có 1 owner. Một đối tượng có thể có nhiều owner. Nhưng tốt hơn hết là cố gắng làm sao để một đối tượng chỉ có 1 hoặc 2 owner thôi. Vì sao sẽ nói ở phần weak.

Ta sử dụng strong đối với các UIView, UITableView,... khi tự tạo View bằng code, chứ không phải bằng storyboard hay xib.

# Weak
Cũng tương tự như strong, ngoại trừ việc nó không tăng số owner tới đối tượng . Weak không trực tiếp can thiệp vào việc hủy của đối tượng, vốn là công việc của strong. Vậy tác dụng của weak là gì? Nếu ta có nhiều owner (strong) đến đối tượng thì sẽ rắc rối cho việc giải phóng bộ nhớ, thế nên ta cần dùng đến weak.

![Kipalog](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/173cc1b2eae8a7c4.png_tn5mgte0n5)

Như ta thấy ở hình trên, Person và Apartment có reference qua nhau. Khi john và number73 không còn owner đến 2 thực thể này nữa thì 2 thực thể này vẫn không bị hủy. Lý do là vì owner tới 2 đối tượng này chưa về 0. (vẫn đang strong qua lại)

Phương án giải quyết như sau

![Kipalog](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/0f3a1ba4a040045d.png_w7vzy9u63n)

Liên kết giống như trên hay bắt gặp khi sử dụng delegate. Vậy nên ta hay dùng weak đối với delegate. Nếu các bạn tạo file xib và kéo thả giao diện vào trong class thì tất cả các đối tượng đều đã có owner trong file xib. Đấy là lý do Properties trong class chỉ liên kết weak với đối tượng, tránh xảy ra hiện tượng nhiều liên kết strong.

# Assign
Dùng cho các biến nguyên thủy như NSInteger, CGFloat, CGPoint, CGRect...

# Copy
Copy được sử dụng đối với những đối tượng Mutable như NSMutableArray, NSMutableDictionary...hoặc Immutable. Giả sử ta có đối tượng A, ta muốn tạo ra đối tượng B giống hệt A rồi thay đổi B mà không làm ảnh hưởng đến A thì lúc đó copy phát huy tác dụng.

NSString có thể sử dụng thuộc tính strong nhưng nhiều lập trình viên sử dụng copy hơn. Ta có đoạn code như sau

```
NSMutableString *pStrMutable = [[NSMutableString alloc] initWithCapacity:100];
[pStrMutable setString:@"HelloWorld"];
NSString *pStrImmutable = pStrMutable; // helloWorld
[pStrMutable setString:@"another text"];
NSLog(@"%@", pStrImmutable); // another text
```

NSMutableString *có thể làm NSString* trỏ đến một text khác, làm sai lệch text mà ta mong muốn. Ta nên sử dụng copy với những đối tượng có cả Immutable và mutable như String, Dictionary.

Ngoài ra copy còn sử dụng với block ^{}. Block cũng là một đối tượng trong obj-c nhưng có một điều đặc biệt, đối tượng này lưu ở stack. Như giải thích ở phần stack thì ta sẽ không quản lý được lifetime của đối tượng lưu trong stack mà việc này diễn ra tự động mỗi khi hàm return hay ra khỏi scope. Thuộc tính copy để tạo ra 1 bản...copy của block, lưu ở heap. Lúc này ARC sẽ quản lý việc giải phóng bộ nhớ giúp ta.

Một số link hay để tham khảo:
[objc.io](http://www.objc.io/issue-7/value-objects.html)