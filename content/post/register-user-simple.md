+++
author = ""
comments = true
date = "2017-08-09T16:41:23+07:00"
draft = true
image = ""
menu = ""
share = true
slug = "post-title"
tags = ["tag1","tag2"]
title = "register user simple"

+++

### Giới thiệu

Việc xác thực quá trình đăng kí mới user là công việc mà coder nào cũng phải gặp, thậm chí nó quen thuộc đến mức như cầm đũa hàng ngày vậy. Chúng ta có thể thực hiện việc validate thông tin ngay chính màn hình đăng ký, tuy nhiên cách này khó test và sẽ khó quản lý nếu có nhiều điều kiện đầu vào. Cách hay hơ là chúng ta có thể tách ra 1 object validator nhận thông tin đầu vào và trả ra thông báo thành công nếu thông tin nhập vào đúng, trả ra mã lỗi nếu thông tin nhập vào sai. 

Mình sẽ làm một ví dụ đơn giản thực hiện việc đăng ký mới user với các điều kiện: xác thực format email, xác thực password lớn hơn 8 ký tự có ít nhất 1 kí tự dạng số và ít nhất 1 ký tự dạng chữ và code test cho việc validate.

### Chuẩn bị toy
Mình tạo một enum Error chứa các case lỗi có thể phát sinh, ở đây để đơn giản chỉ có 2 case. Tiếp theo là một protocol hỗ trợ check thông tin và trả ra mã lỗi, tất cả các object validator sẽ implement protocol này.

{{< highlight objc "style=monokai" >}}
enum ONRegisterUserError : Error {
    case invalidEmail
    case invalidPassword
}

protocol ONRegisterUserProtocol {
    var error: ONRegisterUserError { get }
    func isValid() -> Bool
}
{{< /highlight >}}

### Xác thực Email

Mình sẽ bắt đầu với việc xác thực format email, sử dụng protocol ONRegisterUserProtocol như ở trên

{{< highlight objc "style=monokai" >}}
class ONEmailValidateItem: ONRegisterUserProtocol {
    var error: ONRegisterUserError = .invalidEmail
    private let email: String
    
    init(email: String) {
        self.email = email
    }
   
    func isValid() -> Bool {
        let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
        let emailTest = NSPredicate(format: "SELF MATCHES %@", emailRegEx)
        return emailTest.evaluate(with: self.email)
    }
}
{{< /highlight >}}

Để nhanh chóng thì ta sử dụng NSPredicate và regEx cho việc filter (regEx thì copy qua google xài luôn thôi :D).
check password thì cũng tương tự thôi, câu lệnh RegEx cho ta filter password với 8 kí tự trở lên, ít nhất 1 ký tự số, 1 ký tự chữ.

{{< highlight objc "style=monokai" >}}
class ONPasswordValidateItem: ONRegisterUserProtocol {
    var error: ONRegisterUserError = .invalidPassword
    let password: String
    
    init(password: String) {
        self.password = password
    }
    
    func isValid() -> Bool {
        let passRegEx = "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{8,}$"
        let passTest = NSPredicate(format: "SELF MATCHES %@", passRegEx)
        return passTest.evaluate(with: self.password)
    }
}
{{< /highlight >}}

OK, chúng ta đã có 2 class thực thi chức năng validate riêng biệt. Tiếp theo, mình xây dựng đối tượng đảm nhận việc quản lý các validator.





