---
title: "Di, IoC hay DIP"
date: 2021-06-22T02:06:08+07:00
lastmod: 2021-06-22T02:06:08+07:00
draft: false
keywords: ["oop"]
description: ""
tags: ["oop"]
categories: ["oop"]
author: "VietHQ"

---

## Lý thuyết

Hướng đối tượng có 5 nguyên tắc cơ bản, viết tắt là **SOLID**. Trong đấy **D** là keyword quan trọng nhất, hay được hỏi khi interview. Tuy nhiên khá nhiều bạn nhầm đây là từ viết tắt của *Dependency Injection (DI)*. Tên gọi chính xác của nó là *Dependency inversion principle (DIP)*.

Mục đích của DIP là giảm thiểu sự phụ thuộc lẫn nhau giữa các đối tượng (tight coupling), đảm bảo tính linh động, dễ dàng thay thế. Một nguyên tắc bất di bất dịch của DIP là các class liên kết với nhau thông qua *interface*. Tức là ta không quan tâm đến implement cụ thể, mà chỉ tập trung vào behavior.

Ngoài nguyên tắc DIP, chúng ta còn được nghe đến từ khóa khác cũng có chữ *inversion*, đó là *Inversion of Control (IoC)*. Mục đích của IoC cũng như DIP, tuy nhiên nó không giới hạn việc chỉ sử dụng *interface*. 

Như vậy, có thể coi IoC là nguyên tắc có tính bao quát hơn DIP. Còn DI chỉ là một pattern, một dạng triển khai cụ thể cho IoC.

Nghe lý thuyết có vẻ hơi mông lung, loạn đầu nhỉ :D? OK đoạn trên chỉ phục vụ cho interview. Còn bây giờ mình sẽ tập trung vào thứ quan trọng hơn là *thực hành*. Cứ làm nhiều là quen là hiểu lý thuyết ngay, nhể?



## Thực hành

```swift
class ViewController: UIViewController {
    var loading: URLSession!
  	var errorHandler: ErrorHandler!
    override func viewDidLoad() {
        super.viewDidLoad()
        // loading
        let loading = URLSession.shared
        loading.configuration.timeoutIntervalForRequest = 30.0
        loading.configuration.timeoutIntervalForResource = 30.0
        self.loading = loading
        
      	// error handler
      	self.errorHandler = ErrorHandler()
      
      	// model parser
        self.parser = ModelParser()
      
      	// views
        // ...
    }
}
```

Class *ViewController* cần truyền vào một đối tượng *network* để call API, một đối tượng để xử lý *error* và *parse model* nữa. Giờ hàm viewDidLoad đang quá tải vì xử lý nhiều. Ngoài việc khởi tạo đối tượng, nó còn phải xử lý layout, gắn style cho UI. Ví dụ này có lẽ chúng ta đã gặp đâu đó trong thực tế, thậm chí là gặp tương đối nhiều :cry:

Chúng ta sẽ refactor nó dần dần để đáp ứng IoC. Đầu tiên hãy tách nhỏ viewDidLoad hơn nữa bằng cách sử dụng *factory method*

```swift
extension ViewController {
    func makeNetwork() -> URLSession {
        let loading = URLSession.shared
        loading.configuration.timeoutIntervalForRequest = 30.0
        loading.configuration.timeoutIntervalForResource = 30.0
        return loading
    }
    
    func makeErrorHandler() -> ErrorHandler {
        return ErrorHandler()
    }
    
    func makeParser() -> ModelParser {
        return ModelParser()
    }
}
```

Khi đó code ở viewDidLoad trông gọn gàng hơn, dù cách giải quyết chưa tối ưu, tuy nhiên đã khá hơn chút.

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.loading = self.makeNetwork()
        self.errorHandler = self.makeErrorHandler()
        self.parser = self.makeParser()
    }
```

Các câu lệnh khởi tạo được phân chia thành từng hàm riêng, chúng ta dễ dàng tìm, sửa chữa mà không gây ảnh hưởng đến các khối lệnh khác. Bằng cách tạo subClass của *ViewController* ta cũng có thể override để tùy biến *network, errorHandler và modelParser* theo ý chúng ta.

Tuy nhiên vẫn có gì đó hơi sai sai. Chẳng lẽ mỗi lần cần config khác cho *network, error handler hay model parser* ta lại phải tạo...sub class. *ViewController* cũng không nên biết quá nhiều về cách khởi tạo, config các đối tượng kể trên. Thêm nữa, trường hợp ta cần *network*, không lẽ lại phải tạo *ViewController* trước. Muốn nói :fu: quá.

### Service location

Để tiếp tục *refactor*, ta cần đưa việc khởi tạo ra bên ngoài, *ViewController* sẽ chỉ nhận các đối tượng mà nó phụ thuộc từ bên ngoài

```swift
extension ViewController {
    func make(
      loading: URLSession, 
      errorHandler: ErrorHandler, 
      parser: ModelParser) {
        self.loading = loading
        self.errorHandler = errorHandler
        self.parser = parser
    }
}
```

Các đối tượng *network, error handler và parser* sẽ được đưa vào trong *Factory* như sau

```swift
class ConfigFactory {
    func makeNetwork() -> URLSession {
        let loading = URLSession.shared
        loading.configuration.timeoutIntervalForRequest = 30.0
        loading.configuration.timeoutIntervalForResource = 30.0
        return loading
    }
    
    func makeErrorHandler() -> ErrorHandler {
        return ErrorHandler()
    }
    
    func makeParser() -> ModelParser {
        return ModelParser()
    }
}
```

Khi đó khởi tạo *ViewController* cũng phải sửa lại chút

```swift
    // viewcontroller
    var factory: ConfigFactory
    
    init(factory: ConfigFactory) {
        self.factory = factory
        super.init(nibName: nil, bundle: nil)
        self.make(
            loading: self.factory.makeNetwork(),
            errorHandler: self.factory.makeErrorHandler(),
            parser: self.factory.makeParser()
        )
    }
```

Như vậy ta sẽ tạo *ViewController* như sau

```swift
let factory = ConfigFactory()
let vc = ViewController(factory: factory)
```

Cách trên gọi là *service location* (SL). Tất cả những thứ *ViewController* cần, chúng ta có thể gom nó vào 1 class khác, gọi chung là *Config* đi. Để ý kỹ thì *plist* cũng là một file config trong iOS. Chúng ta điền các thông số cần thiết vào *plist*. App đọc file đó và nhận cấu hình ta cài đặt.

Với cách trên ta đã tách được việc khởi tạo các đối tượng mà *ViewController* phụ thuộc ra ngoài. Tuy nhiên cách làm này cũng có một số vấn đề dễ gặp phải mà ta cần lưu ý. Giả sử file *config* dày lên (cụ thể ở trường hợp này là *ConfigFactory*) thì file này sẽ dần trở nên rác. *ViewController* sẽ nhìn thấy cả những thứ nó không cần và việc chọn lựa cái *ViewController* thực sự cần cũng trở nên khó hơn.

Ví dụ một trường hợp cụ thể hơn, giả sử một thư viện vừa đảm nhận tính toán tọa độ 3 điểm của tam giác theo sin, cos, tan, vừa vẽ UI của tam giác. Nếu ta gắn thư viện đó cho 1 chương trình trên *terminal* thì phần vẽ UI thừa thãi. Ngược lại, nếu gắn thư viện cho 1 chương trình đảm nhận vẽ UI thì tính toán tọa độ lại thừa ra. Một rắc rối nữa đi kèm là ta sẽ khó tìm thấy cái mình thực sự cần vì có quá nhiều thứ hiển hiện trước mắt. 

Trong một app, hoàn toàn có thể có *App Factory* và việc gắn quá nhiều thứ vào *factory* cũng sẽ gây vấn đề cho việc maintain sau này. Cái này chúng ta nên tránh.

Chúng ta đang đi khá từ tốn, giải quyết từng vấn đề nhỏ một. Bài còn dài...vậy nên hãy kiên nhẫn nhé :smile:

### Dependency injection

Mình biết đến từ khóa này khi đi phỏng vấn từ rất lâu lâu lắm rồi. Khi *google* thì thấy nhan nhản các ví dụ, các bài tutorial từ...java là nhiều, tập trung nhiều vào backend.

Đấy cũng là thiệt thòi dành cho những người code iOS. Tuy nhiên, không phải ta không thể học và gắn nó vào thế giới của iOS.

Chúng ta sẽ refactor tiếp ví dụ ở trên, đầu tiên xóa hàm init của *ViewController*

```swift
class ViewController: UIViewController {
    var loading: URLSession?
    var errorHandler: ErrorHandler?
    var parser: ModelParser?

    override func viewDidLoad() {
        super.viewDidLoad()
        // ...
    }
}

```

Tiếp đến tạo *class Builder* cho *ViewController*

```swift
class ViewControllerBuilder {
    let config: ConfigFactory = {
        return ConfigFactory()
    }()
    
    func makeViewController() -> ViewController {
        let vc = ViewController()
        vc.make(
            loading: config.makeNetwork(),
            errorHandler: config.makeErrorHandler(),
            parser: config.makeParser()
        )
        return vc
    }
}

```

Lúc này việc khởi tạo *ViewController* chỉ đơn giản như sau

```swift
let builder = ViewControllerBuilder()
let vc = builder.makeViewController()
```

Giờ *ViewController* không phụ thuộc vào một lớp config và chúng ta còn có thể giấu việc khởi tạo vào trong  *builder*. Mọi thứ trở nên cực kỳ đơn giản và cũng dễ tận dụng lại.

Khi áp dụng DI, chúng ta có thể làm theo các hướng sau:

- Init injection (khi việc phụ thuộc là bắt buộc)
- Property injection (khi việc phụ thuộc không bắt buộc)
- method injection (giống property injection nhưng ta có thể thêm action)

Mình sẽ không đi sâu chi tiết vào phần implement theo 3 hướng trên. Thay vào đó các bạn có thể tự tìm hiểu thông qua *google*

### DIP

Như đã nói từ đầu, để theo rule *DIP* ta cần phải truyền các thành phần phụ thuộc vào class dưới dạng *interface*. 

```swift
class ViewController: UIViewController {
    var loading: LoadingProtocol?
    var errorHandler: ErrorHandlerProtocol?
    var parser: ModelParserProtocol?
    // ...
}
```

Khi các đối tượng truyền vào từ bên ngoài là *interface* thì class sẽ không phụ thuộc vào một đối tượng hay template. Chúng ta chỉ quan tâm đến behavior mà *interface* định nghĩa. Như vậy ta có thể truyền một nhóm đối tượng theo *interface* đó. Việc thay thế và tùy biến càng trở nên dễ dàng hơn.

### Mặt hạn chế

Trong lập trình không có viên đạn bạc, cái gì cũng có mặt hạn chế của nó. Ở trên, đơn thuần ta nói về mặt lợi. Giờ là lúc nói đến mặt hại :smile:

Để thỏa mãn các điều kiện khi tuân thủ *DIP* là chúng ta phải tạo nhiều *interface*. Ở một project lớn, độ phức tạp cao, đòi hỏi càng có nhiều *interface*. Khi đó việc build app sẽ trở nên chậm chạp hơn.

Tiếp theo, việc tuân thủ *DIP* cũng khiến ta phải tách khá nhiều lớp, factory lồng factory. Để tạo ra được một phương thức khởi tạo đơn giản đòi hỏi một quá trình phức tạp với nhiều class được tạo ra. Như ví dụ trên ta có factory cho các đối tượng con của *ViewController*, factory cho riêng *ViewController*.



## Tổng kết

Chữ **D** trong **SOLID** là một nguyên tắc khá quan trọng. Để đảm bảo được nguyên tắc *single-responsibility* hay *ope- close* ta cũng cần thuần thục *Dependency injection*.

**IoC** cũng là nguyên tắc trong lập trình hướng đối tượng, dù không được đề cập trong **SOLID**. Tuy nhiên nó mang tính bao quát hơn **DIP** (DIP phải sử dụng interface). Trong bài đã đề cập đến 2 cách triển khai tuân thủ **IoC** là *Service Location, DI*, tuy nhiên còn một cách nữa là *Event*. Từ cách này cũng nảy sinh ra 1 trường phái lập trình là *Event-driven programming*. Event là chủ đề khá dài, mình sẽ đề cập đến nó chi tiết hơn khi có dịp (không biết bao giờ :smile:)

Lâu lắm mới viết một bài dài như này LoL.

---

Tham khảo:

[1] https://shareprogramming.net/gioi-thieu-inversion-of-control-va-dependency-injection-trong-spring/

[2] https://stackoverflow.com/questions/6550700/inversion-of-control-vs-dependency-injection

[3] https://en.wikipedia.org/wiki/Inversion_of_control
