---
title: Routing
draft: false
tags:
  - ror
  - intro
---

Route giúp ánh xạ đường dẫn với Controller Action. Các Routes trong Rails được định nghĩa trong tệp _config/routes.rb_

Khi một request tới Server, Rails sẽ thực hiện tìm kiếm route khớp với request đó, bao gồm khớp cả HTTP Method và URL Format, **thứ tự tìm kiếm chính là thứ tự khai báo**.

Ta có thể kiểm tra danh sách các routes đã được định nghĩa (theo thứ tự) qua:
- Development Only: `http://localhost:3000/rails/info/routes`
- `rails routes`. Một số Options:
  - `-g`: Tìm kiếm
  - `-c`: Lọc theo Controller

Danh sách này sẽ hiển thị theo dạng bảng theo thứ tự các cột:
- Route Name
- HTTP Method
- URL Patterns
- Route Parameters

# Non-Resourceful Routes

Cú pháp khai báo một route:
```ruby
<method> <path>, to: <controller_action>
```

- `<method>`: HTTP Methods: `get, post, delete, put, patch`
  - Có thể sử dụng `match` Method và `via` Option khi muốn ánh xạ đồng thời nhiều Methods (`:all` cho tất cả)
  - VD: `match "photos", to: "photos#show", via: %i(get post)`
- `<path>`: Chuỗi mô tả định dạng đường dẫn.
- `<to>`: Chuỗi mô tả Controller Action để xử lý
- Một số Options khác:
  - `controller`: Xác định cụ thể Controller (Symbol)
  - `action`: Xác định cụ thể Action (Symbol)
  - `format`: Phần đường dẫn xác định File Extension. VD: Khi request một tệp `avatar.jpg`.
  - `as`: Đổi tên lại Named Routes
  - `module`: Xác định Namespace cho Controller

Ngoài ra, ta có thể chỉ một Controller Action xử lý cho Root Path ("/") bằng Method: `root` (luôn là GET). VD: `root "pages#main"` sẽ luôn gọi main Action của PagesController để xử lý đường dẫn "/"

## Dynamic Segments

Ta có thể tạo đường dẫn động bằng việc sử dụng **Dynamic Segment** cho `<path>` với chuỗi bắt đầu bằng `:`. 

```ruby
# Route
get "products/:id/:user_id", to "products#show"

# Dynamic Segments: `:id`, `:user_id`
```

Đối số thực sự cho các Segments này khi có requests gửi tới sẽ được chuyển vào `params` Hash.

Khi có request `/products/1/2` thì Action `show` của Controller `ProductsController` sẽ được gọi và cung cấp một biến `params` dạng: `params = {id: "1", user_id: "2", controller: "products", action: "show"}` (chú ý, giá trị đều là String)

Với Dynamic Segments, ta có thể giới hạn giá trị nhận vào cho các đối số bằng Regex với Option **`constraints`**. 

```ruby
# Route
get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ }

GET photos/A12345 # match
GET photos/893 # not match
```

Ngoài ra, ta có thể thiết lập giá trị mặc định cho Segments qua Hash Option **`defaults`** (khi không bắt buộc phải xuất hiện)

## Wildcard Segments

Wildcard Segment để chỉ một Path Segment nhận giá trị là **phần còn lại** của Path. 

```ruby
# Route
get 'photos/*other', to: 'photos#unknown'

# match with
GET /photos/12         # params[:other] = "12"
GET /photos/long/to/12 # params[:other] = "long/to/12"
```

Khi Wildcard Segment nằm ở giữa (phía sau nó vẫn còn Segments khác) thì các Segments phía sau sẽ được khớp trước và Wildcard Segment nhận phần còn lại. 

```ruby
# Route
get 'books/*section/:title', to: 'books#show'

# match with
GET books/some/section/last-words   # params = {:section => "some/section", :title => "last-words",...}
```

Khi có Dynamic Segments, không thể dùng nhiều Wildcard Segments.

## Query String

Rails chỉ thực hiện so khớp phần Path, phần Query String sẽ không dùng để so khớp, Query String sẽ được chuyển vào `params` trực tiếp (không đóng gói thêm cấp nào nữa). 

`products/1?price=10` sẽ ứng với `params = {id: "1", price: "10",...}`

## Naming Routes

Named Routes là các tên đại diện cho đường dẫn, tức là thay vì làm việc trên String, ta làm việc với Method khi muốn xác định một đường dẫn trên trang.

Ta có thể xác định tạo một Named Route cho route hiện tại bằng việc xác định Option **`as`**. VD:

```ruby
get ":username", to: "users#show", as: :user
```

sẽ tạo hai Named Routes dạng `user_path` và `user_url`. Khi gọi `user_path("bob")` sẽ trả về `/bob`

# Resource Routing

Rails cung cấp Resource Routing nhằm hỗ trợ khai báo sẵn tất cả các Routes cho một Controller phục vụ một tài nguyên. Cụ thể Resource Route cung cấp các ánh xạ từ HTTP Method + URL tới Controller Action tương ứng.

Có hai methods phục vụ Resource Routing là `resources` (plural form) và `resource` (singular form). Điểm khác biệt:
- `resources` đại diện cho toàn bộ các phần tử. Tức là cần chỉ định rõ phần tử muốn tương tác (VD: qua `:id`) và cũng cung cấp thêm route `index` để hiển thị danh sách.
- `resource` đại diện cho một phần tử đã xác định. VD: Tài khoản đã đăng nhập thành công

VD: Khai báo `resources :photos` sẽ định nghĩa đồng thời các routes sau:

|HTTP Method|Path|Controller#Action|Description|
|--|--|--|--|
|GET|/photos|photos#index|display a list of all photos|
|GET|/photos/new|photos#new|return an HTML form for creating a new photo|
|POST|/photos|photos#create|create a new photo|
|GET|/photos/:id|photos#show|display a specific photo|
|GET|/photos/:id/edit|photos#edit|return an HTML form for editing a photo|
|PATCH/PUT|/photos/:id|photos#update|update a specific photo|
|DELETE|/photos/:id|photos#destroy|delete a specific photo|

## Options

Dùng **`path_names`** để đổi tên lại `new` và `edit` trong đường dẫn. VD: `resources :photos, path_names: { new: 'make', edit: 'change' }` sẽ khớp `photos/:id/change` với `photos#edit` và `photos/make` với `photots#new`

Dùng **`as`** để đổi tên [Named Routes Helpers](#path-and-url-helpers) cho toàn bộ các routes.

Dùng **`only`** hoặc **`except`** để giới hạn các Routes tự động sinh ra. VD: `resources :photos, only: [:index, :show]` chỉ sinh ra Index và Show Route.

Dùng **`path`** để tạo Path Prefix. VD: `resources :photos, path: "admin"` sẽ khớp với `admin/photos/:id,...`

## Path and URL Helpers

Việc khai báo resourceful routes cũng cung cấp các Path và URL Helpers, các Named Route Helpers, là các tên đại diện cho đường dẫn URL. 

Điểm khác biệt giữa chúng:
- **Path Helper**: Cung cấp đường dẫn tương đối (so với thư mục ứng dụng)
  - Kết thúc bằng `_path`
- **URL Helper**: Cung cấp đường dẫn tuyệt đối (bao gồm giao thức và tên miền)
  - Kết thúc bằng `_url`
  - Ngoài ra còn có thể có thêm Path Prefix.

VD: Khai báo `resources :photos` sẽ cung cấp các Named Routes sau:

|Path|Path Helper|
|--|--|
|/photos|photos_path|
|/photos/new|new_photo_path|
|/photos/:id|photo_path(:id)|
|/photos/:id/edit|edit_photo_path(:id)|

## Nested Resources

Các dữ liệu đôi khi có sự ràng buộc với nhau, đối tượng này là một phần tạo nên đối tượng khác, VD: Một **bình luận** thường nằm trong một **bài viết**, như vậy, để tham chiếu đến một **bình luận**, trước hết ta cần tham chiếu đến **bài viết**. Từ đó đường dẫn tới các tài nguyên cũng như các actions xử lý cũng phụ thuộc nhau. Rails cung cấp Nested Resources để hỗ trợ chức năng này

VD:
```ruby
resources :posts do
  resources :comment
end
```
sẽ tạo ra các Route dạng `/posts/:post_id/comments/` tương ứng với Action: `comments#index` và Path Helpers dạng: `post_comments_path`

Để tránh Deep Nested (đường dẫn quá dài), ta có thể sử dụng `only, except` để giới hạn những routes nhất định cần Nested (thực tế chỉ cần `index`, `new`, `create`) hoặc đơn giản hơn là xác định Option `shallow: true` cho `resources` (`resource`)

# Namespace Routing

Khi muốn nhóm các chức năng của ứng dụng, tức là:
- Nhóm riêng các Controllers files vào một thư mục con trong _app/controllers/_. VD: _app/controllers/admin_ cho các Controllers phục vụ chức năng của trang Admin.
  - Các Classes định nghĩa Controllers đặt trong Module. VD: 
  ```ruby
  class Admin::ProductsController < ApplicationController
  end
  ```
  hoặc
  ```ruby
  module Admin
    class ProductsController < ApplicationController
    end
  end
  ```
- Nhóm riêng View files (bao gồm cả Layout, Partials,...) vào một thư mục con trong _app/views/_, theo đúng cấu trúc của Controllers.
- Và ta cũng cần nhóm chúng trên đường dẫn, tức là `https://example.com/admin/products/`

Để thể hiện các Routes thuộc vào cùng một nhóm (và các đường dẫn cũng được Prefix), ta sử dụng Namespace Routing.

VD:
```ruby
namespace :admin do
  resources :products
end
```
sẽ tạo ra các routes sau:

|HTTP Verb|Path|Controller#Action|Named Route Helper|
|--|--|--|--|
|GET|/admin/articles|admin/articles#index|admin_articles_path|
|GET|/admin/articles/new|admin/articles#new|new_admin_article_path|
|POST|/admin/articles|admin/articles#create|admin_articles_path|
|GET|/admin/articles/:id|admin/articles#show|admin_article_path(:id)|
|GET|/admin/articles/:id/edit|admin/articles#edit|edit_admin_article_path(:id)|
|PATCH/PUT|/admin/articles/:id|admin/articles#update|admin_article_path(:id)|
|DELETE|/admin/articles/:id|admin/articles#destroy|admin_article_path(:id)

Chú ý, chúng sẽ tìm kiếm Controllers `Admin:ArticlesController`

Việc phân nhóm này cũng có thể thực hiện phân nhóm luôn trong _routes.rb_ thông qua method `draw`.
- Tổ chức các Routes cho mỗi nhóm trong một tệp riêng tại thư mục _config/routes_ (hoặc các thư mục con)
- Các routes khai báo trong tệp đó không cần bao trong `Rails.application.routes.draw do...end`
- Thêm các khai báo của tệp đó vào _config/routes.rb_ qua `draw`

VD:
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'foo', to: 'foo#bar'

  draw(:admin) # Will load another route file located in `config/routes/admin.rb`
end

# config/routes/admin.rb
namespace :admin do
  resources :comments
end

# config/routes/api.rb
namespace :api do
  resources :comments
end
```

# Scope

`scope` giúp gom nhóm một số routes có chung một số cấu hình. Khi không xác định các Options, một scope chỉ thay đổi URI (thêm Path Segment tương ứng). Ta có thể xác định các Options: 
- `module`: Xác định vị trí (module chứa) của Controller
- `as`: Xác định prefix cho Named Routes
- `path`: Xác định prefix cho URI.

VD:

```ruby
# Controller không đổi: ArticlesController
# Đường dẫn được Prefix: /admin/articles/:id,...
# Helpers không đổi
scope '/admin' do
  resources :articles
end

# Controller + Module: Admin::ArticlesController
# Đường dẫn không thay đổi: /articles/:id,...
# Helpers không đổi
scope module: 'admin' do
  resources :articles
end

# Controller không đổi
# Đường dẫn không đổi
# Helpers được Prefix: post_articles_path,...
scope as: "post" do
  resources :articles
end
```

Khi xác định cả 3 Options trên thì `scope` tương đương `namespace`. 
- Cả hai đều dùng để nhóm các cấu hình của nhiều routes.
- `scope` cho phép tùy chỉnh các cấu hình của nhóm. 
- `namespace` giúp đóng gói các cấu hình này lại 

# More Routes

Bên cạnh các Resource Routings sinh sẵn khi dùng `resources` và `resource`, ta có thể cấu hình một số Action Route sử dụng `member` và `collection` bên trong Resource Block.
- `member` dùng để tạo route cho Action tương tác với một phần tử, tức là đường dẫn tự động được thêm Segment `:id`
- `collection` dùng để tạo route cho Action tương tác với toàn bộ tập hợp. VD: Chức năng `search`

Bên trong các Blocks này sử dụng cú pháp tương tự [Non-Resourcesful Routes](#non-resourceful-routes)

```ruby
resources :articles do
    member do
        get "preview"
    end
    collection do
        get "search"
    end
end
```

Khi định nghĩa thêm các routes này, Rails cũng sinh ra các Named Route dạng

|Named Route|URL|Controller Action|
|--|--|--|
|`MEMBER_RESOURCE_path`|`/RESOURCEs/:id/MEMBER`|`RESOURCEs#MEMBER`|
|`COLLECTION_RESOURCEs_path`|`/RESOURCEs/COLLECTION`|`RESOURCEs#COLLECTION`|


