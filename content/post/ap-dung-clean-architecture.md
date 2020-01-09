---
title: "Áp dụng clean architecture"
date: 2020-01-09T11:41:03+07:00
lastmod: 2020-01-09T11:41:03+07:00
draft: false
keywords: ["Architecture"]
description: ""
tags: ["Architecture"]
categories: ["Architecture"]
author: "VietHQ"

---

# Áp dụng Clean Architecture trong lập trình mobile

## Ngoại truyện

    - Tại sao bạn làm theo mô hình này
    - Vì uncle Bob nói zậy

Đã từ rất lâu rồi, giới lập trình phân chia làm 2 thái cực. Một bên theo tư tưởng của bác Bob, một bên theo Martin Fowler. Tôi đã tìm thấy khá nhiều thứ hay ho trong các giáo án của hai bác ấy thông qua những tờ giấy...gói xôi :D. Bác Bob là tác giả của một series, bác ưa sạch sẽ nên cuốn nào cũng mở đầu bằng chữ "sạch" (VD: Clean coder), đồng thời bác cũng là fan cuồng theo trường phái TDD. Martin Fowler không kém cạnh, tác giả cuốn refactor kinh điển, sách của bác ấy bán rất chạy và bác theo trường phái tất cả bắt nguồn từ architecture.

Hai ông này có thể ví như hai tượng đài, lời nào thốt ra thì developer note lại câu đấy. Tuy nhiên, hai ông cũng rất thích cà khịa lẫn nhau. Một ông bảo TDD là vớ vẩn, phá vỡ architecture, tất cả đều phải bắt nguồn từ cái tổng quan rồi mới đến cái tiểu tiết bên trong. Một ông khăng khăng di từ cái nhỏ mới xây dựng được cái to. Thậm chí agile cũng được hai ông lôi ra đá đểu nhau.

Về TDD https://twitter.com/martinfowler/status/460182896817733632

Về Agile https://twitter.com/unclebobmartin/status/1034532580798787584?lang=en

Hai ông cà khịa nhau là một câu chuyện vui để hóng, cũng có thể hai ông chỉ tung hứng cho nhau thôi. Còn với chúng ta những lập trình viên, học lóm được cái hay cái đẹp từ những ông ấy là ok rồi.

## Mô tả ban đầu

Bác Bob nói về mô hình *Clean architecture* ở mức khá trừu tượng, thế nên mỗi người tư duy theo một cách khác nhau. Ngay cả đối với mô hình VIPER lấy cảm hứng từ clean *architecture* áp dụng vào trong lập trình mobile, cũng có nhiều cách diễn tả về nó. Thôi thì giờ mình cũng nói theo cách của mình vậy :D.

Đầu tiên hãy nói về cách phân tầng, chính là cái vòng tròn nhiều lớp này

https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html

Nguyên tắc quan trọng nhất là tầng bên ngoài biết về tầng bên trong, nhưng ngược lại thì không được, để đảm bảo giảm thiểu sự phụ thuộc lẫn nhau.

Như đã nói ở trên, VIPER là mô hình lấy cảm hứng từ *Clean architecture* và nó cũng là mô hình đầu tiên có gắn *Router* vào trong. Bạn cũng có thể tạo *Router* trong mô hình khác như MVC, MVP, tuy nhiên từ *Router* không được nhắc đến ngay ở bản thân mô hình đó.

Để áp dụng *Clean architecture* vào trong lập trình mobile, mình sẽ phân chia tách biệt phần mềm thành những module con bên trong, theo từng lớp từ trong ra ngoài như sau:

## Common:

Để gắn log, validator impl, operation overload,...
Nói chung là tất cả những cái có thể dùng chung ở các tầng tiếp theo và không liên quan đến UI.

## Domain:

Tầng domain chứa *Entity*, cái này có thể hiểu đơn giản là 1 model chứa một số logic nhỏ bên trong. Giả sử bạn call 1 api và sử dụng response để hiển thị dữ liệu lên app. Khổ một nỗi api này public cho nhiều tình huống sử dụng khác nhau nên đối với app của bạn, dữ liệu response trả về có rất nhiều thông tin thừa, không cần thiết. Bạn phải lựa những cái mình dùng rồi gói gém nó vào 1 model trong app của bạn. Model đó chính là *Entity*.

*Entity* có thể chứa một số logic con, tỉ như format date, gộp firstname, lastname thành username,...blah blah. Diễn đạt theo một cách ngắn gọn nhất, màn hình bạn cần thông tin gì để hiện thị lên thì đấy chính là *Entity*.

Để lấy được *Entity*, ta cần một *Entity Gateway*, có người sẽ gọi là *Provider*. Áp vào trong phần mềm, *Entity Gateway* có thể là Network interface phục vụ lấy dữ liệu. Interface này sẽ được implement ở tầng tiếp theo

Domain sẽ có: *Entity* và *Entity Gateway(interface)*

## Platform:

Tầng này sẽ chứa các đối tượng implement *EntityGateway* như đã nói ở phía trên. Đúng như mô hình clean architecture, tầng dưới sẽ chìa ra interface cho tầng trên implement để giao tiếp với bên dưới.

Trong tầng này chứa *Usercase* interface, có chỗ sẽ gọi là *Interaction*. *Usercase* sẽ định nghĩa logic nghiệp vụ để tầng tiếp theo bên ngoài thực thi, bản thân nó sẽ chứa một *EntityGateway implement* để lấy ra entity cần thiết, đưa qua cách kênh phân phối để hiển thị lên app.

Để chìa output ra bên ngoài thì *Usercase* sử dụng interface *Output*, tạm gọi là *UsercaseOuput*. Mục đích để cho usercase có thể giao tiếp với bên ngoài. *Presenter* của chúng ta sẽ implement cái *UsercaseOuput* và sử dụng ở tầng bên ngoài cùng, mình gọi là *App*

Như vậy tầng Platform sẽ có: Usercase interface, Gateway/Provider/Network/Responsitory implement

## App:

Tầng này chứa các thứ liên quan đến UI như View, các common của view, layout cho view, presenter và các wrapper cho thư viện bên thứ 3. Ngoài ra quan trọng không kém nó sẽ implement *UsercaseOuput* bằng cách tạo ra đối tượng *Presenter*.

*Presenter* sẽ là cầu nối giữa phần còn lại của thế giới với những logic nghiệp vụ ở tầng dưới. Đồng thời nó sẽ chứa cả router/coordinator để điều hướng view.

Tầng App: ...phần còn lại của thế giới.

Đây là cách phân tầng mà mình đang sử dụng. Một app sẽ được chia module theo các tầng như trên để dễ dàng tái sử dụng các thành phần của app.

## Tổng kết:

Túm váy lại, gói gọn nó vào trong 1 vài từ:

- Tầng Application:
    + Presenter implement
    + Router interface, Router implement
    + ViewOutPut interface
    + ViewOutPut implement / UI
- Tầng Platform:
    + Gateway / Network/Provider implement
    + Usercase interface
    + Output / Presenter interface
- Tầng Domain:
    + Entity
    + Provider / Gateway interface
- Tầng Common: Chứa những cái dùng chung không liên quan đến UI

Áp dụng nhiều tầng nhiều lớp thì việc triển khai ban đầu sẽ phức tạp nhưng dễ dàng cho việc maintain và tái sử dụng code, cả cho test nữa. Thế nên nó nên được sử dụng cho môi trường doanh nghiệp, nơi có business logic phức tạp hoặc bạn tự làm pet project để học :D. Ngoài ra thì cứ áp dụng phương châm 

> Make MVC great again

Hoặc dùng cái này https://iosarchitecture.top/

