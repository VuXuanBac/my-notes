---
title: Action View
draft: false
tags:
  - ror
  - view
---

Sau khi xử lý request, Action trong Controller có thể tạo một response theo một trong ba cách sau:
- Gọi `render` để trả về một **Full Response**
- Gọi `redirect_to` để trả về một **HTTP Redirect Response**
- Gọi `head` để tạo một **HTTP Header Only** làm Response

# Default Render

Theo "Convention over Configuration", Rails tự động sử dụng một View có tên trùng với Action, trong thư mục có cùng tên với Controller, trong thư mục _app/views_, để làm **Full Response** cho một request gửi tới.

Các tệp Views này có định dạng `.html.erb` (HTML + Ruby) - vì thế mà được gọi là View Template. Ta hoàn toàn có thể sử dụng định dạng khác, và Rails sẽ sử dụng một Template Handler tương ứng (nếu được cấu hình) để xử lý tệp đó.

VD: Với Request `GET /products`, qua Routing, nó ánh xạ tới Action **index** của **ProductsController**. Nếu như Action này không có bất kỳ xử lý nào, Rails tự động dùng View _app/views/products/index.html.erb_ để trả về cho User.

Đặc biệt, trong một Action, nếu như không chỉ định cụ thể một trong ba cách tạo Response trên, Rails sẽ tạo một `render` Response, với cơ chế tìm kiếm như trên.

# render

Ta có thể gọi `render` để trả về một Full Response khác với mặc định ở trên. VD: Khi UPDATE Action thất bại, thường ta sẽ trả về lại giao diện của EDIT.

**Chú ý, `render` là chỉ thị việc tạo HTTP Response, nó không thực thi một Action mới, nên nếu View phụ thuộc vào một dữ liệu tạo từ Action, ta cũng cần tạo dữ liệu đó trước khi `render`**

Dưới đây là một số cách dùng `render`

**1.** Nếu muốn dùng View Template ứng với một Action khác, ta có thể truyền tên của Action đó.

Nếu Action này nằm khác Controller, ta cần truyền đường dẫn tương đối (theo _app/views_) của View Template đó.

**2.** Nếu không muốn dùng View Template đã định nghĩa, tức là trả về kết quả ngay trên Controller, ta có thể xác định kết quả trả về - với cú pháp tương tự như trong View Template - cho `inline` Option. **Tuy nhiên, cách này không nên sử dụng**

VD: 
```ruby
render inline: "<% products.each do |p| %><p><%= p.name %></p><% end %>"
```

**3.** Nếu kết quả trả về chỉ là một chuỗi, ta có thể xác định giá trị đó cho `plain` Option.

Một khác biệt giữa `plain` và `inline` là kết quả từ `plain` sẽ không được nhúng trong View Layout (trừ khi xác định thêm Option `layout: true`)

**4.** Khi muốn trả về một định dạng cụ thể, như HTML, JSON, XML, JS..., ta sử dụng các Options: `html`, `json`, `xml`, `js`...

**5.** Khi muốn trả về một kết quả nằm trong HTTP Body (Raw Response), tức là không xác định Content-Type, ta xác định giá trị cho `body` Option

**6.** Khi muốn trả về nội dung của một tệp, ta xác định đường dẫn tuyệt đối tới tệp đó (có thể sử dụng `Rails.root` để chỉ đường dẫn tuyệt đối tới project)

Mặc định, nội dung này cũng sẽ được nhúng trong View Layout.

## render Option

Ta có thể ghi đè mặc định thiết lập `Content-Type` trong HTTP Response bằng việc xác định MIME Type cho `content_type`

Mặc định, View Layout theo Convention sẽ được sử dụng để nhúng các kết quả trả về. Ta có thể xác định một View Layout cụ thể qua Option `layout`, với đối số là đường dẫn xác định tệp của Layout đó.

Thông thường HTTP Response Status Code là 200 OK, ta có thể xác định khác thông qua `status` Option với đối số là mã hoặc tên. **Với việc trả về một View Template khác, ta nên xác định Status Code**.

## Layout

Layout là cấu trúc chung cho các View Template, nó nằm trong thư mục _app/views/layouts_ 

Convention: Tên Layout giống với tên Controller. VD: `PhotosController` sẽ tương ứng với Layout _app/views/layouts/photos.html.erb_

**Rails sẽ tìm kiếm Layout mặc định cho một Action dựa trên Convention này.** Nếu không có Layout như vậy, **Layout mặc định là _app/views/layouts/application.html.erb_**

Ta có thể chỉ định cụ thể một Layout dùng cho một Controller qua method `layout`, đối số của nó có thể là 
- Đường dẫn xác định tệp Layout hoặc 
- Một Method/PROC (cho phép xác định Layout lúc Runtime)
- `false` (nếu không muốn dùng Layout)

`layout` method có thể sử dụng các Options `only`, `except` với mục đích giới hạn Actions nào sử dụng Layout hiện tại.

# redirect_to & redirect_back

`redirect_to` tạo một HTTP Redirect Response dùng để chỉ thị Browser điều hướng request tới một Action khác, hoặc một Controller khác, hoặc ứng dụng khác xử lý.

Ngoài ra, ta có thể sử dụng `redirect_back` để điều hướng tới trang mà User tới ngay trước khi được chuyển tới trang hiện tại. Cụ thể, Rails sẽ sử dụng địa chỉ URL từ trường Referrer trong HTTP Request. Do không đảm bảo trường này luôn được Browser xác định, ta cần chỉ định Option `fallback_location` cho method này, với giá trị là đường dẫn mặc định sẽ được điều hướng đến nếu không tìm thấy Referrer.

Chú ý, hai methods này chỉ thực hiện tạo một Response, nên ta có thể hủy thao tác thông qua việc trả về một giá trị khác cho Action.

Ta có thể xác định Option `status` để ghi đè Status Code mặc định được sử dụng (302)

# head

`head` giúp tạo một HTTP Response chỉ có phần Header

Đối số của method này là Status Code.

Ngoài ra ta có thể sử dụng các Options tương ứng với một trường trong HTTP Response Header.