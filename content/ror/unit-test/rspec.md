---
title: RSpec
draft: false
tags:
  - ror
  - unit-testing
  - rspec
---

[RSpec](https://rspec.info/) là một testing framework có hỗ trợ Ruby on Rails. 

RSpec không chỉ hướng đến tạo các đoạn chương trình kiểm thử mà còn giúp đặc tả chương trình sử dụng ngôn ngữ tự nhiên.

RSpec được hỗ trợ bởi các thư viện sau:
- **rspec-core**: Runner + Command Line + Report + API.
- **rspec-expectations**: API giúp diễn tả một kết quả kỳ vọng cho một đoạn chương trình.
- **rspec-mocks**: Test Double Framework.

Gem cho ứng dụng RoR là: [`gem 'rspec-rails'`](https://github.com/rspec/rspec-rails). Ta nên thêm Gem này cho cả `test` và `development` environment (để việc generate Model, Controller,... sẽ tự động sinh cả RSpec files).

Để khởi tạo RSpec cho ứng dụng, sử dụng `rails generate rspec:install`, lệnh này sẽ sinh các tệp và thư mục:
- `.rspec`: Chứa các lệnh cấu hình cho RSpec, tự động được nạp trước khi chạy.
- `spec_helper.rb`: Chứa các lệnh cấu hình cho RSpec và các thư viện của nó.
- `rails_helper.rb`: 

# Basic

## Generate Spec Files

Khi thêm RSpec cho `development` environment thì khi chạy các lệnh `rails generate` cho models, controllers,... sẽ tự động sinh các RSpec files tương ứng cho chúng.

Ta có thể chạy lệnh sinh RSpec file thủ công: `rails generate rspec:[type] [name]`

Với `[type]` là một trong 10 kiểu specs hỗ trợ (bởi RSpec-Rails), mỗi kiểu tương ứng với một phần cần kiểm thử trong ứng dụng. Mỗi kiểu cũng tương ứng kế thừa một `TestCase` Class có sẵn trong Rails, tức là chúng có sẵn các tiện ích và assertions được định nghĩa sẵn trong các Classes này, bên cạnh các hỗ trợ từ RSpec.

## Rails Spec Type

Với mỗi kiểu, các RSpec file thường được đặt trong thư mục tương ứng. VD: Các `model` RSpec files sẽ nằm trong thư mục _spec/models/_.

| Type  | Base | Description |
|---|---|---|
| `model`  | | |
| `controller` | `ActionController::TestCase` | Kiểm thử cho một HTTP Request - Controller Action |
| `mailer` | `ActionMailer::TestCase` | ||
| `job` | ||	
| `view` | `ActionView::TestCase` ||
| `routing` | ||
| `helper` | `ActionView::TestCase` ||
| `request` | `ActionDispatch::IntegrationTest` ||
| `feature` | ||
| `system` | ||

## Execute Spec Files

- Chạy toàn bộ Spec Files, ta sử dụng: `bundle exec rspec`
- Chạy toàn bộ Spec Files trong một thư mục (tức là một Spec Type): `bundle exec rspec spec/models`
- Chạy một Spec File: `bundle exec rspec spec/models/user_spec.rb`
- Chạy từ một dòng (tìm Example gần nhất) trong Spec File: `bundle exec rspec spec/models/user_spec.rb:15`

# RSpec Syntax - DSL

RSpec sử dụng khái niệm Example để chỉ một Test Case. Một (Example) Group dùng để nhóm các Examples có liên quan đến nhau.

- `describe`: Tạo một Group, thường để **nhóm theo Class, Module hoặc Method**.
- `it`: Tạo một Example.
- `context`: Tương tự `describe`, nhưng thường dùng để **nhóm theo Conditions**.

Mỗi Example/Group đều bắt đầu bằng một đoạn Plain Text để mô tả cho chính nó (dùng để tạo specification) và một Block chứa nội dung (đoạn mã) thực thi của nó.

Khi thực hiện chạy các Spec, các đoạn mô tả sẽ được hiển thị theo thứ tự từ ngoài vào trong (từ Group, đến các Nested Groups và cuối cùng là Example). Đoạn mô tả này, khi nối với nhau sẽ giống một mệnh đề, mô tả đầu vào, cách thức thực hiện và đầu ra mong muốn.

## `describe`

`describe` thường dùng để nhóm các Examples liên quan với nhau theo **Class, Module hoặc Method**.

Description của `describe` thường dùng Convention sau:
- Nếu nhóm không phải Method: Mô tả về nhóm. VD: Một HTTP Request, một Class Name
- Nếu nhóm là một **Instance Method**: Sử dụng `#` bắt đầu tên Method.
- Nếu nhóm là một **Class Method**: Sử dụng `.` bắt đầu tên Method.

## `context`

`context` thường dùng để nhóm các Examples liên quan với nhau theo **Conditions**. VD: Khi tạo request tới một Product không tồn tại, ta yêu cầu có Flash Message và Redirect, tức có 2 Examples cùng liên quan tới điều kiện `Product không tồn tại`.

Description của `context` thường bắt đầu bằng **`when`** hoặc **`with`**.

## `it`

`it` dùng để khai báo một Example bên trong một Group.

Description của `it` thường mô tả kết quả mong đợi của Test Case.

# Expectation

RSpec::Expectations cung cấp API dùng để biểu diễn một mệnh đề kỳ vọng một đối tượng (**Expect Target**) sẽ thỏa mãn một điều kiện (**Matcher**). Nếu không thỏa mãn, Expectation được gọi là **Fail**, và ngược lại là **Pass**.

Cú pháp:

```ruby
expect(<object>).to <matcher>, <message>
expect(<object>).to_not <matcher>, <message>

expect { <expression> }.to <matcher>, <message>
```
VD:

```ruby
expect(object).to eq(3) # Pass if object == 3
```
 
Theo sau `to` hoặc `to_not` (hoặc `not_to`) là:
- [Matcher](matchers): Giúp diễn tả một điều kiện. Tất cả các Matcher đều trả về Boolean.
  - Nếu trả về True thì Expectation **Pass**. Ngược lại tức là **Fail**.
  - Đó cũng là giá trị trả về của `to`, `to_not`.
- `message`: String hoặc Proc dùng để trả về thông báo hiển thị (khi chạy Spec) nếu như Expectation Fail.

Hai cú pháp đầu dùng để tạo Expectation cho một object mang dữ liệu.

Cú pháp thứ ba dùng để tạo Expectation cho đối tượng khác. Các Matchers [`raise_error, throw_symbol, yield_*`](#errorsthrowsyield) và [`change`](#change) chỉ có thể sử dụng cho nhóm này.

# Hooks

Hooks là các đoạn mã được thực thi trước và/hoặc sau các Examples. Điều này đặc biệt hữu dụng khi các toàn bộ các Examples (trong phạm vi nào đó) có chung các đoạn mã khởi tạo hoặc dọn dẹp.

Hook có thể khai báo bên trong **Example Group** hoặc **Global** (bên trong `RSpec.configure`).

Một số phạm vi
- `:example` (hoặc `:each`): Chạy Hook trước/sau **mọi** Examples trong Group hiện tại.
- `:context` (hoặc `:all`): Chạy Hook **một lần** trong Group hiện tại, thời điểm ngay trước/ngay sau khi mọi Examples thực thi.
- `:suite`: Chỉ hỗ trợ khi đăng ký Hook Global, chỉ chạy **một lần** trước khi toàn bộ các Example Group chạy

Bên trong các Hooks cũng có thể khai báo các instance variables `@`, có thể truy cập trong mọi Examples, và nếu một Example thay đổi giá trị thì các Example phía sau cũng bị ảnh hưởng.

VD:

```ruby
RSpec.describe Thing do 
    before(:example) do
        # this code run before every examples
    end

    after(:context) do
        # this code run once, after all group's examples finished
    end

    # other examples
end
```

Thứ tự chạy các Hooks theo phạm vi là:
**before(:suite) -> before(:context) -> before(:example) -> after(:example) -> after(:context) -> after(:suite)**

## Conditional Hooks

Ta có thể giới hạn Hooks chỉ áp dụng cho một số Examples bằng việc thêm metadata cho Example đó, và kiểm tra Example Metadata được truyền cho Hook Block.

VD: Tạo Request Format tùy vào Metadata của Example/Group

```ruby
describe "DELETE #destroy" do
  before do |example|
    delete :destroy, params: {
      format: example.metadata[:turbo] ? :turbo_stream : :html
    }
  end

  context "with turbo_stream format", :turbo do
    # All Example in here using `delete :destroy, params: {format: :turbo_stream}` request
  end

  context "with html format" do
    # All Example in here using `delete :destroy, params: {format: :html}` request
  end
```

# Helpers

## `let`, `let!`

`let` giúp tạo một Helper Method. Giá trị trả về của Helper này sẽ được cached giữa các lời gọi bên trong cùng một Example, tức nó giống như một biến hằng số bên trong Example.

Theo sau `let` là Block khai báo nội dung của Helper, Helper này sẽ chỉ được gọi một lần bên trong mỗi Example, và giá trị sẽ được cache lại cho các lời gọi bên trong cùng Example.

`let!` là phiên bản Eager Load của `let`, tức là nó chỉ được thực thi duy nhất một lần ngay lúc khai báo, giá trị sẽ được cache lại cho mọi lời gọi sau.

VD: Các Examples sau đều pass

```ruby
$count = 0
RSpec.describe "let" do
  let(:count) { $count += 1 }

  it "memoizes the value" do
    expect(count).to eq(1) # evaluate here: $count = 1
    expect(count).to eq(1) # load cache here: count -> 1, $count = 1
  end

  it "is not cached across examples" do
    expect(count).to eq(2) # evaluate here: $count = 2
  end
end
```

Chú ý, khi các `let` Helper có cùng tên thì khai báo sau sẽ được ưu tiên. VD: Nếu một Shared Example có khai báo một `let` Helper và được thêm vào cùng một context thì sẽ có các `let` Helper cùng tên.


## Custom

Ta có thể tạo các Helper Methods như thông thường thông qua `def` hoặc `define_method`

Các Helpers này có thể được gọi trong Group được khai báo hoặc Nested Group của nó. Tất nhiên, các lời gọi sẽ làm Helper được thực thi từ đầu, khác với `let` và `let!`

# Subject

`subject` là một đối tượng đặc biệt trong RSpec, nó đại diện cho một đối tượng dữ liệu cần kiểm thử, có thể truy xuất trong mọi Examples bên trong một Group.

Khi truyền một ClassName cho phần Text của một Group, `subject` sẽ được gán giá trị là New Instance của Class đó.

Ta có thể định nghĩa một `subject` cho Group với một Block theo sau nó, giá trị trả về sẽ được gán cho `subject`. Hoạt động của `subject` giống với `let` - cache giữa các lời gọi bên trong cùng Example.

Bên cạnh việc định nghĩa nội dung, ta có thể đặt một tên có ý nghĩa hơn (trong Group hiện tại) cho `subject` (Named Subject). Khi đó ta có thể dùng tên này để truy xuất thay cho `subject`.

Phiên bản `subject!` cũng hoàn toàn tương tự `let!`

Điểm khác biệt giữa `subject` và `let`
- `subject` có thể khởi tạo ngầm (dựa vào tên Class cho Example Group)
- Có thể gọi Expectation ngầm trên `subject` (và Named Subject)

# Sharing

Có một số cách triển khai khi một số Example/Group có chung một số logic

## Shared Examples

Khi có nhiều **Group** có chung một số **Examples** với nhau, ta có thể đặt Examples chung vào một **Shared Examples**.

VD: `GET Index` giữa các Post và User Controllers yêu cầu **trả về OK** và **render Index**, chúng chỉ khác nhau phần Data dùng để render, Shared Examples sẽ chứa hai Examples này.

Shared Examples được khai báo qua `shared_example` method, nó giống với `describe` và `context`, song cho phép nhận vào đối số.

Để gọi một Shared Examples bên trong một Example Group, ta sử dụng `include_examples`, theo sau là Name của Shared Examples và đối số tương ứng cho nó.

Method `include_examples` sẽ thêm Shared Examples vào **trong Context hiện tại** của Group. Trong một số trường hợp có thể gây xung đột giữa các Examples được thêm vào cùng Context (VD: Sử dụng một `let`, Hooks trong Shared Examples và thêm chúng vào cùng một Context thì giá trị của `let` có thể xung đột - Xem thêm [`let`](#let-let)).

Ta có thể sử dụng `it_behaves_like` để thêm Shared Examples vào **Context bên trong Context hiện tại**.

Chú ý, nếu muốn nhúng một Shared Examples từ một RSpec file khác thì ta cần đảm bảo RSpec file này được nạp trước file hiện tại. Hoặc có thể sử dụng `require path/to/spec/file/.rb` để khai báo. Với đường dẫn là tương đối so với thư mục **spec/**

### Context for Shared Examples

Ta có thể truyền một Context cho Shared Example bằng việc truyền Block khi nhúng Shared Examples đó.

```ruby
RSpec.shared_examples "a collection object" do
  describe "<<" do
    it "adds objects to the end of the collection" do
      collection << 1
      collection << 2
      expect(collection.to_a).to match_array([1, 2])
    end
  end
end

RSpec.describe Array do
  it_behaves_like "a collection object" do
    let(:collection) { Array.new }
  end
end

RSpec.describe Set do
  it_behaves_like "a collection object" do
    let(:collection) { Set.new }
  end
end
```

### Include Shared Examples Automatically

Một Example Group có thể chủ động nhúng một Shared Examples bên trong nó bằng việc thiết lập metadata trùng với metadata của Shared Examples.

```ruby
RSpec.shared_examples "shared stuff", :a => :b do
  it 'runs wherever the metadata is shared' do
  end
end

RSpec.describe String, :a => :b do
end
```

Ở đây, mặc định bên trong Group `String` đã có một Example được nhúng từ Shared Examples `shared stuff` vì có metadata trùng nhau

## Shared Context

Khi có nhiều **Group** có chung một số **Hooks, Helpers,...** với nhau, ta có thể đặt chúng vào một **Shared Context**.

VD: Các chức năng yêu cầu phải đăng nhập trước khi thực hiện có thể đặt `sign_in(user)` vào một `before(:example)` Hook và để Hook này vào một Shared Context.

```ruby
RSpec.shared_context "shared admin controller" do
  let(:admin){create(:admin)}

  before do
    allow(controller).to receive(:authenticate_admin_account!).and_return(true)
    allow(controller).to receive(:current_admin_account).and_return(admin)
  end
end
```

Sau đó, ta có thể nhúng Shared Context này tương tự Shared Examples. VD: `include_context "shared admin controller`

hoặc ta có thể cấu hình tự động nhúng Shared Context vào các Group có metadata khớp với mẫu cho trước. VD:

```ruby
RSpec.configure do |config|
  # Tự động nhúng `shared admin controller` Shared Context vào các Group với `admin: true` HOẶC `api: true` 
  config.include_context "shared admin controller", :admin, :api
end
```

Chú ý: **Điều kiện lọc cho `include_context` là HOẶC**

# Metadata

Mỗi Example và Group đều có Metadata bao gồm: 
- `description`: Plain Text mô tả
- `described_class()`: Khi `description` cho **Group** là Class thì method này sẽ trả về Class đó.
- Custom Metadata: Hash Options truyền cho `it, describe, context,...` khi khai báo Example/Group sẽ được coi là Custom Metadata.
  - Một số cấu hình RSpec dựa vào Metadata để thực thi.
  - Hash Options có thể chỉ cần xác định Symbol cho key, value mặc định là `true`

Trong `it, subject, let` và Hooks, ta có thể truy cập vào chính Example hiện tại qua Block Argument truyền cho chúng

```ruby
RSpec.describe Symbol do
  # Access current Example as Block Argument
  before do |example|
      expect(example.description).to eq("is available as described_class")
  end

  # Access current Class Group as `described_class()` method
  it "is available as described_class" do
    expect(described_class).to eq(Symbol)
  end
end

# Declare a metadata with key `foo` for entire Group
RSpec.describe "a group with user-defined metadata", :foo => 17 do
  it 'has access to the metadata in the example' do |example|
    expect(example.metadata[:foo]).to eq(17)
  end

  it 'does not have access to metadata defined on sub-groups' do |example|
    expect(example.metadata).not_to include(:bar)
  end
end
```

# Configuration

## Command Line Configuration

`/.rspec` là một trong các tệp mà RSpec tự động nạp vào để nhận các cấu hình dưới dạng Command Line Option:
  - **`--require spec_helper`**: Tự động khai báo `spec_helper` cho các RSpec files.
  - **`--tag ~skip`**: Bỏ qua các Example/Group chứa metadata `skip: true`

VD: Thay vì chạy `rspec --tag ~skip`, ta có thể để phần Option `--tag ~skip` vào `/.rspec`

## `RSpec.configure`

Các Helpers files như _spec/spec\_helper.rb_ hay _spec/rails\_helper.rb_ có khai báo các cấu hình cho RSpec dưới dạng

```ruby
RSpec.configure do |config|
  config.<option> = <value>
end
```

## Some Configuration

### Pending and Skip Example/Group

Khi có Example/Group 
- Không muốn thực thi, ta có thể cấu hình **skip** nó
- Thực thi nhưng không muốn Fail của nó ảnh hưởng đến toàn bộ Suite, ta cấu hình **pending** nó.

Mặc định, các Example/Group sau sẽ được **skip**:
- Chưa triển khai
- Sử dụng `skip` để khai báo Example thay vì `it`
- Prepend `x` trước `it` (hoặc `specify`, `example`) khi khai báo Example
- Thêm `skip: true` Metadata

### Run some Example/Group only

Ta có thể giới hạn Example/Group nào được thực thi thông qua việc khai báo Metadata cho chúng và thêm cấu hình cho `RSpec.configure`

```ruby
RSpec.configure do |c|
  # Only run Example/Group declare `:some_key` metadata
  c.filter_run_including :some_key => true

  # Dont run Example/Group declare `:another_key` metadata
  c.filter_run_excluding :another_key => true
end
```