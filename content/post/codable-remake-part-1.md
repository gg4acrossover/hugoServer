---
title: "Codable remake part 1"
date: 2019-04-26T10:46:46+07:00
lastmod: 2019-04-26T10:46:46+07:00
draft: true
keywords: ["codable"]
description: "codable"
tags: ["swift"]
categories: []
author: "crossover"
---

Dạo này mình hơi bận nên bỏ bê blog quá. Hôm nay quyết định chăm chút trở lại bằng một series về Codable, coi như làm nóng bản thân :D. Đã có một bài mình đề cập đến vấn đề này. Tuy nhiên bài viết đó chỉ ở mức giới thiệu, lần này mình sẽ tăng độ khó cho game thêm chút nữa.

# Codable

```swift
let json = """
{
    "userName": "crossover",
    "position": "SG",
    "id": 234
}
""".data(using: .utf8)!

struct Baller {
    let userName: String
    let position: String
    let id: Int
}
```

Giả sử ta có 1 json đơn giản và struct tương ứng như trên. Để **deserialization** json nói trên, trước đây, ta phải parse một cách thủ công. Đối với những dự án lớn, dữ liệu trả về phức tạp, thực hiện việc parse json bằng tay vừa tiêu tốn của ta kha khá thời gian, vừa khiến ta thấy nhàm chán. Tuy nhiên từ khi swift 4 xuất hiện, chúng ta đã có giải pháp mang tên **Codable**.

Codable là tính năng mới được đưa thêm vào trong Swift 4 nhằm hỗ trợ **serialization** và **deserialization**, giúp ta tự động hóa công việc nhàm chán này. Bản thân **Codable** là sự kết hợp của 2 protocol *Encode* và *Decode* mà mình sẽ trình bày ngay sau đây.

# Decodable

Decodable là protocol được định nghĩa như sau

```swift
public protocol Decodable {
    // ...
    init(from decoder: Decoder) throws
}
```

Theo định nghĩa này ta sẽ implement hàm init trong protocol để thực hiện **deserialization** cho đối tượng.

Trở lại ví dụ ngay đầu bài viết, ta sẽ gán protocol *Decodable* cho *Baller* để tự động việc parse json sang model.

```swift
extension Baller: Decodable {
    // 1
    enum CodingKeys: String, CodingKey {
        case userName
        case position
        case id
    }
    
    // 2
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        userName = try container.decode(String.self, forKey: .userName)
        position = try container.decode(String.self, forKey: .position)
        id = try container.decode(Int.self, forKey: .id)
    }
}

// 3
let decoder = JSONDecoder()
let instance = try decoder.decode(Baller.self, from: json)
instance.userName // "crossover"
```
Có 3 bước cơ bản để làm công việc trên:

+ B1: tạo enum kế thừa lại *CodingKey*. Để ý một chút, các case của enum *CodingKeys* mapping với key của json. Đây là điều kiện bắt buộc nếu ta muốn parse json thành công. Tên properties của *struct Baller* không nhất thiết phải trùng với tên key của json, nhưng để tránh nhầm lẫn và khó hiểu, ta đặt tên properties trùng với các key trong json. Chúng ta sẽ thống nhất cách thức này xuyên suốt bài viết.

+ B2: implement hàm *init(from decoder: Decoder) throws*. Ở bước này ta phải tạo *container* trước khi decodable. Container có thể chia làm 3 loại, tuy nhiên ở bước này mình sẽ không đi sâu chi tiết cả 3 loại mà chỉ đề cập đến trọng tâm trong ví dụ ở trên. Container đang dùng có kiểu *KeyedDecodingContainer*, áp dụng trong trường hợp properties trong đối tượng cần parse đều có các key tương ứng. Công thức tổng quát sẽ như sau:

```swift
let container = try decoder.container(keyedBy: CodingKeys.self)
// decode to initialize properties ...
```

+ B3: Bước này cũng là bước quan trọng nhất, cho ta kết quả của cả quá trình. Ở bước này ta sử dụng *JSONDecoder* như đoạn code ở trên.

## Decodable ngắn gọn

Trên đây là các bước cần thiết khi ta sử dụng *Decodable* để parse dữ liệu. Tuy nhiên ta có thể để compiler trợ giúp ta một số công đoạn viết code.

```swift
extension Baller: Codable {}
let decoder = JSONDecoder()
let instance = try decoder.decode(Baller.self, from: json)
instance.userName // "crossover"
```

Một số trường hợp đặc biệt, convention ở backend sử dụng snake_case (key "userName" ở trên thay đổi thành "user_name")

```json
// ...
"user_name": "crossover",
// ...
```

Trong khi đó mobile sử dụng camelCase, ta cũng có thể xử lý ngắn gọn dựa vào thuộc tính *keyDecodingStrategy* của *JSONDecoder* mà không phải đặt tên property theo convention của backend.

```swift
let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
```

# Encodable

Giờ đến lúc ta bàn về chức năng **serialization** bằng cách sử dụng *Encodable*. Cũng như *Decodable*, *Encodable* là protocol được định nghĩa như sau:

```swift
public protocol Encodable {
    func encode(to encoder: Encoder) throws
}
```
Ta sẽ implement lại phương thức này đối với *Baller*

```swift
extension Baller: Encodable {
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(userName, forKey: .userName)
        try container.encode(position, forKey: .position)
        // ...
    }
}
```

Chúng ta cũng sẽ tạo *container*, tuy nhiên thay vì sử dụng *let* thì ở đây ta phải sử dụng *var*. Các bước encode thực hiện như ở trên.

Cuối cùng ta sử dụng JSONEncoder để đưa ra dữ liệu mong muốn

```swift
let encoder = JSONEncoder()
let data = try encoder.encode(instance)
print(String(data: data, encoding: .utf8)!)
```

Kết quả của đoạn code trên sẽ in ra dòng

    {"position":"SG","userName":"crossover"}

Để kết quả trông đẹp hơn ta có thể sử dụng thuộc tính tronng JSONEncoder như sau:

```swift
encoder.outputFormatting = .prettyPrinted
```

Khi đó kết quả sẽ ra

    {
        "position" : "SG",
        "userName" : "crossover"
    }

# Nested json

## Decode

Mình đã hoàn thành trường hợp đơn giản nhất, giờ mình sẽ đi vào trường hợp phức tạp hơn.
Như ta đã biết, trong các dự án thực tế, dữ liệu trả về không đơn giản như ở ví dụ đầu tiên, cấu trức của nó thường phức tạp hơn.
Chúng ta sửa lại json đầu tiên một chút:

```swift
let json = """
{
    "payload": {
        "userName": "crossover",
        "position": "SG",
        "id": 234
    }
}
""".data(using: .utf8)!
```

Json ban đầu được lồng vào trong 1 json khác, có key là "payload". Giờ không thể **deserialization** trực tiếp nữa. Ta phải sửa lại code như sau

```swift
// let instance = try decoder.decode(Baller.self, from: json)
let instance = try decoder.decode([String:Baller].self, from: json)
```

Đây là cách đơn giản nhất, tuy nhiên lúc này dữ liệu trả về không phải là chính đối tượng mà nó đã được lồng vào trong 1 dictionary. Để lấy dữ liệu ra ta phải làm như sau

```swift
instance["baller"] // Baller
instance["baller"]?.userName // "crossover"
```

Có một cách khác mà mình thích hơn, viết ra một cách tường minh hơn và dữ liệu được lấy ra trực tiếp chứ không bị lồng vào trong 1 đối tượng khác.

Ta phải làm thêm một bước nữa so với trường hợp cơ bản ban đầu. Ngoài việc định nghĩa các key mapping với properties của đối tượng, ta định nghĩa cả key mapping bao bên ngoài đối tượng. Có bao nhiêu tầng lớp bên ngoài thì có bấy nhiêu **CodingKey** được định nghĩa thêm.

``` swift
extension Baller: Decodable {
    enum OuterKeys: CodingKey {
        case payload
    }

    enum InnerKeys: CodingKey {
        case userName
        case position
        case id
    }

    //...
}
```

Trường hợp cụ thể trong bài viết, ta có duy nhất một key bao bên ngoài, nên mình định nghĩa **OuterKeys** cho key bao ngoài (payload), và **InnerKeys** cho đối tượng cần xử lý.

```swift
init(from decoder: Decoder) throws {
    // 1.
    let outer = try decoder.container(keyedBy: OuterKeys.self)
    // 2.
    let inner = try outer.nestedContainer(keyedBy: InnerKeys.self, forKey: .payload)
    
    userName = try inner.decode(String.self, forKey: .userName)
    position = try inner.decode(String.self, forKey: .position)
    id = try inner.decode(Int.self, forKey: .id)
}
```

Ta chú ý 2 bước ban đầu:

> let outer = try decoder.container(keyedBy: OuterKeys.self)

Bước đầu tiên này ta cũng lấy ra container nhưng cho **OuterKeys**. Lúc này ta lấy được nested json bên trong *payload*

```json
{
    "payload": ...
}
```

Bước đầu này đưa ta trở về trường hợp cơ bản nêu đầu bài viết, lúc này ta cần phải lấy container cho đối tượng cần xử lý. Nó nằm trong **outer** nên mình sử dụng phương thức **nestedContainer**

> let inner = try outer.nestedContainer(keyedBy: InnerKeys.self, forKey: .payload)

## Encode

Encode chúng ta làm tương tự, cần tạo outer container và inner container như lúc decode.

``` swift
extension Baller: Encodable {
    func encode(to encoder: Encoder) throws {
        var outter = encoder.container(keyedBy: OuterKeys.self)
        var inner = outter.nestedContainer(keyedBy: InnerKeys.self, forKey: .payload)
        try inner.encode(userName, forKey: .userName)
        try inner.encode(position, forKey: .position)
        // ...
    }
}
```

Chúng ta tạm kết thúc bài viết ở đây, khi nào có hứng mình lại viết tiếp :D.
