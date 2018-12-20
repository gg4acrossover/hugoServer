---
title: "Nếu thì và những cách thay thế"
date: 2018-12-19T13:58:26+07:00
lastmod: 2018-12-19T13:58:26+07:00
draft: false
keywords: []
description: ""
tags: ["swift"]
categories: []
author: ""

---

Bạn có biết rằng trên thế giới có một cộng đồng anti if,else. Trên đường đi tìm đường cách mệnh, coi như cũng tạo chút thử thách cho bản thân, mình đã tìm ra một số phương pháp tương đối hữu ích.

# 1. Thủ thuật refactor

Các phương pháp này có thể coi như là trick để code trông gọn hơn, đồng thời loại bỏ *if*.

## Dictionary

Đây là phương pháp mình sử dụng từ lâu. Với phương pháp này thì ta có thể refactor hàm sau:

```
func getEpisode(_ id: String) -> Episode? {
    if id = "E1" {
    	return episode_1
    } else if id = "E2" {
    	return episode_2
    } else if {
    	...
    }

    ...
}
```

Thành dạng như sau

```
let dict = ["E1": episode_1,
            "E2": episode_2,
            "E3": episode_3]

func getEpisode(_ id: String) -> Episode? {
    return dict[id]
}
```

## Enum

Nên sử dụng enum với những cái nào có chung format như ở trường hợp sau

Thay vì

```
func play(type: Format) {
    switch type {
    	case .video: self.playVideo()
    	case .audio: self.playAudio()
    	...
    }
}
```

Tên các method ở trên đều có chung format play**%TYPE%**. Ta có thể gọi nó thông qua *selector* để loại bỏ hẳn *if*

```
enum Format: String {
    case video
    case audio
    case image
}

class Player: NSObject {
    func play(type: Format) {
        let selector = Selector.init("play\(type.rawValue.capitalized)")
        self.perform(selector)
    }
}

// Result
Player().play(type: .video)
```

Để sử dụng **selector**, các phương thức cần gắn thêm tiền tố **@objc** và class cần kế thừa **NSObject**.

# 2. Sử dụng functional programming

## Filter

Đây là phương thức trong bộ tam *filter, map, reduce* ra đời kèm ngay ở phiên bản đầu của swift. Param truyền vào filter có dạng tổng quát như sau:

> (T) -> Bool // hàm truyền vào 1 giá trị và return bool

Như vậy ta có thể truyền vào tên hàm thay thay vì định nghĩa ra nó.

Giả sử bài toán ta đang giải quyết là tìm số nguyên dương trong mảng, thay vì viết như:

```
arr.filter { (value) -> Bool in
    return value > 0
}
```

Ta có thể áp dụng công thức tổng quát như trên để hàm ngắn gọn, rõ nghĩa hơn

```
// (Int) -> Bool
func positive(x: Int) -> Bool {
	return x > 0
}

arr.filter(positive)

```

Nếu có nhiều hơn một điều kiện, ta có thể tổng hợp lại các hàm làm một. Giả sử ta muốn tìm số vừa lớn hơn 0 vừa nhỏ hơn 100, ta viết thêm điều kiện

```
func lessThan100(x: Int) -> Bool {
    return x < 100
}
```

Sử dụng phương pháp composite, ta sẽ có 1 hàm dạng như: **condition = lessThan100 & positive**

```
public class func allPass<A>(_ array: [(A) -> Bool], value: A) -> Bool {
    let predicate = array.map({ V.bind(value: value, to: $0) }) // [() -> Bool]
    return checkAll(predicate)
}

public class func allPass<A>(_ array: [(A) -> Bool]) -> (A) -> Bool {
    return { v in
        let predicate = array.map({ V.bind(value: v, to: $0) }) // [() -> Bool]
        return checkAll(predicate)
    }
}
```

Mình tạo 2 phương thức đều có tên **allPass**, phương thức đầu trả về *Bool*, còn phương thức thứ hai có dạng *curry function* (hàm nhận vào 1 param và trả ra 1 hàm khác). Cả 2 hàm chỉ khác nhau về cách thức tạo nhưng cùng chung 1 mục đích sử dụng.

Với 2 phương thức nêu trên ta có thể composite 2 điều kiện **positive** và **lessThan100** theo cách như sau

> allPass([positive,lessThan100]) // (Int) -> Bool

Và hàm filter sẽ được viết ngắn gọn như sau:

```
func doFilterAll(_ value: Int) -> Bool {
    return V.allPass([positive,lessThan100], value: value)
}
arr.filter(doFilterAll)
```

Ta có thể làm tương tự để có phương thức **Or** để tìm ra giá trị thỏa mãn 1 điều khiện trong tất cả các điều kiện cung cấp.

Bạn thấy đó, với *functional* ta dễ dàng tách các đoạn kiểm tra điều kiện ra thành hàm riêng, dễ dàng tổng hợp chúng, đồng thời dễ test và tái sử dụng.

Để hiểu rõ hơn về hàm **bind** và **checkAll**, mình có đưa nó vào trong code ví dụ.

## Refinement type

Thế nào là [refinement type](https://en.wikipedia.org/wiki/Refinement_type)

> refinement type = value + predicates

Như định nghĩa trên, *refinement type* sẽ có 2 thành phần:

	+ Giá trị
	+ Điều kiện

Hai thành phần này có mối liên hệ chặt chẽ, còn liên kết thế nào thì có thể xem đoạn code ngay sau đây.

```
Positive<Double>.of(100.0)?.value // 100
Positive<Int>.of(-1)?.value // nil
Both<Positive<Float>,LessThan100<Float>>.of(99)?.value // 99
Both<Positive<Float>,LessThan100<Float>>.of(101)?.value // nil
``` 

Như bạn thấy đã thấy, 100 thỏa mãn điều kiện lớn hơn 0, 99 thỏa mãn điều kiện nhỏ hơn 100 nên ta có thể sử dụng, trong trường hợp không thỏa mãn (-1,101) giá trị nil được trả về.

Thậm chí ta chưa dùng tí **if** nào :D

Đoạn code định nghĩa *refinement type* như sau

```
public protocol Refinement {
    associatedtype RefinedType
    static func pass(_ value: RefinedType) -> Bool
}

public struct Refined<A,R: Refinement> where A == R.RefinedType {
    public let value: A
    public init?(_ value: A) {
        guard R.pass(value) else { return nil }
        self.value = value
    }
}

public extension Refinement {
    static func of(_ value: RefinedType) -> Refined<RefinedType,Self>? {
        return Refined(value)
    }
}
```

Code ví dụ [source](https://github.com/gg4acrossover/NoIf)

# 3. Mở rộng

Ngoài những cách trên thì còn rất nhiều cách để giảm thiểu if else:

+ Sử dụng đa hình
+ Tách hàm
+ switch case :v
+ Sử dụng hàm cond (như trong Lisp)
+ Blah blah


---

Tham khảo 

+ Thư viện ramdajs [https://ramdajs.com/](https://ramdajs.com/)

+ Ví dụ của [Peter tomaselli] (https://github.com/peter-tomaselli/swift-refined)









