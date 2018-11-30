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

Điều chỉnh tên phần mềm theo phiên bản build









