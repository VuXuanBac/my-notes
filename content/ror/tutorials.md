---
title: RoR Michael Tutorial
draft: true
date: 2024-03-04
tags:
  - ror
  - example
---

# Installation

Config GEM (Ruby Package Manager)
- Turn off document generator for GEM: `echo "gem: --no-document" > ~/.gemrc`

Install [Bundler](https://bundler.io/) provides a consistent **environment** for Ruby projects by tracking and installing the exact gems and versions that are needed.
-  `gem install bundler`
- Install dependencies for project: `bundle install`

Install Rails:
- `gem install rails [-v <version>] --no-document`

Create and Run new project with Rails:
- `rails new <name> && cd <name>`
- `rails server`

# Directory Structure

|Directory|Description|
|--|--|
|**app/**|core application (app) code, including models, views, controllers, and helpers|
|**app/assets/**|application assets such as Cascading Style Sheets (CSS) and images|
|**bin/**|binary executable files|
|**config/**|application configuration|
|**db/**|database files|
|**doc/**|documentation for the application|
|**lib/**|library modules|
|**log/**|application log files|
|**public/**|data accessible to the public (e.g., via web browsers), such as error pages|
|**bin/rails/**|a program for generating code, opening console sessions, or starting a local| server
|**test/**|application tests|
|**tmp/**|temporary files|
|_Gemfile_|gem requirements for this app|
|_Gemfile.lock_|a list of gems used to ensure that all copies of the app use the same gem versions|
|_config\.ru_|a configuration file for Rack middleware|


# Tools

- Scaffold: `rails generate scaffold <resource_name> <atrribute:dtype>s`
    - Ex: `rails generate scaffold User name:string email:string`
- Console: `rails console` [Interact with the application]
- Controller & Actions: `rails generate controller <controller> [<actions>]` (controller name in plural)
  - Generate _app/controllers/\<controller>\_controller.rb_ with action methods
  - Generate _app/views/\<controller>/\<action>.html.erb_
  - Generate _test/controllers/\<controller>\_controller.rb_
  - Generate _app/helpers/\<controler>\_helper.rb_
  - Add GET route to _config/routes.rb_
- Model: `rails generate model <model> [<atrribute:dtype>s]`  (model name in singular)
  - Generate _db/migrate/\<timestamp>\_create\_\<model>.rb_
  - Generate _app/models/\<model>.rb_
  - Generate _test/models/\<model>\_test.rb_
  - Generate _test/fixtures/\<model>.yml_
- Migration: `rails generate migration <name>`  (migration name in snake_case)
- Layout Test:  `rails generate integration_test <name>`
  - Generate _test/integration/\<name>\_test.rb_

# MVC

Rails follow MVC architecture pattern
- Macro: Singleton methods

## Models

**Active Record** is the Rails default library for interacting with the database. Much like _Entity Framework_ for ASP.

### Migration

Migration allow data definitions to be written in pure Ruby, not DDL. It provides a way to alter the structure of database incrementally. Migration file for one model is located at _db/migration/\<timestamp>\_create\_\<model>.rb_

Migration object is an instance of `ActiveRecord::Migration`, with the `change` method that determines the change to be made to the DB.
- `create_table` receive a name for the table and accept a block with variable represent the table itself.
    - By default, the table contains 2 columns: **created_at** and **updated_at**, that are generated with **timestamps** attribute defined in migration object.
- `add_index` receive Table name, column name and hash options (`unique`)
- `add_column` receive Table name, column name and datatype.

File _db/schema.rb_ stores the structures of all the model migrated in database.

Migrated DB can be Rollback to last migration state. 

### MySQL

- Install 
  - On Linux: `sudo apt-get install mysql-server mysql-client libmysqlclient-dev`
  - On Rails: `gem install mysql2`
  - Config for Rails: Edit _config/database.yml_, set the password.
- Enter Shell: `sudo mysql -u root -p`
  - Change password: `ALTER USER 'root'@'localhost' IDENTIFIED BY '<new_password>';`
  - Change password 2: `update user set authentication_string=password('<new>') where user='root'; FLUSH PRIVILEGES;`
  - Show DBs: `show databases;`
  - Go to DB: `use <database>`
  - Show Tables in DB: `show tables;`
  - View Table Structure: `describe <table>;`

### Model Object

All Model classes inherite from `ActiveRecord::Base` class
- `valid?`: check if the model instance is valid
  - `errors.full_messages`: Invalid Error
- `save`: store the model instance to DB, return boolean. 
  - After `save`, the instance can be changed (auto-incremental ID, create timestamp, update timestamp,...)
- `reload`: refresh instance with data from DB.
- `Model.create`: Like `Model.new` -> `save`. Accept a **HASH**.
- `destroy`: delete from DB, the instance itself is remain in memory.
- `Model.find`: return record with ID or raise RecordNotFound.
- `Model.find_by`: return record with attribute or return nil. Accept a **HASH**.
- `Model.first`, `Model.all` (Array, ActiveRecord::Relation)
- `update`: update instance to DB. Accept a **HASH**.
- `update_attribute`

### Callback

Callback is a method that gets invoked at a particular point in the lifecycle of an Active Record object.

## View

### Form

Create Form with Helper `form_with` with following arguments:
- `model`: Model object
- `local`: True if not want to send XHR request (use for localhost)

This method also accept a block with argument represent the form object itself
- From this object, we can create other form elements like: `label`, `text_field` (input)
  - HTML Attribute `id` = `<model>_<field>`.
  - HTML Attribute `name` = `<model>[<field>]`
- Use attribute name for access Model object field.

The form is auto generated with
- `<form action"/\<controller\> class="\<action\>_\<model\>  method="post" >`
- An input hidden contain the authentication token for mitigrate CSRF attack.

## Summary

- Partial Template:
  - Located at _app/views/layouts/\_\<name>.html.erb_
  - Embedded with: `<%= render 'layouts/<name>' %>`
- Provide data from page to template:
  - Define data: `<% provide(<key>, <value>) %>`
  - Retrieve data: `<% yield(<key>) %>`
- Helpers: Methods that can be called in Views
  - Define in _helpers/\<controller>\_helper.rb_ or _helpers/application\_helper.rb_
- Model Associations: `has_many` (N), `belongs_to` (1)
- Add REST routes to `config/routes.rb`:
  - `resources :<controller>`
  - GET => `show`
- Populate model to view:
  - Define an instance variable in respective Controller Action: `@user`
    - Using Model to populate the variable
    - `params` represent the param on URL

# Q&A

## Gem config

Gem config giúp quản lý các cấu hình dùng trong project, như là các cấu hình cho môi trường và ứng dụng.
- Các cấu hình được nhóm lại trong các tệp .yml hoặc .rb (VD: nhóm theo môi trường: production, development,...)
- Các cấu hình đó sẽ được đóng gói lại trong một Ruby object và có thể truy cập toàn cục.

### Config Syntax

- If the value is Regex, prepend it with `!ruby/regexp`

## nil?, empty?, blank?, present?

- nil?: Kiểm tra object có là `nil` object không.
- empty?: Kiểm tra Collection (array, hash, string,...) có rỗng (số lượng phần tử là 0).
- blank?: [Rails] Trả về true nếu
  - nil, false
  - String chứa toàn ký tự khoảng trắng ' ', '\t', '\n',...
  - Empty Array, Hash.
Kiểm tra chuỗi có chứa toàn ký tự khoảng trắng hoặc nil, false không
- present?: [Rails], ngược với blank?

## Uniqueness by Scope

Trong nhiều trường hợp, ta không cần thiết phải thiết lập một trường là duy nhất toàn cục. VD: Tên dự án (Github) của cá nhân chỉ cần là duy nhất đối với từng tài khoản. Xem xét cụ thể hơn thì chính là yêu cầu Unique với một tổ hợp nhiều trường dữ liệu.

VD:
```ruby
# (Guest, Restaurant, Date) must be unique -> A guest could have one reservation at a restaurant per day

# Validation
validates :guest_id, uniqueness: {scope: [ :restaurant_id, :reservation_date ]}

# Index
add_index :reservations, [:guest_id, :restaurant_id, :reservation_date], unique: true,
```
## Callback

Callback là các methods gắn vào vòng đời của một object cho phép kiểm soát các thao tác (create, update, destroy) tác động lên object đó. Cụ thể, callback là đoạn mã được kích hoạt trước hoặc sau thao tác thay đổi trạng thái trên object.

VD: Các callbacks được gọi cho các thao tác liên quan đến:
- Tạo object: `before_validation`, `before_save`, `before_create`,...
- Cập nhật object: `before_validation`, `before_save`, `before_update`,...
- Xóa object: `before_destroy`,...

- `after_create`: Được gọi ngay trước khi lưu object vào DB (**save**, **create**).=> có thể rollback, chưa có record.
- `after_save`: Được gọi sau khi lưu object vào DB (**save**, **create**) nhưng trước khi kết thúc giao dịch => có thể rollback, đã có record.
- `after_commit`: Được gọi khi giao dịch đã kết thúc (**create**, **update**, **destroy**) => KHÔNG thể rollback. 
  - Có thể chỉ định thao tác qua hash option với key `:on`. 

## Environment

- 3 environments: `test`, `development` and `production`
- Check current environment: `Rails.env` (string), `Rails.env.<name>?` (bool)
- Run console within env: `rails console <env>`
- Run server within env: `rails server --environment <env>`
- Migrate DB within env: `rails db:migrate RAILS_ENV=<env>`

## Form

Để tạo một Form từ Rails code, ta có thể sử dụng một trong ba Helpers:
- `form_tag`: Một Form Element thông thường, không Binding với một đối tượng dữ liệu nào cả. Bên cạnh đó thì các Form Controls được tạo trực tiếp từ các `..._tag` Helpers.
- `form_for`: Form có sử dụng Binding tới một Model Object. Các Form Controls cũng được tạo dựa trên Form Builder (đối số của Block truyền vào method này)
- `form_with`: Kết hợp chức năng của cả hai. Nó ra đời để thay thế hai methods trước.
  - Ngoài ra, mặc định các Form sẽ Submit sử dụng XHR. Có thể thay đổi bằng việc thiết lập Option `local: true`

Thông thường ta sẽ sử dụng `form_with`, với các Options sau:
- `url`: Giá trị cho HTML Form `action`
- `method`: Giá trị cho HTML Form `action`
- `model`: Xác định Model Object muốn Binding với Form (lấy từ Controller Action)
  - Việc xác định Option này sẽ thực hiện tự động ánh xạ cho Action, Method.
  - Các tên trường dữ liệu cũng được thu lại bên trong phạm vi của Object. VD: `user[name]`
Ngoài ra ta cũng truyền cho nó một Block với đối số đại diện cho Form tạo được. Trên biến này, ta có thể gọi một số Helpers để tạo các Form Controls như Radio, Input,... Các Helpers nhận vào các đối số sau:
- Tên hoặc trường dữ liệu mà Control đó phục vụ. Từ đó, khi Form Submit, dữ liệu sẽ được thu gom từ các Form Controls và đóng gói vào Hash với key tương ứng.
- Nội dung của Control đó

Các Form Control Helpers: `check_box`, `radio_button`, `text_area`, `hidden_field`, `password_field`...

# Chapter 11

- `figaro` hỗ trợ quản lý tập trung các thông tin cấu hình cần che dấu trong ứng dụng hoặc mở rộng ra là các thông tin về môi trường.
  - Định nghĩa các cấu hình vào tệp _config/application.yml_
- Mailer:
  - `deliver_later`: Lời gọi bất đồng bộ => Đưa email vào hàng đợi của ActionJob. Khi ActionJob sẵn sàng sẽ lần lượt gửi các emails trong hàng đợi của nó.
  - `deliver_now`: Gửi email ngay lập tức, không phụ thuộc vào ActionJob.

# Chapter 12

- Chức năng lấy lại mật khẩu
- Thêm Custom Validation không cho phép mật khẩu mới trùng với mật khẩu cũ

# Chapter 13

- Tạo, Xóa, Hiển thị danh sách cho Micropost
- i18n cho Models, i18n-js
- File Upload, Upload Validation
- ActiveStorage Eager Load: `with_attached_image`

# Chapter 14

- Following
- Associations Advance

- AJAX Rails vs AJAX jQuery
  - AJAX Rails: Tự động kích hoạt khi submit form, link, button,... với `remote:true`
  - jQuery AJAX: Kích hoạt thông qua jquery code
- **Associations**:
  - `class_name` và `foreign_key` dùng khi muốn đặt tên lại cho một tham chiếu bên trong Model hiện tại. VD: Trong hệ thống quản lý bản quyền, User đại diện cho tài khoản đăng ký, song bên trong Goods Model (đại diện cho tác phẩm) thì tài khoản đóng vai trò là tác giả, do đó, cần phải đặt tên lại khi tham chiếu.
    - `class_name`: Xác định tên Model Class muốn tham chiếu.
    - `foreign_key`: Xác định tên của cột muốn dùng làm Foreign Key cho việc tham chiếu.
  - `dependent`: Xác định cách thức xử lý khi một record - đang được tham chiếu từ một bảng khác - bị xóa. VD: Một User bị xóa thì xử lý thế nào với các Posts của người đó
    - `destroy`: Loại bỏ lần lượt các records phụ thuộc (Post). Các callbacks được gọi.
    - `delete_all`: Loại bỏ tất cả cùng lúc các records phụ thuộc. **Không gọi callback**
    - `nullify`: Thiết lập NULL cho cột Foreign Key. **Không gọi callback**
    - `restrict_with_exception`: Raise Exception
    - `restrict_with_error`: Thêm Errors vào record đang muốn xóa (User)
  - `through` giúp tạo liên kết với Model [B] thông qua một Model trung gian [C]
    - Nói cách khác, giúp Model hiện tại [A] biết được nó có thể truy cập vào một Model [B] thông qua [C]
      - VD: Một Bác sĩ [A] có thể truy cập danh sách Bệnh nhân [B] thông qua danh sách Lịch khám [C] mà anh ta biết.
    - `source`: Xác định tên trường trong Model [C] dùng để lấy giá trị của [B]
