+++
author = "VietHQ"
comments = false
date = "2016-09-05T16:32:39+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "uu-tien-toan-tu"
tags = ["c++"]
title = "Ưu tiên toán tử"

+++
Hôm nay mình sẽ nói về một số cái hay nhầm lẫn khi lập trình, tập trung vào những vấn đề như trên tựa đã ghi smiley (làm cái tựa for fun tý)

```
mọi người thấy            mọi người hay nghĩ               thực tế
*p.f                      (*p).f                           *(p.f)
int *ap[]                 int (*ap)[]                     int *(ap[])
a & b != 0                (a & b) != 0                    a & (b != 0)
a << 4 + b                (a << 4) + b                    a << (4 + b)
```

!= và == sẽ được ưu tiên hơn các toán tử bitwise.
Các phép tính toán học sẽ ưu tiên hơn phép dịch bit.
Con trỏ tới mảng, và mảng con trỏ chắc gặp nhiều.

```
int main(int argc, const char * argv[])
{
    int a[] = { 2, 4, 6};
    
    // 1, con tro toi mang
    int (*pta)[] = &a;
    printf("%i \n", *(*pta +2)); // phan tu thu 3
    
    // 2, mang con tro
    int *pa[3];
    
    pa[0] = &a[0];
    pa[1] = &a[1];
    pa[2] = &a[2];
    
    printf("%i \n", **(pa+2)); // phan tu thu 3
    
    return 0;
}
```

P/S: :d vừa rồi cty mình nhận 1 dự án maintain (code c) toàn bitwise, khoai lòi mắt T_T.


