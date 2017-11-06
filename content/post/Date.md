+++
author = ""
comments = true
date = "2017-11-06T16:50:29+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "date-la-gi"
tags = ["swift"]
title = "Date là gì, có ăn được không?"

+++

## Date

Date là một đối tượng gần như không thể thiếu trong các app hiện nay. Nó xuất hiện trong lịch, trong thương mại điện tử, trong các app nhắc việc.
Làm việc với date tưởng chừng như đơn giản nhưng lại là vấn đề hết sức đau đầu vì nó mang tính tương đối :D. Lúc này ở Việt Nam đang là buổi chiều, nhưng ở nước khác lại là buổi sáng chẳng hạn. Có những nước lớn thời gian lại phân theo nhiều timezone khác nhau (như Nga chẳng hạn), thế nên một quốc gia cũng có thể có nhiều khung giờ khác nhau.

IOS hỗ trợ làm việc với time tương đối tốt, thậm chí cũng đã có nhiều thư viện đơn giản hóa việc này. Tuy nhiên việc hiểu bản chất của Date cũng giúp bạn suy ra cách thức vận hành của thư viện hoặc có thể tự tạo cho mình một thư viện riêng phù hợp với yêu cầu. Chúng ta gọi dễ thương đấy là *lightweight* :D

Hôm nay mình sẽ trình bày những vấn đề cơ bản nhất của Date để các bạn có được cái nhìn overview

### Now

Lấy ra thời gian hiện tại đơn giản là 
let now = Date
