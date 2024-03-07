---
title: Unit Testing in RoR
draft: false
date: 2024-03-04
tags:
  - ror
  - unit-testing
  - intro
---

Mặc định, một ứng dụng Rails có 3 môi trường: **development, test** và **production** và cũng tương ứng có 3 DB (chúng được cấu hình trong **config/database.yml**).

# Fixtures

Fixtures là cách gọi khác cho dữ liệu mẫu. Fixtures cho phép populate dữ liệu (dược định nghĩa sẵn) vào Test DB, trước khi chạy Test.

Fixtures được viết theo YAML, VD:

```yml
# lo & behold! I am a YAML comment!
david:
  name: David Heinemeier Hansson
  birthday: 1979-10-15
  profession: Systems development
 
steve:
  name: Steve Ross Kellock
  birthday: 1974-09-27
  profession: guy with keyboard
```

Fixtures cũng được tiền xử lý bởi ERB Processor, tức là ta có thể nhúng Ruby Code trong Fixtures.

# Unit Test for Model

Kiểm thử cho các Models thường bao gồm:
- Các Validations
- Các Methods

# Functional Test for Controllers

Functional Test là kiểm thử các Actions trong một Controller.

Các kiểm thử thường có trong Functional Test
- Request được xử lý với phản hồi thành công hay thất bại
- Việc điều hướng có đến đúng địa chỉ
- Người dùng xác thực thành công hay thất bại
- Dữ liệu dùng để sinh Template có đúng hay không
- Thông báo gửi về có đúng hay không.
- ...

## Make a Request

Để mô phỏng một Request gửi đến Server, trong Functional Test có 6 methods `get, post, patch, delete, put, head` tương ứng với các HTTP Methods

Đối số cho các methods này là:
- **`action`**: Tên của Action muốn gọi
- **`params`**: Optional Hash cho các tham số trong request gửi đến.
- **`body`**: Optional String đã được mã hóa biểu diễn phần Request Body.
- **`session`**: Optional Hash cho `session` object.
- **`flash`**: Optional Hash cho `flash`

## Object for Use

Sau khi tạo Request với các methods trên, có các Objects có thể truy cập từ Test là
- `assigns`: Lưu trữ các instance variables của Controller (VD: `@user`)
- `cookies`: Nội dung `cookies`
- `flash`: Nội dung `flash`
- `session`: Nội dung `session`

Ngoài ra có các instance variables bên trong Test:
- `@controller`
- `@request`
- `@response`

