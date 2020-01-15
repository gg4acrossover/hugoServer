---
title: "Một số tip cho ngày cuối năm"
date: 2020-01-16T00:33:52+07:00
lastmod: 2020-01-16T00:33:52+07:00
draft: false
keywords: ["Share"]
description: "Share"
tags: ["Share"]
categories: ["Share"]
author: "VietHQ"
---

Lâu rồi không viết các tip nho nhỏ, hôm nay tranh thủ lúc tâm tư thư thái, viết một bài.

## Git

Chẳng mấy khi viết về các lệnh git.

+ Check remote đến responsitory nào

> git remote -v

+ Add thì đơn giản hơn

> git remote add origin <link>

+ Xóa thì sao, trước hết cần check xem đang remote đến đâu mới biết mà xóa

> git remote rm origin

+ Check xem ssh đến git đã hoạt động ok chưa

> ssh -T git@github.com

## iOS

Composite là một kĩ năng hay được áp dụng trong lập trình, chắc ai cũng nghe đến *composite over inheritance*. Nếu chưa nghe thì nên search thêm về từ khóa này mà học

VD: cần call nhiều API

``` swift
protocol Task {
    func run(_ completion: @escaping (_ task: Task) -> Void)
}
```

Tạo composite

``` swift
class CompositeTask: Task {
    
    let tasks: [Task]
    
    init(_ tasks: [Task]) {
        self.tasks = tasks
    }
    
    func run(_ completion: @escaping (Task) -> Void) {
        let group = DispatchGroup()
        
        for task in tasks {
            group.enter()
            
            task.run({ task in
                group.leave()
            })
        }
        
        group.notify(queue: DispatchQueue.main) {
            completion(self)
        }
    }
}
```

## Vài lời chia sẻ

Đây có lẽ là bài viết cuối cùng cho năm cũ, nếu hứng thì mình sẽ viết thêm :D. Năm nay mình viết blog tương đối đều tay, nói chung cũng gần được 1 bài/ tháng.

Đầu tư viết blog về tech không phải là điều đơn giản, trung bình một bài trong một tháng đối với mình là ổn. Nhất là khi lập trình không phải là tất cả, cuộc sống của bạn còn có những thứ khác cần lưu ý nữa.

Năm này sức khỏe của mình không được tốt lắm, mình nhận thấy rằng tạo sự cân bằng giữa sức khỏe và công việc là điều cần thiết. Có những lúc mình phá sức code đến tối muộn và sáng hôm sau thì bơ phờ. Vậy nên năm nay mình sẽ làm việc khoa học hơn, nó thực sự rất cần thiết với mình. (Cảm thấy khâm phục những thằng 1 tuần ra 1 bài viết :D, bám sát plan đề ra và vẫn giữ sức khỏe tốt).

Chúc bạn nào đọc blog mình một năm mới an lành, code đẹp :D.
