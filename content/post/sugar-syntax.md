---
title: "Sugar Syntax"
date: 2019-06-12T16:12:19+07:00
lastmod: 2019-06-12T16:12:19+07:00
draft: false
keywords: ["swift","sugar syntax"]
description: "swift"
tags: ["swift"]
categories: []
author: "VietHQ"

---

Swift có một số phương pháp để code viết gọn hơn, hay còn gọi là sugar syntax. Hôm nay mình sẽ giới thiệu một số phương pháp. Các cách này có thể không mang nhiều tính áp dụng, tuy nhiên sẽ giúp chúng ta hiểu sâu hơn về ngôn ngữ mình đang sử dụng

# dynamicallyCallable

Áp dụng cấu trúc này cho class, struct, enum hoặc protocol. Sử dụng code ví dụ để dễ hình dung

```swift
@dynamicCallable
struct TelephoneExchange {
    func dynamicallyCall(withArguments phoneNumber: [Int]) {
        if phoneNumber == [1,1,3] {
            print("công an")
            return
        }
        
        if phoneNumber == [1,1,5] {
            print("cấp cứu")
            return
        }
        
        if phoneNumber == [1,1,4] {
            print("cứu hỏa")
            return
        }
        
        print("unknow")
    }
}
```

Áp dụng

```swift
let dial = TelephoneExchange()
dial(1,1,3) // công an
```

Chúng ta gắn *keyword* **@dynamicCallable** trước kiểu dữ liệu muốn áp dụng.
Dữ liệu muốn sử dụng **@dynamicCallable** cần implement một trong 2 hàm sau:

``` swift
dynamicallyCall(withArguments:)
dynamicallyCall(withKeywordArguments:) 
```
Với phương thức *dynamicallyCall(withArguments:)* ta cần truyền vào param thỏa mãn protocol *ExpressibleByArrayLiteral*. Trong trường hợp này là [Int]

Ngoài ra chúng ta có thể sử dụng phương thức *dynamicallyCall(withKeywordArguments:)* với param là *KeyValuePairs<T,V>*. Lấy ví dụ trên [docs](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html) của swift luôn

```swift
@dynamicCallable
struct Repeater {
    func dynamicallyCall(
        withKeywordArguments pairs: KeyValuePairs<String, Int>) -> String {
        return pairs
            .map { label, count in
                repeatElement(label, count: count).joined(separator: " ")
            }
            .joined(separator: "\n")
    }
}

let repeatLabels = Repeater()
print(repeatLabels(a: 1, b: 2, c: 3, b: 2, a: 1))
// a
// b b
// c c c
// b b
// a
```

# dynamicMemberLookup

Những type sử dụng *dynamicMemberLookup* cần implement hàm *subscript(dynamicMemberLookup:)*. Dữ liệu params truyền vào của hàm này phải thỏa mãn điều kiện mang một trong các type *KeyPath*, *WritableKeyPath* hay *ReferenceWritableKeyPath*. Ta cũng có thể lấy giá trị thông qua dữ liệu thỏa mãn *ExpressibleByStringLiteral*. Nói chung đa số ta dùng kiểu *String*. 

Giải thích dài dòng không bằng viết code (sử dụng [ví dụ](https://oleb.net/blog/2018/06/dynamic-member-lookup/))

```swift
@dynamicMemberLookup
struct Uppercaser {
    subscript(dynamicMember input: String) -> String {
        return input.uppercased()
    }
}
```

Cách dùng:

```swift
Uppercaser().hello \\ HELLO
```

Ngoài ra áp dụng vào dictionary cũng rất hợp lý (sử dụng [ví dụ](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html))

``` swift
@dynamicMemberLookup
struct DynamicStruct {
    let dictionary = ["someDynamicMember": 325,
                      "someOtherMember": 787]
    subscript(dynamicMember member: String) -> Int {
        return dictionary[member] ?? 1054
    }
}
let s = DynamicStruct()

// Use dynamic member lookup.
let dynamic = s.someDynamicMember
print(dynamic)
// Prints "325"

```

# Cái gì tiếp theo

Năm nay ở WWDC 2019, thêm một keyword mới được giới thiệu, có vẻ tính áp dụng nhiều hơn 2 keywords trên. 

Đó là **propertyWrapper**. Cái này mới chỉ xuất hiện ở XCode11 beta, mọi người có thể tự tìm hiểu cập nhật thêm qua bài viết chi tiết [ở đây](https://github.com/DougGregor/swift-evolution/blob/property-wrappers/proposals/0258-property-wrappers.md)

# Mục tiêu bài này là gì

Bài này nhằm mục đích giới thiệu một số keywords trong swift. Mình cũng chưa sử dụng mấy keywords này trong các dự án đã và đang làm. Tuy nhiên nó sẽ hữu ích nếu bạn gặp một ai đó thích viết kiểu syntax sugar :D.
