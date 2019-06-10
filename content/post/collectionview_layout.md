---
title: "Collection view layout"
date: 2019-06-10T10:40:29+07:00
lastmod: 2019-06-10T10:40:29+07:00
draft: false
keywords: ["swift","collectionview","frame","bounds","basic"]
description: "swift"
tags: ["swift"]
categories: ["swift"]
author: "VietHQ"

---

# CollectionView hoạt động như nào

*UICollectionView* là subclass của UIScrollView. Nói một cách khác, nó là *UIView* nhưng có thêm chức năng thay đổi *bounds*. Khi chúng ta di chuyển cell, chúng ta thay đổi giá trị contentOffset, đồng thời thay đổi không gian hiển thị của *UICollectionView*.

Trăm nghe không bằng một thấy, tốt nhất là trình diễn bằng hình ảnh cho dễ hiểu.

![img1](/hugosite/images/note/collectionview/scroll_1.gif)

Trên là hình ảnh mắt chúng ta nhìn thấy khi di chuyển collection view (ô chữ nhật xanh lá cây tượng trưng cho màn hình điện thoại). Tuy nhiên cuộc sống không đơn giản như cách chúng ta tưởng tượng.

![img](/hugosite/images/note/collectionview/scroll2.gif)

Trên thực tế, các cell không di chuyển, phần di chuyển là ô chữ nhật xanh lá. Ô xanh lá cây lúc này chính là *bounds* của collection view. Đối với các view bình thường, *bounds* và *frame* không khác nhau, tuy nhiên ở collectionview, hay các class kế thừa *UIScrollView* nói chung, khi ta thực hiện thao tác vuốt, ta thay đổi giá trị *origin* của *bounds* (*frame* không thay đổi). Thông số origin này chính là *contentOffset* mà chúng ta hay sử dụng.

*UICollectionView* là ví dụ sinh động nhất cho sự khác nhau giữa *frame* và *bounds*. Giải thích kĩ hơn chút, *frame* định nghĩa khung hình của đối tượng trên hệ trục tọa độ xác định dựa trên superview của chính đối tượng đó. Còn *bounds* cũng tương tự nhưng hệ trục tọa độ lại xác định trên chính đối tượng đang xét.

Như vậy để tạo collection view layout, chúng ta cần chú ý đến 2 điều:

- Tính toán vị trí các cell.
- Tính toán vị trí của bounds.

# Tính toán layout

Tính toán layout các cell của collection view không đơn giản như cách chúng ta gắn frame cho từng view. Thay vào đó, chúng ta sử dụng một mảng *UICollectionViewLayoutAttributes* (xem ra quá nhiều tham số :D).

![img](/hugosite/images/note/collectionview/attributes.png)

Bi ơi đừng sợ, để làm quen, chúng ta sẽ tiếp cận với trường hợp đơn giản nhất...tạo cell như *UITableView* sử dụng *UICollectionViewLayout*. 

Có 4 bước quan trọng:

- Tính toán frame của tất cả các cell qua hàm *prepare*.
- Tính toán visible cells bằng phương thức *layoutAttributesForElements(in:)*
- Trả về thông số của các cells qua phương thức *layoutAttributesForItem(at:)*
- Trả về size của collection view qua *collectionViewContentSize*

Tựu chung 4 hàm trên cũng chỉ phục vụ 2 công việc: tính toán frame cells và bounds.
Trước hết tạo class hỗ trợ tính toán các bước trên

``` swift
class TableLayoutCache {

    // MARK: - Calculation
    func recalculateDefaultFrames(numberOfItems: Int) {
        defaultFrames = (0..<numberOfItems).map {
            defaultCellFrame(atRow: $0)
        }
    }

    func defaultCellFrame(atRow row: Int) -> CGRect {
        let y = itemSize.height * CGFloat(row)
        let defaultFrame = CGRect(x: 0, y: y,
                                  width: collectionWidth,
                                  height: itemSize.height)
        return defaultFrame
    }

    // MARK: - Access
    func visibleRows(in frame: CGRect) -> [Int] {
        return defaultFrames
            .enumerated() // Index to frame relation
            .filter { $0.element.intersects(frame)} // Filter by frame
            .map { $0.offset } // Return indexes
    }

    var contentSize: CGSize {
        return CGSize(width: collectionWidth,
                      height: defaultFrames.last?.maxY ?? 0)
    }

    static var zero: TableLayoutCache {
        return TableLayoutCache(itemSize: .zero, collectionWidth: 0)
    }

    init(itemSize: CGSize, collectionWidth: CGFloat) {
        self.itemSize = itemSize
        self.collectionWidth = collectionWidth
    }

    private let itemSize: CGSize
    private let collectionWidth: CGFloat
    private var defaultFrames = [CGRect]()
}
```

Công việc trở nên đơn giản hơn đối với *custom layout* mà ta tạo ra ngay sau đây

``` swift
class TableLayout: UICollectionViewLayout {
    var itemSize: CGSize = .zero  {
        didSet {
            invalidateLayout()
        }
    }
    
    private let section = 0
    var cache = TableLayoutCache.zero
    
    // required
    override var collectionViewContentSize: CGSize {
        return cache.contentSize
    }
    
    // required
    override func prepare() {
        super.prepare()
        
        let numberOfItems = collectionView!.numberOfItems(inSection: section)
            
        cache = TableLayoutCache(itemSize: itemSize,
                                 collectionWidth: collectionView!.bounds.width)
        cache.recalculateDefaultFrames(numberOfItems: numberOfItems)
    }
    
    // required
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        let indexes = cache.visibleRows(in: rect)
        
        let cells = indexes.map { (row) -> UICollectionViewLayoutAttributes? in
            let path = IndexPath(row: row, section: section)
            let attributes = layoutAttributesForItem(at: path)
            return attributes
        }.compactMap { $0 }
        
        return cells
    }
    
    // required
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        
        let attributes = UICollectionViewLayoutAttributes(forCellWith: indexPath)
            attributes.frame = cache.defaultCellFrame(atRow: indexPath.row)
        
        return attributes
    }
}
```

Sau các bước này mọi người sẽ có 1 uitableview không có header, footer :D. Nói chung cũng tạm đủ dùng. Dù sao mục đích bài này chủ yếu để chỉ ra các bước cần thiết cho một *custom layout*. Sau bài này mọi người có thể rút ra những thứ sau:

- Cách thức hoạt động của UICollectionView (UIScrollView nói chung)
- Sự khác nhau giữa *frame* và *bounds*
- Các bước cần thiết để tạo ra custom layout.

---

Tham khảo:

- [https://www.raywenderlich.com/527-custom-uicollectionviewlayout-tutorial-with-parallax](https://www.raywenderlich.com/527-custom-uicollectionviewlayout-tutorial-with-parallax)

- [https://developer.apple.com/documentation/uikit/uicollectionviewlayout](https://developer.apple.com/documentation/uikit/uicollectionviewlayout)

- [https://stackoverflow.com/questions/1210047/cocoa-whats-the-difference-between-the-frame-and-the-bounds](https://stackoverflow.com/questions/1210047/cocoa-whats-the-difference-between-the-frame-and-the-bounds)
