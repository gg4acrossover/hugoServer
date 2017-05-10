+++
author = "VietHQ"
comments = false
date = "2016-09-05T16:21:13+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "c-plus-11-nhung-cai-hay-dung"
tags = ["c++"]
title = "C++11 những cái hay dùng"

+++
Bài tham khảo từ link: 

> [http://www.codeproject.com/Articles/570638/Ten-Cplusplus-Features-Every-Cplusplus-Developer](http://www.codeproject.com/Articles/570638/Ten-Cplusplus-Features-Every-Cplusplus-Developer)

# Sử dụng auto

Trong C++11 từ khóa auto được dùng để compiler có thể tự nhận diện type của dữ liệu đầu vào thông qua rvalue

VD:

```
auto a = 1; // is equal with int a = 1
auto a = 1f // is equal with float a = 1.f
auto a = new foo(); // is equal with *a = new foo();
```

Có một lưu ý là khi sử dụng auto cho biến bất kì thì ta luôn phải initialize cho biến đó.
VD: > auto a; // error

# Sử dụng nullptr

Để gán giá trị null cho con trỏ

```
int *p = nullptr;
```

# ForEach
Duyệt tất cả các phần tử của mảng
VD:

```
int arr[5] = {1, 2, 3, 4, 5};
for( auto &a : arr)
{
    cout<< a << endl; // in ra màn hình 1, 2, 3, 4, 5
}
```

# Override và final

Cái này là một định danh.
Giả sử với vd sau:

```
class B
{
public:
    virtual void f(short)
    {
        std::cout << "B::f" << std::endl;
    }
};

class D : public B
{
public:
    virtual void f(int)
    {
        std::cout << "D::f" << std::endl;
    }
};
```

Trường hợp này, function f ở class D là overload (vì param truyền vào là khác kiểu nhau).
Hoặc với vd khác:

```
class B
{
public:
    virtual void f(int) const
    {
        std::cout << "B::f " << std::endl;
    }
};

class D : public B
{
public:
    virtual void f(int)
    {
        std::cout << "D::f" << std::endl;
    }
};
```

Ở trường hợp này function f ở class D vẫn là overload chứ ko phải là override. Để tránh gây nhầm lẫn, C++11 thêm định danh là override và final giống như trong java. 2 từ khóa này có thể đi liền nhau "override final"

```
class B
{
public:
    virtual void f(short)
    {
        std::cout << "B::f" << std::endl;
    }
    void g(short)
    {
        std::cout << "B::g" << std::endl;
    }
};

class D : public B
{
public:
    virtual void f(int) override
    {
        std::cout << "D::f" << std::endl;
    }
    virtual void g(int) override
    {
        std::cout << "D::g" << std::endl;   // error
    }
};
```

Lưu ý: override chỉ có tác dụng nếu function ở class base là virtual.

# Enum class

VD có 2 enum

```
enum TV { on, off};
enum LAMP { on, off};
```

Cả 2 enum này đều có key giống nhau, dẫn đến compiler không phân biệt
được. Cách giải quyết ở phiên bản cũ là dùng namespace, tuy nhiên ở
C++11 cái này đơn giản hơn bằng cách sử dụng enum class như sau:

```
enum class TV
{
    on, off
};
enum class LAMP
{
    on, off
};

TV::on;
LAMP::on; // no error
```

# Lambda

Đây là điểm mới mà C++03 không có

Cấu trúc nó là *capture-list* { body };

VD:

```
int a = 10;
auto func1 = [=]()
{
    int x = a;
    std::cout<< x << std::endl;
    //a += 10; // error
};
auto func2 = [&]()
{
    a += 10;
};

func1(); // x = 10;
func2(); // a = 20;
```

[&] sẽ giúp complier hiểu được là biến a được khai báo
bên ngoài lambda function sẽ được tham chiếu vào trong lambda function.
Còn [=] có nghĩa là copy giá trị của a vào trong hàm lambda, tuy nhiên
không thể thay đổi giá trị của a. (chỉ có tác dụng copy).

