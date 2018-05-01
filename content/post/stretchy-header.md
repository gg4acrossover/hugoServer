+++
author = ""
comments = true	# set false to hide Disqus
date = "2018-04-28T01:44:57+07:00"
draft = false
image = "images/cover/3.jpg"
menu = ""
share = true
slug = "stretchy-header"
tags = ["swift4","ios"]
title = "Stretchy header"
+++

### 1.Tản mạn

Tạo stretchy header là kỹ thuật không mới nhưng cho hiệu ứng khá đẹp mắt, hiện nay vẫn được nhiều app ưa chuộng dùng lại. Trước đây, hồi còn chưa ra swift, thấy trò này hay quá, mình cũng mày mò tự tạo một cái, tuy nhiên chưa có cơ hội áp dụng vào dự án thực tế. Ở công ty hiện tại, mình được chủ động hơn trong công việc, nên cũng tùy tiện gắn stretchy header vào cho có chút phá cách. Ngoài ra mình cũng thấy một bạn post hỏi cách tạo stretchy header trên cộng đồng IOS, nên tiện lúc ngứa tay, mình post lên dây chia sẻ *mình đã tạo stretchy header ra sao*.

### 2. Hướng đi

Mình không sử dụng *header* mặc định của *UITableView*. Thế nên mình triển khai theo cách tạo một khoảng trống phía trên UITableView(UIScrollView nói chung), khoảng trống này đủ rộng để hiển thị *header* mình tạo. Sau đó điều chỉnh *frame* của *header* cập nhật theo sự kiện *scroll*. Có nhiều cách để cập nhật lại frame của header ví dụ như sử dụng hàm *scrollViewDidScroll* trong *UIScrollViewDelegate*, hoặc sử dụng *KVO* để bắt sự thay đổi của thuộc tính contentOffset. Trong ví dụ lần này mình sử dụng cả 2 cách trên.

### 2. Cách đơn giản nhất

Viết code chú trọng giản tiện, code càng ít bug càng nhỏ. Như hướng đi trên, mình phân ra làm 3 bước quan trọng:

  - Bước 1: Tạo content inset top cho tableview, mục đích là tạo ra 1 khoảng trống để gắn header.

  ![image2](/hugosite/images/note/stretchy-header-1.png)

  - Bước 2: Gắn 1 custom view vào đúng vị trí trống do bước 1 hình thành. Nếu view bạn tạo ra là *UIImageView* thì nên để thuộc tính content là *scaleAspectFill* (scale ảnh giữ đúng tỉ lệ). Đối với trường hợp này, gắn trực tiếp vào viewcontroller, chứ không phải *tableview*

  ![image2](/hugosite/images/note/stretchy-header-2.png)

  - Bước 3: Co giãn frame dựa vào hàm *scrollViewDidScroll*, nhìn code thì dài nhưng thực ra chỉ tính toán lại frame.

  ![image2](/hugosite/images/note/stretchy-header-3.png)

### 3. Cách tạo trường hợp tổng quát

Để tạo trường hợp tổng quát thì mình gắn header vào trong *scrollview(tableview)(. Đây là cách mà các thư viện hay sử dụng. Để làm được theo cách này ta cần phải biết thêm về KVO, biết cách thêm property vào extension của các class có sẵn trong *Cocoa*. Nghe thôi cũng đã thêm nhiều thứ rồi nhỉ? Các bước tiến hành cũng na ná như cách đầu tiên, tuy nhiên có lưu ý nho nhỏ: 

  - Vị trí originY của stretchy header lúc này thường là *contentOffset.y* của scrollview.
  - Vì gắn custom view vào scroll view nên ta cần điều chỉnh frame của header theo sự thay đổi của *contentOffset.y*. Sử dụng KVO trong trường hợp này là hợp lý.

### 4. Gắn vào trong Tableview

Cách này tương tự cách làm tổng quát, tuy nhiên không cần dùng đến KVO, không cần đến extension, thế nên việc xử lý đơn giản đi khá nhiều. Nhược điểm của nó là khi cần áp dụng vào tableview khác thì Ctrl Only C từ chỗ này qua chỗ khác :D, nói chung là cũng nhanh. Vậy nên mọi người dựa vào 2 cách trên để tự làm cách này nhé. Good luck!

[Source code](https://github.com/gg4acrossover/stretchy-header) để mọi người tham khảo.

Dưới đây là thành quả cho có tí động lực

![image2](https://media.giphy.com/media/5tkNzzBCp7GCyLAIYT/giphy.gif)







