---
title: "Config Project"
date: 2018-11-30T14:33:34+07:00
lastmod: 2018-11-30T14:33:34+07:00
draft: true
keywords: []
description: ""
tags: ["swift"]
categories: []
author: ""

---

Hồi mới tiếp xúc với lập trình IOS, mỗi khi build sản phẩm để công ty test, mình lại phải điều chỉnh một số configuration ngay chính trong code. Lúc config thế này, lúc config thế kia, tùy chỉnh cho nó hợp với môi trường theo cách **#define** như này

```
#ifdef DEBUG
#define BASE_URL ...
#else
#define BASE_URL ...
#endif
```

hoặc kiểu comment như này

```
// Product
// static NSString* kBaseURL ...

// Test
static NSString* kBaseURL ...
```

Tuy nhiên những cách đó có rất nhiều hạn chế:

+ Mỗi lần build mình phải đổi flag hay xóa comment

+ Bản production bị ghi đè bởi bản test

Những tùy chỉnh như vậy dễ tạo ra sai sót. Về sau, mình tìm ra một cách để cải thiện, vừa không ghi đè bản test lên production, app cho test và app cho production có tên và icon khác nhau, thuận tiện cho việc phân biệt :d.

## Sử dụng nhiều Schemes

### B1:

Ta chọn manage scheme như hình sau

![img-1]()

Duplicate scheme có sẵn và đổi tên như trong hình, bạn có thể tùy biến thêm scheme phụ thuộc vào bao nhiêu môi trường bạn muốn tạo. Ở đây mình chỉ có *Development* và *Production*

![img-2]()

Có thể điều chỉnh lại build configuration cho hợp với scheme đang sử dụng. 

VD: *Development scheme* đi với *Debug build configuration*, hoặc *Production scheme* đi với *Release build configuration*

![img-3]()

### B2:

Vào **Build Settings** của **Targets**, điều chỉnh tên phần mềm theo phiên bản build.

![img-4]()

### B3:

Vẫn ở **Build Settings**, thay đổi **product bundle identifier** theo ý bạn muốn.

Lưu ý: khi 1 phần mềm có nhiều *bundle identifier* cũng đồng nghĩa với việc bạn phải tạo từng ấy provisioning. Vậy nên nếu mục đích của bạn không cần tạo nhiều môi trường khác nhau thì có thể bỏ qua bước này.

![img-5]()


### B4:

Bước này khá thú vị, ta thay đổi ảnh AppIcon cho từng phiên bản để dễ phân biệt.

Chọn **Asset Category App Icon Set Name** thay đổi như hình sau.

Bản Beta và dev sử dụng chung *AppIcon-Test*, đây là tên icon do mình tự custom, hãy nhớ tên của nó để mình dùng ngay sau đây.

Bản Release sử dụng AppIcon, đây là giá trị default của icon trong **Asset.xcassets**

![img-6]()

### B5:

Ta vào **Asset.xcassets** để thêm icon cho app, đồng thời add thêm 1 loại icon khác dành riêng cho bản dev.

![img-7]()

Qua 5 bước kể trên ta đã có thể tạo ra các phiên bản khác nhau cho cùng 1 app: 1 bản cho dev, 1 bản cho production, mỗi phiên bản 1 tên và icon riêng biệt.










