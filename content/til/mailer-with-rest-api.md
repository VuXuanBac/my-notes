---
  title:  Send email with REST API (instead of SMTP)
  draft: false
  date: 2024-03-14
  tags:
    - mail
    - ror
  description: How to setup Rails ActionMailer to use REST API to send emails?
  aliases: 
---

> 🐯 Một số Hosting Server, với mục đích bảo vệ, thiết lập cấu hình chặn các cổng của dịch vụ gửi mail (như 25, 587, 465,...), vậy cần giải quyết như thế nào? 🙂

🎈 Sau khi thử deploy một ứng dụng Rails lên Hosting Server (trong [bài này](til/deploy-rails-to-render.md)), mình phát hiện rằng nền tảng này **chặn** các cổng gửi email.

🔎 Vậy nên mình có đi tìm hiểu các cách xử lý, và một giải pháp cho vấn đề này là gửi mail theo **REST API**, tức là ủy quyền cho một Mail Server gửi mail cho mình.

Trong số các nền tảng cung cấp tính năng này, mình thấy có [MailerSend](https://www.mailersend.com/) là ít chặt chẽ trong vấn đề đăng ký tài khoản (so với [SendGrid](https://sendgrid.com/en-us) - không hiểu sao các yêu cầu tạo tài khoản đều bị từ chối).

### 📬 Mailsend with Rails

Để đơn giản, mình sử dụng một SDK trên Ruby giúp tương tác với API của Mailsend (thay vì dùng code Ruby để tạo Request)

```ruby
gem "mailersend-ruby"
```

Sau đó, tạo một Class để gói lại các xử lý

```ruby
require "mailersend-ruby"

class MailSender
    def initialize config
        @client = Mailersend::Client.new <token_from_mailersend>
        @sender = Mailersend::Email.new @client
        @sender.add_from(email: <address_in_accepted_domain>)
    end

    def send recipients, subject, parts
        recipients.each do |email|
            @sender.add_recipients(email:)
        end
        @sender.add_subject(subject)
        parts.each do |type, data|
            @sender.public_send("add_#{type}", data)
        end

        @sender.send
    end
end
```

🪝 Xem thêm các methods tại [github](https://github.com/mailersend/mailersend-ruby)

🎈 Đến đây, chức năng gửi mail đã hoàn thành, tuy nhiên vẫn chưa phải là linh động, việc gửi mail vẫn phải gọi thủ công method `send`. Vì thế, mình đi tìm cách tích hợp nó vào **ActionMailer**, hỗ trợ sẵn trong Rails.

### 🖇 REST API and ActionMailer

Trong cấu hình cho ActionMailer, ta có thể xác định một trong các giá trị `smtp`, `sendmail`, `file` và `test` cho

```ruby
config.action_mailer.delivery_method = :smtp
```

💡Vậy liệu mình có thể tự tạo một delivery method dạng `restapi` được không?

Mình tìm được một câu trả lời rất quý giá trên [Stackoverflow](https://stackoverflow.com/questions/64436237/how-can-rails-applicationmailer-be-configured-to-use-an-api-or-restclient-inst)

Vì thế, mình triển khai lại code như sau:

- Tạo một tệp trong `initializers` để thêm một `delivery_method` vào **`ActionMailer::Base`**

```ruby
# config/initializers/mail_sender.rb
require "mailersend-ruby"

class MailSender
    def initialize config
        @client = Mailersend::Client.new <token_from_mailersend>
        @sender = Mailersend::Email.new @client
        @sender.add_from(email: <address_in_accepted_domain>)
    end

    def deliver! mail
        mail.to.each do |email|
            @sender.add_recipients(email:)
        end
        @sender.add_subject mail.subject
        mail.parts.each do |part|
            @sender.public_send("add_#{part.sub_type == "html" ? :html : :text}", part.body.raw_source)
        end

        @sender.send
    end
end

ActionMailer::Base.add_delivery_method :restapi, MailSender
```

- `MailSender` lúc này đóng vai trò là một delivery method cho `ActionMail::Base`, và nó cần override lại method `deliver!`, với đối số là một [**`Mail::Message`** object](https://www.rubydoc.info/github/mikel/mail/Mail/Message)

Sau khi khai báo một delivery method tên là `:restapi` và ánh xạ tới Class `MailSender`, ta đi cấu hình

```ruby
# config/environments/<env>.rb
config.action_mailer.delivery_method = :restapi

config.action_mailer.restapi_settings = {
    # arguments for `initialize` in `MailSender` class
}
```

> [!note] > [Resend](https://resend.com/overview) cũng là một sự lựa chọn tốt, với slogan **_Resend is the email API for developers._**. Gói miễn phí của nền tảng này có thể gửi tối đa 100 emails/ngày
