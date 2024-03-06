---
title: Session, Cookie and Flash
draft: false
tags:
  - ror
  - controller
---

# Session

Phiên hoạt động của mỗi người dùng được lưu trong một Session Object, giúp duy trì dữ liệu giữa các requests. Session Object chỉ có thể truy xuất từ Controller và View.

## Storage for Session

Về bộ nhớ cấp lưu Sessions, ta có thể sử dụng một số cơ chế:
- **CookieStore**: Lưu ở phía Client, dưới dạng Cookie [**Mặc định**]
    - Luôn mã hóa với Secret Key được lưu tại _config/credentials.yml.enc_
    - Tuy nhiên, kích thước lưu trữ nhỏ, thời gian ngắn, không an toàn
- **CacheStore**: Lưu ở phía Server, trong Rails cache
    - Phù hợp với dữ liệu có vòng đời ngắn
- **ActiveRecordStore**: Lưu ở phía Server, trong DB, sử dụng Active Record

Ta có thể cấu hình Storage cho Session Object qua

```ruby
# config/application.rb
Rails.application.config.session_store :cookie_store, key: '_your_app_session', domain: ".example.com"
```

## `session`

Một Session ứng với một User sẽ được định danh bởi ID, một giá trị string 32 ký tự. Khi người dùng truy cập lần đầu, Rails tự động sinh ID cho người dùng đó và gửi nó về Browser qua Cookie. **Rails luôn cần Browser gửi Cookie cùng Session ID để xác định Session Object trong bộ nhớ**.

Từ Request gửi lên (cùng Cookie), Rails có thể truy xuất Session Object và gán nó cho `session` method. [Delegate từ `request.session`]. _Session Object sẽ chỉ được nạp từ bộ nhớ lên khi cần sử dụng (lazily loaded)_.

Ta có thể truy cập, cập nhật và xóa dữ liệu của Session Object hiện tại (trong bộ nhớ) thông qua `session` method này. 

Để xóa toàn bộ Session Object, ta sử dụng `reset_session`

# Cookie

Rails cung cấp method `cookies` để tương tác với Cookie nhận từ request (Cookie) gửi đến Action hoặc từ Action gửi về cho Client (Set-Cookie)

Việc đọc, ghi `cookies` tương tự làm việc với Hash. Tuy nhiên, chú ý là giá trị cho một trường trong Cookie chỉ có thể là String, nên ta cần sử dụng `JSON.generate` để chuyển một Object về String.

Riêng việc xóa một phần tử trong `cookies`, ta cần sử dụng method `delete`.

Ngoài ra, Rails cũng cung cấp hai chức năng cho Cookies: Mã hóa + Xác thực

Chức năng xác thực được thực hiện qua việc đọc/ghi từ `cookies.signed`. Giá trị của phần tử tương ứng sẽ được gắn thêm chữ ký để tránh giả mạo.

Chức năng mã hóa được thực hiện qua việc đọc/ghi từ `cookies.encrypted`. Giá trị của phần tử tương ứng sẽ được mã hóa để tránh tiết lộ.

Khi muốn lưu một giá trị dài hạn, ta sử dụng `cookies.permanent`

Xem thêm: [Cookies](https://api.rubyonrails.org/v7.1.2/classes/ActionDispatch/Cookies.html)

# Flash

Flash là một phần của Session Object, chỉ khác là nó tự động xóa với mỗi request gửi đến, do đó, phù hợp để mang các thông báo.

Vòng đời của Flash 
- Bắt đầu từ lúc được gán.
- Kết thúc ngay **sau** khi một Action nữa (so với Action gán nó) thực thi xong (tức là đã trả về Response).

Helper `flash` trả về một **FlashHash** object, dùng để truy cập vào nội dung của `flash`.

Để hiển thị Flash, ta cần gọi nó trên View, VD:

```ruby
<% flash.each do |type, msg| %>
  <div>
    <%= msg %>
  </div>
<% end %>
```

Đọc/Ghi trên **FlashHash** hoạt động tương tự Hash, với hai key đặc biệt là `:alert` (cho Error Message), `:notice` (cho Confirmation Message). Ngoài ra có các methods sau:
- `each`: Duyệt qua các phần tử trong Flash
- `empty?`: Kiểm tra số lượng phần tử trong Flash hiện tại.
- `discard`, `delete`: Đánh dấu một phần tử/toàn bộ sẽ cần loại bỏ sau Action hiện tại.
- `keep`: Đánh dấu một phần tử/toàn bộ sẽ có trong Action ngay tiếp theo.

Đặc biệt, **FlashHash** còn cung cấp method `now` để đánh dấu một phần tử sẽ chỉ có vòng đời
- Bắt đầu từ lúc được gán.
- Kết thúc ngay **sau** khi Action hiện tại thực thi xong  (tức là đã trả về Response).

Như vậy, để đảm bảo **Flash chỉ xuất hiện trong một View**:
- Thường sử dụng `flash` cho trường hợp `redirect_to`
- Thường sử dụng `flash.now` cho trường hợp `render`.
  - Nếu sử dụng `flash` thì View hiện tại và View của Action tiếp theo cũng đều truy cập được Flash.
