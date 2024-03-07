---
title: Hotwire - Turbo
draft: false
date: 2024-03-04
tags:
  - ror
  - extension
---

Úng dụng Rails thường triển khai theo kiến trúc MVC, tức là tập trung vào SSR (Server-side Rendering). Để cải thiện trải nghiệm người dùng, ta có thể kết hợp sử dụng AJAX và jQuery hoặc triển khai theo SPA (Single Page Application). Tuy nhiên, các giải pháp này có nhược điểm khi mà phải sử dụng code JS nhiều cũng như vấn đề về SEO.

[Hotwire](https://hotwired.dev/) là một FE Framework và được tích hợp mặc định trong Rails 7, giúp xử lý trên Server (SSR) mà vẫn có được tốc độ xử lý của SPA. 

Hotwire bao gồm các công cụ
- [**Turbo**](https://turbo.hotwired.dev/): Giúp tăng tốc xử lý Navigations và Form Submissions + Cập nhật một phần trang + Quảng bá cập nhật tới các người dùng khác. Tất cả các chức năng này có thể triển khai mà không cần sử dụng JS code. **80% vấn đề cần sử dụng JS có thể giải quyết với Turbo**.
- [**Stimulus**](https://stimulus.hotwired.dev/): Trong trường hợp Turbo không xử lý được, ta có thể sử dụng công cụ này để triển khai JS code theo hướng HTML, sử dụng `data-`
- [**Strada**](https://strada.hotwired.dev/): Hỗ trợ cho Mobile Apps

Với Hotwire, Server chỉ cần gửi các đoạn mã HTML (tức là vẫn có thể sử dụng SSR) thay vì JSON như SPA. Chính vì thế mà hướng tiếp cận này được gọi là **"HTML Over The Wire" - Hotwire**.

# Turbo

Turbo (JS Framework) giúp giảm thiểu lượng code JS ứng dụng cần triển khai, mà vẫn đạt được tốc độ của SPA, với các công cụ sau:
- **Turbo Drive**: Tăng tốc xử lý khi nhấn Links hoặc Submit Form, không cần nạp lại toàn bộ trang.
- **Turbo Frame**: Chia trang thành nhiều phần, cho phép cập nhật một phần trang - Lazy Loading và độc lập giữa các thành phần.
- **Turbo Streams**: Cho phép cập nhật bất đồng bộ một phần (bất kỳ) trong trang và quảng bá.

## Turbo Drive

Turbo Drive thực hiện bắt sự kiện click trên toàn bộ các `<a>` có đường dẫn trong Domain của trang:
- Ngăn Browser nạp các trang trên đường dẫn đó
- Sử dụng JS History API để cập nhật URL trên Browser
- Nạp nội dung trang mới qua JS `fetch` API
- Render trang mới

Tương tự với Form Submission, sử dụng `fetch`, và cập nhật trong Background.

Turbo Drive được phát triển từ thư viện Turbolinks.

### Navigation

Việc điều hướng tới một địa chỉ bao gồm một chuỗi các thao tác xử lý: Cập nhật Browser History, Request, Cache Restoring, Rendering và cập nhật Scroll Position. Có hai dạng điều hướng là **Application Visit** và **Restoration Visit**.

**Restoration Visit** là việc điều hướng khi người dùng nhấn vào Back hoặc Forward, tức là ta không thể kiểm soát một Restoration Visit.

**Application Visit** là việc điều hướng trên trang, như nhấn vào link. Trang kết quả sau điều hướng có thể được nạp từ Cache hoặc gửi Request để lấy Response.

Có hai dạng điều hướng ứng dụng là **Advance** (một History Entry được thêm vào History Stack) và **Replace** (thay thế Entry ở đầu History Stack)

Tự động, các thẻ `<a>` sẽ được quản lý bởi Turbo Drive, ta có thể thực hiện điều hướng bằng JS code thông qua hàm `Turbo.visit`. Với Replace Visit, ta sử dụng Annotation `data-turbo-action` cho `<a>` hoặc Option `{ action: "replace"}` cho `Turbo.visit`

Trước khi việc điều hướng diễn ra, sự kiện **`turbo:before-visit`** sẽ được kích hoạt, và ta có thể sử dụng
- `event.detail.url` để lấy thông tin URL
- `event.preventDefault()` để hủy việc điều hướng

### Form Submission

Việc xử lý Form Submission cũng tương tự với Link Click, song hỗ trợ thêm các HTTP Request Method khác an toàn hơn.

Tuy nhiên, với Turbo Drive (với chức năng chính về điều hướng) thì nó kỳ vọng Server sẽ trả về một trong các mã phản hồi
- 303: Redirect Response - Điều hướng đến trang kết quả
- 422: Unprocessable Entity - Khi Form có lỗi Validation
- 500: Internal Server Error - Lỗi trên Server

Trong trường hợp muốn cập nhật một phần trang, ta cần sử dụng Turbo Stream.

### Rendering

Mặc định, khi nhận được Response (hoặc sử dụng Cache) thì Turbo sẽ thay thế toàn bộ nội dung của `<body>` và nối nội dung của `<head>`, giữ lại toàn bộ JS, window, document,...

Ta có thể kiểm soát quá trình Rendering qua sự kiện **`turbo:before-render`**
- Override method `event.detail.render` để xác định nội dung sử dụng để Render kết hợp như nào giữa nội dung cũ và nội dung mới (các đối số của method này)
- `event.preventDefault()` để tạm dừng quá trình Rendering
- `event.detail.resume()` để tiếp tục quá trình Rendering.

VD:

```js
import morphdom from "morphdom"

addEventListener("turbo:before-render", (event) => {
  event.detail.render = (currentElement, newElement) => {
    // Return the combined content
    morphdom(currentElement, newElement)
  }
})
```

### Load

Mặc định khi gửi Request theo `<a>` hoặc `<form>`, thay vì Reload Page khi nhận được Response (cách Browser xử lý), Turbo Drive sẽ chỉ thay thế nội dung của `<body>`, nối (merge) nội dung của `<head>`, còn lại không thay đổi (giữ nguyên `window`, `document`,... nhìn chung là phần JS). Vấn đề là khi trong phần trang mới có chứa một số element cần khởi tạo để hoạt động như mong muốn 
- VD: Một `<select>` cần sử dụng **select2**: `$('select.selector-single').select2();`

Khi đó, do JS không được chạy lại, nên trừ khi tệp JS được nhúng trong trang Response, phần JS nạp trước sẽ không có tác dụng (vì element này chỉ xuất hiện trong trang mới). 

Vấn đề này có thể được xử lý theo một trong hai cách

- C1: **Disable Turbo Drive** cho `<a>` hoặc `<form>` để bắt buộc Reload Page khi có Response cho Visit.
  - `<a data-turbo="false" href="...">`

- C2: Bắt sự kiện `turbo:load`. Sự kiện này kích hoạt khi nạp trang lần đầu và sau mỗi Visit.

```js
$(document).on('turbo:load', function () {
  $('select.selector-single').select2();
  $('select.selector-multiple').select2();
});
```
Tuy nhiên, cần chú ý khi sử dụng cách này, vì rất có thể sẽ gắn một Event Handlers nhiều lần cho các elements (đặc biệt là `document` hay `window`).

### Request

Ta bắt sự kiện **`turbo:before-fetch-requets`** để kiểm soát `fetch` Request trước khi nó được gửi tới Server. Ta cũng có thể sử dụng `event.preventDefault()` và `event.detail.resume()` tương tự **`turbo:before-render`**

### Request Method & Confirmation

Ta sử dụng `<a>` Annotations
- `data-turbo-method` để xác định HTTP Request Method được dùng để gửi `fetch` Request tới Server.
    - Mặc định là `get`
    - Khi đó, `<a>` sẽ sinh ra một `<form>` ẩn phía sau nó.
    - Nên sử dụng `button` hoặc `form` nếu muốn dùng non-GET method.
- `data-turbo-confirmation` để yêu cầu người dùng phải xác nhận `confirm()` trước khi quá trình điều hướng được tiếp tục.
    - Gán method tự tạo dùng thay cho JS `confirm()` thông qua `Turbo.setConfirmMethod`

### ProgressBar

Turbo Drive cũng hỗ trợ hiển thị ProgressBar khi yêu cầu điều hướng được xử lý và trước khi được render. ProgressBar tự động hiển thị khi kết quả trả về lâu hơn 500ms (hoặc cấu hình lại qua `Turbo.setProgressBarDelay`)

Turbo sẽ tìm kiếm `<div class="turbo-progress-bar">` để hiển thị

### Track Assets

Turbo có thể theo dõi phiên bản cho các tài nguyên tĩnh được đặt trong `<head>` như CSS (phiên bản của tệp được đặt trong tên tệp - thường gắn với nội dung của tệp - Xem thêm: Asset Pipeline).

Ta có thể chỉ định Turbo theo dõi phiên bản cho một tệp thông qua Annotation `data-turbo-track="reload"` cho `<link>` hoặc `<script>` tới tệp đó. Khi được gắn Annotation này, phiên bản của tệp được theo dõi giữa các trang, nếu trang mới có phiên bản khác đi thì Turbo tự động nạp lại tệp đó (gửi yêu cầu)

### Root Location

Mặc định, Turbo Drive sẽ quản lý các liên kết tới cùng Domain, tuy nhiên, nếu muốn đảm bảo chỉ sử dụng Turbo Drive trong phạm vi một Path cụ thể trong Domain (việc điều hướng từ Path đó sang Path khác sẽ sử dụng cơ chế của Browser), ta thêm thẻ `<meta>` vào `<head>` như sau:

```html
<head>
  ...
  <meta name="turbo-root" content="/app">
</head>
```
Ở đây, chỉ các liên kết điều hướng giữa các URL bên trong `https://domain.com/app` mới sử dụng Turbo Drive, việc điều hướng sang đường dẫn khác (như: `https://domain.com/help`) sẽ sử dụng Browser.

## Turbo Frame

Turbo Frame giúp chia nhỏ một trang thành các phần độc lập với nhau - Component, bao đóng mỗi Component trong một Frame Element, từ đó có thể Scope Navigation và Lazy Loading.

**Scope Navigation** tức là các sự kiện xảy ra trong một Frame thì chỉ ảnh hưởng tới một Frame. Cụ thể, khi có một sự kiện trong một Navigation Context mà kích hoạt Server trả về một Response thì Turbo sẽ thực hiện trích xuất trong Response đó phần nội dung ứng với Frame hiện tại và chỉ dùng nó làm nội dung mới thay thế cho Frame Element tương ứng, các phần còn lại trong Response là không cần thiết.

Để xác định một Frame, ta bao đóng nội dung của nó bên trong `<turbo-frame>` element, với một `id` xác định - dùng để xác định vị trí của Frame trong Response. Một trang có thể bao gồm nhiều Frames, và đảm bảo các Frame phải có ID khác nhau - không nhất thiết mọi nội dung phải bao đóng trong một Frame nào đó.

Việc chia Frame nên hướng đến việc bao đóng các elements mà: **Các sự kiện xảy ra bên trong Frame chỉ ảnh hưởng đến các elements trong Frame đó**.

```html
<body>
  <div id="navigation">Links targeting the entire page</div>
  <!-- Frame for Messages -->
  <turbo-frame id="message_1">
    <h1>My message title</h1>
    <p>My message content</p>
    <a href="/messages/1/edit">Edit this message</a>
  </turbo-frame>

<!-- Frame for Comments -->
  <turbo-frame id="comments">
    <div id="comment_1">One comment</div>
    <div id="comment_2">Two comments</div>

    <form action="/messages/comments">...</form>
  </turbo-frame>
</body>
```

Nội dung của Frame không nhất thiết phải có ngay khi trả về Response cho trang. Ta có thể chỉ định Attribute `src` để chỉ thị đường dẫn chứa nội dung của Frame. Turbo sẽ tự động tạo các Requests phụ (sau khi tạo Request để lấy nội dung trang) tới đường dẫn xác định nội dung cho Frame đó, lấy nội dung và thực hiện trích xuất phần Frame tương ứng trong Response để Render.

Quá trình trên gọi là **Eager Loading**. Ta có thể cấu hình **Lazy Loading** - tức là không tự động tạo Request để lấy nội dung `src` Frame - bằng việc gán Attribute `loading="lazy"`. Khi đó, việc Loading chỉ thực hiện khi Frame nằm trong vùng hiển thị.

Trong quá trình nạp nội dung, các Attribute `aria-busy=true`, loại bỏ `aria-busy=true` và `complete` sẽ được thêm vào Frame Element.

```html
<body>
  <h1>Imbox</h1>

  <div id="emails">
    ...
  </div>

  <!-- Eager Loading -->
  <turbo-frame id="set_aside_tray" src="/emails/set_aside">
  </turbo-frame>

  <!-- Lazy Loading -->
  <turbo-frame id="reply_later_tray" src="/emails/reply_later" loading="lazy">
  </turbo-frame>
</body>
```

## Turbo Stream

Nếu như Turbo Frame cập nhật toàn bộ một Frame thì Turbo Stream cho phép cập nhật một phần bất kỳ trong trang, thậm chí các phần này là rời rạc. Nội dung cập nhật vẫn là HTML.

Turbo Stream bao đóng một **thông điệp** mô tả cách thức thay đổi trang qua `<turbo-stream>` element. Thông điệp này xác định 
- Một Action Name thể hiện cách thức thay đổi.
  - Xác định qua `action` Attribute.
- Một Target thể hiện element cần thay đổi.
  - Xác định qua `target` Attribute
  - Giá trị có thể là Element ID hoặc CSS Selectors.
  - Tức là cập nhật có thể xảy ra trên một hay nhiều element.
- Nội dung (HTML) mới dùng để cập nhật, nội dung này được đặt trong thẻ `<template>` bên trong `<turbo-stream>`

Ta có thể đóng gói nhiều thông điệp cập nhật (stream element) vào một thông điệp (stream message). Thông điệp này sẽ được gửi qua WebSocket, SSE,..., giúp ứng dụng tự động được cập nhật khi có các thay đổi từ bên ngoài như từ người dùng khác, hoặc ứng dụng khác.

### Actions

Turbo Stream hỗ trợ 7 Actions sau:

- **append**: Nối nội dung mới sau nội dung cũ.
- **prepend**: Nối nội dung mới trước nội dung cũ.
- **replace**: Thay thế toàn bộ element
- **update**: Thay thế nội dung của element
- **remove**: Loại bỏ element.
- **before**: Nối nội dung mới trước element
- **after**: Nối nội dung mới sau element.

### Form Submission

Thông thường, các thông điệp cập nhật sẽ được gửi về khi có sự kiện Submit Form, với Request chứa **Accept** Header (định dạng của Response mà Client mong muốn nhận) là **text/vnd.turbo-stream.html**. 
- Turbo sẽ tự động thêm giá trị này cho **Accept Header** này cho các Form có `method` là POST, PUT, PATCH hoặc DELETE. 
- Các Anchor và GET Form sẽ không được gán. Song ta có thể chỉ thị thông qua Attribute `data-turbo-stream=true`

# Turbo-Rails

[Gem `turbo-rails`](https://github.com/hotwired/turbo-rails) được phát triển để tích hợp Turbo vào Rails Framework.

Khi tải Gem này về ứng dụng, có một số chức năng sau được triển khai

**Turbo Drive**
- Toàn bộ các thẻ `<a>` với `href` tới cùng Domain và các `<form>`
  - Không được Browser xử lý. Thay vào đó Turbo sẽ xử lý
  - Trong quá trình Rendering:
    - Nội dung `<body>` cũ bị thay thế
    - Nội dung `<head>` được Merge
    - Giữ nguyên: `window`, `document`, `<html>`
- Thiết lập `Turbo.session.drive = false` trong JS để không sử dụng Turbo Drive
- Thiết lập `data-turbo="false"` cho element để không sử dụng Turbo Drive cho element đó.

**Turbo Frame**
- Cung cấp `turbo_frame_tag` Helper để tạo `<turbo_frame>`
  - Đối số là ID cho Frame Element
  - Dùng Block để định nghĩa nội dung cho Frame Element.
  
**Turbo Stream**
- Cung cấp `turbo_stream_from` Helper để tạo `<turbo_stream>` element.
- Trên Controller, ta có thể chỉ định Format cho Turbo Stream Message là `format.turbo_stream`
  - Nếu không chỉ định nội dung cụ thể cho format response này, Rails tự động tìm kiếm tệp có định dạng **.turbo_stream.erb** trong _app/views/<controller>_ để làm Response (tương tự các Format khác)
  - Ta có thể trả về Inline Response cho Format này qua method `render`với Option là `:turbo_stream`

VD:
```ruby
def create
  @post = Post.new(post_params)

  respond_to do |format|
    if @post.save
      format.turbo_stream { render turbo_stream: turbo_stream.prepend('posts', partial: 'post') }
    else
      format.html { render :new, status: :unprocessable_entity }
    end
  end
end
```