---
title: "Tiếng Việt có dấu"
date: 2020-05-11T02:09:09+07:00
lastmod: 2020-05-11T02:09:09+07:00
draft: false
keywords: ["swift","ascii","unicode"]
description: ""
tags: ["swift","unicode"]
categories: ["swift"]
author: "VietHQ"
---

Dear all,

Lâu lắm mình mới có hứng thò mặt ra ngoài giang hồ :D. Lý do thì cực kì đơn giản, mình hứng thú với bàn toán xử lý **String** khi mở thẻ ngân hàng trên app.

Trong lúc thao tác mở thẻ, phần nhập *tên ghi trên thẻ* khiến mình đặc biệt chú ý vì yêu cầu nghiệp vụ của nó khá là thú vị. Ô text field không cho nhập tiếng Việt có dấu, ngay cả khi mình cố tình copy paste vào nó cũng tách các kí tự có dấu ra, vd: **â** thành **aa**, **ơ** thành **ow**

Đây cũng là một bài toán hay về **String** nên mình cũng xắn tay áo làm thử một cái xem sao.

## Hướng đi

Máy tính vốn được tạo thành từ những bảng mạch, nó chỉ hiểu 2 số 0 và 1 nhưng lại có thể biểu diễn rất nhiều thứ, nó còn có thể trình diễn hình ảnh, chơi nhạc, bật video. Quá là vi diệu đối với những gì xuất phát từ 2 con số 0 và 1, và càng hay hơn nữa khi những con số này tạo ý tưởng cho mình giải quyết vấn đề.

Việc đầu tiên mình nghĩ đến là qui các kí tự ra mã như mã ascii như hồi mới bỡ ngỡ lập trình. Hồi đó chắc hẳn ai cũng làm bài tập biến hoa thành chữ thường bằng cách qui đổi thành những con số (mã ascii) rồi cộng nó lên với 32. Nếu bạn nào quên thì mình gợi nhớ lại A-Z tương ứng với 65-90 và a-z tương ứng với 97-122.

Tuy nhiên bảng mã ascii giới hạn từ 0 - 127, thế nên chúng ta sẽ sử dụng thằng khác to hơn mang tên **unicode**

Với bảng mã này ta có đầy đủ tất cả các kí tự ta cần, từ chữ không dấu đến chữ có dấu, không thiếu cái gì cả. Tuy nhiên làm thế nào để tách **â** thành **aa** mới là vấn đề vì mã của 2 thằng này khác nhau, không lẽ liệt kê từng trường hợp một. Mình không khoái cách này lắm vì nó quá dài và phức tạp, chẳng may lọt mất một trường hợp thì sao nhỉ?

Thật may vì bảng mã unicode được xây dựng trên một qui chuẩn nhất định. Có 2 kiểu form cơ bản trên qui chuẩn này là **Canonical** và **Compatible**. Cả 2 form này đều có thể biểu diễn dưới 2 dạng **composed** và **decomposed**. Các bạn có thể Google để tự tìm hiểu thêm :D.

Đến đây thì bạn hiểu rồi chứ, **â** là **composed** còn **aa** là **decomposed**. Khi hiểu nguyên lý này thì tách text không còn phức tạp nữa, khi ta tách được chữ và dấu thì việc còn lại khá đơn giản. Ta chuyển dấu sang các kí tự liên quan tương ứng, vd: dấu sắc -> s, nặng -> j, ngã -> x, ...

Riêng với các dấu như **^** ta cần xử lý phực tạp hơn vì cần biết kí tự đằng trước nó là gì để chuyển đổi cho phù hợp.

## Viết code

Đầu tiên là convert từ chữ sang số

``` swift
let testStr = "em về tóc xanh"

let test = testStr.decomposedStringWithCanonicalMapping.unicodeScalars
```

Khi bạn chạy đoạn code trên thì nó ra một mảng số như này:

> [101, 109, 32, 118, 101, 101, 102, 32, 116, 111, 115, 99, 32, 120, 97, 110, 104]

Mảng số này có độ dài lớn hơn chữ nhập vào ban đầu. Lý do bạn có thể dễ dàng đoán ra, các kí tự đại diện cho dấu đã bị tách ra, được biểu diễn dưới dạng mã unicode.

Mình có liệt kê các dấu trong 1 enum sau đây:

``` swift
enum DiacriticsVietnamese: UInt32 {
    case dot = 803 // nang
    case acute = 769 // sac
    case graveAccent = 768 // huyen
    case question = 777 // hoi
    case tilde = 771 // nga
    case composeAW = 774
    case composeAA = 770
    case composeOW = 795

    func toString(with decomposeValue: UInt32) -> String {
        switch self {
        case .dot:
            return "j"
        case .acute:
            return "s"
        case .graveAccent:
            return "f"
        case .question:
            return "r"
        case .tilde:
            return "x"
        case .composeAA:
            if let v = UnicodeScalar(decomposeValue) {
                return String(v)
            }
            
            return " "
        case .composeAW, .composeOW:
            return "w"
        }
    }
}
```

Công việc tiếp đến cực kì đơn giản, chỉ cần thay thế các kí tự dấu bằng chữ cái tương ứng.

```swift
let cab: [Unicode.Scalar] = test.enumerated().compactMap{ idx, unicodeScalar in
    if let currentValue = DiacriticsVietnamese(rawValue: unicodeScalar.value) {
        let preIdx = test.index(test.startIndex, offsetBy: idx-1)
        return currentValue.toString(with: test[preIdx].value).unicodeScalars.first!
    } else {
        return unicodeScalar
    }
}

let toString = String(String.UnicodeScalarView(cab))

```
## Lưu ý

Đoạn code trên chưa xử lý chữ Đ tách thành dd, tuy nhiên việc xử lý cái này cũng đơn giản, các bạn tự xử lý thêm.