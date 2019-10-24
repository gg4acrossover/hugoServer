---
title: "Decorator và thực tiễn áp dụng"
date: 2019-10-21T02:13:23+07:00
lastmod: 2019-10-21T02:13:23+07:00
draft: false
keywords: ["swift","design pattern"]
description: ""
tags: ["swift","design pattern"]
categories: ["swift"]
author: "VietHQ"
---

Chào mọi người, hôm nay mình sẽ giới thiệu về một pattern khá phổ biến - **decorator** . Nói một cách vắn tắt, đây là pattern thuộc nhóm *cấu trúc* và sử dụng *composition* để tạo ra đối tượng ta mong muốn một cách linh hoạt. Nghe có vẻ hơi...lý thuyết, nhưng mình sẽ đi ngay vào phần tình huống và cách thức giải quyết dựa vào **decorator** để các bạn dễ dàng hình dung hơn.

## Vấn đề gặp phải:

Giả sử công ty giao cho bạn 1 task khá to, dựa vào kinh nghiệm bản thân, bạn chia nhỏ để trị. Khi ấy bạn vừa dễ dàng báo cáo sếp, vừa dễ dàng thực thi công việc. Bạn chia task to đó thành 3 task nhỏ A, B, C. Và giờ bạn bắt đầu triển khai từng task con đó.

## Show me the code:

Bước đầu tiên, bạn định hình công việc cần làm bằng cách tạo interface.

``` swift
protocol Process {
    typealias Handler = () -> Void
    func run(completion: @escaping Handler)
}

class ProcessCommand: Process {
    func run(completion: @escaping Handler) {
        completion()
    }
}
```

Sau đấy bạn bắt đầu định nghĩa 1 lớp abstract class, có tên là *ProcessDecorator*. Nó khác *ProcessCommand* mình tạo ở trên ở chỗ: mình cần truyền vào 1 *Process*.

``` swift
class ProcessDecorator: Process {
    var process: Process
    init(process: Process) {
        self.process = process
    }
    
    func run(completion: @escaping Handler) {
        print("begin process...")
        self.process.run(completion: completion)
    }
}
```

Nếu không có biến *process* thì bạn không thể tạo ra 1 chuỗi các sự kiện để hình thành **decorator** được.

Trong pattern **decorator**, những đối tượng giống như *ProcessCommand* bá đạo nhất, nó chẳng cần nương tựa ai cả và luôn luôn đứng đầu.

Hình dung một cách đơn giản, nếu bạn hay đi tập thể dục buổi sáng ở bờ Hồ, bạn sẽ thấy các cụ đứng đấm lưng cho nhau, người sau đấm cho người trước. Người đầu tiên chẳng phải làm gì cả (ProcessCommand), các người phía sau đấm lưng là decorator :v. Nói đến đây lại thèm 1 bữa đi bộ trên bờ Hồ sáng sớm.

Tiếp đến thì đơn giản thôi, tạo decorator từ lớp abstract class *ProcessDecorator*

``` swift
class ProcessA: ProcessDecorator {
    override func run(completion: @escaping Handler) {
        super.process.run{
            print("A")
            completion()
        }
    }
}

class ProcessB: ProcessDecorator {
    var conditionDoB: (() -> Bool)?
    
    func isNeedToDo() -> Bool {
        return self.conditionDoB?() == true
    }
    
    override func run(completion: @escaping Handler) {
        super.process.run{
            if self.isNeedToDo() {
                 print("B")
            } else {
                print("ko lam nua")
            }
           
            completion()
        }
    }
}

class ProcessC: ProcessDecorator {
    override func run(completion: @escaping Handler) {
        super.process.run{
            print("C")
            completion()
        }
    }
}
```

Thằng ProcessB mình đặt thêm điều kiện để tăng độ khó cho game :D. Tức là nó có thể thực thi hoặc không, tùy điều kiện truyền vào.

Bây giờ ta cần nối các task con A, B, C thành một task to.

``` swift
let decorator = ProcessDecorator(process: ProcessCommand())
let doA = ProcessA(process: decorator)
let doAB = ProcessB(process: doA)
doAB.conditionDoB = { false }
let doABC = ProcessC(process: doAB)
```

Khi sử dụng hàm *run* thì kết quả như sau

``` swift
doABC.run {
    print("end process...")
}

// begin process...
// A
// ko lam nua
// C
// end process...

```

Như vậy là mọi thứ đã xong. Tuy nhiên có một thứ chưa được ổn, khi áp dụng cách này, ta cần biết detail của A, B, C mà sếp thì có bao giờ để ý chi tiết từng bước đâu mà chỉ quan tâm đến kết quả cuối cùng. Để làm mịn công việc được giao, chúng ta có thể áp dụng thêm chút **Builder**

## Kết hợp với Builder:

``` swift
class ProcessBuilder {
    private(set) var process: Process
    
    init() {
        self.process = ProcessDecorator(process: ProcessCommand())
    }
    
    func createProcessA() -> ProcessBuilder {
        let doA = ProcessA(process: self.process)
        self.process = doA
        return self
    }
    
    func createProcessB() -> ProcessBuilder {
        let doB = ProcessB(process: self.process)
        doB.conditionDoB = { false }
        self.process = doB
        return self
    }
    
    func createProcessC() -> ProcessBuilder {
        let doC = ProcessC(process: self.process)
        self.process = doC
        return self
    }
    
    func build() -> Process {
        self.createProcessA().createProcessB().createProcessC().process
    }
}
```

Như vậy khi đó ta không cần phải quan tâm từng task con A, B, C. Ta chỉ chú ý đến tổng thế công việc

``` swift
ProcessBuilder().build().run {
    print("end process using builder")
}
```

## Sử dụng functional programming

Quên design pattern đi và sử dụng  function :v
Mình thích cách này, nên giới thiệu thêm cho các bạn hứng thú với functional programming.

Trước hết ta cần định nghĩa toán tử:

``` swift
precedencegroup ForwardApplication {
    associativity: left
}
infix operator |> : ForwardApplication
func |> <A,B>(_ value: A, f: (A) -> B) -> B {
    return f(value)
}
```

Sau đó để thực hiện việc A, B, C, ta có thể làm như sau:

``` swift
func createDoB(_ process: Process) -> ProcessB {
    let doB = ProcessB(process: process)
    doB.conditionDoB = { false }
    return doB
}

let doABC = ProcessCommand()
                |> ProcessDecorator.init
                |> ProcessA.init
                |> createDoB
                |> ProcessC.init

doABC.run {
    print("end process")
}
```

Nếu muốn tạo ra 1 builder, ta định nghĩa thêm 1 toán tử

``` swift
precedencegroup ForwardComposition {
    associativity: left
    higherThan: ForwardApplication
}
infix operator >>> : ForwardComposition
func >>> <A,B,C>(_ f: @escaping (A) -> B, _ g: @escaping (B) -> C) -> (A) -> C {
    return { a in
        g(f(a))
    }
}
```

Lúc đó code trông sẽ như này, kết quả tương tự

``` swift
let doABCBuilder = ProcessDecorator.init
                    >>> ProcessA.init
                    >>> createDoB
                    >>> ProcessC.init
let doAll = ProcessCommand() |> doABCBuilder

doAll.run {
    print("do all")
}
```

# Tổng kết:

Decorator có nhiều điểm lợi nhưng cũng có mặt hạn chế: 
+ Vì đây là một chuỗi các sự kiên liên tiếp nhau nên một mắt xích cần thay đổi ta cần tạo mới một chuỗi khác.
+ Để hiểu hết một chuỗi các sự kiện hoạt động ra sao cũng mất thời gian đi qua từng mắt xích.

Để hạn chế điểm yếu, tốt nhất là mọi người tạo ra 1 chuỗi sự kiện gồm 4, 5 sự kiện con đi kèm thôi :).
Như vậy việc áp dụng **decorator** sẽ rất tuyệt vời.
Trong dự án thực tế, có thể áp dụng **decorator** cho network layer, chia nhỏ thành các đối tượng xử lý error, show/hide indicator,... 

Một fun fact là sử dụng functional programming thì mọi thứ đều là function :v. Thế nên, theo mình biết là không có khái niệm design pattern trong functional programming. Có thể coi ví dụ về functional là câu chuyện ngoài lề để trà đá chém gió :P.

