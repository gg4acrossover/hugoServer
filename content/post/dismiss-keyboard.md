---
title: "Ẩn Keyboard"
date: 2019-06-18T13:59:48+07:00
lastmod: 2019-06-18T13:59:48+07:00
draft: false
keywords: ["swift"]
description: "dismiss keyboard"
tags: ["swift"]
categories: ["swift"]
author: "VietHQ"

---

Xin chào mọi người, hôm nay mình sẽ trở lại với một vấn đề quen thuộc. Đó là ẩn keyboard, xử lý tuy rằng đơn giản, nhưng để làm "mượt" thì cũng không dễ tẹo nào. Mình sẽ đi từng bước một, từ cách bình dân nhất, rồi đến những xử lý refactor.

## Cách bình dân

Cách chúng ta thường thấy nhất là:

```swift
class ViewController: UIViewController {
    @IBOutlet weak var txtField: UITextField!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        let tap = UITapGestureRecognizer(target: self, action: #selector(tapView(sender:)))
        view.addGestureRecognizer(tap)

        // more code ...
    }
    
    @objc func tapView(sender: UITapGestureRecognizer) {
        self.txtField.resignFirstResponder()
    }
}
```
Phương án này đơn giản, tuy nhiên có nhiều nhược điểm:

- Dễ thấy nhất là ta xử lý logic ngay trong *viewDidLoad* sẽ làm hàm này dễ phình to, khó maintain sau này.
- Không tận dụng lại code được, ngoại trừ việc copy paste.

## Extension

Vẫn phương pháp trên, chúng ta có thể làm mịn hơn nữa bằng cách

```swift
class ViewController: UIViewController {
    @IBOutlet weak var txtField: UITextField!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.addTapToHideKeyboard()
    }
    
    @objc func tapView(sender: UITapGestureRecognizer) {
        self.txtField.resignFirstResponder()
    }
}

extension ViewController {
    func addTapToHideKeyboard() {
        let tap = UITapGestureRecognizer(target: self, action: #selector(tapView(sender:)))
        view.addGestureRecognizer(tap)
    }
}
```
Mình bóc tách phần xử lý gesture, nhóm chúng vào 1 hàm duy nhất, hàm này được đưa vào trong extension, cũng giống như cách ta phân đoạn văn hồi còn mài đít trên ghế nhà trường. Phân chia ra sao, mỗi người một style khác nhau, cái này mình không ý kiến. Miễn là code viết ra dễ đọc, sạch sẽ như kotex vậy (mượn tạm câu này của bạn nào đó chuyên viết tut js).

Với cách này, chúng ta có thể hình dung ra một hướng tổng quán hơn, đó chính là đưa hàm *addTapToHideKeyboard* vào trong extension của UIViewController. Tuy nhiên nếu làm như vậy, ta đối mặt với một vấn đề lớn khác, đó là xử lý hàm *tapView* ra sao, khi mà hàm này đang xử lý một tình huống cụ thể, chứ chưa mang tính khái quát.

Thật may mắn là ngoài *resignFirstResponder*, chúng ta còn có phương thức khác mạnh mẽ hơn mang tên *endEditing*

## Sử dụng End editing

Bài toán trở nên đơn giản hơn như sau:

```swift
extension UIViewController {
    func tapViewToHideKeyboard() {
        let tap = UITapGestureRecognizer(target: view, action: #selector(UIView.endEditing))
        view.addGestureRecognizer(tap)
    }
}
```

Bằng hàm trên thì ta có thể xử lý ngắn gọn hơn, tổng quát hơn ở ViewController như sau

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    self.tapViewToHideKeyboard()
}
```
Cách mình vừa trình bày khá là ổn, dư dả đủ dùng. Tuy nhiên nó đang giới hạn ở UIViewController. Mình có thể đưa phương pháp này đến vùng sâu vùng xa hơn, như là với UIView chẳng hạn

## Sử dụng UIView extension

```swift
extension UIView {  
    func addTapGesToDismissKeyboard() {
        let tap = UITapGestureRecognizer.init(target: self, action: #selector(endEditing))
        self.addGestureRecognizer(tap)
    }
}
```

Cách này có một hạn chế nho nhỏ, và chúng ta sẽ cùng nhau xử lý ngay sau đây ít phút.

...

Nói vậy thôi, chúng ta sẽ nói về vấn đề gặp phải trước. Việc xử lý *tap* (ẩn keyboard) ngay trên UIView chỉ có tác dụng khi đối tượng kích hoạt keyboard nằm trên view đó hoặc là nằm ở subView. Nếu đối tượng kích hoạt keyboard nằm ngoài tầm ảnh hưởng của view thì móm. Thế nên nếu dừng xử lý ở đây, tốt nhất chúng ta gắn sự kiện *tap* ngay trên view của ViewController.

Trường hợp view không chứa đối tượng kích hoạt keyboard có thể mô tả qua ảnh sau (textfield nằm ngoài view *Tap to dismiss*)

![img](/hugosite/images/note/dismiss-keyboard/1.png)

Nhưng ngay từ đầu mình đã nói sẽ xử lý vấn đề này nên chúng ta sẽ đi tiếp. Hướng tiếp cận cũng đơn giản, chúng ta sẽ lần mò đến cái thằng view cha to nhất. Chúng ta sửa lại extension của UIView:

```swift
extension UIView {
    var topView: UIView? {
        var top = superview
        while top?.superview != nil {
            top = top?.superview
        }
        
        return top
    }
    
    func addTapGesToDismissKeyboard() {
        let tap = UITapGestureRecognizer.init(target: self, action: #selector(self.dissmissKeyboard))
        self.addGestureRecognizer(tap)
    }
    
    @objc func dissmissKeyboard() {
        topView?.endEditing(true)
    }
}
```
Đầu tiên, ta lấy ra thằng root view, sau đấy cho root view thực hiện *endEditing*.

## Sử dụng NSObject

Cách cuối cùng là cách không phải sử dụng nhiều đến code. Trong storyboard chúng ta có thể add Object.

![img](/hugosite/images/note/dismiss-keyboard/2.png)

Chúng ta sẽ tạo thêm một đối tượng nữa, ở đây mình đặt tên là *KeyboardHider*.

```swift
class MEKeyboardHider: NSObject {
    @IBOutlet var targets: [UIView]! {
        didSet {
            targets.forEach(addTapGesToHideKeyBoard)
        }
    }
    
    private func addTapGesToHideKeyBoard(to view: UIView) {
        view.addTapGesToDismissKeyboard()
    }
}
```
Mình để @IBOutlet cho *targets* nhằm mục đích gắn hành động xử lý ẩn keyboard cho *targets* ngay trên file giao diện.

Các bước sử dụng lúc này như sau:

Tạo object trên file giao diện, gắn class KeyboardHider (tạo ở trên) cho nó như hình sau

![img](/hugosite/images/note/dismiss-keyboard/3.png)

Bước cuối cùng gắn targets và tận hưởng thành quả

![img](/hugosite/images/note/dismiss-keyboard/4.png)

Lúc đó thì hàm *viewDidLoad* ở ví dụ trên sẽ trông như này:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
}
```
Chúng ta đã loại bỏ được hoàn toàn phần code xử lý ẩn keyboard. Quá hay phải không nào?

code tham khảo: [source](https://gist.github.com/gg4acrossover/15ecbf7dc319cf530a32ad0831ca5953)

Sau bài này, ta rút ra được gì:

- Tách code sử dụng extension
- Đưa các hàm dùng chung vào trong extension của các đối tượng trong thư viện chuẩn
- Sử dụng object trong file giao diện
- Ẩn keyboard mà không phải dùng thêm lib nào cả.