---
title: Mock and Stub
draft: false
tags:
  - ror
  - unit-testing
  - rspec
---

RSpec Mocks giúp
- Gọi một method chưa triển khai trên một object
- Xác định giá trị trả về cho methods
- Thiết lập Expectation cho kết quả gọi method

# Objects

## Double

Các chức năng này được triển khai dựa trên Double
- Một Object được tạo sẵn
- Hoặc tạo với một định danh xác định

Double có chức năng mô phỏng lại một object đã xác định nhưng chưa được triển khai. 

Khi một Test Object A có phụ thuộc vào một Object B, Double giúp việc Test A không phụ thuộc vào việc B đã được triển khai hay chưa (**Test Isolation**). Điều này giúp việc phát hiện nguồn gốc lỗi khi kiểm thử.

Ta tạo một Double qua:

```ruby
double_object = double([<name>])
```

Với `[name]` xác định tên của Double, được dùng trong Documentation hoặc hiển thị Fail.

## Stub

Một Double được tạo ra thì chưa mang bất kỳ chức năng nào. Ta có thể gắn chức năng cho Double qua một hoặc nhiều Stubs. 

Stub là một chỉ thị cho một object (có thể là Double hoặc một object đã có sẵn như `controller`) trả về một giá trị khi nó nhận được một message xác định.

```ruby
allow(<object>).to receive(<message>)[.with(<arguments>)] { <return_value>}

allow(<object>).to receive(<message>)[.with(<arguments>)].and_return(<return_value>)
```

VD: `allow(die).to receive(:roll){3}` xác định rằng: **"Khi `die` gọi `.roll` sẽ thu được kết quả `3`"**

### Odd and Ends

- Để gắn đồng thời nhiều Stubs cho một object, ta sử dụng `receive_messages` thay cho `receive`
    - Đối số của nó là một Hash ánh xạ tên method và kết quả trả về (`and_return`) khi gọi method đó trên object.
- Khi cần tạo Chain Calls, ta sử dụng `receive_message_chain` thay vì `receive`
    - Đối số của nó là danh sách các method cần gọi trong Chain, và một Block trả về kết quả.
- Có thể sử dụng `allow_any_instance_of(Klass)`: Để chỉ định toàn bộ instance của Class cùng có một stub


## Verifying Double

Verifying Double là phiên bản nghiêm ngặt hơn của Double, khi mà, nếu phiên bản Double đó đã được triển khai thì các Stubs và Expectations cho các methods (và đối số) không hợp lệ sẽ không được gắn cho Double.

## Message Expectation

Message Expectation là một Expectation: **"Một Double sẽ nhận một message trong quá trình thực thi Example"**, tức là: Example Pass chỉ khi message được gọi trên Double **trước khi Example kết thúc**

Cú pháp của Expectation Messages cũng giống Stubs
```ruby
expect(<object>).to receive(<message>)[.with(<arguments>)] { <return_value>}

expect(<object>).to receive(<message>)[.with(<arguments>)].and_return(<return_value>)
```