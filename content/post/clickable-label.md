---
title: "Clickable label"
date: 2018-11-09T14:29:29+07:00
lastmod: 2018-11-09T14:29:29+07:00
draft: false
keywords: []
description: ""
tags: ["swift"]
categories: []
author: "viethq"

---

Để tạo clickable cho text, ta có thể sử dụng UITextView, đó là cách đơn giản nhất. Ngoài ra ta cũng có thể tận dụng sức mạnh của Textkit :D để áp dụng trực tiếp lên *UILabel* mặc dù chỉ sử dụng một phần rất nhỏ của nó thôi.

Hôm nay lười nên mình show code luôn :), extension của *UILabel* nhé :).

Cách sử dụng đơn giản như sau

```
let text = "By signing up you agree to our Terms & Conditions and show me Privacy Policy"
self.firstTitle.text = text

let termRange = (text as NSString).range(of: "Terms & Conditions")
let privacyRange = (text as NSString).range(of: "Privacy Policy")
self.firstTitle.me_addClickable(to: termRange, privacyRange) { (range) in
    if range.intersection(termRange) != nil {
        print("click terms")
    } else if range.intersection(privacyRange) != nil {
        print("click privacy")
    }
}
```

Bạn định nghĩa đoạn text muốn gắn sự kiện clickable và lấy ra range của nó truyền vào trong hàm **me_addClickable**. Nếu bạn click vào vị trí text được gắn sự kiện thì **range** của nó được trả ra trong **callback** để bạn xử lý.

Ngoài ra đoạn text được gắn range sẽ mang attribute **underlineStyle** để ta xác định được vị trí text **clickable**

[Source code](https://gist.github.com/gg4acrossover/f4dfd4a6e069a3650ff743373e7cce96)


