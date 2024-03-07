---
  title: What are Master Keys in Rails?
  draft: false
  date: 2024-03-07
  tags:
    - til
    - ror
    - security
---
#### ⚡ Vấn đề

> 🦏 **Có bao giờ bạn tự hỏi *master.key* trong Rails 7 dùng để làm gì ?** 🤨

🔎 Hãy bắt đầu tìm hiểu về Master Keys từ [Rails cookie](ror/controller/session_cookie_flash.md#cookie), Rails hỗ trợ tạo chữ ký số (tránh giả mạo cookie) và mã hóa cookie (che giấu). Cả hai cơ chế này, nếu muốn an toàn, đều cần một khóa **bí mật**, **ngẫu nhiên** và **đủ dài**. Rails sử dụng thuật toán sinh khóa [PBKDF2 HMAC](https://ruby-doc.org/stdlib-2.5.1/libdoc/openssl/rdoc/OpenSSL/KDF.html#method-c-pbkdf2_hmac) phục vụ mục đích này. 

> [!info]
> Để xem bộ sinh khóa đang được sử dụng, kiểm tra biến `Rails.application.key_generator`

Thuật toán này cần sử dụng một *password*, do đó, cần lưu lại một *password* riêng cho ứng dụng. Rails gọi *password* này là **`secret_key_base`**. Giá trị này được sinh tự động khi tạo project.

> [!info]
> [`secret_key_base`](https://api.rubyonrails.org/classes/Rails/Application.html#method-i-secret_key_base) có thể truy cập qua `Rails.application.secret_key_base`**

Trong các phiên bản khác nhau của Rails mà `secret_key_base` được lưu trữ khác nhau:
- **Rails 4.1** lưu trữ trong tệp **`secrets.yml`**. Tệp này cần giữ bí mật (git ignore)
- **Rails 5.1** bắt đầu triển khai cơ chế mã hóa cho tệp **`secrets.yml`** - tức là tệp mã hóa có thể public.
  - Để mã hóa thì cũng cần một khóa khác. Rails gọi nó là <span style="color:red;font-weight:bold;">RAILS_MASTER_KEY</span>
  - Khóa này được lưu trong **`secrets.yml.key`** - tệp này cần giữ bí mật
- **Rails 5.2** giới thiệu cơ chế **credentials**


🔐 **Credentials** cho phép lưu trữ các thông tin mật dựa trên mã hóa (ví dụ như JWT secret, API key), thay thế cho *secrets.yml* và *secrets.yml.enc*
- **`RAILS_MASTER_KEY`** được lưu tại **config/master.key** (cần giữ bí mật)
- Các dữ liệu đã mã hóa được lưu tại **`credentials.yml.enc`** (và có thể public)
- Việc thêm dữ liệu cần thực hiện qua lệnh 
```cmd
rails credentials:edit
```
- Lúc này, nội dung tệp chưa mã hóa (đã giải mã) sẽ được hiển thị trên editor để có thể chỉnh sửa.
- Sử dụng lệnh sau để hiển thị nội dung của tệp chưa mã hóa
```cmd
rails credentials:show
```
- Mặc định trong tệp này có một key: `secret_key_base`.

> [!note]
> Chú ý cần thiết lập biến môi trường **`$EDITOR`** để cấu hình Editor cho việc thay đổi nội dung tệp chưa mã hóa. VD: `export EDITOR="nano -w"`

> [!info]
> Truy cập các dữ liệu trong Credentials qua **`Rails.application.credentials.<key>`

🚀 Khi thử triển khai một ứng dụng RoR lên Cloud, ví dụ [Render](https://render.com), ta có thể gặp ngay một lỗi từ bước Build, kiểu như:

```
Missing encryption key to decrypt file with. Ask your team for your master key and write it to config/master.key or put it in the ENV['RAILS_MASTER_KEY']
```

Để thêm Master Key cho môi trường Production, ta thực hiện các bước sau:
- Cấu hình yêu cầu thiết lập master key.
```ruby
# config/environments/production.rb

# Ensures that a master key has been made available in either ENV["RAILS_MASTER_KEY"]
# or in config/master.key. This key is used to decrypt credentials (and other encrypted files).
config.require_master_key = true
```
- Cập nhật biến môi trường **`ENV["RAILS_MASTER_KEY"]`** với nội dung từ `config/master.key`

#### 📬 Tóm lại

> [!important]
> - `RAILS_MASTER_KEY` là khóa bí mật dùng để mã hóa/giải mã dữ liệu nhạy cảm trong ứng dụng.
> - `secret_key_base` là khóa bí mật dùng để sinh các khóa cho các bộ mã hóa (encryptors) và bộ xác thực (verifiers)

> [!note]
> Thứ tự tìm kiếm giá trị `secret_key_base`:
> - `ENV["SECRET_KEY_BASE"]`
> - `credentials.secret_key_base`
> - `secrets.secret_key_base`

#### 🪝Tham khảo

- [Saeloun Blog](https://blog.saeloun.com/2023/08/11/rails-7-1-store-secret-key-base-in-rails-config/)
- [Stackoverflow - Secret key base in Rails 4](https://stackoverflow.com/questions/25426940/what-is-the-use-of-secret-key-base-in-rails-4)
- [Viblo](https://viblo.asia/q/secret-key-base-va-rails-master-key-trong-rails-24lJ3NLb5PM)