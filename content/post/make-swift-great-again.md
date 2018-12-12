---
title: "Make Swift Great Again"
date: 2018-12-11T17:39:09+07:00
lastmod: 2018-12-11T17:39:09+07:00
draft: true
keywords: []
description: ""
tags: ["swift"]
categories: []
author: ""

---

# Make swift great again

Dạo này thời tiết Hà Nội trở lạnh, post một bài cho nóng người :D. Lần này mình sẽ trình bày về một số tip để làm code trông swifty hơn.

## defer

Sử dụng **defer** nếu như bạn là người hay quên :D. Tác dụng của **defer** là nó sẽ chạy sau khi hàm return.

```
    var x = 1
    func doDefer(_ value: inout Int) -> Int {
        // effect after return
        defer {
            value += 1
        }
        
        value += 1
        return value
    }
    
    print(doDefer(&x)) // 2
    print(x) // 3
```

Với tác dụng của hàm defer ta có thể tận dụng để giải phóng bộ nhớ, close file hay bất cứ tác vụ nào đòi hỏi 2 bước ràng buộc lúc bắt đầu và kết thúc.

## Lazy

Sử dụng lazy trong trường hợp lấy ra một vài phần tử của mảng (lấy hết có khi phản tác dụng). Như ví dụ sau đây, ta filter như bình thường thì sẽ mất 11 bước tính toán, còn lazy mất 3 bước tính toán (test trên playground).

```
    let arr = [Int](1...10)
    _ = arr.filter{ $0 % 2 == 0 } // 11 times
    let lazy = arr.lazy.filter{ $0 % 2 == 0 } // 3 times
    lazy.first
    lazy.endIndex
```

## Extension

Rất thích hợp dùng với **Notification**, ta tập hợp tất cả các key vào chung 1 file để quản lý.


```
extension Notification.Name {
    typealias Nf = Notification.Name
    static let showMeTheCode = Nf("show me the code")
}

NotificationCenter.default.addObserver(
	###, 
	selector: ###, 
	name: .showMeTheCode, 
	object: nil)
```

Có thể áp dụng tương tượng với **UserDefault**.

```
extension UserDefaults {
    struct Key: ExpressibleByStringLiteral {
        let drawValue: String
        init(stringLiteral value: StringLiteralType) {
            self.drawValue = value
        }
    }
}

extension UserDefaults {
    func value<T>(key: Key, as: T.Type? = nil) -> T? {
        return self.value(forKey: key.drawValue) as? T
    }
    
    func set(value: Any, forKey key: Key) {
        self.set(value, forKey: key.drawValue)
    }
}

extension UserDefaults.Key {
    typealias Key = UserDefaults.Key
    static let viewCounter: Key = "viewCounter" // ExpressibleByStringLiteral power :D
}

// Example
do {
    let usrDefault = UserDefaults.standard
    usrDefault.set(value: "aa", forKey: .viewCounter)
    if let x: String = usrDefault.value(key: .viewCounter) {
        print(x)
    }
}
```

Lưu ý để viết được là

> static let viewCounter: Key = "viewCounter"

Ta phải sử dụng *ExpressibleByStringLiteral*