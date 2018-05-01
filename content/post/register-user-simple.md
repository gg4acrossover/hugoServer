+++
author = ""
comments = true
date = "2017-08-09T16:41:23+07:00"
draft = false
image = "images/cover/10.jpg"
menu = ""
share = true
slug = "register-user-test"
tags = ["swift3","ios"]
title = "Register user và thực hiện test"

+++

### Giới thiệu

Việc xác thực quá trình đăng kí mới là công việc mà coder nào cũng phải gặp, thậm chí nó quen thuộc đến mức như cầm đũa hàng ngày vậy. Chúng ta có thể thực hiện việc validate ngay trên viewcontroller, tuy nhiên cách này khó test và sẽ khó quản lý nếu có nhiều điều kiện đầu vào. Cách hay hơn là chúng ta có thể tách ra 1 object validator nhận thông tin đầu vào và trả ra thông báo thành công nếu thông tin nhập vào đúng, trả ra mã lỗi nếu thông tin nhập vào sai. 

Mình sẽ làm một ví dụ đơn giản thực hiện việc đăng ký mới user với các điều kiện: xác thực format email, xác thực password lớn hơn 8 ký tự có ít nhất 1 kí tự dạng số và ít nhất 1 ký tự dạng chữ và viết test cho việc validate.

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

### Xác thực và test

Mình sẽ bắt đầu với việc xác thực format email, sử dụng protocol ONRegisterUserProtocol như ở trên

{{< highlight objc "style=monokai" >}}
class ONEmailValidateItem: ONRegisterUserProtocol {
    var error: ONRegisterUserError = .invalidEmail
    private let email: String
    
    init(email: String) {
        self.email = email
    }
   
    func isValid() -> Bool {
        return true
    }
}
{{< /highlight >}}

Ở đây mình chưa implement hàm *isValid* (chỉ để 1 giá trị default là true trước), công việc trước nhất là viết unit test.
Mình tạo lớp *ONTestEmailValid* kế thừa *XCTestCase* trong thư mục [ProjectName]Tests ([ProjectName] là tên project của bạn).
Chú ý: nhớ thêm dòng @testable import [Projectname]

{{< highlight objc "style=monokai" >}}
@testable import ONRegisterUser

class ONTestEmailValid: XCTestCase {
    func testEmailInvalidCharacter() {
        let isValid = ONEmailValidateItem(email: "example@gmail,com").isValid()
        XCTAssertFalse(isValid)
    }
    
    func testEmailInvalidProvider() {
        let isValid = ONEmailValidateItem(email: "example@.com").isValid()
        XCTAssertFalse(isValid)
    }
    
    func testEmailInvalidInComplete() {
        let isValid = ONEmailValidateItem(email: "example@").isValid()
        XCTAssertFalse(isValid)
    }
    
    func testEmailInvalidUserName() {
        let isValid = ONEmailValidateItem(email: "@a.com").isValid()
        XCTAssertFalse(isValid)
    }
    
    func testEmailValid() {
        let isValid = ONEmailValidateItem(email: "example@a.com").isValid()
        XCTAssertTrue(isValid)
    }
}
{{< /highlight >}}

Khi build test (command u) thì các test case hầu hết là fail :D

![testcase](/hugosite/images/note/testcase.png)

Bây giờ mới đến bước implement hàm *isValid* để nó pass tất cả các case mình định nghĩa

{{< highlight objc "style=monokai" >}}
func isValid() -> Bool {
    let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
    let emailTest = NSPredicate(format: "SELF MATCHES %@", emailRegEx)
    return emailTest.evaluate(with: self.email)
}
{{< /highlight >}}

Để nhanh chóng thì mình sử dụng NSPredicate và regEx cho việc filter (regEx thì copy qua google xài luôn thôi :D).
check password làm tương tự như cách triển khai class *ONTestEmailValid*. 

Sau khi implement lại hàm *isValid*, chúng ta chạy lại test, Done!

![testcase](/hugosite/images/note/testcase_pass.png)

OK, chúng ta đã có 2 class thực thi chức năng validate riêng biệt. Tiếp theo, mình xây dựng đối tượng đảm nhận việc quản lý các validator. Mục đích cho việc tạo lớp này là nhận đầu vào input và đưa ra output chung cho tất cả các validator.

{{< highlight objc "style=monokai" >}}
class ONValidateRegisterUser {
    
    enum ONRegisterResult {
        case success(String)
        case fail(String)
    }
    
    let result: ONRegisterResult
    
    init(email: String, password: String) {
        let validator: [ONRegisterUserProtocol] = [ONEmailValidateItem(email: email),
                                                   ONPasswordValidateItem(password: password)]
        let invalids = validator.filter { !$0.isValid() }
        
        if invalids.count > 0 {
            self.result = .fail("Wrong format")
        } else {
            self.result = .success("Done")
        }
    }
}
{{< /highlight >}}

Mình tạo enum *ONRegisterResult* trả về case success khi cả email và pass đều validate thành công, ngược lại ra fail.
Khi đó ở *Viewcontroller* ta chỉ đơn giản làm như sau

{{< highlight objc "style=monokai" >}}
let validator = ONValidateRegisterUser(email: self.userNameTextField.text!, password: self.passwordTextField.text!)

switch validator.result {
case .success(let str):
    SHOW_ALERT(message: str, vc: self)
case .fail(let str):
    SHOW_ALERT(message: str, vc: self)
}
{{< /highlight >}}

### Tổng hợp
Tóm lược bài viết: để thực hiện việc register và TDD ta chú ý 3 việc sau:

* Tạo các lớp validator check đầu vào như username, password,...
* Tạo một lớp quản lý các validator, nhận đầu vào là các dữ liệu cần validate và trả ra result.
* Test trước khi code :D

[Source code](https://github.com/gg4acrossover/register-user-example)



