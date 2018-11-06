---
title: "Toán tử 3 ngôi  hay sugar syntax"
date: 2018-11-06T09:36:44+07:00
lastmod: 2018-11-06T09:36:44+07:00
draft: false
keywords: []
description: ""
tags: ["swift"]
categories: []
author: ""

---

## Tình huống

Hôm nay có chút buồn ngủ nên mình tạo động lực cho bản thân bằng cách tự làm thử toán tử ba ngôi trong swift. Đôi khi chơi theo kiểu hardcore cũng mang lại liều doping khá mạnh, giống như phê thuốc vầy :D.

Bình thường mình không dùng **autoclosure** vì thấy nó cũng không có gì đặc sắc, chỉ là **syntax sugar**. Áp dụng đúng chỗ thì ngon, áp dụng không đúng chỗ thì càng làm code khó hiểu. Đâu phải cứ ngắn là hay. Tuy nhiên ở tình huống này, không dùng **sugar syntax** thì không được. t.t

## Code

Bình thường toán tử ba ngôi sẽ viết kiểu này

```
condition ? trueValue : failureValue
```

Ở trường hợp đơn giản nhất, điều kiện và các giá trị trong toán tử ba ngôi sẽ là value type. Tuy nhiên đời mà, mong muốn của bạn đâu chỉ dừng ở đó.

Thế nên đôi khi nó thành như này

```
<function return Bool> ? <function return trueValue> : <function return failureValue> 
```

Làm thế nào để có thể đáp ứng được cả 2 trường hợp trên chỉ với 1 cách duy nhất, swift có **autoclosure**. Xong vấn đề đầu tiên.

Vấn đề thứ hai, ở đây chúng ta có **2 toán tử**  **"?"** và **":"** như vậy ta cần custom 2 toán tử mới và kết hợp chúng để có thể tạo hiệu ứng tương tự toán tử ba ngôi.

Ok, phân tích bài toán xong, đến giờ show code

```
precedencegroup MyBoolApplication {
    associativity: left
    higherThan: MyConditionApplication
}
infix operator <||>: MyBoolApplication
func <||><T>(lhs: @escaping @autoclosure ()-> T, rhs: @escaping @autoclosure () -> T) -> (Bool) -> T {
    return { condition in
        if condition {
            return lhs()
        } else {
            return rhs()
        }
    }
}

precedencegroup MyConditionApplication {
    associativity: left
}
infix operator <??> :MyConditionApplication
func <??><T>(lsh: @autoclosure () -> Bool, rhs: (Bool) -> T) -> T {
    return rhs(lsh())
}

let x = true <??> "true" <||> "false"
```

## Tác dụng phụ

Bài này có tác dụng **for fun** là chính chứ toán tử ba ngôi đã có sẵn trong bản thân ngôn ngữ rồi. Tuy nhiên, nếu nó có ích cho mọi người thì mình cũng rất vui :D.

Have fun!