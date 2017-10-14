+++
author = ""
comments = true
date = "2017-10-14T10:34:47+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "enum-swift"
tags = ["swift","ios"]
title = "enum result"

+++

### Giới thiệu

Có nhiều cách để viết Enum hơn là làm bánh hay làm tình. Ở bài viết này mình sẽ trình bày một hướng đi, hi vọng mọi người sẽ like :D

### Kiểu phổ thông

Chắc hẳn chúng ta thấy kiểu viết enum này rất quen thuộc, đặc biệt là đối với những ai dùng alamofire

{{< highlight objc "style=monokai" >}}
enum Result<T> {
    case success(T)
    case failure(Error)
    
    public var value: T? {
        switch self {
        case .success(let v): return v
        case .failure: return nil
        }
    }
    
    public var error: Error? {
        switch self {
        case .success: return nil
        case .failure(let e): return e
        }
    }
}
{{< /highlight >}}

và để sử dụng Result ta sẽ switch - case như này:

{{< highlight objc "style=monokai" >}}
switch result {
case .success(let value):
 	// your code
case .failure(let error):
	// your code
}
{{< /highlight >}}


### Kiểu for fun

Mình không thích switch - case cho lắm, vậy nên sẽ "hack" chút theo ý mình thích.
Ở enum vừa rồi, mình sẽ thêm 2 hàm:

{{< highlight objc "style=monokai" >}}
func isSuccess(complete: @escaping (T) -> Void) -> Result<T> {
    guard let value = self.value else {
        return self
    }
    
    complete(value)
    return self
}

func `else`(complete: @escaping (Error) -> Void) {
    guard let err = self.error else {
        return
    }
    
    complete(err)
}
{{< /highlight >}}


Vậy là từ giờ mình có thể check kết quả theo cách sau

{{< highlight objc "style=monokai" >}}
let r: Result<Int> = .success(10)

r.isSuccess { (value) in
    print("show value:", value)
}.else { (err) in
    print(err)
}
{{< /highlight >}}

