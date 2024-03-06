---
title: View Helper
draft: false
tags:
  - ror
  - view
---

Helper là các **methods có thể gọi trên View**, dùng để đưa các đoạn mã xử lý logic phức tạp ra khỏi View, và đồng thời có thể chia sẻ dùng chung giữa các Views. 

Các Helpers được tổ chức theo **Module** đặt trong thư mục _**app/helpers**_.

Dưới đây là một số kiểu Helper:
- [Route Helper](../controller/routing.md#path-and-url-helpers): Các Helpers trả về đường dẫn tới một Route
- [Form Helpers](#form-helpers): Các Helpers giúp tạo Form (và các Form Controls), hỗ trợ tốt hơn việc sinh Form trực tiếp.
- [Asset Helper](../Other/asset_pipeline.md#asset-helper---retrieve): Các Helpers giúp sinh HTML liên kết tới các tài nguyên đã được đưa vào Asset Pipeline: Images, JS, StyleSheets,...
- [URL Helpers](#url-helpers): Các Helpers giúp tạo đường dẫn sử dụng kết quả của Routing.
- [String and Format](#)

Liên kết ngoài:
- [Danh sách Helpers](https://api.rubyonrails.org/v7.1.2/classes/ActionView/Helpers.html)
- [Guide](https://guides.rubyonrails.org/action_view_helpers.html)

# Some useful Helpers

Dưới đây là một số Helpers có thể dùng nhiều

## `distance_of_time_in_words`

Trả về String mô tả sự khác biệt về thời gian giữa hai mốc thời gian:

```ruby
distance_of_time_in_words(Time.now, Time.now + 15.seconds)
# => less than a minute
distance_of_time_in_words(Time.now, Time.now + 15.seconds, include_seconds: true)
# => less than 20 seconds
```

## `time_ago_in_words`

Trả về String mô tả khoảng thời gian đã trôi qua:

```ruby
time_ago_in_words(3.minutes.from_now) # => 3 minutes
```

## Number

- `number_to_currency`: Biểu diễn tiền tệ
- `number_to_human`: Biểu diễn viết gọn của số. 
    - `number_to_human(1234)    # => 1.23 Thousand`
- `number_with_delimiter`: Biểu diễn số phân tách theo phần nghìn.
- `number_with_precision`: Làm tròn số.
- `number_to_percentage`: Biểu diễn phần trăm
- `number_to_human_size`: Biểu diễn theo kích thước. 
    - `number_to_human_size(1234)    # => 1.21 KB`

## Sanitize

- `sanitize`: Mã hóa chuỗi theo HTML
- `strip_tags`: Loại bỏ các Tags.

# URL Helpers

## `url_for`

Dựa trên Routing, trả về đường dẫn tới một Object, cùng với các Options khác

Xem thêm: [UrlFor Module](https://api.rubyonrails.org/classes/ActionDispatch/Routing/UrlFor.html)
VD:

```ruby
url_for @profile
# => /profiles/1

url_for [ @hotel, @booking, page: 2, line: 3 ]
# => /hotels/1/bookings/1?line=3&page=2

url_for @post # given a composite primary key [:blog_id, :id]
# => /posts/1_2
```

## link_to

Tạo thẻ `<a>` với đường dẫn sử dụng `url_for`. Cú pháp:

`link_to(name = nil, options = nil, html_options = nil, &block)`
- **name**: Nội dung (phần Body) của `<a>`
- **options**: Các Options truyền cho `url_for`
- **html_options**: Các HTML `<a>` Attributes khác như `id, class, target, rel, type,...`
    - Khi xác định **html_options** cần bao đóng **options** vào Hash
- **block**: Phần Block làm nội dung cho `<a>`

Cụ thể:

### Basic

Xác định chuỗi làm nội dung (phần Body) cho `<a>`. Nếu nội dung là `nil` thì sử dụng đường dẫn.

```ruby
link_to "Profile", @profile
# => <a href="/profiles/1">Profile</a>

link_to "Profiles", profiles_path
# => <a href="/profiles">Profiles</a>

link_to nil, "http://example.com"
# => <a href="http://www.example.com">http://www.example.com</a>
```

### Block as Body

Sử dụng Block để định nghĩa phần nội dung.

```ruby
<%= link_to(@profile) do %>
  <strong><%= @profile.name %></strong> -- <span>Check it out!</span>
<% end %>
# => <a href="/profiles/1">
       <strong>David</strong> -- <span>Check it out!</span>
     </a>
```

### Turbo Support

Rails 7 hỗ trợ Turbo, và `link_to` sử dụng các `data` Options để xác định
- `turbo_method`: HTTP Method gửi request khi nhấn vào liên kết.
- `turbo_confirm`: Hiển thị một JS `confirm` với nội dung xác định khi nhấn vào liên kết.

```ruby
link_to "Log out", logout_path, data: { turbo_method: :delete, turbo_confirm: "Are you sure to log out?" }
```

### link_to_if, link_to_unless

Chỉ hiển thị dưới dạng `<a>` nếu điều kiện thỏa mãn, nếu không, chỉ hiển thị phần Body.

```ruby
<%=
   link_to_if(@current_user.nil?, "Login", login_path) do
     link_to(@current_user.login, account_path(@current_user))
   end
%>
# If the user isn't logged in...
# => <a href="/sessions/new/">Login</a>
# If they are logged in...
# => <a href="/accounts/show/3">my_username</a>
```

### link_to_unless_current

Điều kiện chỉ hiển thị dưới dạng `<a>` là: Đường dẫn tới View hiện tại **khác** với đường dẫn xác định cho `link_to`

VD: Trên NavBar, mục nào tương ứng với trang đang hiển thị sẽ không cần sử dụng liên kết.

## button_to

Tạo thẻ `<form>` với nội dung là một Submit `<button>`. Cú pháp tương tự `link_to`, trong đó
- Mặc định có thêm `<input type="hidden" name="authenticity_token">`
- **html_options** được truyền cho `<button>`, ngoại trừ
    - `form` xác định các Attributes cho `<form>`
    - `form_class` sẽ xác định `class` Attribute cho `<form>`
    - `params`: Mỗi phần tử key-value sẽ tương ứng một `<input type="hidden">` trong `form`
- **options** có thể chứa thêm (bên cạnh các `url_for` Options):
    - `method`: `<form>` Method, mặc định là POST.


## current_page?

Kiểm tra đường dẫn hiện tại có phù hợp với một số tiêu chí.
- `action`
- `controller`
- Các Options khác tùy thuộc Query String

VD: Giả sử đường dẫn kích hoạt View hiện tại là: **http://www.example.com/shop/checkout?order=desc&page=1**

```ruby
current_page?(action: 'process')
# => false  # [checkout]
current_page?(controller: 'shop', action: 'checkout', order: 'desc', page: '2')
# => false  # [page: '1']

current_page?('http://www.example.com/shop/checkout', check_parameters: true)
# => false # [Thiếu Query String trong khi `check_parameters: true`]
```

# Form Helpers

Form Helpers giúp hỗ trợ tạo `<form>` để tương tác với Resource Routes. Thường là Form tạo và cập nhật một tài nguyên.

Form Helper hỗ trợ chức năng sau:
- Xác định đường dẫn tương ứng với một Action - đường dẫn gửi Form.
- Sinh các thẻ để biểu diễn cho các trường dữ liệu của tài nguyên
- Binding với Object.

Để tạo một Form, ta sử dụng `form_with`, với Helper này, ta có thể sinh một Form với chức năng tổng quát (như Search Box) hoặc một Form gắn với một Object.

`form_with` nhận vào một Block với một đối số. Đối số này là một [**FormBuilder**](https://api.rubyonrails.org/v7.1.2/classes/ActionView/Helpers/FormBuilder.html), từ đây, ta có thể tạo các Form Controls cho `<form>` và gắn với một Object (Binding).

## Form Builder

Từ Form Builder, ta có thể sinh các Form Controls và Binding chúng tới các trường của một Object (nếu xác định cho Form).

Các Form Control này nhận đối số đầu tiên là tên của trường dữ liệu, tên này sẽ được dùng cho `name` và `id` HTML Attribute cũng như là **key** trong `params` gửi về Controller, value sẽ sử dụng `value` HTML Attribute.

Xem thêm: [Data Boxing](#data-boxing)

### check_box & radio_button

Với `check_box`, đối số
- [2] HTML Options
- [3] **checked_value**: Giá trị cho `value` Attribute khi Checked. Mặc định là "1"
- [4] **unchecked_value**: Giá trị cho `value` Attribute khi Checked. Mặc định là "0"

Checkbox sẽ được coi là CHECKED nếu giá trị của trường dữ liệu trùng với `value`.

```ruby
<%= form.check_box :is_robot, {}, "yes", "no" %>
<%= form.label :is_robot, "I am a Robot" %>
```

```html
<input type="checkbox" id="is_robot" name="is_robot" value="yes" />
<label for="is_robot">I am a Robot</label>
```

Với `radio_button`, đối số
- [2] **tag_value**: Giá trị cho `value` Attribute khi Checked.
- [3] HTML Options
    - `checked`: true nếu muốn chọn.

```ruby
<%= form.radio_button :gender, :male %>
<%= form.label :gender_male, "Male" %>
<%= form.radio_button :gender, :female %>
<%= form.label :gender_female, "Female" %>
```

```html
<input type="radio" id="gender_male" name="gender" value="male" />
<label for="gender_male">Male</label>
<input type="radio" id="gender_female" name="gender" value="female" />
<label for="gender_female">Female</label>
```

Chú ý, **label** ở đây sử dụng tên trường nối với giá trị cho RadioButton.

### Text Inputs

Xem thêm: [Form Builder Helpers](https://api.rubyonrails.org/v7.1.2/classes/ActionView/Helpers/FormBuilder.html#method-i-radio_button)

### select

Để tạo một `<select>` cùng với các `<option>` ta sử dụng Helper `select`, với đối số:
- [2] **choices**: Dữ liệu cho các `<option>`
- [3] Options
- [4] HTML Options:
- [5] Block: Tùy chỉnh giao diện cho `<option>`

**choices** xác định dữ liệu cho các `<option>`
- **Tập hợp các String**: Các `<option>` sẽ có `name` Attribute và phần nội dung giống nhau
- **Tập hợp bộ 2 phần tử**: Phần tử đầu sẽ dùng làm nội dung cho `<option>` và phần tử sau sẽ dùng làm `name`
- **Hash của các Tập hợp**: Mỗi phần tử trong Hash sẽ tương ứng một `<optgroup>` với `label` Attribute là Hash Key và nội dung là tập các `<option>` sinh từ Hash Value

Ta có thể lựa chọn một `<option>` bằng việc xác định `selected` theo sau là giá trị `name` của Option đó.
    - Khi xác định Model Object cho Form thì giá trị `selected` tự động được xác định.

VD1: 

```ruby
<%= form.select :city, [["Berlin", "BE"], ["Chicago", "CHI"], ["Madrid", "MD"]], selected: "CHI" %>
```

```html
<select name="city" id="city">
  <option value="BE">Berlin</option>
  <option value="CHI" selected="selected">Chicago</option>
  <option value="MD">Madrid</option>
</select>
```

VD2:

```ruby
<%= form.select :city,
      {
        "Europe" => [ ["Berlin", "BE"], ["Madrid", "MD"] ],
        "North America" => [ ["Chicago", "CHI"] ],
      },
      selected: "CHI" %>
```

```html
<select name="city" id="city">
  <optgroup label="Europe">
    <option value="BE">Berlin</option>
    <option value="MD">Madrid</option>
  </optgroup>
  <optgroup label="North America">
    <option value="CHI" selected="selected">Chicago</option>
  </optgroup>
</select>
```

## Form Object

Để gắn một **Model** Object với Form (Binding), ta xác định Object đó cho `model` Option.

```ruby
<%= form_with model: @article do |form| %>
  <%= form.text_field :title %>
  <%= form.text_area :body, size: "60x10" %>
  <%= form.submit %>
<% end %>
```

```html
<form action="/articles/42" method="post" accept-charset="UTF-8" >
  <input name="authenticity_token" type="hidden" value="..." />
  <input type="text" name="article[title]" id="article_title" value="My Title" />
  <textarea name="article[body]" id="article_body" cols="60" rows="10">
    My Body
  </textarea>
  <input type="submit" name="commit" value="Update Article" data-disable-with="Update Article">
</form>
```

### Convention

Từ Object này, Form xác định được
- Đường dẫn gửi Form `action`: Sử dụng `url_for` trên Object.
- Tự động điền giá trị cho các Form Controls có cùng tên trường dữ liệu trùng với tên thuộc tính của Object.
- `name` Attribute tự động xác định Scope dựa trên Object. Tức là dữ liệu từ các Form Control sẽ được đóng gói thêm một cấp nữa. VD: `params[:product][:name]` khi Model Object là một Product instance.
- Submit Button sẽ tự động nhận nội dung dựa trên việc Object có dữ liệu hay chưa. VD: "Create Product" nếu biến truyền vào chưa có dữ liệu.
- Form Method cũng tự động xác định dựa trên việc Object có dữ liệu hay chưa: POST cho Create (chưa có dữ liệu), và PATCH cho Update.

### Object in Namespace

Với đường dẫn nằm trong Namespace Routing, ta cần xác định phần Namespace cho `model`, cùng với Object.

```ruby
form_with model: [:admin, @article]
```
### Nested Form

Khi một Model Object có nhiều liên kết với Model Object khác (VD: Một Sách có nhiều Tác giả), ta có thể tạo Nested Form.
- Cấp cao nhất sử dụng `form_with`
- Các cấp sau sử dụng `fields_for`

`fields_for` cũng cung cấp Form Builder

VD:

```ruby
<%= form_with model: @person do |form| %>
  Addresses:
  <ul>
    <%= form.fields_for :addresses do |addresses_form| %>
      <li>
        <%= addresses_form.label :kind %>
        <%= addresses_form.text_field :kind %>

        <%= addresses_form.label :street %>
        <%= addresses_form.text_field :street %>
        ...
      </li>
    <% end %>
  </ul>
<% end %>
```

## Options from Database Data

Khi muốn sử dụng dữ liệu đang có trong DB để làm các lựa chọn ta có thể sử dụng một số Helpers sau:

### `collection_select`

Sử dụng `<select>`

Các đối số bao gồm:
- [2]: Tập hợp các Objects, (thường là lệnh truy vấn. VD: `Product.all`)
- [3]: Thuộc tính dùng làm `value` cho `<option>`
- [4]: Thuộc tính dùng làm Body cho `<option>`

```ruby
<%= form.collection_select :city_id, City.order(:name), :id, :name %>
```
```html
<select name="person[city_id]" id="person_city_id">
  <option value="3">Berlin</option>
  <option value="1">Chicago</option>
  <option value="2">Madrid</option>
</select>
```

### `collection_radio_buttons` & `collection_check_boxes`

Tương tự `collection_select`

## Data Boxing

Dữ liệu trong Form được đóng gói vào `params` như sau:
- Các trường thuộc tính trong Model Object được đóng gói vào một key ứng với tên của Model Object đó
    - `params[:product][:name]`
    - `name` Attribute Value: `product[name]`

Xem thêm: [Parameter Naming Convention](https://guides.rubyonrails.org/form_helpers.html#understanding-parameter-naming-conventions)