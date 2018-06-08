+++
author = ""
comments = true	# set false to hide Disqus
date = "2018-06-08T15:04:33+07:00"
draft = false
image = "images/cover/3.jpg"
menu = ""
share = true
slug = "strong-type-va-bai-toan-qui-doi-tien-te"
tags = ["swift4","ios"]
title = "Strong type và bài toán qui đổi tiền tệ"
+++

### Strong type là gì? Tại sao lại cần?

*Strong type* là cách ta định nghĩa ra một kiểu dữ liệu mới, từ những dữ liệu có sẵn nhưng mang tính định danh cao hơn. Ở các ngôn ngữ khác, ví dụ như haskell, người ta hay gọi đấy là Phantom types (kiểu bóng ma)

Một ví dụ điển hình và đơn giản nhất:

Ta có một *struct Bank* để gửi tiền

{{< highlight objc "style=monokai" >}}
struct Bank {
    let money: Double
    init(_ money: Double) {
        self.money = money
    }
}
{{< /highlight >}}

Ngân hàng chỉ cho gửi VND, nhưng chưa có cơ chế check. Ta gửi VND bình thường theo cách sau 

> Bank(100) // VND

Nhưng một hôm, ta bận và nhờ bạn gửi hộ, bạn lại gửi $

> Bank(100) // vẫn được chấp nhận

Vậy nên ta cần một kiểu dữ liệu mới để người dùng chỉ nhập được duy nhất VND. Đây là lúc **Strong type** phát huy tác dụng.


### Cách thức tạo Strong type

VND, USD, AUD là đơn vị tiền tệ của mỗi quốc gia. Ta coi mỗi loại tiền là 1 *Unit*, Ta cần một cái gì đó để xác định 3 *Unit* trên cùng chung một nhóm. 
Thế nên ta định nghĩa 2 protocol: *UnitFamily* tạo nhóm, *MyUnit* để định nghĩa các đơn vị trong nhóm.


{{< highlight objc "style=monokai" >}}
// Define UnitFamily to group all of units have the same family
protocol UnitFamily {
    associatedtype BaseUnit
}

// Define MyUnit, own unit will be based on MyUnit protocol
protocol MyUnit {
    associatedtype Family: UnitFamily
    static var symbol: String { get }
    static var converted: UnitConverter { get }
}
{{< /highlight >}}

Ở đây ta để ý rằng *UnitFamily* có chứa 1 type *BaseUnit* nhưng không sử dụng đến, tại sao lại cần đến nó? Như ta biết trong thực tế, bất cứ công thức qui đổi nào đều cần dùng một đơn vị chuẩn. Giống như **F = ma**, *m* luôn là *kg*. Thế nên *BaseUnit* có thể hiểu là một *Unit* bất kì trong group được lấy ra làm chuẩn.

Tiếp theo ta cần tạo ra **Strong type** để không nhầm lẫn *Unit* này với *Unit* khác. 

{{< highlight objc "style=monokai" >}}
// Unit wrapper, this one help us convert data type easier
struct MyMeasurement<UnitType: MyUnit> {
    let value: Double
    init(_ value: Double) {
        self.value = value
    }
}
{{< /highlight >}}

*UnitType* không được sờ đến trong *MyMeasurement*, tác dụng của nó là **định danh**, qua đó hình thành **Strong type**, cũng có thể gọi là *Phantom Types*. Ta đã có **Strong type** theo cách rất đơn giản phải không? Tiếp theo là vấn đề convert.

Để convert được giữa các đơn vị theo cách chung nhất thì ta tạo *extension* cho *MyMeasurement* và có sử dụng *generic*.

{{< highlight objc "style=monokai" >}}
extension MyMeasurement {
    func converted<TargetUnit>(to: TargetUnit.Type) -> MyMeasurement<TargetUnit> where
        TargetUnit: MyUnit, UnitType.Family == TargetUnit.Family {
            let valueInBase = UnitType.converted.baseUnitValue(fromValue: self.value)
            let convertValue = TargetUnit.converted.value(fromBaseUnitValue: valueInBase)
            return MyMeasurement<TargetUnit>(convertValue)
    }
}
{{< /highlight >}}

Bây giờ ta bắt đầu định nghĩa các loại tiền với kiểu dữ liệu Enum. Các *Unit* VND, USD, AUD đều chung 1 nhóm (Family) *MyCurrency* 

{{< highlight objc "style=monokai" >}}
// Group
enum MyCurrency: UnitFamily {
    typealias BaseUnit = VND
}

// Unit
enum VND: MyUnit {
    typealias Family = MyCurrency
    static var symbol: String {
        return "VND"
    }
    
    static var converted: UnitConverter = UnitConverterLinear(coefficient: 1)
}

enum USD: MyUnit {
    typealias Family = MyCurrency
    static var symbol: String {
        return "$"
    }
    
    static var converted: UnitConverter = UnitConverterLinear(coefficient: 1*22.5)
}

enum AUD: MyUnit {
    typealias Family = MyCurrency
    static var symbol: String {
        return "AUD"
    }
    
    static var converted: UnitConverter = UnitConverterLinear(coefficient: 1*17.3)
}
{{< /highlight >}}

OK, mọi thứ đã được định nghĩa một cách ổn thỏa. Bây giờ sếp trả lương cho ta theo 2 đợt, đợt 1 lấy vnd, đợt 2 lấy dollar Úc, ta muốn qui đổi ra VND để tiêu cho dễ.

{{< highlight objc "style=monokai" >}}
let vnd = MyMeasurement<VND>(20)
let aud = MyMeasurement<AUD>(10)

let salaryValue = vnd.value + aud.converted(to: VND.self).value
let salary = MyMeasurement<VND>(salaryValue)
{{< /highlight >}}

Ta phải qua ngân hàng để chuyển đổi đồng dollar Úc qua VND nhưng điều đó quá bất tiện, ta yêu cầu ngân hàng tạo phương thức qui đổi tiền tệ cho mình. Tất nhiên là ngân hàng đáp ứng

{{< highlight objc "style=monokai" >}}
func + <Unit1,Unit2>(lhs: MyMeasurement<Unit1>, rhs: MyMeasurement<Unit2>) -> MyMeasurement<VND>
    where
    Unit1: MyUnit
    , Unit2: MyUnit
    , Unit1.Family == Unit2.Family
    , Unit1.Family == VND.Family {
        let lhsToVND = Unit1.converted.baseUnitValue(fromValue: lhs.value)
        let rhsToVND = Unit2.converted.baseUnitValue(fromValue: rhs.value)
        return MyMeasurement<VND>(lhsToVND + rhsToVND)
}
{{< /highlight >}}

Giờ ta chỉ cần thực hiện lệnh qua phần mềm để tự động chuyển về VND, nhẹ hơn 1 dòng :v

{{< highlight objc "style=monokai" >}}
let vnd = MyMeasurement<VND>(20)
let aud = MyMeasurement<AUD>(10)
let newSalary = vnd + aud
{{< /highlight >}}

OK giờ ta có lương, ta muốn tiết kiệm nên làm sổ tiết kiệm VND. Ta bận nên nhờ bạn đến gửi hộ (như lần trước), bạn mang theo $ nhưng lần này ngân hàng nâng cấp hệ thống kiểm tra đầu vào nên bạn không gửi tùy tiện được. Bank cảnh báo lỗi luôn

![image](/hugosite/images/note/strong-type.png)

{{< highlight objc "style=monokai" >}}
struct Bank {
    let money: MyMeasurement<VND>
    init(_ money: MyMeasurement<VND>) {
        self.money = money
    }
}
{{< /highlight >}}

Để ý rằng *money: Double* được thay thế bằng MyMeasurement là **Strong type**

Nhưng về sau ta không muốn ra tận ngân hàng để gửi tiền nữa, muốn gửi trực tuyến. Ngân hàng lại cải tiến đáp ứng khách hàng bằng cách sử dụng tool *ExpressibleByIntegerLiteral* và *ExpressibleByFloatLiteral* mà Apple cung cấp.


{{< highlight objc "style=monokai" >}}
extension MyMeasurement: ExpressibleByIntegerLiteral {
    init(integerLiteral value: IntegerLiteralType) {
        self.value = Double(value)
    }
}

extension MyMeasurement: ExpressibleByFloatLiteral {
    init(floatLiteral value: FloatLiteralType) {
        self.value = Double(value)
    }
}
{{< /highlight >}}


Bây giờ thì ta chỉ cần thao tác

> Bank(5.2) // ngân hàng vẫn check format VND

Mọi người có thể lấy full [Source code](https://gist.github.com/gg4acrossover/2e62a7f67bc86c3bca48f81f8b508f28) ở đây.


Tham khảo:
[Measurement](https://oleb.net/blog/2016/08/measurements-and-units-with-phantom-types/) ở blog [https://oleb.net/](https://oleb.net/)











