+++
author = "VietHQ"
comments = true
date = "2016-09-05T16:36:42+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "code-sach-code-kho-thoang"
tags = ["objc", "ios"]
title = "Code sạch, code khô thoáng"

+++
Đây là một số phương pháp khi tớ làm việc với obj c. Với mỗi người, tổ chức có thể khác, thế nên cái này chỉ mang tính tham khảo. :smile: 

### Sử dụng immutable

Đối với array hay dictionary nếu không cần sử dụng mutable thì ta nên viết như sau:

```
	NSArray *pFruit = @[@"orange", @"apple", @"lemon", @"strawberry"];
	NSDictionary *pDict = @{ @"name" : @"Tom",  @"age" : @18};
```

Thay vì viết:

```
	NSArray *pFruit = [NSArray arrayWithObjects:@"orange", @"apple", @"lemon", @"strawberry", nil];
	NSDictionary *pDict2 = [NSDictionary dictionaryWithObjectsAndKeys:@"value1",@"key1",@"value2", @"key2", nil];
```

Như thế code nhìn trông gọn và dễ đọc hơn, đối với các biến dạng số hoặc bool được đưa vào trong dictionary để gửi lên server ta cũng nên làm như vậy.

```
	NSNumber *pIsNew = @YES;
```

Thay vì

```
	NSNumber *pIsNew = [NSNumber numberWithBool:YES];
```

Sử dụng category

Frame được sử dụng nhiều khi viết code iOS, ta có thể tạo ra một file chứa các c function chuyên làm việc với frame, hay tạo ra một macro. Ví dụ như muốn lấy originX của 1 uiview:

```
	CGFloat ORIGIN_X( CGRect rec)
	{
            return rec.origin.x;
	}
```

Hoặc ngắn hơn với macro

```
#define posX(v)                  v.frame.origin.x
```

Tuy nhiên sử dụng category cũng là một lựa chọn thời thượng. Giả sử bạn có thể lấy originX bằng cách sau:

```
CGFloat originX = [PopUpView getX];
```

Khá ngắn gọn, clear, dễ chỉnh sửa. Category còn đặc biệt với những ai hay gửi nhận dữ liệu lên server, đôi khi dictionary sẽ trả về một cái object null. Việc check null ở nhiều chỗ khác nhau khá là tốn thời gian nếu cứ phải viết đi viết lại như thế này

```
if(dict[@"key"] && ![dict[@"key"] isEqual: [NSNull null]])
```

Ta có thể đưa vào category theo cách sau

```
- (id)safeValueForKey:(NSString *)key {
    id value = [self valueForKey:key];
     
    if (value && ![value isEqual: [NSNull null]]) {
        return value;
    }
     
    return nil;
}
```

Sau đó ở tất cả mọi nơi chỉ cần gọi

```
NSString *pValue = [dict safeValueForKey: @"key"];
```

### Sử dụng block

```
UIView *pCircle2 = ({
	UIView *pView = [[UIView alloc]
					initWithFrame:CGRectMake(80.0f, 80.0f, 90.0f, 90.0f)];
	pView.backgroundColor = [UIColor redColor];
	CAShapeLayer *shape = [CAShapeLayer layer];
	CGFloat r = CGRectGetWidth(pView.frame)*0.5f;
	CGPoint center = CGPointMake( r, r);
	UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
							    radius:r
							startAngle:0
							  endAngle:2*M_PI
							 clockwise:YES];
	shape.path = path.CGPath;
	pView.layer.mask = shape;
	pView;
});
```

Gộp các đoạn code sử dụng chung cho một mục đích vào trong 1 block, với cách này thì giá trị return tương ứng với dòng cuối trong block.

### Cách đặt tên

Nhiều tài liệu của apple cũng có nói về cách đặt tên. Mọi người hãy chú ý delegate của uitableview hay uicollectionview sẽ thấy có nhiều điểm tương đồng trong cách đặt tên hàm.
Ví dụ guide của apple [datten](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-BCIGIJJF)

Từ cách đặt tên trên ta có thể tạo ra enum như sau

```
typedef NS_ENUM(NSUInteger, GGAnimationViewType) {
    GGAnimationViewTypeFadeIn,
    GGAnimationViewTypeFadeOut,
    GGAnimationViewTypeEaseIn,
    GGAnimationViewTypeEaseOut
};
```

Công thức chung là [tên class][tên enum], cụ thể như ví dụ trên [GGAnimationView][Type].
Các giá trị enum [tên class][tên enum][các giá trị]. Các giá trị FadeIn, FadeOut,....

Tóm lại là code càng ngắn gọn càng dễ hiểu, trừ cách đặt tên biến ra :v.

