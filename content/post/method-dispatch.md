+++
author = ""
comments = true
date = "2017-11-25T11:36:01+07:00"
draft = false
image = "images/cover/9.jpg"
menu = ""
share = true
slug = "method-dispatch"
tags = ["swift","ios"]
title = "Method dispath trong protocol"

+++

## Method dispatch là gì?

Method dispatch là thuật toán xác định cách thức vận hành method thông qua compiler. Nhắc đến method dispatch, thường người ta sẽ nói đến 2 kiểu điển hình:

* Static dispatch: xác định hàm được chạy trong quá trình biên dịch.
* Dynamic dispatch: xác định hàm được chạy trong quá trình runtime, cái này thể hiện rõ nhất qua tính đa hình trong OOP.

Với mỗi ngôn ngữ, tùy thuộc vào thiết kế mà có sự khác nhau về cách thức vận hành phương thức. Tỉ như, C sử dụng mô hình biên dịch tĩnh, javascript sử dụng mô hình biên dịch động, c++ phức tạp hơn, vừa có biên dịch tĩnh như C, vừa có biên dịch động (vitual function), còn Swift thì sao? 

Nó cũng là sự kết hợp, nhưng khác với C++

>
Swift is another case of a hybrid model: its semantics provide predictability between obviously static (structs, enums, and global funcs) and obviously dynamic (classes, protocols, and closures) constructs


## Tại sao lại nói đến method dispatch?

Theo như thông tin ở trên, protocol sử dụng *dynamic dispatch*, tuy nhiên kể từ khi *protocol extension* ra đời, thế giới không còn đơn giản như ta tưởng nữa.
Đối với 1 hàm không được khai báo ở protocol mà được khai báo ở extension thì nó là *static dispatch*. Nói theo một cách khác, *protocol extension* khiến protocol trở nên...lưỡng tính. Đó là khởi nguồn của 2 kiểu bug khi sử dụng protocol

* Bug khi khai báo func không được định nghĩa trong original protocol
* Bug khi tạo class kế thừa từ 1 class sử dụng default func trong protocol

## Kể chuyện bằng code

Trăm nghe không bằng một thấy, ta tạo một protocol đơn giản như sau

{{< highlight objc "style=monokai" >}}
protocol Worker {
    var name: String { get }
}

extension Worker {
    // default value
    var name: String {
        return "worker"
    }
    
    // not in original Worker protocol
    func sayHi() {
        print("hi, \(self.name)")
    }
}
{{< /highlight >}}

Hãy chú ý đến hàm **sayHi** không được khai báo ở *protocol* **Worker** mà chỉ được khai báo trong *extension*.

Giờ ta tạo một struct kế thừa *protocol* **Worker**

{{< highlight objc "style=monokai" >}}
struct Dev: Worker {
    var name: String {
        return "dev"
    }
    
    func sayHi() {
        print("hello, \(self.name)")
    }
}

let dev = Dev()
dev.sayHi() // hello, dev
{{< /highlight >}}

Kết quả ra là *"hello, dev"*. Mọi thứ vẫn OK, giờ ta sửa một chút.

{{< highlight objc "style=monokai" >}}
let anotherDev: Worker = dev
anotherDev.sayHi() // hi, dev
{{< /highlight >}}

Tư duy thông thường, ta dự đoán kết quả là *"hello, dev"*, tuy nhiên ma thuật xảy ra, kết quả bất ngờ ra *"hi, dev"*. Dù ta gán *anotherDev* chính bằng *dev*, hàm **sayHi** không được gọi trong struct *Dev*, mà lại được gọi ở extension.
Thử tiếp, ta đưa 2 đối tượng *dev* và *anotherDev* đã tạo ở trên vào 1 mảng

{{< highlight objc "style=monokai" >}}
let listDev = [dev, anotherDev]
for persion in listDev {
    print("say hi in loop")
    persion.sayHi() // hi, dev
}
{{< /highlight >}}

Kết quả in ra vẫn là...*"hi, dev"*. Do protocol extension có chứa hàm **sayHi** không được khai báo ở protocol khởi tạo, thế nên nó là *static dispatch*. Do *static dispatch* thực thi trong lúc biên dịch nên độ ưu tiên sẽ cao hơn *dynamic dispatch*. Điều đó giải thích vì sao khi đưa 2 đối tượng cùng type là *Dev* vào trong *array* nó lại cho ra kết quả như trên.

Ngoài ra việc sử dụng protocol extension còn phát sinh ra thêm 1 bug nữa

{{< highlight objc "style=monokai" >}}
class PHPDev: Worker {
    func sayHi() {
        print("Hiii, \(self.name)")
    }
}

class WebDev: PHPDev {
    var name: String {
        return "WebDev"
    }
}

let phpDev = PHPDev()
phpDev.sayHi() // Hi, worker

let webDev = WebDev()
webDev.sayHi() // Hi, worker
{{< /highlight >}}

Mình sẽ để đây và không nói gì thêm :"), đùa thôi.

Class PHPDev kế thừa protocol **Worker** và sử dụng default name trong extension protocol. Ta tạo tiếp class WebDev, khai báo lại name

{{< highlight objc "style=monokai" >}}
var name: String {
    return "WebDev"
}
{{< /highlight >}}

Khi gọi phương thức **sayHi**, kết quả của đối tượng webDev không phải là *"hiii, WebDev"*. Đây là bug thứ hai dễ mắc phải mà mình nhắc đến đầu bài.
Cách đơn giản nhất để khắc phục là...không sử dụng protocol extension, cách tiếp theo là vẫn implement lại phương thức default của protocol (định nghĩa trong extension).

Toàn bộ ví dụ trong bài [source](https://gist.github.com/836c3a99ad12d5d46713542d5afa32f6.git)

## Kết luận

Protocol extension quả thực rất tiện lợi nhưng nó là con dao 2 lưỡi, cần chú ý khi sử dụng.

--

Tham khảo

[https://medium.com/@leandromperez/protocol-extensions-gotcha-9ef1a42c83b6](https://medium.com/@leandromperez/protocol-extensions-gotcha-9ef1a42c83b6)
[https://www.raizlabs.com/dev/2016/12/swift-method-dispatch/](https://www.raizlabs.com/dev/2016/12/swift-method-dispatch/)


