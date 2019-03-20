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

# Generic

Trước hết tách những trường mặc định ra 1 struct riêng

```swift
struct ResponseInfo {
    let timestamp: String
    let error: Error?
}

struct Response {
    let info: ResponseInfo
    let payload: Any
}
```

Tùy chỉnh lại một chút

```swift
struct Response<T> {
    let info: ResponseInfo
    let payload: <T>
}
```

Như vậy ứng với dữ liệu trả về khác nhau, ta có thể viết như sau

```swift
extension Response where T == User {
    init(info: ResponseInfo, user: User) {
        self.info = info
        self.payload = user
    }
}

extension Response where T == Book {
    init(info: ResponseInfo, book: Book) {
        self.info = info
        self.payload = book
    }
}

let bookResponse = Response(info: info, book: book)
let userResponse = Response(info: info, user: user)
```

Dùng *generic* khi gọi đến property payload ta không phải ép kiểu nữa.

```swift
bookResponse.payload // Book
userResponse.payload // User
```

# protocol

```swift
// 1. define protocol
protocol ResponseType {}

// 2. inherited ResponseType protocol
struct User: ResponseType
struct Book: ResponseType

struct Response {
    let info: ResponseInfo
    let payload: ResponseType
}

extension Response {
    init(info: ResponseInfo, user: User) {
        self.info = info
        self.payload = user
    }
}
```

Dùng *protocol* có vẻ phức tạp hơn dùng *generic* khá nhiều. Chúng ta phải gán các payload kế thừa protocol *ResponseType*. Hơn nữa khi sử dụng payload cũng vẫn phải ép kiểu. Như vậy cực chẳng đã, thà dùng *Any* còn đơn giản hơn :v.

```swift
let result = v.payload as! User
```

# Gán thêm phương thức vào Response

Bây giờ mình muốn thêm phương thức vào trong response để tiện debug, ứng với mỗi response ta có thêm description.

VD:

Với payload là *User* tương ứng với description "response" + User.description 
với payload là *Book* tương ứng với description "response" + Book.description

Muốn như vậy ta phải implement như sau

```swift
// 1 
struct Response<T>: CustomStringConvertible where T: CustomStringConvertible {
    var description: String {
        return "response: " + payload.description
    }
    
    let info: ResponseInfo
    let payload: T
}

// 2
struct User: CustomStringConvertible {
    var description: String {
      return "{username: \(username), password: \(password)}"
    }
    
    //...
}

//...
```

Vì không thể xác định được T trước khi khởi tạo đối tượng, chúng ta phải gán protocol cho cả Response và T để tạo description cho Response.
Như vậy để add thêm phương thức ứng vào Response với cách sử dụng **generic** là tương đối to tay.




