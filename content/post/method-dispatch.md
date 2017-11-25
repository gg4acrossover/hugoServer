+++
author = ""
comments = true
date = "2017-11-25T11:36:01+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "method-dispatch"
tags = ["swift","ios"]
title = "Method dispath trong protocol"

+++

## Method dispatch là gì?

Method dispatch là thuật toán quyết định hàm nào sẽ được chạy bởi compiler. Việc xác định hàm được chạy được thực hiện trong lúc biên dịch là static dispatch, trong lúc runtime là dynamic dispatch. Tỉ như, C sử dụng mô hình biên dịch tĩnh, javascript sử dụng mô hình biên dịch động, còn Swift thì sao? 

Nó là sự kết hợp.

>
Swift is another case of a hybrid model: its semantics provide predictability between obviously static (structs, enums, and global funcs) and obviously dynamic (classes, protocols, and closures) constructs

# Dynamic dispatch

Thuật ngữ này khá quen thuộc đối với các ngôn ngữ hỗ trợ OOP. Khi mà một phương thức ở lớp cha có khả năng overried ở lớp con.
