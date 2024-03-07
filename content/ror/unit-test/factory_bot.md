---
title: Factory Bot
draft: false
date: 2024-03-04
tags:
  - ror
  - unit-testing
  - test-data
---

Factory Bot là một thư viện thay thế cho Rails Fixtures, tức là dùng để định nghĩa dữ liệu mẫu dùng cho kiểm thử.

Cấu hình Factory Bot với RSpec Framework
```ruby
# Gem
group :development, :test do
  gem "factory_bot_rails"
end

##### RSpec ######
# spec/support/factory_bot.rb
RSpec.configure do |config|
  # Auto Preface `FactoryBot` into all Factory Bot methods (like: `FactoryBot.create` -> `create`)
  config.include FactoryBot::Syntax::Methods
end

# spec/rails_helper.rb
require "support/factory_bot"
```

Dữ liệu mẫu tương ứng với mỗi Model Class được gọi là **Factory**, Factory Bot sẽ tự động nạp các Factories định nghĩa trong các tệp sau:
- _factories.rb_
- _test/factories.rb_
- _spec/factories.rb_
- _factories/*.rb_
- _test/factories/*.rb_
- _spec/factories/*.rb_

Để sinh một Factory, ta sử dụng 

# Syntax

Mỗi Model Object được định nghĩa trong `factory` method, với
- Tên của Object, tên này cũng dùng để ngầm định Model Class sử dụng, hoặc có thể chỉ định Option `class`
    - Thường thì mỗi Model chỉ cần một Factory
    - Có thể sử dụng Option `aliases` để khai báo các tên gọi khác cho Factory
    - Dựa vào tên của object này ta sẽ dùng để tạo instance với [Build Strategy](#build-strategy)
- Theo sau là một Block dùng để định nghĩa các object attributes, và giá trị tương ứng cho chúng.

VD:

```ruby
factory :user, class: "User" do
  first_name { "John" }
  last_name { "Doe" }
  date_of_birth { 18.years.ago }
  email { "#{first_name}.#{last_name}@example.com".downcase } # Dependent Attribute
end
```

**Best Practices:**
- Mỗi Model Class nên định nghĩa một **Basic Factory**.
- **Basic Factory** chỉ bao gồm các attributes cần thiết (có `presence` Validation và không có `default`)
- Sau đó sử dụng [Inheritance](#inheritance) để định nghĩa các Factories cụ thể hơn.
- Các attributes này có thể override sau này.

# Inheritance

Ta có thể sử dụng Inheritance tạo nhiều Factories cho một Model Class, và chúng chỉ khác nhau ở một số attributes.

Inheritance tức là định nghĩa một Factory bên trong một Factory

VD: 

```ruby
factory :post do
  title { "A title" }

  factory :approved_post do
    approved { true }
    # title { "A title" }
  end
end
```

hoặc có thể định nghĩa độc lập và gán Option `parent`:

```ruby
factory :post do
  title { "A title" }
end

factory :approved_post, parent: :post do
  approved { true }
end
```

# Association

Khi các Models có Association với nhau, bên trong Factory có thể khai báo Association như sau:

```ruby
# Association: Each Post has one Author
factory :author do
end
# IMPLICIT from Factory Name
factory :post do
  # ...
  author 
end

# EXPLICIT
factory :post do
  # ...
  association :author
end
```

Ta có thể override attribute cho Association

```ruby
factory :post do
  # ...
  association :author, last_name: "Writely"
end
```

## `has_many`

Với `has_many` Association, ta có thể sinh dữ liệu cho attribute tương ứng cho Model instance với một trong các cách sau

1. Sử dụng Build Strategy và định nghĩa một Ruby function để populate data cho `has_many` attribute
2. Định nghĩa ngay trong Factory, và sử dụng Callback [`after(:create)`](#callback)

VD:

```ruby
FactoryBot.define do
  factory :post do
    title { "Through the Looking Glass" }
    user
  end

  factory :user do
    name { "John Doe" }

    # user_with_posts will create post data after the user has been created
    factory :user_with_posts do
      # posts_count is declared as a transient attribute available in the
      # callback via the evaluator
      transient do
        posts_count { 5 }
      end

      # the after(:create) yields two values; the user instance itself and the
      # evaluator, which stores all values from the factory, including transient
      # attributes; `create_list`'s second argument is the number of records
      # to create and we make sure the user is associated properly to the post
      after(:create) do |user, evaluator|
        create_list(:post, evaluator.posts_count, user: user)

        # You may need to reload the record here, depending on your application
        user.reload
      end
    end
  end
end

create(:user).posts.length # 0
create(:user_with_posts).posts.length # 5
create(:user_with_posts, posts_count: 15).posts.length # 15
```

# Build Strategy

Factory Bot hỗ trợ một số chiến lược tạo object
- `build`: Tạo một Model instance nhưng **chưa lưu vào DB**.
- `create`: Tạo một Model instance và **lưu vào DB**.
- `attributes_for`: Tạo một Hash cho các attributes của object.
- `build_stubbed`: Trả về Model instance tương tự `create` (lưu DB - cập nhật ID, updated_at...) nhưng thực chất chỉ là `build`

Association tự động sử dụng Build Strategy của parent object của nó

Ta có thể sinh nhiều Model instance cho cùng một Factory, sử dụng `build_list` hoặc `create_list`.
- Đối số là tên Factory và số lượng.
- Các method này trả về Array.
- Chúng cũng có thể nhận vào các **override attributes** - Option Hash (áp dụng cho toàn bộ)
- Hoặc khai báo một Block với hai đối số là `(object, index)`

# Other Objects

## Sequence

Sequence là một đối tượng để sinh nhiều dữ liệu có cùng format.
- Tạo một Sequence với `sequence` method
- Build một Sequence với `generate` method. Mỗi lần gọi sẽ tạo một Unique object

Một Sequence có thể định nghĩa Global (bên trong `FactoryBot.define`) hoặc Local bên trong một Factory

VD:

```ruby
# Defines a new GLOBAL sequence
FactoryBot.define do
  sequence :email[, initial_value] do |n|
    "person#{n}@example.com"
  end
end

generate :email
# => "person1@example.com"

generate :email
# => "person2@example.com"
```

Dữ liệu cung cấp cho Sequence có thể là một Enumerable, chỉ cần nó cung cấp `next`

Với mỗi lần gọi Sequence object sẽ dùng một giá trị tăng tự động, mặc định là 1. Ta có thể xác định một giá trị khác (VD: 'a'), miễn là giá trị này có thể gọi `next`

VD:

```ruby
# Defines a new LOCAL sequence
factory :task do
  sequence :priority, %i[low medium high urgent].cycle
end
```

## Trait

Trait cho phép định nghĩa một Attribute đóng gói nhiều Attributes khác cho Factory. 

Khi tạo Model instance (qua tất cả các [Build Strategy](#build-strategy), hoặc qua [Association](#association)), ta chỉ cần xác định tên của Trait thì tự động sẽ được gán giá trị cho mọi Attributes định nghĩa bên trong Trait đó.

```ruby
factory :user do
  name { "Friendly User" }

  trait :active do
    name { "John Doe" }
    status { :active }
  end

  trait :admin do
    admin { true }
  end
end

# creates an admin user with :active status and name "Jon Snow"
create(:user, :admin, :active, name: "Jon Snow")
```

Với Enum định nghĩa bên trong một Model, Factory Bot sẽ tự động sinh một Trait cho mỗi giá trị của Enum đó

```ruby
# app/models/task.rb
class Task < ActiveRecord::Base
  enum status: {queued: 0, started: 1, finished: 2}
end

# rspec/factories/task.rb
FactoryBot.define do
  factory :task
end

FactoryBot.build(:task, :queued)
FactoryBot.build(:task, :started)
FactoryBot.build(:task, :finished)
```

# Callback

Bên trong các Factory có thể gọi một trong 4 callbacks mặc định sau đây:
- `after(:build)`: Dùng cho cả `build` và `create` Strategy
- `after(:create)`, `before(:create)`: Dùng cho `create` Strategy
- `after(:stub)`: Dùng cho cả `build_stubbed` Strategy

Các Callbacks này cũng có thể gọi Global, áp dụng lên toàn bộ Factory