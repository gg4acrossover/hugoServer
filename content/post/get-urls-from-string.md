---
title: "Get Urls From String"
date: 2019-07-11T11:08:05+07:00
lastmod: 2019-07-11T11:08:05+07:00
draft: false
keywords: ["swift"]
description: "lấy url từ string"
tags: ["swift"]
categories: ["swift"]
author: "viethq"

---

# Lấy url từ string

Bạn đã bao giờ tự hỏi làm thế nào để lấy ra url từ string ko?
Cách đơn giản hướng đến đầu tiên là split string ra rồi đưa vào hàm *URL.init*

``` swift
var str = "goto https://www.google.com/"
let urls = str.components(separatedBy: .whitespaces)
              .compactMap(URL.init)

// [goto, https://www.google.com/]
```

Kết quả ra cả "goto", có gì đó sai sai?
Hóa ra thằng *URL.init* vốn dễ dãi. Nó nhận cả đường dẫn, tên thư mục nó lấy, url nó lấy, tên folder nó lấy, blah blah.
Vậy là ta nên tìm một cách khác chặt chẽ hơn.

# Sử dụng NSRegularExpression

Để áp dụng cách này, bạn nên biết chút về expression hoặc là sử dụng có sẵn như [ở đây](https://emailregex.com/).
Code triển khai cũng không dài không ngắn

```swift
let pt = "https?:\\/\\/(www\\.)?[-a-zA-Z0-9@:%._\\+~#=]{1,256}\\.[a-zA-Z0-9()]{1,6}\\b([-a-zA-Z0-9()@:%_\\+.~#?&//=]*)"
let regex = try! NSRegularExpression(pattern: pt, options: [])
regex.matches(in: str, options: [], range: NSMakeRange(0, str.count)).map { (range) -> String? in
    // your code
}
```

Closure trả về range của đoạn text thỏa mãn điều kiện bạn đưa ra ban đầu từ chuỗi pattern regex. Từ đó bạn có thể lấy url theo như mình muốn.

# Sử dụng NSDataDetector

*NSDataDetector* có anh em họ hành với *NSRegularExpression*. Thằng này tương đối dễ dùng, chỉ có một nhược điểm duy nhất: nó là hàng ăn sẵn nên không phải services nào bạn muốn nó cũng đáp ứng được như cô chị của nó. Mỗi người một sở thích, dùng hay không là tùy ở mình.

```swift
let detector = try NSDataDetector(types: NSTextCheckingResult.CheckingType.link.rawValue)
            detector.enumerateMatches(in: str, options: [], range: NSMakeRange(0, self.count), using: { (result, _, _) in
                // your code
            })
```

Điểm hay của *NSDataDetector* là nó định nghĩa ra các type để mình sử dụng. Danh sách phong phú, đa dạng để bạn lựa chọn, thế nên cũng không cần biết *expression* làm gì.

``` swift
extension NSTextCheckingResult {

    
    public struct CheckingType : OptionSet {

        public init(rawValue: UInt64)

        
        public static var orthography: NSTextCheckingResult.CheckingType { get }

        public static var spelling: NSTextCheckingResult.CheckingType { get }

        public static var grammar: NSTextCheckingResult.CheckingType { get }

        public static var date: NSTextCheckingResult.CheckingType { get }

        public static var address: NSTextCheckingResult.CheckingType { get }

        public static var link: NSTextCheckingResult.CheckingType { get }

        // ...
    }
```

# Bài học rút ra là gì?

1. Không có cách nào tối ưu nhất, chỉ có cách nào hợp với mình nhất thôi.

2. Như cái title của bài viết.
