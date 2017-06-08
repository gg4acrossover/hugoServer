+++
author = ""
comments = true
date = "2017-06-07T16:14:26+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "slim-tableview"
tags = ["ios","swift3"]
title = "Slim tableview"

+++

### 1.Tản mạn

Cách đơn giản nhất để giảm bớt bug là viết code ít đi. Chân lý đó đã được đưa vào một định luật nổi tiếng, hồi phổ thông ai cũng từng kinh qua.

> e = mc2

![image2](/hugosite/images/note/emc_1x.png)

Dịch một cách chân phương là error = more code (càng nhiều code càng gây lỗi). Khoảng cách giữa coder và tester phụ thuộc vào việc có bao nhiêu bug sinh ra. Einstein khi đưa ra định luật đó, chắc không ngờ rằng nó có thể dịu dàng đúng (theo một cách mới) với cả ngành IT này. Thế nên viết ngắn đi mà vẫn dễ hiểu luôn luôn được những lười biếng coder coi là tôn chỉ.

Khi sử dụng *tableview* chúng ta luôn đi theo một format chung, thậm chí gõ đến thuộc lòng, không cần *autocomplete*.

{{< highlight objc "style=monokai" >}}
self.listTbl.register( UINib( nibName: String(describing: GGArtistryCell.self), bundle: nil), 
	forCellReuseIdentifier: "cell")
{{< /highlight >}}

Cả những khi dequeue nữa

{{< highlight objc "style=monokai" >}}
let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
{{< /highlight >}}

Những đoạn code này hơi dài, vậy nên chúng ta, những lười biếng coder, nên làm ngắn nó đi tí.
Mình sẽ sử dụng **protocol** và **generics** để việc đăng kí và dequeue cell ngắn gọn hơn.

### 2. Make up

Làm việc với objc, mình hay tạo baseCell có 2 hàm *getNib* và *getNibName* (cho identifier) để việc đăng kí cell trở nên ngắn gọn hơn. Tuy nhiên ở swift, mình sẽ sử dụng **protocol**.

{{< highlight objc "style=monokai" >}}
/**
    cell protocol for uicollectionview, uitableview
 */
protocol GGBaseCellProtocol {
    static func getNibName() -> String // return nibname of cell
    static func getNib() -> UINib // return nib of cell
}


/**
    default implement for cell
 */
extension GGBaseCellProtocol where Self : UIView {
    /// collectionView get nibname
    static func getNibName() -> String {
        return String(describing: self)
    }
    
    /// collectionView get nib
    static func getNib() -> UINib {
        return UINib.init(nibName: self.getNibName(), bundle: nil)
    }
}
{{< /highlight >}}

Tạo extension cho protocol để ta không phải khai báo đi, khai báo lại 2 hàm *getNib* và *getNibName* ở mỗi class implement lại protocol này. Khi đó ta chỉ cần cho custom cell của mình kế thừa lại protocol. Khá là linh động. Code lúc đó sẽ ngắn đi chút ít nhưng nhìn clear hơn.

Ví dụ như thế này

{{< highlight objc "style=monokai" >}}
self.listTbl.register(GGArtistryCell.getNib(), 
	forCellReuseIdentifier: GGArtistryCell.getNibName())
{{< /highlight >}}

Khá là đẹp rồi phải không? tuy nhiên ta có thể code ngắn hơn nữa bằng cách sử dụng **generics**.

{{< highlight objc "style=monokai" >}}
extension UITableView {
    func on_register< T : GGBaseCellProtocol>(type: T.Type)  {
        self.register(T.getNib(), forCellReuseIdentifier: T.getNibName())
    }
    
    func on_dequeue< T : GGBaseCellProtocol>(idxPath : IndexPath) -> T {
        guard let cell = self.dequeueReusableCell(withIdentifier: T.getNibName(), for: idxPath) as? T else {
            fatalError("couldnt dequeue cell with identifier \(T.getNibName())")
        }
        
        return cell
    }
}
{{< /highlight >}}

Để cho gọn, mình tạo extension cho UITableView, với 2 hàm *on_register* và *on_dequeue* làm 2 việc mà ai cũng biết là việc gì rồi.

Khi đó bước đăng kí cell sẽ ngắn đi như sau:

{{< highlight objc "style=monokai" >}}
self.listTbl.on_register(type: GGArtistryCell.self)
{{< /highlight >}}

So với trước thì ngắn gọn đi khá nhiều, coder không phải viết dài dòng nữa.

Tiếp đến là dequeue cũng ngắn không kém phần

Thay vì:

{{< highlight objc "style=monokai" >}}
if let cell = self.listTbl.dequeueReusableCell(withIdentifier: GGArtistryCell.getNibName(), for: indexPath) as? GGArtistryCell {
	// your code
}
{{< /highlight >}}

Ta chỉ cần

{{< highlight objc "style=monokai" >}}
let cell : GGArtistryCell = self.listTbl.on_dequeue(idxPath: indexPath)
{{< /highlight >}}

Bây giờ mình đã có thể lười đi tí :smile:

Ta có thể áp dụng phương pháp này cho cả collectionView

### 3. Kết

Lời cuối sẽ là một [siêu tip](https://twitter.com/realbadiostips/status/867791220323500032) để code ngắn gọn

> MVC = Most Viable Cocoapod.



