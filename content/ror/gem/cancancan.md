---
title: CanCanCan
draft: false
tags:
  - ror
  - gem
---

CanCanCan là một thư viện hỗ trợ **phân quyền** cho Ruby. Các quyền có thể được định nghĩa trong các tệp gọi là **Ability**, có thể gộp chung các quyền trong một tệp Ability duy nhất.

Thư viện này gồm 2 phần:
- _Authorization Library_: Hỗ trợ định nghĩa các **Rules** để truy cập các tài nguyên khác nhau và các **Helpers** để kiểm tra các quyền được định nghĩa.
- _Rails Helpers_: Hỗ trợ Rails Controllers trong việc kiểm tra quyền truy cập Models

Cấu hình:

```ruby
## Gemfile
gem "cancancan"

## CMD

# Generate Ability File : /app/models/ability.rb
rails generate cancan:ability
```

File Ability có nội dung ban đầu như sau:

```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
  end
end
```
- Trong đó `user` là người dùng cần cấp quyền. 
- Khi [kiểm tra quyền](#check-abilities), mặc định **CanCanCan sẽ sử dụng `current_user`** để truyền cho `user` trong `initializer`
- Với nội dung trống này, không có quyền nào được cho phép.

# Define Abilities

Các quyền trong Ability Files được định nghĩa qua macro `can`:

```ruby
can <actions>, <subjects>, <conditions>
```

Nó sẽ định nghĩa **"User hiện tại có quyền thực hiện `<action>` trên tài nguyên `<subject>` chỉ khi điều kiện `<condition>` trả về true"**

Các quyền trong Ability File nên được định nghĩa theo mức tăng quyền hạn, phụ thuộc vào vai trò của `user`. VD: Từ `user` chưa đăng nhập (`user = nil`) đến `user` đã đăng nhập và đến Admin `user` (`user.is_admin = true`)

## Can Actions

Các Actions trong CanCanCan định nghĩa các thao tác thực hiện trên tài nguyên, nó cũng tương ứng với các Controller Actions và Model Callbacks.

Mặc định, thư viện này chỉ cung cấp 4 ngữ nghĩa cho Actions: `:read`, `:create`, `:update` và `:destroy`.

Ngoài ra, ta có thể tạo một **Can Action bất kỳ**, với một Symbol. VD: Admin có thể `lock` một tài khoản. `can :lock, Account if user.is_admin`

Với riêng Rails, CanCanCan sẽ tạo các aliases để chúng tương ứng với các Rails Controller Actions (7 kiểu):

```ruby
read: [:index, :show]
create: [:new, :create]
update: [:edit, :update]
destroy: [:destroy]
```

Từ đây, khi kiểm tra quyền `can? :edit, @resource`, CanCanCan tự động ánh xạ về `can? :update, @resource`. Song, khi định nghĩa Abilities, thường chỉ dùng 4 Actions có sẵn.

Ngoài ra, một quyền đặc biệt là `manage` có thể dùng để định nghĩa quyền thực hiện bất kỳ thao tác nào (có toàn quyền)

## Can Subjects

Subjects cho Ability **thường** là một Ruby Class, ngoài ra ta có thể sử dụng Symbol để biểu thị một Object bất kỳ.

VD: `can :read, :dashboard if user.is_admin` và trong Views, ta có thể kiểm tra `can? :read, :dashboard` trước khi trả về một đường dẫn đến Dashboard Page.

Symbol `:all` để chỉ toàn bộ tài nguyên.

## Conditions

### Hash Options

Conditions cho các Rules đơn giản nhất là sử dụng một Hash Options
- key là attributes của `subject` object: DB Columns, Associations
  - Ta có thể sử dụng các attributes của associations
- value là giá trị dùng để so sánh bằng hoặc sử dụng Array/Range

### Block

Ta có thể truyền Block cho `can` để xác định điều kiện cho Rule.

```ruby
can :update, Project do |project|
  project.priority < 3
end
```

Tuy nhiên, **Block này chỉ được gọi trong [Check Abilities](#can-cannot) `can?` nếu như `subject` truyền cho nó là instance**, nếu như là Class thì mặc định trả về `true` (không gọi Block)

Do Block chỉ được dùng với Conditions, nên với [Fetching Records](#fetching-records), ta cần cung cấp thêm mệnh đề cho điều kiện WHERE cùng với Block. Nếu không CanCanCan sẽ sinh lỗi.

```ruby
# Truyền String để xác định điều kiện WHERE cho Fetching Records
can :update, Project, ["priority < ?", 3] do |project|
  project.priority < 3
end

# Điều kiện WHERE có thể đơn giản nếu dùng ActiveRecord Scope
can :read, Article, Article.published do |article|
  article.published_at <= Time.now
end
```

## Combine Abilities

Khi định nghĩa Rules, thông thường ta sử dụng `can` và sử dụng `cannot` chỉ nếu muốn giới hạn quyền đã định nghĩa trước.

Trong trường hợp các Rules cho cùng một `subject`, các quy tắc sau sẽ được áp dụng:

**1.** Khi định nghĩa các `can` Rules, chúng sẽ được nối với nhau theo OR.

```ruby
# `can? :update, @project` trả về true nếu
#     - `current_user` là Project Owner
# HOẶC
#     - `Project.locked = true`
can :manage, Project, owner: user
can :update, Project, locked: false
```

**2.** Khi có thêm `cannot` Rules, **Rule định nghĩa sau sẽ override Rule định nghĩa trước**, tức là thứ tự các Rules là quan trọng.

```ruby
# `current_user` có toàn quyền với Project NGOẠI TRỪ `:destroy`
can :manage, Project
cannot :destroy, Project
```

```ruby
if user.moderator?
  can :manage, Project
  cannot :destroy, Project
  can :manage, Comment
end

if user.admin?
  can :destroy, Project
end

# 
```

# Check Abilities

Sau khi định nghĩa các Abilities, bên trong Controllers/Views/Models ta có thể kiểm tra quyền được cấp phát cho User

Mặc định, CanCanCan sử dụng method `current_user` để lấy `user` truyền cho `Ability.initialize`

## Controller Helpers

### `can?`, `cannot?`

Hai methods này để kiểm tra `user` có thể thực hiện `action` trên `subject` không.

```ruby
can? <action>, <subject>

cannot? <action>, <subject>
```

Thông thường `can?` và `cannot?` được dùng trong Controllers và Views, song có thể dùng ở bất cứ đâu.

### `authorize!` and Other Helpers

Bên trong Controller, ta có thể sử dụng `authorize!` để đơn giản hơn việc sử dụng `can?`. Methods này sẽ raise `CanCan::AccessDenied` nếu không có quyền.

Ta có thể thêm cấu hình mặc định xử lý cho Exeption trên trong _config/application.rb_

```ruby
config.action_dispatch.rescue_responses.merge!('CanCan::AccessDenied' => :unauthorized)
```

CanCanCan cũng cung cấp các Helper Methods cho Controller:
- `authorize_resource`: Tự động gọi `authorize! action, @resource` cho mọi Controller Actions
- `load_resource`: Tự động nạp `@resource` từ `Model`
- `load_and_authorize_resource`: Kết hợp hai methods trên
- `@resource` và `Model` ở đây được suy ngầm định từ tên Controller

VD:

```ruby
class ArticlesController < ApplicationController
  load_and_authorize_resource

  def index
    # @articles are already loaded...see details in later chapter
  end

  def show
    # the @article to show is already loaded and authorized
  end

  def create
    # the @article to create is already loaded, authorized, and params set from article_params
    @article.create
  end

  def edit
    # the @article to edit is already loaded and authorized
  end

  def update
    # the @article to update is already loaded and authorized
    @article.update(article_params)
  end

  def destroy
    # the @article to destroy is already loaded and authorized
    @article.destroy
  end

  protected

  def article_params
    params.require(:article).permit(:body)
  end
end
```

## Check for Others

Ta có thể kiểm tra quyền cho một User khác (không dùng ngầm định `current_user`), bằng việc tạo một Ability instance:

```ruby
Ability.new(some_user).can? <action>, <subject>
```

hoặc đơn giản hơn là sử dụng `delegate` để gắn `can?` từ Ability cho User Model.

```ruby
# app/models/user.rb
class User
  delegate :can?, :cannot?, to: :ability

  def ability
    @ability ||= Ability.new(self)
  end
end

some_user.can? :update, @article
```

# Fetching Records

CanCanCan cũng hỗ trợ lấy toàn bộ các objects mà `user` có quyền truy cập qua method:

`Model.accessible_by(current_ability, <action>=:index)`

- `current_ability` tự động được thêm vào Controller

CanCanCan sẽ tự động sinh câu truy vấn dựa trên các điều kiện `<conditions>` định nghĩa trong Ability File.

# Configuration

References:
- [R1](https://github.com/CanCanCommunity/cancancan/blob/develop/docs/changing_defaults.md)

## `current_user`, `current_ability`

Mặc định, CanCanCan mong muốn
- Định nghĩa một `Ability` Class
- Có method `current_user` cho Controllers sử dụng các _Controller Helpers_

Nếu không có `current_user`, ta có thể định nghĩa một method trả về tài khoản người dùng hiện tại và tạo `alias_method`

```ruby
class ApplicationController < ActionController::Base
  alias_method :current_user, :name_of_your_method # Could be :current_member or :logged_in_user
end
```

Sau đó, CanCanCan sẽ cung cấp cho các Controllers method `current_ability` để trả về Ability object cho `current_user`

```ruby
def current_ability
  @current_ability ||= Ability.new(current_user)
end
```