+++
date = "2016-09-05T15:51:57+07:00"
draft = false
title = "Enum flag c++"
image = ""
slug = "Enum-flag-c-plus-plus"
tags = ["c++"]
comments = false

+++
Hôm nọ đọc trên DNH có bài hướng dẫn về bit field

> [Daynhauhoc](http://www.wikiwand.com/en/Bit_field)

Trong bài đó cũng có phần nói về bit operators. Cái này trong IOS áp dụng tương đối nhiều, nên hôm nay mình sẽ viết 1 cái tip nho nhỏ về bitwise

# Bitwise

Sử dụng bitwise trong lập trình nói chung làm cho code của bạn trở nên huyền bí, khó đọc. Tuy nhiên cũng có một số trường hợp áp dụng nó sẽ khiến code trông ngắn gọn, sáng sủa. Giả sử bạn làm game, nhân vật của bạn có thể có nhiều hơn một thuộc tính như CHAO, NORLMAL, MAGIC...Bạn sẽ tạo mấy biến để lưu trữ thuộc tính nhân vật?

Đây là một trường hợp tuyệt vời để áp dụng enum flag, khi đó ta chỉ cần một biến duy nhất để lưu trữ thuộc tính, tránh code rườm rà.
Giống như cách define sau

```
#define KEY_UP       (1 << 0)  // 000001
#define KEY_RIGHT    (1 << 1)  // 000010
#define KEY_DOWN     (1 << 2)  // 000100
#define KEY_LEFT     (1 << 3)  // 001000
#define KEY_BUTTON1  (1 << 4)  // 010000
#define KEY_BUTTON2  (1 << 5)  // 100000
```

Tuy nhiên mình sẽ không dùng define mà dùng enum làm cờ.

```
enum FLAG
{
         UNKNOW    = 0, 
         NORMAL    = 1 << 0,
         MAGIC     = 1 << 1,
         CHAO      = 1 << 2
};
```

Hàm main

```
int main(int argc, const char * argv[])
{
         FLAG flag = MAGIC | CHAO;
   
         if( flag & NORMAL)
         {
         	cout<<"Normal"<<endl;
         }   
   
         if( flag & CHAO)
         {
   		cout<<"Chao"<<endl;
         }
   
         return 0;
};
```

Tuy nhiên nếu bạn chạy ngay code này thì sẽ lỗi. Ta cần phải overload toán tử \| (OR)

```
inline FLAG operator|(FLAG a, FLAG b)
{
    return static_cast<FLAG> ( static_cast<int>(a) | 
                               static_cast<int>(b));
};
```

OK, bây giờ code đã có thể chạy được. Giải thích một chút nhé:
Khi ta thực hiện phép toán | (OR) giữa CHAO và MAGIC thì ta được (gán cờ để nhân vật mang 2 thuộc tính)

```
MAGIC             00000010 
CHAO              00000100 
MAGIC | CHAO      00000110
```

Khi ta thực hiện phép toán & giữa CHAO và (MAGIC \| CHAO) thì ta được (kiểm tra cờ được gán)

```
MAGIC | CHAO      	00000110 
CHAO              	00000100
CHAO & (MAGIC | CHAO)   00000100
```

Kết quả giống như biến CHAO thế nên kết quả của bài trên là hiện ra CHAO.
Để biết ta gắn những cờ nào thì FLAG sẽ thực hiện phép & với cờ đó.

P/S: Lập trình ios thì các bạn sẽ thấy cái này khá nhiều.

