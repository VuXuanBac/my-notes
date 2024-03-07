---
title: Active Job
draft: false
date: 2024-03-04
tags:
  - ror
  - system
  - extension
---

Active Job là một Framework cho phép quản lý các Jobs (clean-ups, mailings,...) cho ứng dụng. 

Các Jobs được chạy bởi Job Runner, đây là thành phần quản lý việc thực thi các Jobs, thông thường sử dụng Queue. Mỗi Job sẽ cần được nạp vào Queue và sẽ được Job Runner sắp xếp để chạy.

Các Job Runners cũng có thể triển khai nhiều Queue khác nhau, đại diện cho mức độ ưu tiên khác nhau.

Active Job cung cấp nền tảng bao phủ sự khác biệt giữa Job Runners khác nhau (với API khác nhau)

Job Runner mặc định trong Rails là **Asynchronous** (`:async`). Runner này thực hiện các Jobs trong các Threads nằm trong cùng Process với ứng dụng. Điều này giúp cho các Jobs có thể chạy bất đồng bộ nhưng sẽ bị xóa nếu chạy lại ứng dụng.

# Create a Job

Mỗi Job trong Rails được mô tả qua một Ruby file bên trong thư mục _app/jobs/_, có thể tạo qua: `rails generate job <name> --queue <queue>`

Các Jobs kế thừa từ `ApplicationJob < ActiveJob::Base` và cần phải override method `perform`. Method này là nội dung của Job, chứa các lệnh sẽ được thực thi khi Job được chạy.

`perform` có thể nhận vào đối số bất kỳ nhưng thông thường ta chỉ nên truyền vào các đối số với kiểu dữ liệu cơ bản. VD: Thay vì truyền vào một `User`, ta có thể truyền `id` và bên trong sẽ thực hiện tìm `User` tương ứng.

Ta có thể xác định Queue mà Job này sẽ được nạp vào thông qua macro `queue_as`, mặc định là `:default`

```ruby
class ReportJob < ActiveJob::Base
  queue_as :default

  def perform(*args)
    # Do something later
  end
end
```
# Run a Job

Để một Job được thực thi, ta chỉ cần thêm nó vào Queue của Job Runner thông qua `perform_later`

Gọi method này trên Class Object với đối số tương ứng như khai báo cho `perform`.

Ta có thể cấu hình thời gian thêm Job vào Queue với method `set` với các Options:
- `wait`: Chờ một khoảng thời gian rồi mới nạp.
- `wait_until`: Chờ tới một thời điểm rồi mới nạp.
- `queue`: Thay đổi lại Queue Type
- `priority`: Độ ưu tiên cho Job (càng lớn càng ưu tiên)

```ruby
ReportJob.set(wait_until: Time.now.tomorrow, queue: :urgent).perform_later
```

# Queuing Backends

Job Runner mặc định trong Rails là `:async`, tức là các Jobs sẽ được lưu vào trong RAM, rõ ràng là không phù hợp với ứng dụng thực tế.

Do đó, ta cần chọn một Job Runner, cùng với Queuing Backends. 

## Configuration

Rails cung cấp sẵn một số Adapters cho Queuing Backends như **Sidekiq**, **Resque**, **Delayed Job**,... Để cấu hình sử dụng một Queuing Backend, ta thiết lập cấu hình trong _config/application.rb_

```ruby
# config/application.rb
class Application < Rails::Application
    # Be sure to have the adapter's gem in your Gemfile
    # and follow the adapter's specific installation
    # and deployment instructions.
    config.active_job.queue_adapter = :sidekiq
  end
```

hoặc cấu hình cho từng Job qua:

```ruby
class ReportJob < ActiveJob::Base
  self.queue_adapter = :resque
  # ...
end
```

## Run

Vì các Jobs cần được thực thi song song với ứng dụng nên hầu hết các Job Runner sẽ cần được khởi động cùng với ứng dụng.

VD: Với Sidekiq, ta chỉ định chạy Sidekiq cùng với khởi tạo các Queues theo thứ tự ưu tiên giảm dần

```cmd
bundle exec sidekiq -q [queue1] -q [queue2]...
```

# Sidekiq

Sidekiq là một Framework cho các tác vụ chạy nền với 3 thành phần:
- **Client**: Cung cấp API cho việc tạo và nạp Job vào Queue
- **Redis**: Redis cung cấp Data Storage cho Sidekiq, nó chứa dữ liệu về các Jobs cũng như dữ liệu về lịch sử
- **Server**: Thực hiện lấy các Jobs trong hàng đợi (lưu tại Redis) để xử lý.

Sidekiq thực hiện lưu các đối số của `perform_later`  vào Redis dưới dạng JSON.

Sidekiq chạy các Job (Worker - tên cũ) trên **các luồng** thay vì trên tiến trình (như Resque) và **hỗ trợ Retry khi Job Fail** (ActiveJob mặc định không hỗ trợ nhưng nếu dùng Sidekiq vẫn sẽ có)

Ta có thể tạo một Sidekiq Job thay vì Job trong Active Job. Điểm khác biệt là:
- ActiveJob hỗ trợ **GlobalID để Serialization một Object**, nên ta có thể truyền vào `perform`  một Active Record (thay vì chỉ truyền vào ID và sau đó Deserialization)
  - Vấn đề là Object nếu bị xóa trước khi truyền cho Sidekiq thì có lỗi.
- ActiveJob hỗ trợ **Callback** cho quá trình thực thi Job.
- Chính vì hai đặc điểm này làm việc thực thi Job qua Active Job tiêu tốn nhiều CPU hơn, nên **tốc độ xử lý chậm hơn**.
- Tuy nhiên, do ActiveJob là interface chung cho các Job Runner, nên nó **không hỗ trợ Retry**, trong khi Sidekiq thì hỗ trợ.
- Sidekiq hỗ trợ **Bulk Queueing** (đẩy lượng lớn Jobs vào Redis một lần - giảm trễ khi kết nối tới Redis)

# Whenever

Gem `whenever` cung cấp cách thức tạo và deploy Cronjob vào hệ thống, sử dụng cú pháp Rails

Cấu hình:
```ruby
# Gem
gem 'whenever', require: false

# Command Line
cd path/to/project
bundle exec wheneverize .
```

Sau bước cấu hình này, ứng dụng sẽ được tạo một tệp _config/schedule.rb_. Đây là nơi chứa các lệnh tạo Cronjob

## Job Type

Các Jobs định nghĩa trong Whenever, mặc định, được phân thành các nhóm
- `command`: Command Line
- `rake`: Rake Task
- `runner`: Thực thi các Class Method
- `script`: Chạy script hệ thống

Tương ứng với chúng là các lệnh hệ thống khác nhau

```ruby
job_type :command, ":task :output"
job_type :rake,    "cd :path && :environment_variable=:environment bundle exec rake :task --silent :output"
job_type :runner,  "cd :path && bin/rails runner -e :environment ':task' :output"
job_type :script,  "cd :path && :environment_variable=:environment bundle exec script/:task :output"
```

Với các cấu hình
- `:path`: Mặc định trỏ đến thư mục mà `whenever` được thực thi (VD: Lệnh deploy Cronjob)
- `:environment`: Mặc định là `:production`
- `:environment_variable`: Mặc định là `RAILS_ENV`
- `:output`: Vị trí lưu file kết quả việc thực thi Job

Các cấu hình này có thể được xác định bằng việc sử dụng method `set` bên trong _config/schedule.rb_. VD:

```ruby
# Ghi các kết quả thực hiện các Jobs vào trong tệp "#{path}/log/cronjob.log"
set :output, "#{path}/log/cronjob.log"
```

## Job Syntax

Mỗi Job được định nghĩa để chạy (lặp lại) vào một thời điểm xác định (VD: Cứ mỗi 8:00 AM vào Thứ hai) sử dụng `every` method

```ruby
every 3.hours do # Every 3 hours run the following Tasks
  # the following tasks are run in parallel (not in sequence)
  runner "MyModel.some_process"
  rake "my:rake:task"
  command "/usr/bin/my_great_command"
end

every 1.day, at: '4:30 am' do # Every day, at 4:30 am
  runner "MyModel.task_to_run_at_four_thirty_in_the_morning"
end
```

## Cronjob Update

Sau khi khai báo các Jobs bên trong _config/schedule.rb_, ta thực hiện cập nhật Cronjob trong hệ thống qua lệnh: `whenever --update-crontab`

Để kiểm tra cập nhật thành công chưa, ta chạy lệnh `crontab -l` để xem các cronjob trong hệ thống.

Ta có thể thực hiện truyền đối số khi `whenever` thực thi _config/schedule.rb_ bằng Option `--set`.

```cmd
whenever --update-crontab --set "environment=development&report_day=monday&report_time=8:00am"
```

Các đối số truyền cho `--set` Option sẽ được đóng gói vào biến `@set_variables` trong Schedule File.

VD:

```ruby
# config/schedule.rb
env :PATH, ENV["PATH"] # incase `bundle command not found`
set :output, "#{path}/log/cronjob.log"

report_day = @set_variables[:report_day] || :monday
report_time = @set_variables[:report_time] || "8:00am"

every report_day, at: report_time do
  runner "ReportJob.perform_now"
end
```

Chú ý:
- Mặc định `whenever` sử dụng Rails environment là `:production` nên có thể gây lỗi, cần sử dụng `--set` Option để cập nhật lại
- Một số Job Type sử dụng `bundle` Command, trong một số trường hợp khi chạy Job, `shrims` chưa được nạp, ta cần sử dụng lệnh `env :PATH, ENV["PATH"]` để cập nhật lại biến môi trường cho `whenever`
