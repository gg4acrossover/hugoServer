---
title: "Tạo view từ xib"
date: 2018-11-20T16:50:01+07:00
lastmod: 2018-11-20T16:50:01+07:00
draft: false
keywords: []
description: ""
tags: ["swift"]
categories: []
author: ""

---

## 1. Tạo uiview từ xib

Khi tạo giao diện cho 1 app, chắc hẳn ai cũng gặp trường hợp có các view con xuất hiện đi, xuất hiện lại giữa các màn hình khác nhau. Để tránh việc copy qua lại nhàm chán, ta nên tách view con đó ra thành 1 xib duy nhất và tái sử dụng khi cần thiết.

Đầu tiên ta tạo class kế thừa từ UIView và Xib đi kèm như hình sau.

![img1](/hugosite/images/note/xib/xib-1.png)

Ta gắn class cho file's owner trong xib như hình dưới. Đây là bước bắt buộc nếu ta muốn kéo thả reference outlet vào trong code.

![img2](/hugosite/images/note/xib/xib-2.png)

Tạo **reference outlet** cho view trong Xib (kéo thả vào file tương ứng với xib, ở đây mình kéo thả vào file AvatarView.swift và đặt tên view là contentView).

![img3](/hugosite/images/note/xib/xib-3.png)


Trong file code lúc đó sẽ trông như này

![img4](/hugosite/images/note/xib/xib-4.png)

Ta nhớ để thuộc tính cho size của xib là freeform.

![img5](/hugosite/images/note/xib/xib-5.png) 

Ok các bước cài đặt đã hoàn thành, giờ ta viết thêm 1 số đoạn code cần thiết trước khi bắt đầu sử dụng.

```
// step 2: init
override init(frame: CGRect) {
    super.init(frame: frame)
    self.commonInit()
}

required init?(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
    self.commonInit()
}
```

Ta tạo 2 hàm init tương ứng với 2 cách khởi tạo view, **required init?(coder aDecoder: NSCoder)** sẽ chạy khi ta tạo view từ file giao diện, hàm còn lại dùng khi tạo view từ trong code.

Hàm **commonInit** được sử dụng chung trong cả 2 cách khởi tạo view trên được implement như sau

```
// step 3:
private func commonInit() {
    // load nib first
    Bundle.main.loadNibNamed(String(describing: AvatarView.self), owner: self, options: nil)
    
    // add view from nib
    self.addSubview(self.contentView)
    
    // add constraints
    self.contentView.translatesAutoresizingMaskIntoConstraints = false
    self.contentView.topAnchor.constraint(equalTo: self.topAnchor).isActive = true
    self.contentView.bottomAnchor.constraint(equalTo: self.bottomAnchor).isActive = true
    self.contentView.leadingAnchor.constraint(equalTo: self.leadingAnchor).isActive = true
    self.contentView.trailingAnchor.constraint(equalTo: self.trailingAnchor).isActive = true
}
```

Chắc không phải nói thêm nhiều về đoạn code này :).

## 2. Một class chung nhiều xib

Ta tùy biến một chút để việc reuse code nâng lên 1 tầm mới.

Trong hàm **commonInit** ta thấy giao diện phụ thuộc vào việc ta load xib nào, vậy nên ta có thể cho nhiều xib chung 1 class, miễn là nó có cấu trúc và cách thức xử lý tương tự nhau. Ví dụ như khung chat, ta sử dụng ô bên phải, người chat cùng ô bên trái, cả 2 ô chat giống hệt nhau, khác nhau mỗi căn lề trái hoặc phải. Lúc đó có thể sử dụng 2 xib chung 1 class.

Mình tạo 1 thuộc tính gắn vào code để tùy biến xib nào sẽ được sử dụng kèm với view.

```
@IBInspectable var nibByString: String = kLeft {
    didSet {
        self.commonInit()
    }
}
```

Lúc đó ta chỉ cần sửa lại dòng code sau để tùy biến

```
// xib name hard code
Bundle.main.loadNibNamed(xibName, owner: self, options: nil) 
```

```
// xib name tùy biến, ta tự truyền vào
Bundle.main.loadNibNamed(self.nibByString, owner: self, options: nil) 
```

Done!

[Code ví dụ](https://github.com/gg4acrossover/MEReuseView) cho bạn nào cần.

## 3. Mở rộng:

Ta có thể tái sử dụng **UIViewController** dựa vào **Storyboard**. Về cơ bản nó cũng tương tự như cách ta tạo view với xib, tuy nhiên ở đây ta thay **UIView** bằng **UIViewController**, sử dụng phương thức **addChildViewController** của **UIViewController** để làm việc. Đây là một hướng để mọi người tìm hiểu thêm.






