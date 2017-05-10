+++
author = ""
comments = true
date = "2016-09-20T16:01:11+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "transition-animation"
tags = ["objc"]
title = "Transition Animation"

+++

### Giới thiệu

Bắt đầu từ IOS7, apple hỗ trợ tạo animation lúc chuyển viewcontroller. Quá trình chuyển đổi được hỗ trợ bao gồm:

+ NavigationController
+ TabbarController
+ Presentations và dismiss

Mình sẽ tập trung vào custom animation *push* và *pop* với *NavigationController*. Với các phương thức chuyển viewcontroller khác thì làm tương tự.

### Mô hình hóa

Giả sử bạn có 2 viewcontroller A và B, bây giờ bạn muốn push từ A sang B.

![image1](/hugosite/images/note/1.jpg)

Đầu tiên, ta cần khởi tạo *transtion* và xác định được 2 viewcontroller cần di chuyển, lúc này A đang được chứa bởi *container*(là view chứa để control việc di chuyển)

![image2](/hugosite/images/note/2.jpg)

Tiếp theo add viewcontroller B vào container đang chứa A.

![image2](/hugosite/images/note/3.jpg)

Cuối cùng là thực hiện animation chuyển đổi giữa 2 màn hình, quá trình kết thúc khi A được remove khỏi container

![image2](/hugosite/images/note/4.jpg)

### Implement

Để thực hiện được mô hình trên, ta sử dụng những protocol sau

#### UINavigationControllerDelegate 

Protocol có 2 method giúp ta xác định được cách thức transition

{{< highlight cpp "style=emacs" >}}
- (nullable id <UIViewControllerInteractiveTransitioning>)navigationController:(UINavigationController *)navigationController
                          interactionControllerForAnimationController:(id <UIViewControllerAnimatedTransitioning>) animationController NS_AVAILABLE_IOS(7_0);

- (nullable id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC  NS_AVAILABLE_IOS(7_0);

{{< / highlight >}}

Hàm đầu tiên dùng để bắt các sự kiện touch vào màn hình, trả về object implement *UIViewControllerInteractiveTransitioning*. 

Hàm thứ hai xác định cách thức push/pop khi ta click action (next/back) trên thanh navigation bar, trả về object implement *UIViewControllerAnimatedTransitioning*. Các params giúp ta xác định được *fromVC* là viewcontroller nào, *toVC* là viewcontroller nào. Ở trường hợp di chuyển từ A sang B thì A là *fromVC* và B là *toVC*.
Cuối cùng là param *operation*, đây là enum gồm có 3 value để xác định trạng thái push hay pop

{{< highlight cpp "style=emacs" >}}
 UINavigationControllerOperationNone,
 UINavigationControllerOperationPush,
 UINavigationControllerOperationPop,
{{< / highlight >}}

#### UIViewControllerAnimatedTransitioning 
có 2 phương thức

{{< highlight cpp "style=emacs" >}}
- (NSTimeInterval)transitionDuration:(nullable id <UIViewControllerContextTransitioning>)transitionContext;

- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;
{{< / highlight >}}

Để thực hiện animation ta cần 2 thông số, thông số đầu tiên là duration, hàm đầu tiên giúp ta xác định được điều đó. Phương thức thứ hai dùng để implement animation chuyển đổi. Trong hàm này ta cần đưa ra thông báo để biết được quá trình push/pop được hoàn thành


{{< highlight cpp "style=emacs" >}}
[transitionContext completeTransition:![transitionContext transitionWasCancelled]];
{{< / highlight >}}

#### Example
**Bước 1:** Tạo ra object implement protocol *UIViewControllerAnimatedTransitioning*, trước hết xác định duration

{{< highlight cpp "style=emacs" >}}
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
{
    return 0.25;
}
{{< / highlight >}}

Định nghĩa một animation đơn giản

{{< highlight cpp "style=emacs" >}}

/**
 * using example in 
 * https://www.objc.io/issues/5-ios7/view-controller-transitions/ 
 */

- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext
{
    //1.
    UIViewController* toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    UIViewController* fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];

    //2.
    [[transitionContext containerView] addSubview:toViewController.view];
    toViewController.view.alpha = 0;

    //3.
    [UIView animateWithDuration:[self transitionDuration:transitionContext] animations:^{
        fromViewController.view.transform = CGAffineTransformMakeScale(0.1, 0.1);
        toViewController.view.alpha = 1;
    } completion:^(BOOL finished) {
        fromViewController.view.transform = CGAffineTransformIdentity;

        //4.
        [transitionContext completeTransition:![transitionContext transitionWasCancelled]];

    }];

}
{{< / highlight >}}

Phân tích chút nào:

+ Bước 1: xác định toViewController và fromViewController
+ Bước 2: add toViewController.view vào trong container
+ Bước 3: thực hiện animation.
+ Bước 4: thông báo animation kết thúc qua câu lệnh *completeTransition*

**Bước 2:**
Tạo object implement protocol *UINavigationControllerDelegate*, trong object này ta cần truyền vào object đã implement *UIViewControllerAnimatedTransitioning* trước đó.

{{< highlight cpp "style=emacs" >}}
-(id<UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                 animationControllerForOperation:(UINavigationControllerOperation)operation
                                              fromViewController:(UIViewController *)fromVC
                                                toViewController:(UIViewController *)toVC
{
    // return your custom animation transition
}
{{< / highlight >}}

**Bước 3**
Setting ở viewcontroller, như ở trên ta có viewcontroller A -> B. Ở A ta cần làm như sau trước khi push.

{{< highlight cpp "style=emacs" >}}
//1.
self.navigationController.delegate = /* your custom UINavigationControllerDelegate */;
//2.
[self.navigationController pushViewController:B animated:YES];
{{< / highlight >}}

Ở bước này ta đã có thể push/pop với animation, nhưng để *full service* ta sẽ gắn thêm *interactive animations*. Tuy nhiên ở giới hạn bài viết này mình sẽ chỉ nói về animation cho push và pop. Bạn có thể tìm hiểu chi tiết hơn ở [ví dụ](https://github.com/objcio/issue5-view-controller-transitions) của trang [objc.io](https://www.objc.io/)

Một animation mình làm

![gif](https://media.giphy.com/media/l2Sq1x6bWR3i9S8sU/giphy.gif)

Bài viết có tham khảo

+ [https://www.objc.io/issues/5-ios7/view-controller-transitions/](https://www.objc.io/issues/5-ios7/view-controller-transitions/)
+ [http://whoisryannystrom.com/2013/10/01/View-Controller-Transition-Orientation/](http://whoisryannystrom.com/2013/10/01/View-Controller-Transition-Orientation/)




