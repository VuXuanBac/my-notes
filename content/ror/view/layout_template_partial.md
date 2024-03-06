---
title: View
draft: false
tags:
  - ror
  - intro
---

Kết quả HTML trả về từ một Controller Action có thể bao gồm 3 thành phần: **Templates**, **Layouts** và **Partials**

Thực tế, tùy vào góc nhìn mà một tệp có thể được gọi tên. Ta sẽ quy ước như sau:
- Gọi chung là View
- **Layout** là đối tượng cho phép nhúng nội dung cho nó
- **Partial** là một phần nội dung, cần nhúng vào đối tượng khác.
- **Template** là các đối tượng còn lại. Mặc dù, thực tế, cả Layout và Partial đều là Template.

# Template

Template là các tệp có sự kết hợp giữa Markup Language và Programming Language. VD: ERB là một Template kết hợp giữa HTML và Ruby.

Bên trong ERB, Ruby code được nhúng vào HTML theo cú pháp:
- `<%= ... %>`: Nội dung bên trong được thực thi và kết quả trả về được chuyển về String và gán lại cho tệp.
- `<% ... %>`: Nội dung được thực thi nhưng kết quả trả về không được gán lại.

# Layout

Layout là cấu trúc chung cho nhiều View (Layout, Template, Partial)

Nhìn chung, bất kỳ một View nào cho phép nhúng một View khác bên trong nó đều được gọi là Layouts.

## Convention

Khi Controller trả về một Response, thông thường Response này sẽ được nhúng trong một Layout mặc định của Action tương ứng.
- Tệp nằm trong thư mục _app/views/layouts_
- Tên giống với tên Controller. 
- Nếu không có thì sử dụng: **_app/views/layouts/application.html.erb_

VD: `PhotosController` sẽ tương ứng với Layout _app/views/layouts/photos.html.erb_

# yield & content_for & provide

`yield` giúp xác định **vị trí nào trong Layout** cần nhúng một nội dung: Tức là phần nội dung cần nhúng sẽ thay thế vị trí của các methods này trong Layout.

`content_for` và `provide` giúp xác định **nội dung cần nhúng của View**.

Nội dung cần nhúng có thể **có định danh** và được truy xuất theo định danh, tức là _View cần nhúng có thể chia thành các phần, và mỗi phần được nhúng trong vị trí khác nhau trên Layout_.

Phần nội dung có định danh cần truy xuất theo định danh. Phần nội dung không có định danh (chỉ có một) thì luôn được coi là nội dung nhúng mặc định - sử dụng `yield` không có định danh.

VD:

```ruby
# view.html.erb

<% content_for :head do %>
  <title>A simple page</title>
<% end %>

<p>Hello, Rails!</p>

# layout.html.erb
<html>
  <head>
  <%= yield :head %>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

Ở đây, Layout sử dụng `yield :head` để truy xuất nội dung có định danh `:head`; trong khi `yield` sẽ truy xuất phần nội dung không có định danh.

sẽ sinh kết quả là:

```html
<html>
  <head>
  <title>A simple page</title>
  </head>
  <body>
  <p>Hello, Rails!</p>
  </body>
</html>
```

## content_for vs provide

Cả hai method này có cú pháp như nhau
- Nhận vào một định danh cho nội dung
- Nhận vào một Block để xác định nội dung

Điểm khác biệt là `provide` không hỗ trợ cơ chế [Streaming](https://api.rubyonrails.org/classes/ActionController/Streaming.html). Điều này rõ nhất khi **phần nội dung bị chia nhỏ** trong View - tức là có nhiều đoạn nội dung có chung định danh.

`provide` sẽ chỉ thị Layout biết không cần tìm kiếm thêm đoạn nội dung nữa (vì nó đã có đủ)

VD:

```ruby
# layout.html.erb
<div class="heading"><%= yield :surprise %></div>
<div class="body">
   <p><%= yield %></p>
</div>

# template.html.erb
<%= XXXXX :surprise, "Hello" %>
More Content
<%= XXXXX :surprise, ", World!" %>
```

Nếu `XXXX` là `content_for` thì kết quả sẽ là:

```html
<div class="heading">Hello, World!</div>
<div class="body">
   <p>More Content</p>
</div>
```
Nếu `XXXX` là `provide` thì kết quả sẽ là:

```html
<div class="heading">Hello</div>
<div class="body">
   <p>More Content</p>
</div>
```
## content_for vs yield

Thực tế, `content_for` cũng có thể dùng trong Layout để chỉ định vị trí cần nhúng nội dung.

Điểm khác biệt giữa `yield` và `content_for` là `yield` không thể gọi trong Helper Modules.

# Nested Layouts

Khi muốn một Layout kế thừa một Layout khác, thì trong Layout kế thừa, ta xác định các phần nội dung khác thông qua `content_for`, sau đó gọi `render template: <base_layout>` để render ngay Base Layout ra Layout hiện tại.

VD:

```ruby
# app/views/layouts/application.html.erb
# BASE LAYOUT

<!DOCTYPE html>
<html>
  <head>
    <title><%= yield :title %></title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= javascript_importmap_tags %>
    <%= stylesheet_link_tag "common", "data-turbo-track": "reload" %>
    <%= javascript_import_module_tag "common" %>
    <%= yield :head %>
  </head>
  <body>
    <%= yield :body %>
  </body>
</html>
```

Đây là Base Layout, ta chỉ định hai vị trí có thể tùy chỉnh là `:head` và `:body`. Khi đó, Layout kế thừa sẽ là:

```ruby
# app/views/layouts/base.html.erb
# NESTED LAYOUT

<% content_for :head do %>
  <%= stylesheet_link_tag "base", "data-turbo-track": "reload" %>
  <%= javascript_import_module_tag "base" %>
<% end %>

<% content_for :body do %>
  <div class="container-scroller">
    <%= render "sidebar" %>
    <div class="container-fluid page-body-wrapper">
      <%= render "navbar" %>
      <div class="main-panel">
        <div class="content-wrapper">
          <%= yield %>
        </div>
        <%= render "footer" %>
      </div>
    </div>
  </div>
<% end %>
<%= render template: "layouts/application" %>
```

Trong Nested Layout này, ta cũng có dùng `yield` để các View có thể nhúng nội dung vào. Đồng thời sử dụng `render template` để sinh lại nội dung của Base Layout vào vị trí tương ứng.

# Partials

Partials giúp chia View thành các phần nhỏ hơn, tức là có thể dùng lại. Các tệp Partials được đặt tên bắt đầu bằng `_`.

Định danh cho Partial là tên tệp (không bao gồm `_` ở đầu). Ta có thể xác định thêm thư mục chứa tệp đó, vị trí của thư mục này là tương đối so với _app/views_. VD: "products/product" sẽ tương ứng với tệp _app/views/products/\_product.html.erb_

Xem thêm: [Inheritance](#inheritance)

Bên trong một View (Template, Layout, Partial) ta nhúng một Partial với `render`, theo sau là định danh của Partial đó với Option `partial`

Dưới đây là một số Options cho `render`

## Pass Variable to Partials

Mặc định, Partial không có bất kỳ biến cục bộ nào, ta có thể sử dụng các khai báo sau, chúng sẽ tự động tạo các biến cục bộ cho Partials. 

### locals

Ta có thể truyền dữ liệu cho Partial thông qua Option `locals` của `render`, với giá trị là một Hash chứa dữ liệu. 

Sau định nghĩa này, **mỗi phần tử key-value trong `locals` sẽ tương ứng với một biến cục bộ** trong Partial, tức là, ta có thể sử dụng trực tiếp key để truy cập dữ liệu.

Ngoài ra, ta có thể sử dụng `local_assigns` Helper để truy cập dữ liệu này giống như Hash.

VD:

```ruby
# app/views/products/show.html.erb
<%= render partial: "products/product", locals: { product: @product } %>

# app/views/products/_product.html.erb
<%= tag.div id: dom_id(product) do %>
  <h1><%= product.name %></h1>
<% end %>
```

Trong VD trên: `product` tương đương với `local_assigns[:product]`

Điểm khác biệt là nếu không truyền giá trị với key tương ứng, cách sử dụng biến sinh lỗi, còn sử dụng `local_assigns` sẽ trả về `nil`.

### object

Ta có thể thay thế Option `locals` bằng `object`. Khi đó, Partial sẽ được cung cấp một biến cục bộ với **cùng tên Partial** và giá trị là giá trị truyền cho `object`.

VD: Hai cách sau là tương đương
```ruby
<%= render partial: "account", object: @buyer %>

<%= render partial: "account", locals: { account: @buyer } %>
```

Ta có thể sử dụng thêm Option `as` (theo sau là String hoặc Symbol) để đổi tên biến cục bộ này

### collection

Một mong muốn thường có là render một tập dữ liệu với cùng một giao diện. VD: Render các sách trong thư viện, mỗi sách render trên một Card.

Trong những trường hợp này, ta sử dụng Option `collection` (thay vì cách xử lý truyền thống là duyệt tập hợp).

VD: Hai cách sau là tương đương
``` Ruby
<%= render partial: "ad", collection: @advertisements %>

<% @advertisements.each do |ad| %>
  <%= render partial: "ad", locals: { ad: ad } %>
<% end %>
```

Sử dụng `collection`, trên Partial ta được cung cấp các biến sau, với `XXX` là định danh cho Partial:
- `XXX`: Phần tử hiện tại của tập hợp.
- `XXX_iteration`: Một biến dùng để duyệt, chứa các thông tin sau:
  - `index`: Chỉ số của phần tử hiện tại.
  - `first?`, `last?`: Xác định phần tử này có ở đầu hay cuối tập hợp không.

Sử dụng `as` Option để đổi tên mặc định `XXX` này.

Sử dụng `spacer_template` Option để xác định Partial dùng để phân tách giữa hai Partials ứng với hai phần tử liên tiếp.

Trong trường hợp muốn sử dụng một giao diện mặc định khi tập hợp rỗng, ta có thể sử dụng cú pháp OR (vì khi tập hợp rỗng `render` sẽ trả về `nil`)

VD:
```ruby
<%= render(partial: "ad", collection: @advertisements) || "There's no ad to be displayed" %>
```

### Shorthand Syntax

Dưới đây là một số cú pháp viết gọn khi sử dụng `render`:

**1.** Khi chỉ sử dụng `partial` và `locals`
``` Ruby
# <%= render partial: "account" %>
<%= render "account" %>

# <%= render partial: "account", locals: { account: @buyer } %>
<%= render "account", account: @buyer %>
```
**2.** Đường dẫn tới Partial được xác định tự động từ Object truyền cho nó (thông qua `to_partial_path` gọi trên Model)
```ruby

# @account.to_partial_path returns 'accounts/account', so it can be used to replace:

# <%= render partial: "accounts/account", locals: { account: @account} %>
<%= render @account %>

# @posts is an array of Post instances, so every post record returns 'posts/post' on +to_partial_path+,
# that's why we can replace:

# <%= render partial: "posts/post", collection: @posts %>
<%= render @posts %>
```

## Partial + Layout 

Ta có thể sử dụng Option `layout` để xác định một Layout chứa Partial hiện tại. 

`render` lúc này sẽ trả về Layout đã được nhúng Partial. Như vậy, **Layout này cũng là một Partial**.

VD:

```ruby
# app/views/users/index.html.erb
Here's the administrator:
<%= render partial: "user", layout: "administrator", locals: { user: administrator } %>

Here's the editor:
<%= render partial: "user", layout: "editor", locals: { user: editor } %>

# app/views/users/_user.html.erb
Name: <%= user.name %>

# app/views/users/_administrator.html.erb
<div id="administrator">
  Budget: $<%= user.budget %>
  <%= yield %>
</div>

# app/views/users/_editor.html.erb
<div id="editor">
  Deadline: <%= user.deadline %>
  <%= yield %>
</div>
```
Ở đây, _\_user.html.erb_ là Partial, _\_administrator.html.erb_ và _\_editor.html.erb_ là hai Partial Layouts.

Nội dung của Template _index.html.erb_ sẽ được chuyển thành:

```ruby
Here's the administrator:
<div id="administrator">
  Budget: $<%= user.budget %>
  Name: <%= user.name %>
</div>

Here's the editor:
<div id="editor">
  Deadline: <%= user.deadline %>
  Name: <%= user.name %>
</div>
```

Chú ý, các cơ chế truyền dữ liệu [ở trên](#pass-variable-to-partials) vẫn có thể dùng. 
- Trên Layout cũng được cung cấp các biến cục bộ tương tự Partial.
- Với `collection`, `layout` cũng sẽ được render với mỗi phần tử.

### Partial as Block 

Khi kết hợp Layout và Partial, thay vì sử dụng một tệp riêng, ta có thể định nghĩa Partial trực tiếp như một Block cho `render`.

VD: Viết lại VD trên

```ruby
# app/views/users/index.html.erb
Here's the administrator:
<%= render(layout: "administrator", locals: { user: administrator }) do %>
  Name: <%= user.name %>
<% end %>

# app/views/users/_administrator.html.erb
<div id="administrator">
  Budget: $<%= user.budget %>
  <%= yield %>
</div>
```

Lúc này, nội dung Partial _\_user.html.erb_ được truyền vào `render` dưới dạng Block.

Dễ thấy cách này không hiệu quả khi Partial _\_user.html.erb_ được dùng lại nhiều lần.

# View Mechanisism

## Rendering

Khi trả về một View cho một Action, quá trình tạo View đầy đủ sẽ diễn ra theo cơ chế  **Inside-out - từ trong ra ngoài**, tức là:
- Các thành phần nhỏ nhất sẽ được xử lý trước: Partials
- Xử lý lan dần đến Layout.

## View Paths

Dưới đây là một số Conventions khi Rails tìm kiếm các tệp cho Views:
- Chỉ tìm kiếm các tệp trong thư mục _app/views_
- Tất cả các View đều thực hiện tìm kiếm theo hướng kế thừa:

Cụ thể:
- Với Layout:
  - Tìm kiếm tệp cùng tên Controller, bên trong thư mục _app/views/layouts_
  - Nếu không thấy, sử dụng Layout của Controller cha của Controller hiện tại
- Với Template và Partial (bao gồm Partial Layout):
  - Tìm kiếm tệp có tên được xác định.
    - Template Convention: Tệp cùng tên với Action
    - Đối số xác định qua `render`
  - Tệp này bên trong thư mục của Controller hiện tại: _app/views/<controller>_
  - Nếu không thấy, tìm trong thư mục của Controller mà Controller hiện tại kế thừa.

VD:
```
# app/views/advertiser/buy.html.erb
<%= render partial: "account", locals: { account: @buyer } %>

<% @advertisements.each do |ad| %>
  <%= render partial: "ad", locals: { ad: ad } %>
<% end %>
```
sẽ tìm kiếm Partial _\_account.html.erb_ và _\_ad.html.erb_ trong các thư mục:
- _advertiser_
- _application_

## Specific View

Ta có thể chỉ định cụ thể một Layout theo các cách sau:
- Sử dụng `layout` Method cho Controller
  - Đối số là String xác định tệp Layout sử dụng
  - Đối số là Symbol tới Method trả về String xác định tệp Layout sử dụng
- Sử dụng `layout` Option trong `render` cho mỗi Action