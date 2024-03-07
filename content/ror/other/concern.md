---
title: Concern
draft: false
date: 2024-03-04
tags:
  - ror
  - extension
---

References:
- [How it works](https://www.writesoftwarewell.com/how-rails-concerns-work-and-how-to-use-them/)

Concern là một **Module** dùng để chia nhỏ Class hay Module.

Rails Concern là một Ruby Module kế thừa từ _ActiveSupport::Concern_

Một Class khi _include_ một Module thì Class đó sẽ nhận toàn bộ các methods được định nghĩa bên trong Module, như thể đó là các methods mà Class đó định nghĩa.

Với Concern, Rails mở rộng thêm hai tính năng
- Include cả Class Methods
- Định nghĩa cả Validations, Callbacks, Constants,...

Concern cung cấp hai Blocks sau:
- **_included_**: Nhúng toàn bộ code trong phần này vào Class: Validations, Associations, Scopes,...
- **_class\_methods_**: Định nghĩa các Class Methods

Ta vẫn có thể định nghĩa các instance methods ở bên ngoài hai Blocks trên.

```ruby
module Taggable
  extend ActiveSupport::Concern

  included do
    # any code that you want inside the class
    # that includes this concern
  end

  class_methods do
    # methods that you want to create as
    # class methods on the including class
  end
end
```

Để dùng Concern, ta _include_ nó như Module thông thường.