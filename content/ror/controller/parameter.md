---
title: Controller
draft: false
date: 2024-03-04
tags:
  - ror
  - intro
---

**Convention**: Tên controller đặt dưới dạng **số nhiều** và tệp mã nguồn, tên Class kết thúc bằng `Controller`

Tất cả các WebApp Controllers đều kế thừa từ `ActionController::Base` - [Module ActionController](https://api.rubyonrails.org/v7.1.2/classes/ActionController.html). Các **public methods** trong Controllers được gọi là các Actions.

Khi nhận được một request, quá trình routing sẽ ánh xạ nó tới một Action của một Controller và Rails sẽ thực hiện tạo một instance cho Controller đó và gọi Action tương ứng để xử lý.

Các Actions đều cần trả về một Response cho Client. Mặc định, Rails tự động thực hiện một số Response cho một Action xác định. VD: 
- `index` trả về View `index.html.erb`.
- `create` trả về `redirect_to` với status code 302

# Parameters

Dữ liệu được Client gửi cùng request (Query String + Request Body) đều được đóng gói vào đối tượng `ActionController::Parameters` và được truy xuất qua method `params`. Đối tượng này tương tự một Hash, tất cả các giá trị của nó đều là String.

Để truy xuất một trường dữ liệu, ta có thể
- Sử dụng cú pháp của Hash (trả về `nil` nếu không có key tương ứng)
- Sử dụng `dig` nếu muốn truy xuất dữ liệu ở các cấp sâu hơn (trả về `nil` nếu giữa chừng `nil`).

Khi muốn gửi một dữ liệu Array, ta sử dụng `[]` trong tên của trường dữ liệu gửi lên. 

```ruby
# request
GET /clients?ids[]=1&ids[]=2&ids[]=3

# params
params = {ids: ["1", "2", "3"]}
```

Khi muốn gửi một dữ liệu Hash, ta cũng sử dụng `[<key>]` trong tên của trường dữ liệu gửi lên. 

```ruby
# request
GET /clients?info[name]=Acme&info[address][city]=Carrot

# params
params = {info: {name: "Acme", address: {city: "Carrot"}}}
```

Ngoài ra, quá trình Routing cũng tự động gán dữ liệu cho `params`, bao gồm `:controller`, `:action`, `:id`,...

```ruby
# Route
get '/clients/:status', to: 'clients#index', foo: 'bar'

# Request
GET clients/active

# `params` cho `index` Action trong `ClientsController`
params ={controller: "clients", action: "index", status: "active", foo: "bar"}
```

## Strong Parameters

Cung cấp cơ chế bảo vệ ứng dụng từ dữ liệu người dùng nhập vào, đặc biệt là dữ liệu dùng cho các thao tác cập nhật DB. Tính năng của Strong Parameters bao gồm giới hạn các trường dữ liệu cần thiết và yêu cầu phải xác định các trường dữ liệu cần có.

Các methods sau đây được gọi trên `ActionController::Parameters` và trả về một `ActionController::Parameters`, tức là có thể tạo Chain Call.

### `require`

Method `require` giúp lấy một/nhiều trường dữ liệu với key xác định và trả về các `Parameters` mới với nội dung là phần value của trường dữ liệu đó. Nếu value này là `nil, "", {}, []...` (ngoại trừ false) thì sinh lỗi `ActionController::ParameterMissing`

```ruby
ActionController::Parameters.new(person: { name: "Francesco" }).require(:person)
# => #<ActionController::Parameters {"name"=>"Francesco"} permitted: false>
```

Khi đối số cho `require` là một mảng các keys, việc tìm kiếm sẽ theo thứ tự truyền vào, kết quả là một `Array` các `Parameters`

### `permit`

Method `permit` nhận vào một tập các quy tắc giúp lọc các trường dữ liệu cần thiết và loại bỏ toàn bộ các trường dữ liệu còn lại.
- `:key`: Trường dữ liệu với key tương ứng, và value là **Scalar** thì sẽ pass.
- `key: []`: Trường dữ liệu với key tương ứng, và value là **Array of Scalar** thì sẽ pass.
- `key: {}`: Trường dữ liệu với key tương ứng, và value là **Hash of Scalar** thì sẽ pass.
Ta có thể kết hợp để tạo các Nested Parameters, lọc ra các trường ở mức sâu hơn.

Method này trả về một `Parameters` mới với thuộc tính `permitted? = true`

VD:
```ruby
params.permit(
    :name,
    { emails: [] },
    friends: [:name, { family: [:name], hobbies: []}]
)
```
- Lọc `:name` với giá trị Scalar
- Lọc `:email` với giá trị Array of Scalar
- Lọc `:friends` với giá trị là một Hash, bao gồm
    - Một key `:name`
    - Một key `:family` là một Hash với key `:name`
    - Một key `:hobbies` là một Array of Scalar

Ngoài ra, có method **`permit!`** dùng để buộc một `Parameters` nhận `permitted? = true`

# Response

Sau khi xử lý request, Action trong Controller có thể tạo một response theo một trong ba cách sau:
- Gọi `render` để trả về một Full Response
- Gọi `redirect_to` để trả về một HTTP Redirect Response
- Gọi `head` để tạo một HTTP Header Only làm Response

# Metal

`ActionController::Base` kế thừa từ `ActionController::Metal`, class này cung cấp các chức năng cốt lõi cho Controller, liên kết giữa Routing và Controller Action (**dispatch**), tuy nhiên, không có các chức năng Rendering hay Redirecting của `Base`

Một số attributes và methods mà Class này cung cấp:
- `request`: [Request](https://api.rubyonrails.org/v7.1.3/classes/ActionDispatch/Request.html) object tương ứng với Action hiện tại
- `response`: [Response](https://api.rubyonrails.org/v7.1.3/classes/ActionDispatch/Response.html) để tương tác với response trả về.
- `middleware`: Các Middleware mà Controller sử dụng
- `params`: `Parameters` từ `request`
- `session`: [Session] để tương tác với session.
- `controller_name`: Tên controller hiện tại (không bao gồm suffix "Controller")
- `action_name`: [Kế thừa từ `AbstractController::Base`]