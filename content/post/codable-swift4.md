+++
author = "crossover"
comments = true
date = "2017-11-18T10:10:10+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "codable-swift4"
tags = ["swift","ios"]
title = "Codable swift 4"

+++

## Codable là gì?

Chắc hẳn ở swift 3 đa số chúng ta sử dụng lib *ObjectMapper* để parse json thành model. Thư viện này khá tiện dụng, tuy nhiên có một nhược điểm là chúng ta vẫn phải viết hàm map key. Điều này mình không thích lắm, trước đây objc có *JSONModel* parse thông minh hơn hẳn, ai dùng qua chắc đều biết. Đáng tiếc là ở swift không có thằng nào như vầy cả.

Tuy nhiên điều đó đã thay đổi khi swift 4 xuất hiện đi kèm với **Codable**. Vậy **Codable** là gì?

Theo như Apple quảng cáo, **Codable** giúp ta được 2 việc:

* Encodable dùng để encoding
* Decodable dùng để decoding

> /// A type that can convert itself into and out of an external representation.

> public typealias Codable = Decodable & Encodable

Về bản chất **Codable** là một protocol kết hợp giữa 2 protocol *Encodable* và *Decodable*, mỗi protocol lại đảm nhận chức năng như nói ở trên. Có thể coi *Codable* tương đương với một chiêu hai thức :D

## Decodable như thế nào?

Giả sử ta có một json string như sau

{{< highlight objc "style=monokai" >}}
let json = """
[{
    "name":"cross",
    "age":10,
    "birthDay":"2017-06-21 20:20+0000"
}]
"""
{{< /highlight >}}

Chúng ta sẽ định nghĩa struct kế thừa *Codable*

{{< highlight objc "style=monokai" >}}
struct User : Codable {
    var name: String?
    var age: Int = 0
    var birthDay: Date
}
{{< /highlight >}}

Giờ là lúc thức thứ nhất *Decodable* thể hiện sức mạnh

{{< highlight objc "style=monokai" >}}
// parse json with JSONDecoder
let decoder = JSONDecoder()

let dateFormater = DateFormatter()
dateFormater.timeZone = TimeZone.current
dateFormater.dateFormat = "yyyy-MM-dd HH:mmZ"
decoder.dateDecodingStrategy = .formatted(dateFormater)

if let data = json.data(using: .utf8) {
    if let items = try? decoder.decode([User].self, from: data),
        let date = items.first?.birthDay {
        print(dateFormater.string(from: date))
        print(items.first?.username ?? "something wrong")
    } else {
        print("error")
    }
}
{{< /highlight >}}

Ở đây mình cố tình cho biến date vào trong json string, nên parse hơi dài (có thêm bước format data), tuy nhiên về cơ bản chỉ cần 2 dòng code tương ứng 2 bước:

* Convert json thành data
* tạo đối tượng JSONDecoder để thực hiện decode

{{< highlight objc "style=monokai" >}}
// step 1
if let data = json.data(using: .utf8) {
	// step 2
	let items = try? JSONDecoder().decode([User].self, from: data)
	...
}
{{< /highlight >}}

Vì công thức chỉ có 2 bước, tương ứng với mọi đối tượng, thế nên mình có thể tạo Generic cho việc decoding như sau:

{{< highlight objc "style=monokai" >}}
typealias DecoderConfig = (JSONDecoder) -> ()
func parseDecodableObject<T: Decodable>(data: Data,config: DecoderConfig?) throws -> T {
    let decoder = JSONDecoder()
    config?(decoder)
    let injectObject = try decoder.decode(T.self, from: data)
    return injectObject
}
{{< /highlight >}}

Tương ứng với code sẽ rút ngắn đi như thế này

{{< highlight objc "style=monokai" >}}
let item: [User] = try parseDecodableObject(data: data, config: config)
{{< /highlight >}}

Đôi khi, mình muốn đặt tên thuộc tính theo ý mình, không theo key trên server trả về, có được không nhỉ? *Decodable* bất chấp hết nhưng cần thêm 1 bước:

Giả sử ở **struct User** đầu ta đổi property name thành...username, ta sẽ phải thêm 1 enum *CodingKeys* kế thừa *CodingKey* vào trong struct User

![image1](/hugosite/images/note/decodable.png)

Có 2 điều bắt buộc đối với enum kế thừa *CodingKey*: 

* Phải có type là *String*.
* Các case phải trùng với tên thuộc tính của đối tượng. Với những thuộc tính cần mapping thì ta gán giá trị, vd: username gán giá trị là "name"(trùng với json key).


## Encoding như thế nào?

Thức thứ hai trong *Codable* là *Encodable*, về cơ bản nó ngược lại với *Decodable*, thế nên mình chỉ show code thôi.

{{< highlight objc "style=monokai" >}}
var user: User?
if let date = dateFormater.date(from: "2017-06-21 20:20+0000") {
    user = User(username: "cross", age: 10, birthDay: date)
}

let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .formatted(dateFormater)

do {
    let userData = try encoder.encode(user)
    let userJson = try JSONSerialization.jsonObject(with: userData, options: [])
    print(userJson)
} catch {
    print(error)
}
{{< /highlight >}}

Về tính ứng dụng: với *Encodable* ta dễ dàng lưu struct vào *UserDefaults* hoặc tạo params để post lên server chẳng hạn.

Source code của toàn bộ ví dụ: [source](https://gist.github.com/gg4acrossover/f27b8b0c2336c5bb7fc7f1a311a29537)




