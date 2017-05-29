+++
author = ""
comments = true
date = "2017-05-29T15:11:07+07:00"
draft = false
image = ""
menu = ""
share = true
slug = "tao-api-client-swift-3"
tags = ["swift"]
title = "Tạo APIClient với swift 3"

+++

### 1. Giới thiệu

Từ hồi mới bắt đầu làm IOS, thằng nào cũng hỏi mình có biết sử dụng AFNetworking không? Khổ nỗi lúc đó, mình mới chuyển từ làm game sang, rất ít dùng đến lib, đa số tự viết nên lơ tơ mơ không biết chúng nó nói đến cái gì. Dùng anh google tìm hiểu thì mình mới ngộ ra AFNetworking là một tool wrap lại urlsession, hỗ trợ developer thao tác nhanh gọn, đỡ mất công viết đi viết lại những đoạn code thủ tục (lib đã hỗ trợ bạn làm việc đó). Từ đó project nào mình cũng gắn AFNetworking vào. 

Tuy nhiên AFNetworking được viết bằng obj-c và không có phiên bản swift. May mắn thay, ta có một lib thay thế, phổ biến không kém trên nền swift.
Thậm chí chúng nó còn quảng cáo qua lại cho nhau :smile:

> Programming in Swift? Try Alamofire for a more conventional set of APIs.

Tận dụng sức trẻ của ngôn ngữ mới nên Alamofire có cách viết bóng bẩy, hiện đại hơn so với đàn anh AFNetworking. Vậy nên mình cũng tạo nên 1 lớp wrap mới cho Alamofire để sử dụng chung giữa các project của mình.

### 2. Hướng đi

Có lẽ hầu hết ai cũng wrap lại AFNetworking bằng cách viết 1 lớp *APIClient* sử dụng singleton với hàm init

{{< highlight objc "style=monokai" >}}
- (instancetype)initWithBaseURL:(NSURL *)url
{{< /highlight >}}

Ngay từ hàm init ta có thể tạo baseURL cho toàn bộ những API của mình. Cách này khá tiện dụng khi project chỉ sử dụng duy nhất một base url. Tuy nhiên project mới đây của mình có tận 3 base url. Thế nên việc sử dụng singleton với base url có vẻ không hợp lý lắm. Có lẽ chính vì thế ở Alamofire ta không thấy có hàm tương tự nữa.

Trên đường tìm kiếm hướng đi mới, mình thấy một cách khá là hot trend: sử dụng router. Alamofire cung cấp cho ta protocol *URLRequestConvertible* phục vụ mục đích đấy

Lấy luôn một ví dụ triển khai *URLRequestConvertible*

{{< highlight objc "style=monokai" >}}
enum Router: URLRequestConvertible {
    case search(query: String, page: Int)

    static let baseURLString = "https://example.com"
    static let perPage = 50

    // MARK: URLRequestConvertible

    func asURLRequest() throws -> URLRequest {
        let result: (path: String, parameters: Parameters) = {
            switch self {
            case let .search(query, page) where page > 0:
                return ("/search", ["q": query, "offset": Router.perPage * page])
            case let .search(query, _):
                return ("/search", ["q": query])
            }
        }()

        let url = try Router.baseURLString.asURL()
        let urlRequest = URLRequest(url: url.appendingPathComponent(result.path))

        return try URLEncoding.default.encode(urlRequest, with: result.parameters)
    }
}
{{< /highlight >}}

Ở đây ta cần phải đưa ra các thông tin để tạo ra request: base url, path, param. Đây là các trường cơ bản nhất, tuy nhiên ở một popular API ta còn cần biết những thông tin khác như method (get, set, put, delete), token,...Nếu ta triển khai hết trong 1 class implement *URLRequestConvertible* thì viết khá là dài dòng so với cách tạo một BASE_URL duy nhất. Đây là điều mà mình không muốn :joy:.

### 3. Triển khai
Lấy ý tưởng từ [chris.eidhof](http://chris.eidhof.nl/post/typesafe-url-routes-in-swift/), mình viết theo hướng tạo ra 1 router bao gồm các thông tin thiết yếu của 1 API: url, method, token (không đưa thông tin params vào trong router)

{{< highlight objc "style=monokai" >}}
public protocol ONPath {
    var path : String { get }
}

public protocol ONToken {
    
    /// example: "Bearer", "Basic", etc
    var tokenKind : String { get }
    var tokenStr : String { get }
    var isAuthorization : Bool { get }
}

public protocol ONMethod {
    var method : Alamofire.HTTPMethod { get }
}

// protocol container
public protocol ONUrl : ONPath, ONToken, ONMethod {
    var baseURL : String { get }
    var url : String { get }
}
{{< /highlight >}}

Khá là đơn giản. Bây giờ ta sẽ định nghĩa 1 router dựa trên protocol trên

{{< highlight objc "style=monokai" >}}
// MARK: - router for github
enum ONGithubURL : ONUrl {
    var baseURL: String {
        return BASE_URL
    }

    case getGitHubUser(account : String)
    
    var path: String {
        switch self {
        case .getGitHubUser(let acc):
            return "users/\(acc)"
        }
    }
}

// MARK: - method for github urls
extension ONGithubURL : ONMethod {
    var method: HTTPMethod {
        switch self {
        case .getGitHubUser(_):
            return .get
        }
    }
}

// MARK: - generate token for github url
extension ONGithubURL : ONToken {
    var isAuthorization: Bool {
        switch self {
        case .getGitHubUser(_):
            return false
        }
    }
    
    var tokenStr : String {
        switch self {
        case .getGitHubUser(_):
            return ""
        }
    }
}
{{< /highlight >}}

Từ enum ONGithubURL, ta có thể tạo ra router như sau

{{< highlight objc "style=monokai" >}}
let router = ONGithubURL.getGitHubUser(account: "gg4acrossover")
router.url // https://api.github.com/users/gg4acrossover
router.method // GET
{{< /highlight >}}

Với cách sử dụng router như trên ta có thể tạo ra url theo ý muốn. Từ đó ta có thể tạo hàm call API sử dụng Alamofire theo cách sau

{{< highlight objc "style=monokai" >}}
public func call(router: ONUrl, params: [String: Any]? = nil, success: @escaping responseJSON, fail: @escaping responseError) -> DataRequest {
    
    // add accept header
    var headers = ["Accept" : "application/json,charset=utf-8,text/html"]
    
    // add authorization if need
    if router.isAuthorization {
        headers["Authorization"] = router.tokenKind + " " + router.tokenStr
    }
    
    debugPrint(router.method.rawValue + " " + router.url)
    debugPrint("Headers: \(headers)")
    
    return sessionMng.request(router.url, method: router.method, parameters: params, headers: headers)
                     .validate()
                     .responseJSON { response in
            switch response.result {
            case .success(let value):
                let json = JSON(value)
                success(json)
            case .failure(let error):
                if error._code == NSURLErrorTimedOut {
                debugPrint("Request Timeout...")
                }
                fail(error)
        }
    }
}
{{< /highlight >}}

Ví dụ sử dụng ONGithubURL enum      

{{< highlight objc "style=monokai" >}}
let router = ONGithubURL.getGitHubUser(account: "gg4acrossover")
ONAPIClient.default.call(router: router, success: success, fail: fail)
{{< /highlight >}}

Code tham khảo: [source](https://github.com/gg4acrossover/swiftForFun/tree/master/Project%2003%20-OnAPIClient)

Các bài viết có thể tham khảo thêm:

* [typesafe url routes in swift](http://chris.eidhof.nl/post/typesafe-url-routes-in-swift/)

* [rethinking routers](https://chaione.com/blog/rethinking-routers-swift-protocol-oriented-programming-part-1/)

* [Moya](https://github.com/Moya/Moya)









