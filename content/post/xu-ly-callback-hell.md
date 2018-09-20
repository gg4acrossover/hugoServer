---
title: "Xử lý callback hell với functional programming"
date: 2018-09-19T00:28:59+07:00
lastmod: 2018-09-19T00:28:59+07:00
draft: false
keywords: []
description: ""
slug: "xu-ly-callback-hell"
tags: ["swift","functional programming"]
categories: []
author: ""

---

## Giới thiệu

*Callback* là kĩ thuật được ưa chuộng trong lập trình hiện nay. Ngặt nỗi, nếu sử dụng không khéo rất dễ xảy ra *callback hell*. Dưới con mắt nghệ thuật, nó lai lái kim tự tháp, nhưng dưới con mắt coder, đặc biệt với những lập trình viên ưa-cái-đẹp thì nó chẳng khác gì một thảm họa. Bí kíp tránh *callback hell* không phải là không có. Nếu chính đạo dùng *Promise* thì tà đạo chơi *functional programming*.

Dạo này mình được khai sáng một chút tà đạo nên mình sẽ dùng nó để giải quyết bài toán *callback hell*

## Ví dụ

Để thấy được kĩ thuật này hữu dụng chỗ nào, mình đưa ra bài toán nho nhỏ và từng bước giải quyết vấn đề theo cả cách truyền thống lẫn cách hiện dại.
Giả sử ta có *String* như sau

![image1](/hugosite/images/note/callback-hell/json-string.png)

Ta cần lấy ra mảng *name* để hiển thị ra màn hình. Bài toán này cũng tương tự việc bạn lấy dữ liệu từ server, parse dữ liệu và hiển thị lên màn hình.

## Tiền xử lý

Ta tạo ra các phương thức và model sử dụng chung cho cả 2 cách

![image2](/hugosite/images/note/callback-hell/define-func.png)

*CompletionHandle* được mình dùng để thực hiện việc callback xuyên suốt bài viết. Đây là *function* với tham số đầu vào dạng enum Result (tương tự như *Result* trong *Alamofire*).

Ba phương thức được tạo ra lần lượt thực hiện các thao tác:

+ *convertUserDataFromString* -> convert *String* thành *Data*
+ *getUsers* -> parse *Data* sang mảng User model
+ *getAllName* -> lấy ra name từ mảng User 

## Cách truyền thống

Ứng với ba phương thức kể trên, ta call lần lượt các function, tạo thành ba callback lồng nhau.

![image3](/hugosite/images/note/callback-hell/callback-hell.png)

Cách làm này quá quen thuộc nên mình không giải thích gì thêm.

## Functional programming

Với cách này, ta phải tư duy theo hướng hoàn toàn mới: đi từ trừu tượng đến cụ thể, chứ không phải từ cụ thể đến trừu tượng (nghe giống công nghệ giáo dục đang áp dụng :D). 

Đầu tiên ta định nghĩa toán tử mới để kết hợp hai *function* thành một function duy nhất thực hiện cả hai nhiệm vụ. Điều này không những làm code gọn hơn, mà còn trông nguy hiểm hơn, nói cách khác *functional style* hơn.

![image4](/hugosite/images/note/callback-hell/define-operator.png)


Sức mạnh của **"~>"** thể hiện rõ qua dòng code sau:

![image4](/hugosite/images/note/callback-hell/combinator.png)

Trong đó

+ **convertUserDataFromString** là function dạng <span style="color:green">**(Result\<Data\> -> Void) -> Void**</span>. có thể đưa về trường hợp tổng quát **(A) -> Void**
+ **getUsers** là function dạng <span style="color:green">**(Data, Result<[User]> -> Void) -> Void**</span> có thể đưa về trường hợp tổng quát **(A, B) -> Void**
+ **getAllName** là function dạng <span style="color:green">**([User], Result<[String]> -> Void) -> Void**</span> có thể đưa về trường hợp tổng quát **(B, C) -> Void**

Function **getNames** là tổng hợp của ba function trên, có thể biểu diễn dưới cách sau:

![image4](/hugosite/images/note/callback-hell/combinator-function.png)

Từ trường hợp cụ thể trên, ta qui ra trường hợp tổng quát cho function **"~>"**.

**"First"** đưa về trường hợp tổng quát là (A) -> Void, nhiệm vụ của function này là nhận dữ liệu input, chuyển nó thành A rồi đưa vào trong callback (giống function *convertUserDataFromString* định nghĩa ở trên)

**"Second"** đưa về trường hợp tổng quát là (A, B) -> Void, có nhiệm vụ transform dữ liệu A nhận từ **First** thành B và đưa vào trong callback (giống function *getUser* hoặc *getAllName* định nghĩa ở trên)

**"~>"** thì return (B) -> Void, đây là callback đưa ra output B sau khi đã đi qua bước chuyển đổi A thành B.

Kết hợp ((A) -> Void) -> ((A, B) -> Void) thành ((B) -> Void) có thể viết rút gọn thành **(A) -> (A,B) -> (B)**.
Ta có thể hiểu theo kiểu: dữ liệu (A) đi qua một bước biến đổi (A,B) thì có (B) để xài.

Cái hay của function **"~>"** là đầu ra của nó có thể sử dụng làm đầu vào của chính **"~>"** dưới dạng **First** param, ta chỉ cần ghép nó với một **Second** hợp lệ là có thể tạo thành một chuỗi các function lồng nhau. 

Bước 1: **(A) -> (A,B) -> (B)**

Bước 2: **(B) -> (B,C) -> \(C\)**

Qui nạp thành bước N: **(N1) -> (N1,N2) -> N2**

Dùng function **~>** giúp ta trải phẳng callback hell và đưa nó về dạng gần với ngôn ngữ tự nhiên:

> Công việc D = công việc A + công việc B + công việc C.

Chúng ta có thể giải quyết bài toán theo hướng khác cũng sử dụng *functional programming*. Mình sẽ trình bày ở bài tiếp theo :). See ya!

---

Tham khảo [slide của Vincent-pradeilles](https://github.com/vincent-pradeilles/slides/blob/master/nsspain-2018-solving-callback-hell-with-good-old-function-composition.pdf)
