---
title: Matcher
draft: false
tags:
  - ror
  - unit-testing
  - rspec
---

# RSpec Matchers

Matcher bao gồm 
- **Built-in Matchers**: Các Matchers tạo sẵn hoặc các Matcher tự động sinh dựa trên các methods của object tương ứng ([Convention Matchers](#convention-matcher))
- **Custom Matchers**

Ta có thể tạo alias cho một Matcher với method `RSpec::Matchers.alias_matcher <alias>, <name>`

## Built-in Matchers

Xem thêm: [Built-in Matchers](https://rubydoc.info/github/rspec/rspec-expectations/RSpec/Matchers)

### Identity/Equivalence/Truthiness

| Matcher | Pass `to` If |
|---|---|
| `eq` | `==` |
| `be`, `equal` | `equal?` |
| `be_a` | `kind_of?` |
| `be_an_instance_of` | `instance_of?` |
| `respond_to` | `respond_to?` |
| `be_nil` | `nil?` |
| `be_falsey`, `be_truthy` ||
| `exist` | `exist?` |

### Comparisons/Membership

| Matcher | Pass `to` If |
|---|---|
| `be >`, `be >=`,... <br>(không có `be ==`) ||
| `be_between`,<br>`be_between().inclusive`,<br>`be_between().exclusive` | `between?` |
| `be_within(delta).of(expected)` | `object == expected +- delta` |
| `start_with` | object (collection/string) bắt đầu bằng *expected |
| `end_with` | object (collection/string) kết thúc bằng *expected |
| `include` | object (collection/string) chứa *expected |
| `contain_exactly` | object (collection/string) có đầy đủ các phần tử trong *expected (không kể thứ tự) |
| `match_array` | Phiên bản khác của `contain_exactly` khi expected là một mảng (thay vì chuỗi đối số *)  |
| `cover` | Range object chứa các phần tử *expected |
| `match` | - Regexp hoặc String: `match`<br>- Collection: So khớp từng cặp phần tử |

### Errors/Throws/Yield

| Matcher | Pass `to` If |
|---|---|
| `raise_error(error_class, message, &block)` | Nếu không có đối số, match với mọi lỗi sinh ra. Nếu không, match với error_class và/hoặc message |
| `throw_symbol` |  |
| `be_between().inclusive` | `between?` |
| `be_within(delta).of(expected)` | `object == expected +- delta` |
| `yield_control` | `expect` Block gọi `yield` |
| `yield_successive_args` | `expect` Block gọi `yield` nhiều lần với các giá trị lần lượt là giá trị *expected |
| `yield_with_args` | `expect` Block gọi `yield` một lần với các giá trị truyền ra nằm trong *expected |
| `yield_with_no_args` | `expect` Block gọi `yield` một lần và không truyền giá trị nào ra ngoài |

### Change

Quan sát sự thay đổi giá trị. `change` có thể nhận vào hai dạng đối số:
- Receiver và Message
- Block: Phải sử dụng cú pháp `{...}` thay vì `do...end` (vì sẽ xung đột với `expect`)

- `receiver.message` sẽ được đánh giá giá trị **trước** `expect` Block.
- `change` Block sẽ được đánh giá giá trị **sau** `expect` Block.

Sau khi đánh giá hai giá trị này, **nếu chúng vẫn là một object**, sẽ đi so sánh giá trị băm của chúng. Nếu giá trị băm khác nhau tức là có sự thay đổi - tức là Pass.

Ta có thể nối theo sau `change()` bằng một trong các nhóm:
- `from`, `to`: Xác định giá trị trước và/hoặc sau (dựa trên thứ tự đánh giá)
- `by`, `by_at_least`, `by_at_most`: So sánh lượng thay đổi (tăng hoặc giảm)

### Convention Matcher

| Matcher | Pass `to` If |
|---|---|
| `be_xxxx` | `xxxx?` |
| `have_xxxx` | `has_xxxx?` |

## RSpec-Rails Matchers

Các Matchers được định nghĩa thêm trong `rspec-rails`

- **`be_a_new(Class)`**: Pass nếu object là `new` instance của `Class`
- **`have_http_status(status)`**: [Controller|Request|Feature] Pass nếu trả về một Response với mã xác định bởi `status` (Code, Name, Custom)
- **`render_template(name)`**: [Controller|Request|View] Pass nếu trả về một Partial/Template/Layout xác định. **Sử dụng với Target: `expect(response)**
- **`redirect_to(destination)`**: [Controller|Request] Pass nếu trả về một Redirect Response với địa chỉ xác định bởi `destination` (URL, Named Route, `url_for` Arguments,...). **Sử dụng với Target: `expect(response)**
- **`route_to(destination)`**: [Controller|Routing] Pass nếu một request sẽ được ánh xạ tới địa chỉ xác định bởi `destination` (Route Pattern, `url_for` Arguments,...)
- **`be_routable`**: [Controller|Routing] Pass nếu địa chỉ (xác định bởi expectation target) có thể được ánh xạ. **Thường sử dụng `expect(...).not_to be_routable**

## Custom Matchers

Ta có thể định nghĩa thêm một số Matchers khi nó cần dùng nhiều cho ứng dụng

Cú pháp: `<actual>` chỉ đối số truyền cho `expect`

```ruby
RSpec.Matchers.define <name> do |<args_of_matcher>|
  match do |<actual>|
     # some code return boolean, true for Pass
     # Use `block_arg` if <args_of_matcher> if a block
  end

  failure_message do |<actual>|
    # override message when example is fail
  end

  # define some helper methods here
end
```

- Ta sử dụng `block_arg` nếu như đối số truyền cho Matcher là Block
- Override `failure_message_when_negate` cho thông báo lỗi nếu Example không Pass với `.to_not`
- Override `match_when_negated` cho logic kiểm tra với `.to_not` (nếu khác với `.to`)

# Shoulda-Matchers

[Shoulda-Matchers](https://github.com/thoughtbot/shoulda-matchers) cung cấp các matchers hỗ trợ cho RSpec và Minitest frameworks, giúp việc viết unit test ngắn gọn hơn (chỉ cần một dòng code - one-liner).

Một số cấu hình cho GEM này với ứng dụng Rails

```ruby
# Gem
group :test do
  gem 'shoulda-matchers', '~> 6.0'
end

# spec/rails_helper.rb
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```

## Syntax

Khác với RSpec khi khuyến khích sử dụng `expect` thì Shoulda-Matchers thường sử dụng `should` và `should_not` (RSpec cũng có cú pháp này).

Cũng chính vì kỳ vọng viết unit test theo dạng one-liner nên Shoulda-Matcher cũng thường tận dụng [`subject`](rspec.md#subject) với đặc tính implicit expectation.

Cú pháp của [`should` trong RSpec](https://rubydoc.info/github/rspec/rspec-expectations/BasicObject:should):

```ruby
[<object>.]should <matcher>, <message>

[<object>.]should_not <matcher>, <message>
```

Vì `<object>` thường ngầm định là `subject` nên thường không xuất hiện.

## Support Matchers

Shoulda-Matchers cung cấp **thêm** nhiều Matchers để là ngắn gọn đoạn code cần viết cho mỗi Example. Các Matchers này cũng được gom nhóm theo [kiểu của Spec](rspec.md#spec-type) và chỉ có thể sử dụng cho kiểu spec tương ứng.

### ActiveModel and ActiveRecord Matchers - `model` Type

Các Matchers nhóm này đều hỗ trợ các Qualifiers sau:
- `.on(context)`: Chỉ xét trong Context xác định (`:create`, `:update`...)
- `.with_message(message)`

Một số Matchers sẽ hỗ trợ Qualifiers:
- `.allow_nil`
- `.allow_blank`

Các Matchers:
- **`allow_values(*expected).for(attr)`**: Pass nếu `attr` nhận một trong các giá trị *expected.
- **`validate_xxxx_of(attr)`**: Pass nếu `validates_xxxx_of(attr)` không có `errors`. `xxxx` có thể là:
    - `absence`, `presence`, `acceptance`, `confirmation`
    - `comparison`: Qualifiers `.is_greater_than(value)`, `is_other_than(value)`,....
    - `numericality`: Qualifiers `.is_greater_than(value)`, `is_other_than(value)`,...., `.even`, `.odd`, `.only_integer`, `.is_in`
    - `exclusion`, `inclusion`: Qualifiers `.in_array`, `.in_range`
    - `length`: Qualifiers `.is_at_least`, `.is_at_most`, `.is_equal_to`
    - `uniqueness`: Qualifiers `.scoped_to(scope)`, `.case_insensitive`, `.ignore_case_sensitivity`

Ngoài ra, các Matchers của ActiveRecord thường liên quan đến Associations, như là: `have_one`, `have_one_attached`, `belong_to`, `have_rich_text`,...

### Controller - `controller` Type

- **`respond_with(status [number/range/symbol])`**: Pass nếu Action trả về Response có mã tương ứng.
- **`permit(*params).for(action, params:, verb:)`**: Pass nếu các `params` parameters của `action` (với HTTP Method `verb`) được định nghĩa trong `permit` Strong Parameters.
    - `.on(required_param)`: Khi có required parameters
- **`redirect_to(destination, &block)`**: Pass nếu Action trả về một Redirect Response tới một địa chỉ xác định bởi `destination` (URL, Named Route, `url_for` Arguments,...)
- **`render_template(options, message)`**: Pass nếu Action trả về một Template/Partial
- **`render_with_layout(layout = nil)`**: Pass nếu Template trả về được render trong Layout xác định
- **`set_flash`**: Pass nếu Action thực hiện thiết lập dữ liệu cho `flash` 
    - `[:key]`: Chỉ định một key kiểm tra
    - `.to`: Chỉ định giá trị thiết lập cho key.
    - `.now`: Xét `flash.now`
- **`set_session`**: Pass nếu Action thực hiện thiết lập dữ liệu cho `session` 
    - `[:key]`: Chỉ định một key kiểm tra
    - `.to`: Chỉ định giá trị thiết lập cho key.
    - `.now`: Xét `flash.now`
- **`use_before|after|around_action(name)`**: Pass nếu Controller thiết lập `before_action`, `after_action`, `around_action`