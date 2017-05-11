+++
author = "gg4acrossover"
comments = true
date = "2017-05-10T11:25:40+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "progress-hud-don-gian"
tags = ["swift"]
title = "Tạo progressHUD đơn giản"

+++

### Giới thiệu

ProgressHUD là thành phần không thể thiếu trong các app mobile, thường được sử dụng mỗi khi load data từ server.
Mình hay dùng tool hỗ trợ, tỉ như thằng [SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD) hay thằng [MBProgressHUD](https://github.com/jdg/MBProgressHUD). Tuy nhiên thi thoảng tự sướng chút cho nó yomost :smile:. Ví dụ này mình sẽ viết bằng swift 3

### Ý tưởng

Mình sẽ tạo 1 window với rootViewController có nhiệm vụ làm container cho thằng HUD. Thằng HUD chính là singleton để sử dụng chung cho toàn app.
HUD sẽ có 2 hàm chính là **show** và **hide** để ẩn hiện.

### Triển khai

**Bước 1:** tạo container, background của container sẽ màu đen và có độ alpha để nhìn thấy phía sau

{{< highlight objc "style=monokai" >}}
	class GGHUDViewController: UIViewController {

	    override func viewDidLoad() {
	        super.viewDidLoad()

	        self.view.backgroundColor = UIColor.black.withAlphaComponent(0.4)
	    }
	}
{{< /highlight >}}

**Bước 2:** tạo class singleton cho HUD

Vì nó là singleton nên ta cần tạo 1 instance **share** và tạo private init.

{{< highlight objc "style=monokai" >}}
static let share = GGProgressHUD()
{{< /highlight >}}

{{< highlight objc "style=monokai" >}}
    private init() {
        // add icon to center of progress view
        self.iconView = NVActivityIndicatorView( frame: kRectIconView,
                                                 type: .audioEqualizer,
                                                 color: UIColor.red,
                                                 padding: 0.0)
        self.iconView.center = CGPoint( x: kRectProgressView.width * 0.5,
                                        y: kRectProgressView.height * 0.5)
        self.progressView.addSubview(self.iconView)
        
        // always show in center of view even when rotate
        // can use constraints instead
        self.progressView.autoresizingMask = [ .flexibleBottomMargin,
                                               .flexibleLeftMargin,
                                               .flexibleRightMargin,
                                               .flexibleTopMargin]
        
        // add progress to container which is a viewcontroller
        let container = GGHUDViewController()
        container.view.addSubview(self.progressView)
        self.window.rootViewController = container
    }
{{< /highlight >}}

Mình dùng lại thư viện [NVActivityIndicatorView](https://github.com/ninjaprox/NVActivityIndicatorView) vì nó có khá nhiều hiệu ứng đẹp. Để đưa indicator vào chính giữa màn hình, mình dùng autoresizingMask, đây là cách đơn giản nhất, cách khác nữa là add constraints, nhưng mình không đề cập ở đây.

Tiếp theo ta cần tạo phương thức ẩn, hiện

Đầu tiên là hiện

{{< highlight objc "style=monokai" >}}
    // save root window of app
    if let root = UIApplication.shared.keyWindow {
        self.rootWindow = root
    }
    
    self.window.alpha = 0
    self.window.makeKeyAndVisible()
    self.iconView.startAnimating()
    
    UIView.animate(withDuration: 0.5) {
        self.window.alpha = 1.0
    }
{{< /highlight >}}

Như đã nói ở trên, mình tạo 1 window mới với rootViewcontroller làm nhiệm vụ chứa HUD. Ta dùng hàm *makeKeyAndVisible()* đưa nó lên top, vậy nên trước đấy ta cần lưu lại keyWin cũ để khi ẩn ta set lại keyWin default của app.

{{< highlight objc "style=monokai" >}}
    // set root window to key
    self.rootWindow?.makeKey()
    
    UIView.animate(withDuration: 0.5, animations: {
        self.window.alpha = 0.0
    }) { (_) in
        self.iconView.stopAnimating()
    }
{{< /highlight >}}

### Cách dùng

Ta đơn giản chỉ cần call ở viewcontroller *GGProgressHUD.show()* / *GGProgressHUD.hide()*

### Hình ảnh

Sau khi hoàn thành thì nó thế này

![gif](https://media.giphy.com/media/l4FGJYdxiYXx3Y7Nm/giphy.gif)

Đơn giản phải không nào.
Source: [GGProgressHUD](https://github.com/gg4acrossover/swiftForFun/tree/master/%20Project%2002%20-%20GGProgressHUD)












