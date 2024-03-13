---
  title:  Action Mailer
  draft: false
  date: 2024-03-13
  tags:
    - ror
    - mail
  description: How to send mail in Ruby on Rails
---

`ActionMailer` hỗ trợ chức năng gửi email cho ứng dụng. ActionMailer tổ chức chức năng tương tự Controllers.

- Các Actions và Views
- Các biến instance có thể truy cập trên Views
- Có thể truy xuất `params`

Sử dụng lệnh để sinh một Mailer với tên tương ứng (đặt tên tương tự Controller)

```cmd
rails g mailer <name>
```

# ActionMailer Actions

Các Mailers đều cần kế thừa từ `ActionMailer::Base`, mỗi method trong đó tương ứng một Action, và có thể gắn với một View Template để làm nội dung email.

Các Mailer Actions có thể **nhận vào đối số**.

Bên trong Mailer Action đó (và View tương ứng), có thể truy xuất vào các biến sau:

| Variable             | Description                                       |
| -------------------- | ------------------------------------------------- |
| `headers`            | Đọc/Ghi nội dung trên Header                      |
| `attachments`        | Gán Attachments                                   |
| `attachments.inline` | Gán Attachments nhúng trong nội dung email        |
| `mail`               | Xác định các trường của mail: `to`, `subject`,... |
| `params`             | Đối số được truyền khi gọi `.with`                |

Mặc định, nội dung email được gắn vào `mail`, nếu không xác định cụ thể `format` thì toàn bộ các tệp có cùng tên với Action sẽ được dùng làm nội dung _multipart/alternative_

## Send Mail

Sau khi định nghĩa Action và View dùng làm nội dung mail, ta có thể kích hoạt việc gửi mail qua:

```ruby
UserMailer.weekly_summary(user).deliver_now

# In `weekly_summary`, can access `user` directly
```

ActionMailer cũng có thể hoạt động cùng ActiveJob để có thể gửi mail trong Background với `deliver_later`

Cách thứ hai để truyền đối số vào Action là qua `params`

```ruby
UserMailer.with(user: user).weekly_summary.deliver_now

# In `weekly_summary`, can access `user` via `params[:user]`
```

## URL in mail

Khi email cần nhúng các links, ta có thể sử dụng `url_for` hay các Named Route URL. Chú ý nên sử dụng đường dẫn dạng **URL** vì Mail Client không thể phân giải đường dẫn tương đối dạng `path`.

Khi xác định đường dẫn URL, ta cũng cần chỉ định địa chỉ Host của đường dẫn.

Thường các đường dẫn trong mail sẽ là một tài nguyên trên Web Server, nên ta có thể cấu hình:

```ruby
config.action_mailer.default_url_options = { host: "example.com" }
```

Với các tài nguyên khác như hình ảnh, có thể cấu hình cho Server phục vụ các tài nguyên này qua

```ruby
config.action_mailer.asset_host = "https://example.com"
```

## Default Options

Ta có thể sử dụng method `default` để xác định các thông tin trong email có thể dùng chung giữa các Actions trong Class hiện tại.

```ruby
class AdminMailer < ApplicationMailer
  # Send emails to multiple recipients
  default to: -> { Admin.pluck(:email) },
          from: 'notification@example.com'

  def new_registration(user)
    @user = user
    mail(subject: "New User Signup: #{@user.email}")
  end
end

```

Hoặc có thể cấu hình

```ruby
config.action_mailer.default_options = { from: "no-reply@example.com" }
```

## Email Preview

Để xem trước email trước khi gửi thực sự, cần cấu hình sau:

```ruby
config.action_mailer.preview_paths << "#{Rails.root}/views/mailer_previews"
```

Mặc định, đường dẫn của Preview Class sinh cùng `rails generate mailer` nằm ở **test/mailers/previews**

Tạo Class kế thừa từ **`ActionMailer::Preview`**, ở đó định nghĩa các Action Method để trả về một `Mail::Message` là nội dung email.

Message này có thể sử dụng dữ liệu thực sự từ DB

```ruby
class NotifierMailerPreview < ActionMailer::Preview
  def welcome
    NotifierMailer.welcome(User.first)
  end
end
```

Để xem Preview, truy cập vào đường dẫn `http://localhost:3000/rails/mailers/<class>/<method>`

# Configurations

| Field               | Description                                                                       | Options                                                                |
| ------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `delivery_method`   | Cách thức gửi mail                                                                | `smtp`, `sendmail`, `file`, `test`                                     |
| `smtp_settings`     | Cấu hình khi cách thức gửi là SMTP                                                | `address`, `port`, `domain`, `user_name`, `password`, `authentication` |
| `sendmail_settings` | Cấu hình khi cách thức gửi là SendMail (executable)                               | `location` (Vị trí tệp thực thi), `arguments`                          |
| `delivery_job`      | Job Class định nghĩa `deliver_later`, mặc định là `ActionMailer::MailDeliveryJob` |                                                                        |
