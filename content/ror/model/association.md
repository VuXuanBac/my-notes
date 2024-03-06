---
title: Association
draft: false
tags:
  - ror
  - model
---

Association là tập các Macro (Class Methods) dùng để gắn kết các Objects qua Foreign Keys.
Sau khi thêm các Associations, Class có thể duyệt và thay đổi các object có quan hệ với nó thông qua các Generated Methods (từ macro tương ứng)

# Relationship Handling

## One-to-One

VD: Một **Account** gắn với một và chỉ một **UserInfo**, ta muốn thêm Foreign Key trong **Account** với tên là `user_info_id`

Sử dụng `has_one` trong Base Model (**UserInfo**), sử dụng `belongs_to` trong Associated Model (**Account**) - Model có Foreign Key.

## One-to-Many

VD: Một **Account** có thể tạo nhiều **Comment**, mỗi **Comment** chỉ gắn với một **Account**. Lúc này, trên **Comment**, ta sử dụng Foreign Key là `account_id`.

Sử dụng `has_many` trong Base Model (**Account**), sử dụng `belongs_to` trong Associated Model (**Comment**) - Model có Foreign Key.

## Many-to-Many

VD1: Một **Account** có thể yêu thích **Post** (của người khác), mỗi **Post** có thể được yêu thích bởi nhiều **Account**. Lúc này, ta cần tạo một bảng liên kết giữa hai bảng này - **Favorite**, bảng này về cơ bản chỉ cần có Foreign Keys: `account_id`, `post_id` tới hai bảng.

VD2: Một **Account** có thể tạo nhiều đánh giá cho mỗi **Post** (của người khác), mỗi **Post** có thể được đánh giá bởi nhiều **Account**. Lúc này, ta cần tạo một bảng liên kết giữa hai bảng này - **Rate**. Bảng này cần có Foreign Keys: `account_id`, `post_id` tới hai bảng, ngoài ra còn cần `star` thể hiện mức độ đánh giá.

### Direct

Cách này không cần tạo **Model** trung gian tương ứng với bảng liên kết trong DB (bảng **Rate**, **Favorite**)

1. Tức là chỉ cần tạo bảng trong Migration. VD:

```ruby
class JoinAccountsPostsFavorite < ActiveRecord::Migration[7.1]
  def change
    # C1: Tạo bảng như thông thường
    create_table :favorites, id: false do |t|
      t.belongs_to :account
      t.belongs_to :post
    end

    # C2: Tạo bảng như thông thường
    create_join_table :accounts, :posts, table_name: :favorites do |t|
      t.index :account_id
      t.index :post_id
    end
  end
end
```

2. Sau đó, trong hai Model **Account** và **Post** ta thêm Associations, sử dụng macro `has_and_belongs_to_many`:
```ruby
class Account < ApplicationRecord
  has_and_belongs_to_many :favorite_posts, 
    join_table: "favorites",
    class_name: "Post"
end

class Post < ApplicationRecord
  has_and_belongs_to_many :favoriters, 
    join_table: "favorites",
    class_name: "Account"
end
```

Tuy nhiên, cách này thường không được ưa chuộng vì 
- Nếu bảng liên kết có một số thuộc tính thêm (ngoài 2 Foreign Keys) thì không thể xử lý nó trên Model.
- Không có Model đồng nghĩa không thể tạo Validations, Callbacks,...
- Cách này tạo trực tiếp liên kết N-N

### Indirect

Cách này chuyển liên kết N-N về hai liên kết 1-N, tức là ta sẽ tạo **Model** cho bảng liên kết.

1. Tạo Table và Model cho hai liên kết

```ruby
# Migration
def change
    create_table :rates do |t|
        t.belongs_to :account, null: false, foreign_key: true
        t.belongs_to :post, null: false, foreign_key: true
        t.integer :value, null: false

        t.timestamps
    end
end

# Model
class Rate < ActiveRecord::Base
  belongs_to :post
  belongs_to :rater, class_name: Account.name,       
                     foreign_key: :account_id
end
```

2. Sử dụng `has_many` và `has_many through` trong hai Associated Models **Account** và **Post**

```ruby
class Account < ActiveRecord::Base
  has_many :rates
  has_many :rated_posts, through: rates, source: :post
end

class Post < ActiveRecord::Base
  has_many :rates
  has_many :raters, through: rates
end
```

# Macros

- **Base Model**: 
    - Model được tham chiếu
    - Không chứa Foreign Key
    - VD: **UserInfo**
- **Associated Model**:
    - Model tham chiếu tới Model khác trong quan hệ 1-1, 1-N
    - Chứa Foreign Key tham chiếu tới Model đó.
    - VD: **Comment**
- **Join Model**:
    - Model trung gian kết nối hai Models trong quan hệ N-N
    - Chứa hai Foreign Keys tham chiếu tới hai Models đó
    - VD: **Favorite, Rate**

## belongs_to

- Dùng trong quan hệ 1-1, 1-N, N-N (Indirect)
- Xác định trong **Associated Model** (1-N) hoặc **Join Model** (N-N)
- Cú pháp:
```ruby
    belongs_to :<association_name>, [scope], [options]
```
- Tên Association ở dạng **số ít**.
- Options:

| Option | Description | Implicit Value | 
|---|---|---|
| `class_name` | **Base Model** | Tên asso |
| `foreign_key` | Tên Foreign Key [Bảng hiện tại] | Tên asso gắn với `_id` |
| `inverse_of` | Tên asso khai báo trong **Model** bên kia | |
| `optional` | `true` nếu muốn Associated Object có thể nhận NULL | false |
## has_one

- Dùng trong quan hệ 1-1
- Xác định trong **Base Model**
- Cú pháp
```ruby
    has_one :<association_name>, [scope], [options]
```
- Tên Association ở dạng **số ít**.
- Options:

| Option | Description | Implicit Value | 
|---|---|---|
| `class_name` | **Associated Model** | Tên asso |
| `foreign_key` | Tên Foreign Key | Tên Model hiện tại gắn với `_id` |
| `inverse_of` | Tên asso khai báo trong **Model** bên kia | |
| `dependent` | Cách thức xử lý Associated Object khi Object hiện tại bị xóa | **nil**: Không xử lý |

## has_many

- Dùng trong quan hệ 1-N
- Xác định trong **Base Model**
- Cú pháp

```ruby
    has_many :<association_name>, [scope], [options]
```
- Tên Association ở dạng **số nhiều**.
- Options:

| Option | Description | Implicit Value | 
|---|---|---|
| `class_name` | **Associated Model** | Tên asso |
| `foreign_key` | Tên Foreign Key | Tên Model hiện tại gắn với `_id` |
| `inverse_of` | Tên asso khai báo trong **Model** bên kia | |
| `dependent` | Cách thức xử lý Associated Object khi Object hiện tại bị xóa | **nil**: Không xử lý |

## has_many through

- Dùng trong quan hệ N-N (Indirect)
- Xác định trong **Base Model**
- Chỉ rằng: Model hiện tại [A] có thể truy xuất tới một Model thứ 3 [C] qua **Join Model** [B]
- Cú pháp
```ruby
    has_many :<association_name>, [scope], through: :<another_asso> *[options]
```
- Tên Association ở dạng **số nhiều**
- Options:

| Option | Description | Implicit Value | 
|---|---|---|
| `class_name`, `foreign_key` | Không có ý nghĩa | |
| `through` | Tên của asso giữa Model hiện tại [A] và **Join Model** [B] (và asso đó bên trong [A]) | |
| `source` | Tên của asso giữa **Join Model** [B] và Model thứ 3 [C] (và asso đó bên trong [B]) | Tên asso hiện tại |

## has_one through

- Dùng trong quan hệ 1-1-1 (Indirect)
- Xác định trong **Base Model**
- Chỉ rằng: Model hiện tại [A] có thể truy xuất tới một Model thứ 3 [C] qua Model [B]
- Cú pháp
```ruby
    has_one :<association_name>, [scope], through: :<another_asso> *[options]
```
- Tên Association ở dạng **số ít**
- Options:

| Option | Description | Implicit Value | 
|---|---|---|
| `class_name`, `foreign_key` | Không có ý nghĩa | |
| `through` | Tên của asso giữa Model hiện tại [A] và Model [B] (và asso đó bên trong [A]) | |
| `source` | Tên của asso giữa Model [B] và Model [C] (và asso đó bên trong [B]) | Tên asso hiện tại |

# Generated Methods

Các Methods được sinh ra sau khi tạo Association Macro

## `has_one` and `belongs_to`

- Gọi `asso` là tên Association được tạo, các methods được sinh ra dựa trên giá trị này

| Generated Method | Description  |
|---|---|
| `asso`  | Đọc Associated Object, hoặc **nill** |
| `reload_asso`  | Tương tự `asso`, nhưng luôn Query lại từ DB |
| `asso=` | Gán lại Foreign Key cho Associated Object là ID của Base Object |
| `build_asso` | Tạo một Associated Object có Foreign Key là ID của Base Object, đối số là các thuộc tính của Object cần tạo |
| `create_asso` | Tương tự `build_asso` nhưng lưu ngay. Trả về `nil` nếu Validate Fail |
| `create_asso!` | Tương tự `create_asso` nhưng sinh lỗi nếu Validate Fail |

## has_many

- Gọi `coll` là tên Association được tạo, các methods được sinh ra dựa trên giá trị này

| Generated Method | Description  |
|---|---|
| `coll`  | Đọc toàn bộ Associated Objects và gán cho một **Relation** |
| `coll<<`  | Thêm trực tiếp phần tử cho Associated Objects (Update DB, Validation, Callback) - gán Foreign Key là ID của Object hiện tại |
| `coll.delete`  | Gán lại Foreign Key là NULL cho các phần tử bị xóa và thực thi cấu hình xác định trong `dependent` |
| `coll.destroy`  | Hủy trực tiếp các phần tử bị xóa (Update DB, Callback) |
| `coll.reload` | Tương tự `coll` nhưng Query lại vào DB |
| `coll.build` | Tạo và thêm một/nhiều phần tử vào Associated Objects, gán Foreign Key là ID của Object hiện tại, đối số là các thuộc tính của Object cần tạo |
| `coll.create` | Tương tự `coll.build` nhưng lưu ngay. Trả về `nil` nếu InValid |
| `coll.create!` | Tương tự `coll.create` nhưng sinh lỗi nếu InValid |

# Nested Attributes

Nested Attributes cho phép cập nhật associated object qua object chứa nó (VD: Cập nhật thông tin người dùng UserInfo thông qua tài khoản đăng nhập Account)

Để khai báo Nested Attributes cho một Model, ta sử dụng macro `accepts_nested_attributes_for`, theo sau là danh sách các associated attributes.

Khai báo macro này cũng tự động cung cấp cho Model một Attribute Writer dưới dạng `<name>_attributes=`

```ruby
class Book < ActiveRecord::Base
  has_one :author
  has_many :pages

  accepts_nested_attributes_for :author, :pages
end

# book.author_attributes=
# book.pages_attributes=
```

Sau khi khai báo Nested Attributes, khi tạo hoặc cập nhật Parent Object, ta có thể cung cấp Hash Params với key dạng `<name>_attributes` để cập nhật cho Associated Object.

VD: One Account - One Avatar

```ruby
class Member < ActiveRecord::Base
  has_one :avatar
  accepts_nested_attributes_for :avatar
end

params = { member: { name: 'Jack', avatar_attributes: { icon: 'smiling' } } }
member = Member.create(params[:member])
member.avatar.id # => 2
member.avatar.icon # => 'smiling'
```

VD: One Member - Many Posts
```ruby
class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts
end

params = { member: {
  name: 'joe', posts_attributes: [
    { title: 'Kari, the awesome Ruby documentation browser!' },
    { title: 'The egalitarian assumption of the modern citizen' },
    { title: '', _destroy: '1' } # this will be ignored
  ]
}}

member = Member.create(params[:member])
member.posts.length # => 2
member.posts.first.title # => 'Kari, the awesome Ruby documentation browser!'
member.posts.second.title # => 'The egalitarian assumption of the modern citizen'
```

* Khi tạo Form cho cập nhật/tạo Nested Attributes, ta thường sử dụng `fields_for`, nó giúp tạo một Scope cho `params`

```ruby
<%= form_for @person do |person_form| %>
  ...
  <%= person_form.fields_for :address do |address_fields| %>
    Street  : <%= address_fields.text_field :street %>
    Zip code: <%= address_fields.text_field :zip_code %>

    Delete: <%= address_fields.check_box :_destroy %>
  <% end %>
  ...
<% end %>
```

## Destroy through Nested Attributes

Mặc định chỉ có thể tạo và cập nhật Nested Attributes, để thực hiện xóa, ta cần xác định option `allow_destroy: true` cho macro. 

Để chỉ định một record bị xóa, ta cung cấp Hash với hai trường `{id: <object_id>, _destroy: true}` khi thực hiện cập nhật Parent Object.

## Reject If

Ta có thể chỉ định một Proc/Method cho Macro với option `reject_if` nếu muốn xác định một điều kiện chỉ cập nhật những Associated Object nếu giá trị (attributes) truyền vào thỏa mãn một tiêu chí xác định

```ruby
class Member < ActiveRecord::Base
  has_many :posts
  accepts_nested_attributes_for :posts, reject_if: proc { |attributes| attributes['title'].blank? }
end

params = { member: {
  name: 'joe', posts_attributes: [
    { title: 'Kari, the awesome Ruby documentation browser!' },
    { title: 'The egalitarian assumption of the modern citizen' },
    { title: '' } # this will be ignored because of the :reject_if proc
  ]
}}

member = Member.create(params[:member])
member.posts.length # => 2
```

# Polymorphic

Polymorphic là một Association đặc biệt, mô tả một Model có quan hệ với nhiều Models khác cùng trên một liên kết (Foreign Key)

Polymorphic Model là Model thuộc về nhiều hơn một Base Models, nói cách khác, Polymorphic được sử dụng khi nhiều Models có chung nhau một Associated Model.

VD: Hệ thống giảng dạy có hai vai trò **`Teacher`** và **`Student`**, các Model này mang thông tin riêng. Và cả hai vai trò này đều sử dụng chung một hình thức đăng nhập qua **`Account`** (email, password). Lúc này, `Account` là **Polymorphic Model**, nó mang một liên kết tới cả hai Model trên.

Vì có liên kết tới nhiều Models cùng lúc, một Polymorphic Model cần phải khai báo thêm (bên cạnh Foreign Key `id`) một trường **`type`** để xác định bản ghi tương ứng liên kết với Model nào.

```ruby
# Migration
class CreateAccounts < ActiveRecord::Migration[7.1]
  def change
    create_table :accounts do |t|
      t.string  :email
      t.string  :password_digest
      t.bigint  :user_id
      t.string  :user_type
      t.timestamps
    end

    add_index :accounts, [:user_type, :user_id]
  end
end

# Models
class Account < ApplicationRecord
  belongs_to :user, polymorphic: true
end

class Teacher < ApplicationRecord
  has_one :account, as: :user
end

class Student < ApplicationRecord
  has_one :account, as: :user
end
```

Option **`as`** cho `has_one` hay `has_many` được dùng khi muốn xác định Association hiện tại liên kết tới một Polymorphic với tên tương ứng.

# STI

Single Table Inheritance (STI) tận dụng cơ chế kế thừa Class giúp triển khai các Models có **chung các thuộc tính (dữ liệu trong DB)** nhưng có hành vi khác nhau.

STI đề cập đến việc chỉ sử dụng một bảng duy nhất cho Base Model và các Models kế thừa sẽ **dùng chung bảng này**. Bảng chung cần có thêm trường **`type`** để xác định Class_Name mà bản ghi thuộc về.

VD: Một **`Review`** (đánh giá của người dùng về sản phẩm) và **`Comment`** (phản hồi từ người bán) chỉ có những thuộc tính của **`Message`** (nội dung, tài khoản...). Khi đó, ta chỉ cần tạo bảng cho **`Message`**, và để cho `Review` và `Comment` kế thừa từ đó. `Review` và `Comment` có thể có Validations, Callbacks, Methods riêng.

```ruby
# Migration
class CreateMessages < ActiveRecord::Migration[7.1]
  def change
    create_table :messages do |t|
      t.string  :content
      t.references  :account
      t.string  :group, null: false

      t.timestamps
    end
  end
end

# Models
class Message < ApplicationRecord
  # Xác định tên cột dùng làm `type`
  self.inheritance_column = "group"
  belongs_to :account
end

class Review < Message
  # Custom here
end

class Comment < ApplicationRecord
  # Custom here
end
```

Khi tạo bản ghi, ta có thể tạo qua SubClass hoặc BaseClass

```ruby
Review.create(account_id: 1, content: "Hello World")

# Nếu không xác định `group` (i.e `type`), sinh lỗi ActiveRecord::SubclassNotFound
Message.create(group: "Review", account_id: 1, content: "Hello World")
```