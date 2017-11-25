+++
author = ""
comments = true
date = "2017-11-25T11:36:01+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "method-dispatch"
tags = ["swift","ios"]
title = "Method dispath trong protocol"

+++

## Method dispatch là gì?

Method dispatch là thuật toán quyết định hàm nào sẽ được chạy bởi compiler. Việc xác định hàm nào được thực thi trong lúc biên dịch là static dispatch, trong lúc runtime là dynamic dispatch. Với mỗi ngôn ngữ, tùy thuộc vào thiết kế mà có sự khác nhau về cách thức vận hành phương thức. Tỉ như, C sử dụng mô hình biên dịch tĩnh, javascript sử dụng mô hình biên dịch động, c++ phức tạp hơn, vừa có biên dịch tĩnh như C, vừa có biên dịch động (vitual function), còn Swift thì sao? 

Nó cũng là sự kết hợp, nhưng khác với C++

>
Swift is another case of a hybrid model: its semantics provide predictability between obviously static (structs, enums, and global funcs) and obviously dynamic (classes, protocols, and closures) constructs


## Tại sao lại nói đến method dispatch?

Theo như thông tin ở trên, protocol sử dụng dynamic dispatch, tuy nhiên kể từ khi protocol extension ra đời, thế giới không còn đơn giản như ta tưởng nữa.
Đối với 1 hàm không được khai báo ở protocol mà được khai báo ở extension thì nó là static dispatch. Trước em dynamic, giờ em uốn éo cả static.

Ta tạo một protocol đơn giản như sau

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

Hàm **sayHi** không được khai báo ở protocol Worker mà chỉ được khai báo trong extension.

Giờ ta tạo một struct kế thừa protocol Worker

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

Mọi thứ vẫn OK, giờ ta sửa một chút.

{{< highlight objc "style=monokai" >}}
let anotherDev: Worker = dev
anotherDev.sayHi() // hi, dev
{{< /highlight >}}

Kết quả khác biệt, dù ta gán *anotherDev* chính bằng *dev*, hàm **sayHi** không được gọi trong struct Dev, mà lại được gọi ở extension.
Thử tiếp, ta đưa 2 đối tượng *dev* và *another dev* đã tạo ở trên vào 1 mảng

{{< highlight objc "style=monokai" >}}
let listDev = [dev, anotherDev]
for persion in listDev {
    print("say hi in loop")
    persion.sayHi() // hi, dev
}
{{< /highlight >}}

Kết quả in ra vẫn là kết quả của phương thức khai báo trong extension. Do protocol extension có chứa hàm **sayHi** không được khai báo ở protocol khởi tạo, thế nên nó là static dispatch. Do static dispatch thực thi trong lúc biên dịch nên độ ưu tiên sẽ cao hơn dynamic dispatch. Điều đó giải thích vì sao khi đưa 2 đối tượng cùng type là *Dev* vào trong *array* nó lại cho ra kết quả như trên.

Ngoài ra việc sử dụng protocol extension còn phát sinh ra thêm 1 bug nữa, mọi người có thể xem chi tiết [tại đây](https://team.goodeggs.com/overriding-swift-protocol-extension-default-implementations-d005a4428bda)

--
Bài viết có [tham khảo](https://medium.com/@leandromperez/protocol-extensions-gotcha-9ef1a42c83b6)



