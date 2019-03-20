---
title: "Làm việc với response"
date: 2019-03-20T16:33:59+07:00
lastmod: 2019-03-20T16:33:59+07:00
draft: true
keywords: ["enum","clean code"]
description: ""
tags: []
categories: ["swift"]
author: ""

---

# Bài toán

Khi nhận response từ server, chúng ta thường thấy json có điểm chung như sau

```swift
struct Response {
    let timestamp: String
    let id: Int
    let payload: Any
}
```

*timestamp* và *id* là những trường mặc định luôn được trả về, chúng có type cố định. Còn type của *payload* lại phụ thuộc vào api bạn call, thế nên mình đang để dưới dạng *Any*.

Tuy nhiên sử dụng type *Any* có nhiều hạn chế: chúng ta phải biết chính xác nó-là-gì để ép kiểu trước khi sử dụng.

Vì vậy chúng ta cần nghĩ ra các phương án khác thay thế *Any*.

## Sử dụng Generic và protocol

Trước hết tách những trường mặc định ra 1 struct riêng

```swift
struct ResponseInfo {
    let timestamp: String
    let error: Error?
}
```

Định nghĩa thêm protocol **ResponseType** để tiện mở rộng cho *payload* về sau. 
Tùy chỉnh lại code cũ một chút, ta có được hình hài mới cho *Response*:

```swift
protocol ResponseType {}

struct Response<T: ResponseType> {
    let info: ResponseInfo
    let payload: T
}
```

Như vậy ứng với dữ liệu trả về khác nhau, ta có thể viết như sau

```swift
let bookResponse = Response(info: info, payload: book)
let userResponse = Response(info: info, payload: user)
//...
```

Dùng *generic* khi gọi đến property payload ta không phải ép kiểu nữa.

```swift
bookResponse.payload // Book
userResponse.payload // User
```

## Gán thêm phương thức vào Response

Bây giờ mình muốn thêm phương thức vào trong response để tiện debug, ứng với mỗi response ta có thêm description.

VD:

Với payload là *User* tương ứng với description "response" + User.description 
với payload là *Book* tương ứng với description "response" + Book.description

Muốn như vậy ta phải implement như sau

```swift
protocol ResponseType: CustomDebugStringConvertible {}

struct User: ResponseType {
    var debugDescription: String {
        return "{username: \(username), age: \(age)}"
    }
    
    // ...
}

extension Response: CustomDebugStringConvertible {
    var debugDescription: String {
        return "response: \(self.payload.debugDescription)"
    }
}

// ...
```

Đoạn code trên cũng khá đơn giản, mình tận dụng lại **CustomDebugStringConvertible** có sẵn để thêm description cho *Response* và cả *payload*.

Việc gán protocol **ResponseType** cho *payload* đảm bảo tất cả đối tượng implement lại protocol này đều phải thêm thuộc tính *debugDescription*.

Nếu ta muốn thêm phương thức khác vào trong *payload*, đơn giản chỉ cần tạo protocol mới và cho **ResponseType** kế thừa

```swift
protocol ResponseType: CustomDebugStringConvertible, Jsonable,... {}
```
Tuy nhiên giờ mình muốn quản lý tất cả *payload* ở một chỗ. Lúc này *Enum* phát huy tác dụng.

## Sử dụng Enum

Ta tạo enum chứa tất cả các type của *payload*. Lúc này Response cũng sẽ chỉnh sửa lại chút, gán **enum type** cho *payload* thay vì *generic*

```swift
enum ResponsePayload {
    case userDetail(User)
    case bookDetail(Book)
    // ...
}

struct Response {
    let info: ResponseInfo
    let payload : ResponsePayload
}

let newUser = Response(info: info, payload: .userDetail(user))
```
Lúc này muốn sử dụng payload, ta cần thêm một công đoạn nữa, chứ không gọi trực tiếp như sử dụng *generic*.

```swift
if case .userDetail(_) = newUser.payload {
    // use payload
} else {
    fatalError()
}
```

Muốn gán thêm phương thức mới cho *payload* cũng làm tương tự như đối với sử dụng *generic*

# Tổng kết

* Sử dụng **generic** code gọn hơn sử dụng **enum**, tuy nhiên nhược điểm là phân tán, khó tập trung vào 1 chỗ để quản lý.

* Sử dụng **enum** code dài hơn chút, nhưng ta gom được tất cả vào 1 chỗ, dễ quản lý hơn.
