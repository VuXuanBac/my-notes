---
title: Traditional way for Authentication
draft: false
tags:
  - ror
  - example
---

Với chức năng Authentication đơn giản, ta có thể triển khai như sau:

# 1. Tạo Model và DB Table cho tài khoản: **`Account`** và thêm Validations

```ruby
# Generate Model
rails generate model Account name:string email:string

# app/models/account.rb
class Account < ApplicationRecord
  before_save { self.email = email.downcase }

  validates :name, presence: true, length: { maximum: 50 }
  validates :email, presence: true, length: { maximum: 255 },
                    format: { with: /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i },
                    uniqueness: true
end
```

# 2. Thêm `has_secure_password` cho Model và thêm trường `password_digest` trong DB

Macro này có trong module `ActiveModel::SecurePassword`, được thêm sẵn vào `ActiveRecord::Base`

Cũng cần thêm gem `bcrypt`.

Macro này thực hiện:
- Khai báo **`password`** và **`password_confirmation`** cho Model.
- Thêm các Validations cho hai thuộc tính này.
- Băm (BCrypt) `password` về **`password_digest`** (cần được thêm vào DB Table)
- Cung cấp method **`authenticate`** cho Model để so sánh một giá trị password đối số có hợp lệ không. Nếu đúng, trả về `Account` instance.

Ngoài ra, ta có thể tùy chỉnh bằng việc xác định đối số cho Macro này:
- Tên của thuộc tính cần băm. Mặc định `password`
- Đối số thứ hai là `false` nếu không muốn tạo sẵn các Validations.

Khi sử dụng tên khác (`XXX`) thuộc tính khai báo trong macro này, ta cần sử dụng method **`authenticate_XXX`** thay vì `authenticate`

# 3. Tạo Session Controller cho chức năng đăng nhập, đăng xuất

Trong `create` Action, sử dụng method `authenticate` để kiểm tra mật khẩu.

Nếu xác thực thành công, lưu ID của `Account` vào `session`. Điều này cũng giúp thêm một Cookie (tạm thời - `Expires = Session`) vào Response của Action này.

Chức năng đăng xuất về cơ bản là xóa `session`

# 4. Chức năng Remember Me

Thêm **`remember_digest`** và DB Table và **`attr_accessor :remember_token`** cho `Account` Model.

Trong `Account` Model, triển khai một số Methods để sinh Remember Token và kiểm tra token truyền vào có khớp với `remember_digest` không

```ruby
class Account < ApplicationRecord
  # Kiểm tra token đối số có khớp với token lưu trong DB không
  def authenticated? remember_token
    BCrypt::Password.new(remember_digest).is_password? remember_token
  end

  class << self
    # Sử dụng `BCrypt` để băm token sinh được
    def digest string
      if ActiveModel::SecurePassword.min_cost
        cost = BCrypt::Engine::MIN_COST
      else
        cost = BCrypt::Engine.cost
      end
      BCrypt::Password.create(string, cost: cost)
    end

    # Sinh một Token ngẫu nhiên.
    def new_token
      SecureRandom.urlsafe_base64
    end
  end
end
```

Trong Model, ta cũng triển khai hai methods để cập nhật `remember_digest` cho hai trường hợp đăng nhập và đăng xuất

```ruby
  def remember
    self.remember_token = self.class.new_token
    update_attribute :remember_digest, self.class.digest(remember_token)
  end

  # Forgets a user.
  def forget
    update_attribute :remember_digest, nil
  end
```

Trong `SessionController`, khi đăng nhập, thực hiện
- Gọi `Account.remember` để tạo mới và lưu Remember Token vào DB
- Lưu Token (và Account ID) vào Cookies: **`cookies.permanent`** (với Account ID, nên để vào **`cookies.permanent.encrypted`**)

Ta cũng cần kiểm tra trường hợp request gửi đến có Remember Token thông qua method **`Account.authenticated?`** định nghĩa ở trên, với đối số là giá trị đọc từ `cookies`.

Chức năng Forget bao gồm:
- Xóa `remember_digest`
- Xóa Cookies

# 5. Authorization

Authorization ở đây chỉ ở mức đơn giản, tức là chỉ cho phép thực hiện tác vụ nếu đã đăng nhập.

Sử dụng `before_action` để kiểm tra

# 6. Activation

Tương tự

# 7. Password Reset

Tương tự