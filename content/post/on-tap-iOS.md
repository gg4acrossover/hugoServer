---
title: "Ôn tập iOS"
date: 2019-05-20T02:21:43+07:00
lastmod: 2019-05-20T02:21:43+07:00
draft: false
keywords: []
description: "Ôn tập iOS"
tags: ["swift"]
categories: []
author: "VietHQ"
---

# Một số câu hỏi ôn tập iOS

## Begin

### Câu 1

``` swift
struct Question {
  var level: Int = 1
}

var question1 = Question()
var question2 = question1
question2.level = 2
```

Giá trị của question1.level sẽ như nào?
Nếu Question là class thì có gì khác không? Tại sao?

### Câu 2

``` swift
var view1 = UIView()
view1.alpha = 0.5

let view2 = UIView()
view2.alpha = 0.5 // compile error?
```

Dòng code *view2.alpha* có báo lỗi không? tại sao?

### Câu 3

``` swift
var animals = ["fish", "cat", "chicken", "dog"]
animals.sort { (one: String, two: String) -> Bool in
    return one < two
}
print(animals)
```

Bạn đang dùng swift mà, hãy làm đoạn code trên ngắn gọn hơn nữa.

### Câu 4

Ta tạo 2 class như và 2 instance của nó như sau

``` swift
class Book {
    var title: String
    var author: String
    
    init(title: String, author: String) {
        self.title = title
        self.author = author
    }
}

class Box {
    var book: Book?
    
    init(book: Book?) {
        self.book = book
    }
}

var book = Book(title: "to mock a mockingbird", author: "Raymond Smullyan")
let box1 = Box(book: book)
let box2 = Box(book: book)
```

Sau đó ta thay đổi title cuốn sách ở *box2*

``` swift
box2.book?.title = "to mock a mockingbird 2"
```

Đoạn code trên không báo lỗi, tuy nhiên khi kiểm tra lại giá trị *box1.book.title* ta thấy nó cũng thay đổi. Điều gì đã xảy ra và làm thế nào để có thể fix?

### Câu 5

**Optional** là gì?

### Câu 6

Sự khác biệt giữa **class** và **struct**

### Câu 7

**Generic** giải quyết vấn đề gì?

## Intermediate

### Câu 1

Sự khác nhau giữa **nil** và **.none**?

### Câu 2

Tạo class và struct như sau:

``` swift
public class NoteClass {
    private(set) var title: String = "unknow"
    public func setTitle(_ title: String) {
        self.title = title
    }
}

let noteClass = NoteClass()
noteClass.setTitle("Monday")

public struct NoteStruct {
    private(set) var title: String = "unknow"
    public mutating func setTitle(_ title: String) {
        self.title = title
    }
}

let noteStruct = NoteStruct()
noteStruct.setTitle("Monday") // error!
```

Lỗi xảy ra ở dòng code cuối cùng. Hãy giải thích tại sao và cách fix?

### Câu 3

``` swift
var animal = "cat"

let closure = { [animal] in
    print("I love \(animal)")
}

animal = "dog"

closure()
```

Đoạn code trên in ra cái gì? Tại sao?

### Câu 4

Ở objc bạn có thể định nghĩa const:

```objc
const int number = 0;
```

Với swift thì:

```swift
let number = 0
```

Có gì khác nhau giữa 2 cách trên không?

### Câu 5

Sự khác nhau giữa **static function** và **class function**

### Câu 6

Sự khác nhau giữa **weak** và **unowned**

## Advance

Câu 1

``` swift
public final class NoteClass {
    var title: String
    init(_ title: String) {
        self.title = title
    }
}
```

Để tạo đối tượng ta có thể viết

``` swift
let note: NoteClass = NoteClass("write better code")
```

Ta có thể viết như này được không? Làm như nào?

```swift
let note: NoteClass = "write better code"
```

Câu 2

Đoạn code này có lỗi gì?

```swift
struct User {
    // ...
}

func login(user: User?) {
    guard let user = user else {
        print("There is no kitten")
    }
    print(user)
}
```

Có 3 cách để giải quyết lỗi trên, nêu cả 3 cách.

Câu 3

Đoạn code sau bị lỗi, giải thích tại sao?

``` swift
enum List<T> {
    case none
    case node(T, List<T>)
}
```

Làm thế nào để fix?

Tạm dừng ở đây nhỉ? Đáp án mọi người tự mò để ôn tập nhé :D.