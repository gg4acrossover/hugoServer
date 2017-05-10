+++
author = ""
comments = true
date = "2017-01-11T14:27:47+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "hugo-tutorial"
tags = ["hugo"]
title = "Tạo static site với hugo"

+++
Blog hiện tại của mình sử dụng hugo.

> [https://gohugo.io/](https://gohugo.io/)

Đây là một static website engine, cài đặt đơn giản, có thể deploy trực tiếp trên github. Mặc dù hugo được quảng cáo bởi hiệu năng tốt nhưng điều kéo mình đến với engine này là thư viện theme khổng lồ mà không kém phần lung linh của nó.

> [http://themes.gohugo.io/](http://themes.gohugo.io/)

Trên trang chủ đã hướng dẫn chi tiết cách tạo site và gắn theme cho website nên mình không trình bày lại nữa.
Blog mình sử dụng theme casper, dưới đây là đoạn template để chuyển page của nó.

```
<nav class="pagination" role="navigation">
	{{if .HasPrev}}
	    <a class="newer-posts" href="{{ "" | absURL -}}{{"page/"}}{{- add .PageNumber -1}}">&larr; Newer Posts</a>
	{{end}}
	<span class="page-number">Page {{ .PageNumber }} of {{.TotalPages}}</span>
	{{if .HasNext}}
	    <a class="older-posts" href="{{ "" | absURL -}}{{"page/"}}{{- add .PageNumber 1}}">Older Posts &rarr;</a>
	{{end}}
</nav>
```



