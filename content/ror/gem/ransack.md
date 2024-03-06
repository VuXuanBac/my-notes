---
title: Ransack
draft: false
tags:
  - ror
  - gem
---

Gem Ransack giúp triển khai chức năng **search** và **sort** cho ứng dụng sử dụng chính cú pháp Ruby và ERB.

Cấu hình:

```ruby
# Gemfile
gem "ransack"
```

# `Ransack::Search`

Với những lệnh tìm kiếm và sắp xếp chỉ đòi hỏi câu truy vấn đơn giản, ta có thể sử dụng ngay Ransack.

Sau khi cài Gem, `ActiveRecord::Base` sẽ được mở rộng với method **`ransack`**, nó nhận vào một _Hash các tiêu chí tìm kiếm/sắp xếp_ và trả về một **`Ransack::Search`** object. 

Từ object này, ta có thể gọi **`result`** để thực hiện các truy vấn tìm kiếm và sắp xếp trên Model (tương tự như gọi các `scope`)

```ruby
def index
  @q = Person.ransack params[:q]
  @people = pagy @q.result.includes(:articles)
end
```

## Combinator

Khi có nhiều tiêu chí tìm kiếm, mặc định Ransack sẽ kết hợp chúng dưới quan hệ AND. Ta có thể xác định quan hệ OR bằng việc xác định key `:m` cho params

```ruby
Post.ransack(params[:q].try(:merge, m: 'or'))
```

## Sort

Với các tiêu chí sắp xếp, mỗi tiêu chí được mô tả bởi một **`Ransack::Nodes::Sort`** và đóng gói thành một mảng, ta có thể truy cập các tiêu chí sắp xếp từ `Ransack::Search` qua method **`sorts`**

```ruby
@q = Post.ransack(params[:q])
@q.sorts = ['title asc', 'created_at desc'] if @q.sorts.empty?

@posts = @q.result(distinct: true)
```

Các Sort Params sẽ được đóng gói bên trong key `:s` trong `:q` params.

## SQL

Ta có thể sử dụng `@q.result.to_sql` để xem lệnh SQL tương ứng mà một `Ransack::Search` dùng để truy vấn.

# Matchers

Matcher chính là các tiêu chí tìm kiếm/sắp xếp.

**Search Matcher** mô tả thuộc tính và cách thức so sánh thuộc tính đó (**predicate**) với một **value** để cho ra kết quả.

Một số Predicates cho Search Matchers (`*` để chỉ thuộc tính dùng để so sánh)

- `*_eq`, `*_not_eq`
- `*_matches`: Sử dụng LIKE
- `*_start`: Sử dụng `LIKE 'value%'`
- `*_cont`: Contains

**Sort Matcher** mô tả thuộc tính và hướng sắp xếp (DESC, ASC) cho thuộc tính đó.

```
/posts?q[title_cont]=ab&q[s]=created_at+desc
```

tương ứng việc tìm kiếm các Post có `Post.title` chứa chuỗi `ab` và sắp xếp kết quả theo trường `Post.created_at` giảm dần.

## Multiple Columns



## Associations

Ta có thể áp dụng các tiêu chí tìm kiếm/sắp xếp trên Association của Model tương tự như với một thuộc tính trực tiếp của Base Model.

VD: Một `Post` có tác giả là một `Account`

```ruby
class Account < ActiveRecord::Base
  has_many :posts

  # has attribute username:string
end

class Post < ActiveRecord::Base
  belongs_to :author, class_name: Account.name, foreign_key: :account_id

  # has attributes title:string and content:string
end
```

Khi tìm kiếm/sắp xếp `Post` ta có thể tìm kiếm/sắp xếp dựa trên các thuộc tính của `Account`

```ruby
<%= search_form_for @q do |f| %>
  <%= f.label :author_username_cont %>
  <%= f.search_field :author_username_cont %>

  <%= f.submit "search" %>
<% end %>
...
<%= content_tag :table do %>
  <%= content_tag :th, sort_link(@q, :author_username) %>
<% end %>
```

hoặc sử dụng String cho Sort

```ruby
<%= content_tag :table do %>
  <%= content_tag :th, sort_link(@q, "author.username") %>
<% end %>
```

## Alias

Khi tạo các Matchers với việc kết hợp các thuộc tính, tên matcher có thể rất dài, ta có thể tạo Alias cho nó qua method `ransack_alias` bên trong Model

```ruby
class Post < ActiveRecord::Base
  belongs_to :author

  # Abbreviate :author_first_name_or_author_last_name to :author
  ransack_alias :author, :author_first_name_or_author_last_name
end
```

Khi đó, Matcher có thể được tạo sử dụng trực tiếp `author` (không xung đột với `Post.author`)

# Helpers

Ransack cũng cung cấp hai Helpers để hỗ trợ tạo các Form và Link cấu hình sẵn các tiêu chí vào `params`.

## `search_form_for`

Helper này thay thế cho `form_for`, hỗ trợ người dùng nhập (`search_field`)/chọn (`select_field`) các tiêu chí tìm kiếm.

Đối tượng truyền cho Helper này là `Ransack::Search`.

```ruby
<%= search_form_for @q do |f| %>

  # Search if the name field contains...
  <%= f.label :name_cont %>
  <%= f.search_field :name_cont %>

  # Search if an associated articles.title starts with...
  <%= f.label :articles_title_start %>
  <%= f.search_field :articles_title_start %>

  # Attributes may be chained. Search multiple attributes for one value...
  <%= f.label :name_or_description_or_email_or_articles_title_cont %>
  <%= f.search_field :name_or_description_or_email_or_articles_title_cont %>

  <%= f.submit %>
<% end %>
```

## `sort_link`, `sort_url`

`sort_link` giúp tạo một `<a>` với đường dẫn chứa sẵn các tiêu chí sắp xếp tương ứng với một trường xác định

```ruby
<%= sort_link(@q, :name, 'Last Name', default_order: :desc) %>
```

Anchor này cũng được hỗ trợ thêm `Arrow Indicator` để chỉ hướng sắp xếp.

Các đường dẫn sắp xếp cũng tự động kết hợp với các tiêu chí tìm kiếm và sắp xếp khác. Tức là có thể sắp xếp theo kết quả tìm kiếm và sắp xếp trên nhiều trường.

`sort_url` chỉ sinh URL thay vì sinh cả `<a>`

Đối tượng truyền cho các Helpers này là `Ransack::Search`.

# Configuration

Các cấu hình cho Ransack có thể được đặt trong _config/initializers/ransack.rb_

```ruby
Ransack.configure do |config|

  # Change default search parameter key name.
  # Default key name is :q
  config.search_key = :query

  # Raise errors if a query contains an unknown predicate or attribute.
  # Default is true (do not raise error on unknown conditions).
  config.ignore_unknown_conditions = false

  # Globally display sort links without the order indicator arrow.
  # Default is false (sort order indicators are displayed).
  # This can also be configured individually in each sort link (see the README).
  config.hide_sort_order_indicators = true
end
```

## Search Key

Mặc định, key cho Hash dùng để chứa các tiêu chí tìm kiếm/sắp xếp là `:q` (tức là `search_form_for` hay `sort_link` sẽ mặc định đóng gói các tiêu chí tìm kiếm vào `:q`), điều này có thể làm xung đột giữa các Search Form khác nhau cùng tồn tại trong Action.

Ta có thể thêm option `search_key` giúp xác định key để đóng gói các tiêu chí.

```ruby
# Controller
@search = Log.ransack(params[:log_search], search_key: :log_search)

# search_form_for
<%= f.search_form_for @search, as: :log_search %>

# sort_link
<%= sort_link(@search) %>
```

# Complex Queries

Khi tiêu chí tìm kiếm/sắp xếp trở nên phức tạp, search Form nên chuyển sang dùng POST thay vì GET

```ruby
<%= search_form_for @q, url: search_people_path,
                        html: { method: :post } do |f| %>
```

## Custom Predicate

Ta có thể tạo một Predicate bằng việc định nghĩa bên trong tệp cấu hình với method **`add_predicate`**, đối số cho nó là
- Tên của predicate
- Options:
    - **`arel_predicate`**: Base Predicate hỗ trợ bởi `Arel`
    - **`formatter`**: Biến đổi lại giá trị **`value`** trước khi so sánh.
    - **`validator`**: Kiểm tra giá trị gốc của **`value`**. Nếu không phù hợp sẽ bị loại bỏ khỏi danh sách tiêu chí.
    - **`type`**: Ép kiểu cho **`value`**
    - **`case_insensitive`**: So khớp dựa trên phân biệt hoa thường hay không

```ruby
# config/initializers/ransack.rb
Ransack.configure do |config|
  config.add_predicate 'equals_diddly', # Name your predicate
    # What non-compound ARel predicate will it use? (eq, matches, etc)
    arel_predicate: 'eq',
    # Format incoming values as you see fit. (Default: Don't do formatting)
    formatter: proc { |v| "#{v}-diddly" },
    # Validate a value. An "invalid" value won't be used in a search.
    # Below is default.
    validator: proc { |v| v.present? },
    # Should compounds be created? Will use the compound (any/all) version
    # of the arel_predicate to create a corresponding any/all version of
    # your predicate. (Default: true)
    compounds: true,
    # Force a specific column type for type-casting of supplied values.
    # (Default: use type from DB column)
    type: :string,
    # Use LOWER(column on database).
    # (Default: false)
    case_insensitive: true
end
```

VD2: Định nghĩa một Date Predicate

```ruby
# config/initializers/ransack.rb
Ransack.configure do |config|
  config.add_predicate 'date_equals',
    arel_predicate: 'eq',
    formatter: proc { |v| v.to_date },
    validator: proc { |v| v.present? },
    type: :string
end
```

## Ransacker

Ransacker giúp tạo các tiêu chí phức tạp, nó giúp trả về một **Arel Node**, từ đó có thể kết hợp với các Predicate để tạo một Matcher.

Ransacker được định nghĩa bên trong Model qua class method `ransacker` với đối số là tên thuộc tính (Arel Node), kiểu trả về của thuộc tính và một Block để trả về Arel Node.

```ruby
# Model:
ransacker :created_date, type: :date do
  Arel.sql('date(created_at)')
end
```

Ở đây, Ransacker tạo một `:created_date` attribute cho Model dựa trên việc biến đổi `created_at` về Date.

Ta cũng có thể thêm arguments cho Ransacker.

Xem thêm: [Ransacker](https://activerecord-hackery.github.io/ransack/going-further/ransackers/)

# Authorization

Việc tìm kiếm và sắp xếp thường chỉ nên giới hạn trong một số trường nhất định. Ta có thể cấu hình bên trong Model giới hạn này thông qua 4 class methods (mở rộng cho `ActiveRecord::Base`)

- **`self.ransack_attributes`**: Các **Attributes** và **Ransackers** có thể dùng trong các Search Matchers.
- **`self.ransack_associations`**: Các **Associations** có thể dùng trong các Search Matchers.
- **`self.ransortable_attributes`**: Các **Attributes** và **Ransackers** có thể dùng trong các Sort Matchers.
- **`self.ransackable_scopes`**: Các **Class methods** và **Scopes** có thể dùng trong các Sort Matchers.

Nếu có các Matcher có thuộc tính không nằm trong các giới hạn này, Matcher đó sẽ bị loại bỏ.

Các methods này nên để trong **private**.

## Authorization Object

Các class methods này cũng nhận vào một đối số **`auth_object`** cho phép cấu hình các giới hạn dựa trên một điều kiện. VD: Các giới hạn khác nhau với tài khoản thông thường và tài khoản admin.

```ruby
class Article < ActiveRecord::Base
  def self.ransackable_attributes(auth_object = nil)
    if auth_object == :admin
      # whitelist all attributes for admin
      super
    else
      # whitelist only the title and body attributes for other users
      super & %w(title body)
    end
  end

  private_class_method :ransackable_attributes
end
```

`auth_object` sẽ được truyền khi gọi `ransack` trên Model

```ruby
class ArticlesController < ApplicationController
  def index
    @q = Article.ransack(params[:q], auth_object: set_ransack_auth_object)
    @articles = @q.result
  end

  private

  def set_ransack_auth_object
    current_user.admin? ? :admin : nil
  end
end
```