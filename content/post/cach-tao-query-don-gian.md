+++
author = ""
comments = true
date = "2017-07-20T10:43:13+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "cach-tao-query-don-gian"
tags = ["swift3","ios"]
title = "Cách tạo query đơn giản"

+++

Có nhiều cách viết blog công nghệ hơn là làm bánh hay làm tình. 

Những ngày này Hà Nội mưa liên miên, được cái mát giời, mình lại tức cảnh sinh tình, bỗng dưng thèm viết blog. Chả là, dự án mình đang làm, phía đối tác cung cấp khá nhiều query để call api được thuận tiện. Để tạo một chuỗi query không phải là khó, có nhiều phương pháp thực hiện như tạo string format chẳng hạn, dùng query builder cũng chẳng ai cấm. Dưới đây mình sẽ trình bày cách mình tạo query builder bằng enum và reduce trong swift.

### Enum

Enum dùng khá nhiều trong swift, với việc tạo 1 chuỗi query thì enum là lựa chọn không hề tồi.

Giả sử ta có các query filter, fields, sort, hints,...ta có thể tạo enum với mỗi case là 1 query mà ta mong muốn như sau

{{< highlight objc "style=monokai" >}}
enum ONNodeQuery : String {
    case filterSport = "filter={\"Title\":\"SPORTS\"}"
    case fieldsParentChild = "fields=[\"Title\",\"parent\", \"children\",\"descendants\"]"
    case sortTitle = "sort=[[\"Title\",1]]"
    case hints = "hints=true&hintLevel=%@"
    case filterAncestors = "\"ancestors\":{\"$in\": [ \"%@\" ]}"
}
{{< /highlight >}}

Với case *hint* hoặc *filterAncestors* ta cần truyền param từ bên ngoài theo string format qua *%@*

Mình tạo tiếp enum khác để truyền param vào những query nói trên và 1 struct helper với nhiệm vụ nối các câu lệnh query

{{< highlight objc "style=monokai" >}}
enum ONNodeQueryParam {
    case query(ONNodeQuery)
    case addParam(str : String)
    case addAnd
}

struct ONMakeQuery {
    var toDrawQuery : String = ""
    
    mutating func handler(cmd : ONNodeQueryParam) {
        switch cmd {
        case .query(let query):
            self.toDrawQuery += query.rawValue
        case .addParam(let str):
            self.toDrawQuery = String(format: self.toDrawQuery, str)
        case .addAnd:
            self.toDrawQuery += "&"
        }
    }
}
{{< /highlight >}}

Biến toDrawQuery gán giá trị default để ko phải tạo init method.

Ok, hiện tại mình đã đủ tool để ghép nối các query thành 1 chuỗi

Ta tạo 1 mảng query như sau

{{< highlight objc "style=monokai" >}}
let query : [ONNodeQueryParam] = [.query(.filterSport), .addAnd,
                                  .query(.fieldsParentChild), .addAnd,
                                  .query(.sortTitle), .addAnd,
                                  .query(.hints), .addParam(str: "20")]
{{< /highlight >}}

Cũng khá dễ hiểu: ta cần lấy ra các node sport với 1 số trường mong muốn (child, parent), lấy ra hints với 20 records, nối các query bằng kí tự "&".

Tạo chuỗi query bằng struct như sau

{{< highlight objc "style=monokai" >}}
var makeQuery = ONMakeQuery()

for i in query {
    makeQuery.handler(cmd: i)
}

print(makeQuery.toDrawQuery) // query we need
{{< /highlight >}}

Tuy nhiên giờ functional programming khá là phổ biến, ta có thể dùng cách khác ngắn hơn mà không cần tạo struct, đó là sử dụng reduce

{{< highlight objc "style=monokai" >}}
let toDrawQuery = query.reduce("") { (text, value) -> String in
    switch value {
        case .query(let query):
            return text + query.rawValue
        case .addParam(let str):
            return String(format: text, str)
        case .addAnd:
            return text + "&"
    }
}
{{< /highlight >}}

Khá là đơn giản phải không :D.

Code [ví dụ](https://gist.github.com/gg4acrossover/7d49d0c2c922486e37f622c73808597c)

Tham khảo [Statements, messages and reducers](http://www.cocoawithlove.com/blog/statements-messages-reducers.html)



