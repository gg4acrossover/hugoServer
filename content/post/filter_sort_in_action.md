+++
author = ""
comments = true
date = "2017-07-03T14:53:55+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "filter-sort-in-action"
tags = ["ios","swift 3"]
title = "Filter, sort in action"

+++

### Back to basic

Hồi học phổ thông, tôi rất ấn tượng với một thằng giải toán bằng định nghĩa. Hắn học chuyên toán tự nhiên, ngồi trên tôi một bàn, vậy nên tôi may mắn liếc được bài :D. Cách giải độc đáo ấy thậm chí không có trong bài giải được đăng tải sau khi thi. Tôi cũng khá...tự hào vì học lóm được cách giải đó. Về sau, khi làm lập trình game, tôi cũng may mắn được chỉ cho một công thức vật lý đơn giản để giải quyết vấn đề trong game.

Có thể nói, những kĩ thuật đơn giản, chúng ta hay bỏ quên, đôi khi lại đưa ra một cách giải hay, độc đáo. Hôm nay, tôi sẽ "back to basic" để xem lại những hướng giải quyết việc **filter** và **sort** trong Array. Tôi sẽ dùng swift để giải quyết vấn đề

### Filter

Giả sử ta có một mảng

{{< highlight objc "style=monokai" >}}
let listStr = ["quoc anh", "Hoang hai", "hoàng giang","anh ngoc", "quoc nguyen"]
{{< /highlight >}}

Ta cần lọc ra những phần tử có chứa từ "hoang". Ta có thể sử dụng những cách sau.

1. Sử dụng *NSPredicate*

Trường hợp ta không cần phân biệt chữ hoa chữ thường ta có thể làm như sau

{{< highlight objc "style=monokai" >}}
let filterC = NSPredicate(format: "self contains[c] %@", "hoang")
let arrFilterC = (listStr as NSArray).filtered(using: filterC) // ["Hoang hai"]
{{< /highlight >}}

Trường hợp ta không cần phân biệt dấu

{{< highlight objc "style=monokai" >}}
let filterD = NSPredicate(format: "self contains[d] %@", "hoang")
let arrFilterD = (listStr as NSArray).filtered(using: filterD) // ["hoàng giang"]
{{< /highlight >}}

Nếu không cần phân biệt cả viết hoa, thường, cả dấu, ta có thể kết hợp điều kiện

{{< highlight objc "style=monokai" >}}
let filterD = NSPredicate(format: "self contains[cd] %@", "hoang")
{{< /highlight >}}

Về query format của NSPredicate, mọi người tham khảo thêm tại [đây](http://nshipster.com/nspredicate/).

2. Sử dụng hàm filter của swift

Cách trên là kế thừa di sản từ ObjC, bây giờ chúng ta sử dụng cách hiện đại hơn mà nhìn cũng gọn hơn

Không cần phân biệt chữ hoa, chữ thường

{{< highlight objc "style=monokai" >}}
let filterStrC = listStr.filter { $0.range(of: "hoang", options: .caseInsensitive, range: nil, locale: nil) != nil }
{{< /highlight >}}

Ngắn hơn có thể dùng cách sau

{{< highlight objc "style=monokai" >}}
let arrFilter = listStr.filter { $0.localizedCaseInsensitiveContains("hoang")}
{{< /highlight >}}

Không cần phân biệt dấu

{{< highlight objc "style=monokai" >}}
let filterStrD = listStr.filter { $0.range(of: "hoang", options: .diacriticInsensitive, range: nil, locale: nil) != nil }
{{< /highlight >}}

### Sort (theo Alphabet)

1. Sử dụng *ComparisonResult*

Trường hợp không phân biệt chữ hoa, chữ thường (Theo mã ascii)

{{< highlight objc "style=monokai" >}}
let arrPre = (listStr as NSArray).sortedArray(comparator: { (s1 ,s2) -> ComparisonResult in
    if let x = s1 as? String, let y = s2 as? String {
        return y.localizedCaseInsensitiveCompare(x)
    }
    
    return ComparisonResult.orderedAscending
})
{{< /highlight >}}

Kết quả nhận được như sau

> ["quoc nguyen", "quoc anh", "Hoang hai", "hoàng giang", "anh ngoc"]

2. Sử dụng *NSSortDescriptor*

{{< highlight objc "style=monokai" >}}
let sortDes = NSSortDescriptor(key: nil, ascending: false, selector: #selector(NSString.localizedCaseInsensitiveCompare))
let arrPre2 = (listStr as NSArray).sortedArray(using: [sortDes])
{{< /highlight >}}

Kết quả cũng ra tương tự như trên

3. Sử dụng hàm *sorted* của swift

{{< highlight objc "style=monokai" >}}
let sortSimple = listStr.sorted {
    $1.localizedCaseInsensitiveCompare($0) == .orderedAscending
}
{{< /highlight >}}

### Kết

Tóm lại bằng một câu mà ai cũng biết :D
> Keep it simple
