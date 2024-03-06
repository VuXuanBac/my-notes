---
title: Validation
draft: false
tags:
  - ror
  - model
---

Validation giúp đảm bảo (mức ứng dụng - Model Level) chỉ những dữ liệu hợp lệ mới có thể lưu vào DB. 

Thực tế có một số cách khác cho đảm bảo toàn vẹn dữ liệu:
- **Database Constraints**: Triển khai trực tiếp trên DB, nhưng khó bảo trì. Thường chỉ triển khai những ràng buộc đơn giản mà dùng nhiều như Unique.
- **Client-side Validation**: Không đáng tin cậy, song phản hồi nhanh.
- **Controller-side Validation**: Triển khai trên Controllers, song khó bảo trì và làm phức tạp Controller.

# How it works

Một số thao tác nhất định trong Model có thể tự động kích hoạt Validation, một số khác thì không. Thông thường Validation chạy khi một Model instance được lưu hoặc cập nhật vào DB. Khi instance đó không qua được Validation, nó sẽ được gán nhãn **InValid** và thao tác tương ứng không được thực hiện.

Ta có thể chủ động chạy Validation trên instance bằng việc gọi `valid?` hoặc `invalid?`.

Khi instance chạy qua Validation, nếu không thành công, các lỗi (lý do không thành công) được thêm vào **`errors`**, nó giống một Hash giữa thuộc tính được kiểm tra và danh sách các thông báo lỗi.

_Thực tế, việc chạy Validation nếu không thành công sẽ thêm lỗi vào `errors`, còn `valid?` hay `invalid?` sẽ kiểm tra `errors` xem có rỗng không._

## Work with `errors`

Ta có thể tương tác với các lỗi Validation qua `errors` như sau:
- `errors` là một Hash ánh xạ: `<attribute> => <messages_array>`
- `errors.full_messages`: Mảng các thông báo lỗi
- `errors.add`: Thêm lỗi, đối số lần lượt là:
    - Attribute Name
    - Error Type [optional]
    - Hash Options:
        - `message` thể hiện lý do lỗi, nó sẽ được nối với Attribute Name để tạo thành `full_message` là dạng có thể render ra View.
- Một key đặc biệt trong `errors` là `:base`, nó liên quan đến các lỗi chung cho instance, không liên quan đến thuộc tính nào cả.
- `any?` Kiểm tra có lỗi nào không
- `count` Trả về số lượng lỗi tổng cộng

# Validator

Validators là một đối tượng định nghĩa cách thức Validate cho một Model instance. Rails cung cấp một số cách để thêm Validator như sau: [ActiveModel::Validations::ClassMethods](https://api.rubyonrails.org/v7.1.2/classes/ActiveModel/Validations/ClassMethods.html)

Khi thêm Validator cho Model, ta có thể xác định một số Options sau (có thể dùng cho mọi Validators):

- `on`: Xác định Validation Context. Giá trị:
    - `:create`: Gọi Validation cho thao tác tạo mới
    - `:update`: Gọi Validation cho thao tác cập nhật.
    - Mặc định là kiểm tra toàn bộ
- `message`: Xác định thông báo lỗi khi Validation không thành công.
    - Có thể sử dụng các placeholder sau: `value`, `attribute`, `model` (Class Name)
    - Tùy vào các Validators có thể có thêm placeholders khác.
    - Ta có thể truyền vào PROC với hai đối số:
        - **object**: Đại diện cho Model instance
        - **data**: Hash với các keys: `model`, `attribute`, `value`
- `allow_nil`: `true` tức là Bỏ qua Validation nếu Attribute tương ứng nil.
- `allow_blank`: `true` tức là Bỏ qua Validation nếu Attribute tương ứng blank?.
- `strict`: `true` nếu muốn sinh Exception khi Validation không thành công (thay vì mặc định là chỉ thêm lỗi vào `errors`)
    - Exception: `ActiveModel::StrictValidationFailed`
- `if`, `unless`: Chỉ gọi Validation trong những điều kiện nhất định.
    - Xác định một PROC hoặc METHOD

# Validation Helpers

Các Validation Helpers là các Validators được định nghĩa sẵn, có thể dùng cho các Models để định nghĩa các Validations cho các thuộc tính của Model đó.

Điểm chung của các Helpers
- Tên có dạng `validates_<name>_of`
    - Với `<name>` là tên rút gọn cho Helper tương ứng.
- Có thể gắn cho cùng lúc nhiều thuộc tính. Các thuộc tính có thể là hoặc không là trường dữ liệu trong DB

Thay vì dùng các Helpers, ta có thể sử dụng `validates` với cú pháp tương đương:

```ruby
# Use Validation Helpers
validates_<name>_of [:attr]s, *[options]

# Use `validates`
validates [:attr]s, [name]: [options]
```

Danh sách các Validators hỗ trợ sẵn: [ActiveModel::Validations::HelperMethods](https://api.rubyonrails.org/v7.1.2/classes/ActiveModel/Validations/HelperMethods.html)

| Helper Name | Description | Hash Options | 
|---|---|---|
| `acceptance` | [Checkbox] `true` yêu cầu thuộc tính tương ứng phải nhận giá trị `true` (checked) | `accept` (giá trị hoặc mảng thể hiện giá trị chấp nhận được) |
| `confirmation` | [Hai dữ liệu nhập vào giống nhau] `true` sẽ tự động tạo một **Virtual Attribute** cho Model có tên `..._confirmation` và kiểm tra giá trị giữa chúng khi chạy Validation | `case_sensitive` (`false` nếu không phân biệt hoa thường) |
| `comparison` | So sánh với một thuộc tính hoặc một hằng | `greater_than`, `greater_than_or_equal_to`, `equal_to`, `other_than`... |
| `format` | Khớp với một REGEX hoặc một Proc trả về REGEX | `with REGEX` (khớp) hoặc `without REGEX` (không khớp)|
| `inclusion`, `exclusion` | Nhận/Không nhận một trong các giá trị trong một Enumerable hoặc một Proc trả về Enumerable | `in ENUMERABLE` |
| `length` | Kích thước của thuộc tính | `in RANGE`, `minimum`, `maximum`, `is`. Ghi đè `messages`: `too_long`, `too_short`, `wrong_length` [Placeholder: %{count}] |
| `numericality` | `true` nếu muốn chỉ nhận giá trị số | `only_integer`, `odd`, `even`, `greater_than`,..., `in RANGE` |
| `presence` | `true` nếu không được blank? | `only_integer`, `odd`, `even`, `greater_than`,..., `in RANGE` |
| `uniqueness` | `true` nếu muốn UNIQUE. Thực hiện Query | - `scope` (giảm mức duy nhất khi xét cùng với các thuộc tính khác),<br>- `case_sensitive` (`false` nếu không muốn phân biệt hoa thường khi so sánh), <br>- `conditions PROC` (giảm mức duy nhất khi xét trong một điều kiện) |

# Custom Validators

Có một số cách để thêm Validators cho Model như sau

## `validate` - Validate for Whole Instance

Method này cho phép thêm các **Method/Block Validators** cho Model, song sẽ kiểm tra trên toàn bộ Model instance.

```ruby
validate [:METHOD], *[options], &BLOCK
```

Method truyền cho `validate`
- Không có đối số (có thể sử dụng trực tiếp các methods và attributes bên trong Model)
- Giá trị trả về không có ý nghĩa. 
- Khi muốn thể hiện Validation không thành công: Thực hiện thêm một `errors` với tên thuộc tính kiểm tra (hoặc `:base` nếu không xác định thuộc tính kiểm tra) và thông báo lỗi.

VD:
```ruby
class Comment
  include ActiveModel::Validations

  validate :must_be_friends

  def must_be_friends
    errors.add(:base, 'Must be friends to leave a comment') unless commenter.friend_of?(commentee)
  end
end
```

Block truyền cho `validate` 
- Đối số duy nhất đại diện cho Model instance hiện tại.
- Giá trị trả về không có ý nghĩa. 
- Thực hiện cập nhật `errors` để thể hiện Validation không thành công.

VD:
```ruby
class Comment
  include ActiveModel::Validations

  validate do |comment|
    comment.must_be_friends
  end

  def must_be_friends
    errors.add(:base, 'Must be friends to leave a comment') unless commenter.friend_of?(commentee)
  end
end
```

## `validates_each` - Validate for each Attributes

Method này cho phép thêm các **Block Validators** cho Model, song sẽ được gọi với mỗi Attributes được truyền vào.

Cú pháp của `validates_each` :

```ruby
validates_each [:attr]s, *[options], &BLOCK
```

Block truyền cho `validates_each` 
- Đối số:
    - _record_: Model instance hiện tại.
    - _attribute_: Thuộc tính kiểm tra.
    - _value_: Giá trị hiện tại của thuộc tính 
- Giá trị trả về không có ý nghĩa. 
- Thực hiện cập nhật `errors` để thể hiện Validation không thành công.

## `validates, validates!` - Validate for each Attributes

Methods này bên cạnh việc thêm các Validators có sẵn (dạng shortcut), còn có thể thêm các **Class Validators**, song sẽ kiểm tra mỗi Attributes truyền vào.

Phiên bản nghiêm ngặt hơn của `validates` là `validates!`, thay vì thêm lỗi vào `errors`, nó sẽ sinh Exception.

Thông thường, ta nên tổ chức các Custom Validators theo thư mục (VD: app/validators/) và các tệp:
- Thêm cấu hình đường dẫn vào _config/application.rb_. VD: `config.autoload_paths += %W["#{config.root}/app/validators/"]`

Sau đó, tạo một Class có tên dạng `[Name]Validator` - Kế thừa từ **`ActiveModel::EachValidator`** 
- Override method **`validate_each`**. Đối số của nó là:
    - _record_: Model instance hiện tại.
    - _attribute_: Thuộc tính kiểm tra.
    - _value_: Giá trị hiện tại của thuộc tính tương ứng trong instance.
- Trong **`validate_each`**, thêm lỗi vào `errors` để chỉ Validation không thành công.

Sau khi tạo Validator như trên, ta thêm nó vào Model qua `validates`

```ruby
validates [:attr]s, [name]: [options]
```

- `[name]` tương ứng với `[Name]` trong tên Class
- `[options]` được đóng gói vào Hash `options` và truyền vào Class
- Với mỗi `[attr]`, `validate_each` sẽ được gọi để thực hiện Validation

VD:

```ruby
# email_validator.rb
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    record.errors.add attribute, (options[:message] || "is not an email") unless
      /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i.match?(value)
  end
end

# person.rb
class Person
  include ActiveModel::Validations
  attr_accessor :name, :email

  validates :name, presence: true, length: { maximum: 100 }
  # Dùng EmailValidator để kiểm tra thuộc tính :email.
  validates :email, presence: true, email: true

  # Lúc này, `options: {message: "Invalid Email"} ` được truyền vào EmailValidator Class
  validates :email, presence: true, email: {message: "Invalid Email"}
end
```

## `validates_with` - Validate for Whole Instance

Method này cho phép thêm các **Class Validators** cho Model, song sẽ chỉ kiểm tra trên toàn bộ instance.

Cụ thể, ta tạo một Class có tên dạng `[Name]Validator` - Kế thừa từ **`ActiveModel::Validator`** 
- Override method **`validate`**. Đối số của nó là:
    - _record_: Model instance hiện tại.
- Trong **`validate`**, thêm lỗi vào `errors` để chỉ Validation không thành công.

Sau đó, ta thêm Validator này cho Model theo cú pháp:

```ruby
validates_with [Name]Validator
```

# Validation for Association

Khi một Model có liên kết với một Model khác, ta có thể xác định `validates_associated`, để chỉ định thực hiện Validation cho cả các associated objects khi chạy Validation cho instance hiện tại. Đối số của method này là danh sách các associated objects.

Chú ý: 
- **`validates_associated` không được đặt ở cả hai đầu của Association, điều đó sẽ dẫn đến lặp vô hạn.**
- **`errors` sẽ được gán cho từng Model instance mà không chuyển qua Association**

VD:
```ruby
class Book < ActiveRecord::Base
  has_many :pages
  belongs_to :library

  validates_associated :pages, :library
end
```

Method này sẽ không chạy Validation cho associated objects nếu instance hiện tại chưa gán giá trị cho associated objects này. Ta cần xác định thêm `validates_presence_of` để đảm bảo associated objects được gắn cùng với instance khi thực hiện Validation.
