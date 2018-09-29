---
title: "Style view"
date: 2018-09-30T01:27:11+07:00
lastmod: 2018-09-30T01:27:11+07:00
draft: false
keywords: []
description: ""
slug: "style-view"
tags: ["swift"]
categories: []
author: ""

---

## Một phút cho quảng cáo

Trong lập trình IOS nói riêng và mobile nói chung, kỹ năng code giao diện là phần rất quan trọng. Độ phức tạp của một task và thời gian thực hiện phụ thuộc nhiều vào UI bạn làm. UI đơn giản thì làm nhanh, UI khó thì làm lâu. Kĩ năng tạo giao diện cần được các thợ code mobile trau dồi, rèn luyện thật kĩ. Nhất là đối với những người lười như mình :)), việc build những style dùng chung sẽ giảm thời gian làm xuống, tăng thời gian chơi lên. YOLO mà. Nhưng làm thế nào để dùng chung đâu phải là việc đơn giản. Người thợ code lại phải lên đường tìm cách gỡ rối tơ lòng.

## Tìm đường cách mệnh

Cách hiệu quả nhất là ta tạo những func dùng chung, ví dụ như muốn làm đậm chữ một label, sau đó cho background màu xanh, rùi cuối cùng là bo cái label đấy nó hơi cong chút xíu. Ta có thể viết mã giả như sau

```
setBold(lbl)
setBlackBackground(lbl)
setRound(lbl)

styleLabel(lbl) {
  setBold(lbl)
  setBlackBackground(lbl)
  setRound(lbl)
}
```

Cách giải quyết rất đơn giản phải không nào. Tuy nhiên bạn đang dùng swift mà, hãy làm nó theo phong cách swift.

## Show me the code

Đầu tiên ta định nghĩa ra một struct làm chức năng wrap style. Trong struct có chứa hàm callback thực hiện thao tác với UI

![img1](/hugosite/images/note/style-ui/style1.png)

Tiếp theo định nghĩa tiếp hàm compose, compose này làm nhiệp vụ tổng hợp lại các thao tác với UI thành một hàm duy nhất có dạng **(T) -> Void**. Trong đó T đóng vai trò UI được truyền vào.

![img2](/hugosite/images/note/style-ui/style2.png)

Ta đã có *function* để tổng hợp lại các bước thao tác với UI, bước tiếp theo là **run** nó thôi. Ta định nghĩa một **function** làm việc này.

![img3](/hugosite/images/note/style-ui/style3.png)

OK! done, giờ ta sẽ tạo example đơn giản để thấy hiệu quả của việc này. Ta tạo 3 style từ struct định nghĩa ở trên làm các nhiệm vụ: tạo đường cong cho label, đổi background của label thành màu đen, cho chữ được bôi đậm nữa.

![img4](/hugosite/images/note/style-ui/style4.png)


Rồi tổng hợp chúng lại như sau

![img5](/hugosite/images/note/style-ui/style5.png)

## Cách overload toán tử

Chúng ta có thể overload toán tử để nhìn phép toán gọn gàng, sáng sủa hơn.

![img6](/hugosite/images/note/style-ui/style6.png)

Và kết quả sẽ được là

```
let compositeStyle2 = borderStyle + blackBgStyle + titleBoldStyle
compositeStyle2.apply(to: lbl2)
```

:D Đơn giản như đan rổ ấy nhỉ?

## Hướng functional

Theo huớng này thì ta định nghĩa ra toán tử **<>**

![img7](/hugosite/images/note/style-ui/style7.png)

Và hàm **+** ở trên có thể sửa lại thành


```
static func + (lhs: StyleComposite, rhs: StyleComposite) -> StyleComposite {
    return StyleComposite(styling: lhs.styling <> rhs.styling)
}
```

Thậm chí ta có thể không cần dùng struct nữa mà qui thành **function** hết. Mã giả như sau


```
func setBold(lbl) { ... }
func setRound(lbl) { ... }
func setBlackBg(lbl) { ... }

let style = setBold <> setRound <> setBlackBg

// in your UI code
style(lbl) // run style
```

Vậy là mình đã trình bày xong 3 cách để tái sử dụng code trong việc style UI :). Bạn có cách nào hay không? Show me the code.
