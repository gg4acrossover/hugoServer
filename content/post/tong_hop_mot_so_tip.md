+++
author = ""
comments = true	# set false to hide Disqus
date = "2018-06-15T17:21:29+07:00"
draft = false
image = "images/cover/5.jpg"
menu = ""		# set "main" to add this content to the main menu
share = true	# set false to hide share buttons
slug = "tong-hop-mot-so-tip"
tags = ["swift4"]
title = "Tổng hợp một số tip Swift"
+++

*Tổng hợp một số tip Swift mà mình hay dùng*

### Measurement

Nếu bạn muốn qui đổi đơn vị một cách nhanh chóng mà không phải tự xây dựng hệ thống chuyển đổi riêng thì measurement là lựa chọn hợp lý.


{{< highlight objc "style=monokai" >}}
typealias Duration = Measurement<UnitDuration>
let s = Duration(value: 60.0, unit: .seconds)
let m = Duration(value: 1.0, unit: .minutes)
let twoMinutes = s + m

twoMinutes.value // 120.0 s
{{< /highlight >}}

Đặc biệt nhất, bạn có thể tự cộng, trừ 2 measurement cùng chung một loại. VD: 1p + 60s = 120s
Có khá nhiều kiểu Unit cho bạn lựa chọn.

### Slim tableview

Mình hay dùng cái này nhất, bạn sẽ tiết kiệm được 2s cuộc đời mỗi khi đăng ký hay sử dụng lại một cell bất kì.

{{< highlight objc "style=monokai" >}}
// register tableview/collection view
protocol ONBaseCellProtocol {
    static func getNibName() -> String // return nibname of cell
    static func getNib() -> UINib // return nib of cell
}

extension ONBaseCellProtocol where Self : UIView {
    /// collectionView get nibname
    static func getNibName() -> String {
        return String(describing: self)
    }
    
    /// collectionView get nib
    static func getNib() -> UINib {
        return UINib.init(nibName: self.getNibName(), bundle: nil)
    }
}

extension UITableView {
    func on_register(type: ONBaseCellProtocol.Type) {
        self.register(type.getNib(), forCellReuseIdentifier: type.getNibName())
    }
    
    func on_register(types: [ONBaseCellProtocol.Type]) {
        for type in types {
            self.register(type.getNib(), forCellReuseIdentifier: type.getNibName())
        }
    }
    
    func on_dequeue< T : ONBaseCellProtocol>(idxPath : IndexPath) -> T {
        guard let cell = self.dequeueReusableCell(withIdentifier: T.getNibName(), for: idxPath) as? T else {
            fatalError("couldnt dequeue cell with identifier \(T.getNibName())")
        }
        
        return cell
    }
}
{{< /highlight >}}

Với đoạn code trên, bạn có thể viết register/deque tableview theo cách ngắn gọn sau

{{< highlight objc "style=monokai" >}}
self.tableview.on_register(YourCell.self)
let cell: YourCell = self.tableview.on_dequeue(idxPath: indexPath)
{{< /highlight >}}

Áp dụng tương tự với CollectionView.

### Self

Nếu bạn đã từng dành cả tuổi thanh xuân để viết weakSelf, strongSelf thì đây là giải pháp cho bạn.
Viết \`self\` sẽ giúp bạn sử dụng *self* như một biến, thay vì dùng nó như keyword.

{{< highlight objc "style=monokai" >}}
guard let `self` = self else {
    // return ...
}

self.tableView.reloadData() // thay vì self?.tableView.reloadData() hay strongSelf.tableView.reloadData()
{{< /highlight >}}

Bạn có thể dùng cách này để khai báo singleton.

### Functional

Sử dụng overload toán tử là một cách hay để tiếp cận swift theo hướng functional.

{{< highlight objc "style=monokai" >}}
precedencegroup ForwardApplication {
    associativity: left
}
infix operator |> : ForwardApplication
func |> <A,B> (x: A, f: @escaping (A) -> B) -> B {
    return f(x)
}


precedencegroup ForwardComposition {
    associativity: left
    higherThan: ForwardApplication
}
infix operator >>> : ForwardComposition
func >>> <A,B,C> (f: @escaping (A) -> B, g: @escaping (B) -> C) -> (A) -> C {
    return { a in g(f(a))}
}

func incr(x: Int) -> Int {
    return x + 1
}

// có thể viết
[1,2,3].map(incr >>> String.init)

// thay vì
[1,2,3]
    .map(incr)
    .map(String.init)
{{< /highlight >}}

