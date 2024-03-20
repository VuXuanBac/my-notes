---
title: Devise
draft: false
date: 2024-03-04
tags:
  - ror
  - gem
---

References:

- [Introduction](https://dev.to/kevinluo201/introduction-to-devise-modules-and-enable-all-of-them-4p25)
- [Behind the scenes](https://www.ombulabs.com/blog/learning/devise/behind-the-scenes-devise.html)

Devise là một Rack-based Engine dựa trên Warden (Middleware) cho giải pháp **Authentication** hỗ trợ ứng dụng Rails, cung cấp **đầy đủ một giải pháp MVC** (model, view, controller) và **thiết kế theo hướng module** (chỉ tích hợp những chức năng mà ứng dụng cần)

Devise tổ chức các giải pháp Authentication thành 10 Modules:

- **Database Authenticatable**: Băm và lưu mật khẩu vào DB, Xác thực mật khẩu khi đăng nhập.
- **Confirmable**: Gửi email xác nhận tài khoản sau khi đăng ký
- **Recoverable**: Cấp lại mật khẩu
- **Registerable**: Đăng ký tài khoản, Cập nhật và Xóa tài khoản
- **Rememberable**: Ghi nhớ đăng nhập dựa trên Cookie
- **Trackable**: Theo dấu đăng nhập người dùng: Số lượt, Thời điểm và IP
- **Validatable**: Cung cấp sẵn Validation cho Email và Password
- **Lockable**: Khóa tài khoản khi đăng nhập thất bại, Mở khóa qua Email/thời gian chờ.
- **Omniauthable**: Hỗ trợ [OmniAuth](https://github.com/omniauth/omniauth) - Đăng nhập qua các APIs khác như Facebook
- **Timeoutable**: Xóa session sau một khoảng thời gian không tương tác hoặc sau một khoảng thời gian.

# Configuration

```ruby
# Gemfile
gem "devise"

# CMD
rails generate devise:install
## Some instructions are shown

# config default mailer address
# config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Devise được thiết kế như một Engine tích hợp đầy đủ MVC nên gần như ta chỉ cần khai báo là có thể sử dụng các chức năng của Devise.

## Model

Sau đó, ta cần chỉ định trong (nhiều, nếu có) Models sử dụng cho chức năng Authentication (VD: `Account`, `User`, `Admin`,...) các Module mà Model đó cần tích hợp từ Devise.

```ruby
# app/models/user.rb
class User < ApplicationRecord
  devise :database_authenticatable
end
```

Hoặc, nếu ứng dụng chưa có các Models nào cho chức năng Authentication, ta có thể sinh dựa trên:

```cmd
rails generate devise MODEL
```

Mỗi Model được tích hợp Devise sẽ đóng vai trò như một **Scope**.

## View

Trong trường hợp cần tùy chỉnh các Views (không dùng Views mặc định của Devise), ta sử dụng lệnh:

```cmd
rails generate devise:views
```

Devise mặc định chỉ dùng một View cho nhiều **Scopes** (Models). Các tệp Views này sẽ nằm trong _app/views/devise/_.

Ta có thể chỉ định Devise sinh Views riêng cho các Scopes (Models) khác nhau bằng cấu hình:

```ruby
# config/initializers/devise.rb
config.scoped_views = true
```

Khi đó, mỗi Scope sẽ có một cấu trúc Views riêng. VD: _app/views/managers/_, _app/views/members/_

Cú pháp sinh Scoped View:

```cmd
rails generate devise:views MODELs -v [modules]
```

Options `-v` theo sau là danh sách các Modules mà Scope này cần sinh Views.

## Controller

Tương tự, nếu muốn tùy chỉnh Controller, ta có thể sinh các Controllers vào cấu trúc thư mục project

```cmd
rails generate devise:controllers [scope] -c [modules]
```

## Helpers

Khi tích hợp Devise cho Model, Devise cũng tự động sinh các Helpers sau:

- **`authenticate_SCOPE!`**: Kiểm tra tài khoản và mật khẩu và trả về tài khoản đã đăng nhập
- **`current_SCOPE`**: Tài khoản đăng nhập hiện tại.
- **`SCOPE_signed_in?`**: Kiểm tra tài khoản đã đăng nhập chưa.
- **`SCOPE_session`**: Trả về `warden.session`

## Route

Devise là một Engine, nên ta cần `mount` nó vào một đường dẫn xác định. Tuy nhiên, Devise khai báo riêng `devise_for` thay cho `mount`.

Method này sẽ tự động sinh đầy đủ các routes cho một Model Scope (dựa trên các Module được tích hợp vào Model)

- Sử dụng Option **`skip`** hoặc **`only`** (theo sau là mảng các tên Devise Module) để giới hạn các Routes sinh ra.

Khi sử dụng `devise_for`, theo sau là một **Scope Name** (plural form), Devise sẽ sử dụng:

- **Controller Action**: Tìm kiếm trong **thư mục** tương ứng với tên Scopes.
  - Sử dụng Option **`controllers`** để xác định cụ thể đường dẫn tới Controller.
  - Sử dụng Option **`module`** để chỉ định thư mục tìm kiếm Controllers. Mặc định là `devise`.
- **Model**: Suy ra từ tên Scope.
  - Sử dụng Option **`class_name`** để chỉ định một Model Class cụ thể.
- **URI Prefix**: Suy ra từ tên Scope
  - Sử dụng Option **`path`** để cấu hình URI Prefix cụ thể.
- **Helper Methods**: Sinh các Helper Methods như `current_XXX`, `authenticate_XXX!`,...
  - Sử dụng Option **`singular`** để cấu hình lại XXX (mặc định là tên Scope (singular))

Ta có thể truyền vào Block cho method này để cấu hình thêm các Routes.

Chú ý, **nếu `devise_for` được khai báo trong một Namespace thì Scope Name cũng được Prefix bởi tên Namespace đó.**

# Traditional way for Authentication

Xem thêm: [has_secure_password](ror/other/authentication.md)

# Warden

Warden là một Middleware dùng để lấy thông tin xác thực từ Request và truy xuất thông tin tài khoản đăng nhập.

Vì Warden là một Middleware nên **quá trình xác thực diễn ra trước khi xử lý bởi Routing và Controller**.

# Database Authenticatable

Module này là nền tảng của Devise và hầu hết mọi trường hợp đều được thêm vào.

DatabaseAuthenticatable cũng giống một Strategy dùng để xác thực của Warden.

## Required Fields

```ruby
# Lưu mã băm BCrypt của password (tương tự `password_digest`)
t.string   :encrypted_password
```

## Generated Methods

- **`.password=`**: Thuộc tính, đồng thời băm BCrypt về `encrypted_password`
- **`.valid_password?`**: Kiểm tra một password có hợp lệ không (tương tự `authenticate`)
- **`.authenticatable_salt?`**: Sử dụng 29 ký tự đầu của `encrypted_password`.
- **`.send_email_changed_notification`**: Gửi thông báo (Flash) khi cập nhật Email.
- **`.send_password_change_notification`**: Gửi thông báo (Flash) khi cập nhật Password.
- **`.update_with_password(params)`**: Cập nhật các trường trong MODEL với `params` nếu như **`params[:current_password]`** là hợp lệ.
- **`.update_without_password(params)`**: Cập nhật các trường trong MODEL không yêu cầu nhập mật khẩu. Ta cần override lại method này để đảm bảo một số trường (VD: `:email`) sẽ không được cập nhật nếu sử dụng method này.

```ruby
def update_without_password(params, *options)
  params.delete(:email)
  super(params)
end
```

## Routes

| Named Route           | Method | URL                        | Controller Action       |
| --------------------- | ------ | -------------------------- | ----------------------- |
| new_MODEL_session     | GET    | /MODELs/sign_in(.:format)  | devise/sessions#new     |
| MODEL_session         | POST   | /MODELs/sign_in(.:format)  | devise/sessions#create  |
| destroy_MODEL_session | DELETE | /MODELs/sign_out(.:format) | devise/sessions#destroy |

Như vậy, việc tích hợp Module này là đủ cho chức năng đăng nhập.

Xem thêm: [DatabaseAuthenticatable](https://www.rubydoc.info/github/heartcombo/devise/main/Devise/Models/DatabaseAuthenticatable)

# Registerable

Module này giúp người dùng Tạo, Cập nhật và Xóa tài khoản.

Thực tế Module này chỉ sinh ra các Routes và Views cần thiết.

# Confirmable

Module này giúp người dùng Xác nhận đăng ký tài khoản qua Email dùng để đăng ký.

Confirmation Email sẽ tự động được gửi khi tài khoản đăng ký thành công, và có thể gửi lại.

## Required Fields

```ruby
# Token được nhúng vào URL trong Confirmation Email
t.string   :confirmation_token
# Thời điểm Server nhận được xác nhận (khi người dùng nhấn vào URL)
t.datetime :confirmed_at
# Thời điểm gửi Confirmation Email. Được dùng khi muốn kiểm tra khoảng thời gian xác nhận.
t.datetime :confirmation_sent_at
# Dùng khi muốn chức năng cập nhật lại Email. Trường này lưu địa chỉ Email mới. Và sẽ dùng để cập nhật lại `email` khi confirmed
t.string   :unconfirmed_email # Require only if config.reconfirmable = true
```

## Generate Methods

- **`.confirmed?`**: Người dùng đã xác nhận chưa.
- **`.send_confirmation_instructions`**: Gửi Confirmation Email.
- **`.confirm`**: Xác nhận tài khoản thủ công

## Configuration

```ruby
# Giới hạn khoảng thời gian mà URL trong Confirmation Email là còn hạn. Mặc định `nil`
config.confirm_within = 3.days

# Cho phép cập nhật `email` và gửi lại Confirmation Email để xác nhận tài khoản mới.
# Yêu cầu có trường `unconfirmed_email`
config.reconfirmable = true
```

Xem thêm: [Confirmable](https://www.rubydoc.info/github/heartcombo/devise/main/Devise/Models/Confirmable)

# Validatable

Module này thêm các Validations cho hai trường `email` và `password` của Model Scope.

## Configuration

```ruby
# Đổi lại mẫu Regex để Validate Email
config.email_regexp = /\A[^@\s]+@[^@\s]+\z/
# Đổi lại số lượng ký tự yêu cầu cho Password. Sử dụng `min..` để không giới hạn tối đa.
config.password_length = 6..128
```

# Recoverable

Module này thêm chức năng Cấp lại mật khẩu. Tương tự như Confirmable: Gửi Email hướng dẫn, sau khi người dùng nhấn vào đường dẫn trong Email sẽ chuyển đến trang cho phép nhập mật khẩu mới.

Controller: **Passwords**

## Required Fields

```ruby
# Token được nhúng vào URL trong ResetPassword Email
t.string   :reset_password_token
# Thời điểm gửi ResetPassword Email. Được dùng khi muốn kiểm tra khoảng thời gian xác nhận.
t.datetime :reset_password_sent_at
```

## Generate Methods

- **`.clear_reset_password_token`**: Xóa các trường trong DB
- **`.send_reset_password_instructions`**: Gửi ResetPassword Email.
- **`.reset_password(new, new_confirm)`**: Đổi mật khẩu thủ công

## Configuration

```ruby
# Giới hạn khoảng thời gian mà URL trong ResetPassword Email là còn hạn. Mặc định `nil`
config.reset_password_within = 12.hours
# Nếu là `false`, không tự động đăng nhập sau khi mật khẩu được đổi thành công.
config.sign_in_after_reset_password = true
```

# Rememberable

Module này quản lý Remember Token cho chức năng Ghi nhớ đăng nhập

## Required Fields

```ruby
# Thời điểm đăng nhập với Remember Token. Dùng để kiểm tra giới hạn thời gian hợp lệ cho Token.
t.datetime :remember_created_at
```

## Generate Methods

- **`.remember_me`**: Thuộc tính
- **`.extend_remember_period`**: Gia hạn thời gian hiệu lực của Remember Token.
- **`.extend_remember_period`**: Gia hạn thời gian hiệu lực của Remember Token.
- **`.forget_me!`**: Xóa Remember Token
- **`.remember_me!`**: Sinh Remember Token và cập nhật `remember_created_at`

## Configuration

```ruby
# Giới hạn khoảng thời gian mà Remember Token còn hợp lệ
config.remember_for = 12.hours
# Nếu là `true`, tự động xóa toàn bộ Remember Token khi đăng xuất
config.expire_all_remember_me_on_sign_out = true
```

## Works

Sau khi đăng nhập thành công với `{"remember_me": "1"}` params, Server sẽ gửi về một Cookie `remember_MODEL_token` ở dạng **Signed Message** (**`cookies.signed["remember_MODEL_token"]`**)

Giá trị Token này bao gồm 2 phần: Message và Signature. Phần Message chứa các thông tin:

- MODEL_id,
- `rememberable_value`: Sử dụng 29 bytes đầu của **`encrypted_password`** (gọi method `authenticatable_salt` của **DatabaseAuthenticatable**)
- Thời điểm tạo Cookie

Chú ý: **Trong các phiên bản cũ của Devise triển khai theo hướng lưu `remember_token` trong DB. Nhưng về sau sẽ không có trường này mà sử dụng trực tiếp `encrypted_password`**

Giá trị Token sẽ được kiểm tra qua `MODEL_instance.remember_me?(token, generated_at)`

# Timeoutable

Module này giúp thu hồi phiên đăng nhập của người dùng nếu không tương tác (gửi Request) trong một khoảng thời gian.

## Generate Methods

- **`.timedout?`**: Kiểm tra thời điểm hiện tại đã quá hạn chưa.

## Configuration

```ruby
# Giới hạn khoảng thời gian mà phiên hợp lệ nếu không tương tác
config.timeout_in = 12.hours
```

## Work

Sau khi tích hợp module này, `session` sẽ lưu thêm một trường:

```ruby
"warden.user.SCOPE.session": {
  "last_request_at": <timestamp>
}
```

dùng để theo dấu thời điểm Server nhận được Request cuối cùng.

Tại mỗi Request gửi đến, Devise sẽ so sánh trường này (thời điểm Request trước đó) với `timeout_in.ago` để biết phiên đã hết hạn chưa.

Chú ý: _**Timeoutable** và **Rememberable** là xung đột với nhau, vì nếu đã Timeout, nhưng Remember Token vẫn hợp lệ thì Request vẫn được chấp nhận._

# Lockable

Module này triển khai chức năng Khóa tài khoản khi đăng nhập thất bại nhiều lần. Chức năng Mở lại tài khoản có thể triển khai theo 2 hướng: **Gửi Email** hoặc **Chờ một khoảng thời gian**.

## Required Fields

```ruby
# Số lượng lần đăng nhập thất bại
t.integer  :failed_attempts, default: 0, null: false
# Token mở khóa tài khoản (gửi kèm email). Dùng để kiểm tra :email unlock strategy
t.string   :unlock_token
# Thời điểm khóa tài khoản. Dùng để kiểm tra :time unlock strategy
t.datetime :locked_at
```

## Generate Methods

- **`.access_locked?`**: Kiểm tra tài khoản có khóa không
- **`.lock_access!`**: Khóa tài khoản thủ công (cập nhật `locked_at`). Có thể truyền vào một Hash `{send_instructions: true}` nếu muốn gửi Email hướng dẫn mở khóa.

## Configuration

```ruby
# Số lượng lần thử cho phép
config.maximum_attempts = 5
# Khóa tài khoản nếu như đăng nhập thất bại (hoặc :none nếu không muốn khóa)
config.lock_strategy = :failed_attempts
# Cách thức mở khóa tài khoản (:time, :email, :both, :none)
config.unlock_strategy = :both

# Nếu cách thức mở khóa là `:time`: Khoảng thời gian chờ
config.unlock_in = 12.hours
```

# Trackable

Module giúp theo dấu đăng nhập của người dùng.

## Required Fields

```ruby
# Số lần đã đăng nhập
t.integer  :sign_in_count, default: 0, null: false
# Thời điểm đăng nhập phiên hiện tại
t.datetime :current_sign_in_at
# Thời điểm đăng nhập lần cuối
t.datetime :last_sign_in_at
# Địa chỉ IP lần đăng nhập hiện tại
t.string   :current_sign_in_ip
# Địa chỉ IP lần đăng nhập cuối
t.string   :last_sign_in_ip
```

# Omniauthable

Module này giúp đăng nhập vào ứng dụng dựa trên một tài khoản bên ngoài, VD: Facebook, Google,... - **Providers** sử dụng OAuth 2.0.

Vì tương tác với hệ thống bên ngoài, nên luồng triển khai chức năng này khác với các chức năng trên.

Trước hết, ta cần tải các Gem hỗ trợ cho Omniauth với Provider tương ứng.

```ruby
# Gemfile
gem 'omniauth-facebook', '~> 9.0'
gem 'omniauth-rails_csrf_protection', '~> 1.0.1'
```

## Required Fields

```ruby
# Provider Name cho lần đăng nhập hiện tại
t.string  :provider
# UID của tài khoản trong Provider
t.string   :uid
```

## Configuration

Ta cần tạo một ứng dụng trên Provider tương ứng, VD: [Facebook App](https://developers.facebook.com/docs/development/create-an-app/)

```ruby
# Chú ý sử dụng Figaro hoặc Credentials cho hai trường này
config.omniauth PROVIDER, "APP_ID", "APP_SECRET"
```

Trên Model, ta cần chỉ định thêm Option **`omniauth_providers: [:facebook,...]`** phía sau `devise`

Trên Route, ta cũng cần xác định **`omniauth_providers: [:facebook,...]`** cho `devise_for`

Khi đó, sẽ có một Route dẫn đến trang đăng nhập của Facebook để lấy quyền truy cập. VD: `MODEL_facebook_omniauth_authorize_path`

Khi đăng nhập thành công, Facebook sẽ phản hồi tới một đường dẫn (xác định trong đường dẫn gửi tới), và đường dẫn callback này chính là một Route xác định sẵn `MODEL_facebook_omniauth_callback_path`

# Authenticatable

Đây là một Module cơ sở cho chức năng xác thực trong Devise, khác với `Database Authenticatable` là xác thực dựa trên mật khẩu.

Ở Module này, ta quan tâm một số methods sau:

## `active_for_authentication?`

Sau khi tài khoản đăng ký, method này sẽ được gọi để kiểm tra tính active của tài khoản đó. Một số Modules khác ghi đè method này:

- **Confirmable**: `super && (!confirmation_required? || confirmed? || confirmation_period_valid?)`
- **Lockable**: `super && !access_locked?`

Ta có thể ghi đè nó (ví dụ có một trường `is_active` và quản trị viên có thể inactive một tài khoản), nhưng cần gọi lại `super`.

Khi ghi đè method này, cũng nên ghi đè method `inactive_message` để trả về I18n key xác định lý do về trạng thái inactive này.

```ruby
class User < ApplicationModel
  devise :database_authenticatable
  def active_for_authentication?
    super & is_active?
  end

  def inactive_message
    is_active? ? super : :inactivate_by_admin
  end
end
```
