---
title: "Tip iOS cho tháng năm"
date: 2019-05-23T10:09:24+07:00
lastmod: 2019-05-23T10:09:24+07:00
draft: false
keywords: []
description: ""
tags: ["swift"]
categories: ["swift"]
author: "VietHQ"
---

# CSS string

Nếu ta muốn sử dụng style có sẵn trên web thì có thể nghiên cứu phương án tận dụng nó lại trên iOS.

Bước đầu implement phương thức gắn css style vào string

``` swift
extension NSAttributedString {
    convenience public init(text: String, styles: [String: String]) {
        let style = styles.compactMap { (key, value) -> String in
            return "\(key): \(value)"
            }.joined(separator: ";")
        
        try! self.init(
            data: Data("<p style=\"\(style)\">\(text)</p>".utf8),
            options: [.documentType: NSAttributedString.DocumentType.html,
                      .characterEncoding: String.Encoding.utf8.rawValue],
            documentAttributes: nil)
    }
}
```

Tiếp theo định nghĩa 1 css từ dictionary.

```swift
enum Styles {
    static let header = [
        "color" : "#89da20",
        "font-size": "24px",
        "font-family": "-apple-system, system-ui"
    ]

    // ...
}
```

Bước cuối là dùng thôi

```swift
let hello = NSAttributedString(text: "Hello", styles: Styles.header)
```

Chú ý: không phải tất cả các thẻ html, css đều được hỗ trợ nên lưu ý trước khi sử dụng.

# Tăng hiệu năng khi xử lý ảnh trên iOS

Đoạn code này lấy từ WWDC2018. Về cơ bản *UIImageView* sẽ phải decode ảnh về dạng bitmap. Với những ảnh dung lượng lớn hoặc size ảnh to, việc decode đòi hỏi lượng lớn bộ nhớ để xử lý.

Ta có thể resize ảnh về với kích thước của *UIImageView* để nâng hiệu năng xử lý lên. Điều này đặc biệt hữu ích khi áp dụng với màn hình gallery.

```swift
func downsample(imageAt imageURL: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    let imageSource = CGImageSourceCreateWithURL(imageURL as CFURL, imageSourceOptions)!
    let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions =
        [kCGImageSourceCreateThumbnailFromImageAlways: true,
         kCGImageSourceShouldCacheImmediately: true,
         kCGImageSourceCreateThumbnailWithTransform: true,
         kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels] as CFDictionary
    let downsampledImage =
        CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions)!
    return UIImage(cgImage: downsampledImage)
}
```

---

Các link tham khảo:

- [css string](https://ilyagru.github.io/cssattributedstring-in-swift)

- [resize image](https://nshipster.com/image-resizing/)