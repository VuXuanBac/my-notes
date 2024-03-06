---
title: Pagy
draft: false
tags:
  - ror
  - gem
---

Cơ chế phân trang thực hiện phân bổ các nội dung trong các đường dẫn khác nhau.

Khi người dùng di chuyển qua các trang, chỉ số trang mong muốn sẽ được đóng gói vào Request để gửi lên Server. Tại đây, dựa trên chỉ số, Server trích xuất một phần tài nguyên (VD: các tài nguyên có chỉ số từ [page_size * page_index, page_size * (page_index + 1) - 1]) và trả về.

Một thư viện gọn nhẹ cho ứng dụng Rails là `Pagy`:
- Pagy định nghĩa class `Pagy` lưu các biến hỗ trợ cho việc phân trang như: Số lượng items trên một trang, Số lượng items tổng cộng, Vị trí trang cần lấy nội dung.

Cấu hình:

- Cấu hình cho Pagy instance: Tạo tệp _config/initializers/pagy.rb_ như trong [pagy.rb](https://ddnexus.github.io/pagy/quick-start/#configure)

```ruby
# application_controller.rb
include Pagy::Backend
# Từ đó có thể sử dụng hàm `pagy` trong các Controllers muốn phân trang

# application_helper.rb
include Pagy::Frontend
# Từ đó có thể sử dụng các hàm tạo Navigation Bar cho HTML như: `pagy_bootstrap_nav @pagy`
```

- Cấu hình I18n cho Pagy:

```ruby
# config/initializers/pagy.rb
Pagy::I18n.load({ locale: "en" }, { locale: "vi" })

# application_controller.rb
before_action { @pagy_locale = params[:locale] }
```