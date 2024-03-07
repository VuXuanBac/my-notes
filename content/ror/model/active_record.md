---
title: Active Record
draft: false
date: 2024-03-04
tags:
  - ror
  - intro
---

**Active Record** là thư viện mặc định của Rails triển khai chức năng của ORM (**Object Relational Mapping**), tương tự _Entity Framework_ của ASP.

Active Record giúp ánh xạ giữa một bảng trong DB với một Class trong ứng dụng (gọi là **Model**), ngoài ra còn một số chức năng:
- Associations
- Validations.
- Model Inheritances

Tương tự các thành phần khác trong ứng dụng Rails, Active Record cũng triển khai theo quy tắc "_Convention over Configuration_":
- Model & Table:
    - Tên Model: Dạng số ít, CamelCase. VD: "CartItem"
        - Tên tệp dưới dạng snake_case.
    - Tên Table: Dạng số nhiều, snake_case. VD: "cart_items"
- Schema:
    - Primary Key mặc định sử dụng tên "id".
    - Foreign Key sử dụng tên bảng tham chiếu tương ứng: "table_id"
    - Tự động tạo các cột "id", "created_at", "updated_at"

# Model

Các Model Classes đều kế thừa từ `ActiveRecord::Base`, nó có thể có những thuộc tính ánh xạ tới các trường dữ liệu trong DB và có những thuộc tính khác - mang dữ liệu mức ứng dụng.

Dựa trên Convention, Model Class tự động ánh xạ sang một bảng của DB, và nó có các thuộc tính tương ứng với các cột.

VD:
```ruby
class Person < ActiveRecord::Base
end

p = Person.new
# <Person id: nil, name: nil, created_at: nil, updated_at: nil>
```

Khi một Model instance được tạo qua `new` method (giống các Ruby Classes khác) thì instance đó chỉ được lưu trong RAM, chưa phải một record trong DB. 

Ta có thể sử dụng method `new_record?` để kiểm tra một instance đã được lưu chưa - `true` tương ứng với chưa lưu.

# CRUD

Bên cạnh việc mỗi Model được ánh xạ với một bảng trong DB, các Models cũng được cung cấp một số methods để thao tác trực tiếp với DB.

Các methods nếu là Class Methods sẽ viết thông thường, nếu là instance method sẽ bắt đầu bằng `.`

Các methods này có phiên bản Bang `!`, thông thường, các Bang Methods này sẽ sinh lỗi nếu **Validation Fail** (còn phiên bản thường sẽ trả về `false` hoặc `nil`)

## Save

|Method|Description|Skip|
|--|--|--|
|`.save`, `.save!`|Lưu instance hiện tại vào DB, trả về `false` nếu không pass Validation||
|`.reload`|Nạp lại dữ liệu từ DB vào instance sử dụng ID||
|`insert`, `insert!`|Lưu một bản ghi sử dụng SQL Command, Bang sinh lỗi nếu trùng bản ghi|**Validations, Callbacks**|

## Create

|Method|Description|Skip|
|--|--|--|
|`create`, `create!`|Tạo và Lưu một/nhiều bản ghi sử dụng Hash (Array of Hash)||
|`build`|Tương đương `new`, tức là tạo instance nhưng chưa lưu vào DB|

## Read

|Method|Description|
|--|--|
|`all`|Danh sách các bản ghi|
|`first`, `last`||
|`find`|Tìm kiếm dựa trên một/nhiều ID. Sinh lỗi `RecordNotFound` nếu không thấy|
|`find_by`, `find_by!`|Tìm kiếm **một bản ghi** (LIMIT 1) theo một/nhiều trường|
|`where`|Tìm kiếm **tất cả bản ghi** theo một/nhiều trường|

## Update

|Method|Description|Skip|
|--|--|--|
|`update`, `update!`|Cập nhật dựa trên ID và Hash, lưu sử dụng `save` (và `save!`), trả về Object||
|`.update_attributes`|Cập nhật nhiều trường||
|`.update_attribute`, `.update_attribute!`|Cập nhật một trường|**Validations**|
|`.update_column`,`.update_columns`|Cập nhật dựa trên Hash theo SQL Command|**Validations, Callbacks, `updated_at`**|

## Delete

|Method|Description|Skip|
|--|--|--|
|`destroy`, `destroy!`, `.destroy`, `.destroy!`|Xóa một dựa trên ID||
|`delete`, `.delete`|Xóa dựa trên ID, sử dụng SQL Command, trả về số records/instance|**Callbacks, dependent**|

# Dirty

Các Active Record Model tự động triển khai chức năng của Module `ActiveModel::Dirty`

Module này dùng để theo dấu sự thay đổi của các thuộc tính, với một số instance methods, với `[attr]` là tên của thuộc tính của Model:
- `changed?`, `[attr]_changed?`: Kiểm tra có sự thay đổi không
- `[attr]_was`: Giá trị cũ của thuộc tính
- `[attr]_change`: Một mảng hai phần tử chứa giá trị cũ và giá trị hiện tại của thuộc tính
- `rollback!`: Quay lại giá trị cũ
- `changed`: Mảng tên các thuộc tính thay đổi
- `changes`: Hash chứa tên thuộc tính và sự thay đổi

```ruby
person = Person.new
person.changed? # => false

person.name = 'Bob'
person.changed?       # => true
person.name_changed?  # => true
person.name_changed?(from: nil, to: "Bob") # => true
person.name_was       # => nil
person.name_change    # => [nil, "Bob"]
person.name = 'Bill'
person.name_change    # => [nil, "Bill"]
```

# Support

## `delegate`

Active Support phần Module Extension cung cấp class method `delegate` với mục đích cho phép một object có thể gọi **trực tiếp** public methods của object bên trong nó.

Điều này rất hữu dụng khi truy xuất methods qua Model Association.

Đối số của macro này là:
- Danh sách các method muốn ủy quyền
- `to`: Object sở hữu các methods trên hoặc **`:class`** nếu muốn instance có thể truy xuất methods gắn cho chính class của nó. 
- `prefix`: Prefix tên các methods lấy được.
- `private`: Chuyển visibility của các methods lấy được về private bên trong Class hiện tại.

VD: Một hóa đơn (Invoice) gắn với một khách hàng (Client)

```ruby
class Client < ActiveRecord::Base
    has_many :invoice
    # has attributes :name, :address
end

class Invoice < ActiveRecord::Base
    belongs_to :client
    
    delegate :name, :address, to: :client, prefix: :customer
end

invoice = Invoice.first
invoice.customer_name # <=> invoice.client.name
invoice.customer_address # <=> invoice.client.address
```