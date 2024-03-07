---
title: Migration
draft: false
date: 2024-03-04
tags:
  - ror
  - model
---

Migration giúp theo dấu quá trình thay đổi Schema **db/schema.rb**. Migration sử dụng Ruby code. Mỗi Migration là một tệp trong thư mục **db/migrate** với tên bắt đầu bằng timestamp, để chỉ thứ tự của Migration.

Có thể sinh tự động Migration qua `rails generate migration <name> <options>`

Việc đặt tên Migration cũng giúp tạo lệnh tương ứng. Các Conventions:

| Convention  |  Description |
|---|---|
| **`CreateXXX`** `[col]:[type]:[index],...` | Tạo bảng với tên **XXX** và các cột tương ứng | 
| **`Add___ToXXX`** `[col]:[type]:[index],...` | Thêm các cột `[col]` (kiểu dữ liệu `[type]`, tạo index tương ứng) vào bảng **XXX** |
| **`Remove___ToXXX`** `[col]:[type],...` | Xóa các cột `[col]` (kiểu dữ liệu `[type]`) từ bảng **XXX** |
| **`Add___RefToXXX`** `[ref]:references,...` | Thêm Reference Column tới bảng `[ref]` vào bảng **XXX** |
| **`CreateJoinTableXXXYYY`** `[xxx] [yyy]` | Tạo bảng liên kết N-N giữa hai bảng `[xxx]` và `[yyy]` |

# Methods

## create_table

- Dùng để tạo một bảng
- Cú pháp

```ruby
create_table :<name>, <table_options> do |t|
    t.<type> :<col>, <modifiers>
end
```

- Mặc định sinh PRIMARY KEY `id`
    - Table Option: `primary_key: [key1, key2...]` để thay đổi mặc định
    - Table Option: `id: false` để không tạo PRIMARY KEY
- Một số [Field Options - Column Modifiers](#column-modifiers)
- Một số [Column DataType](#column-type)

## change_table

- Dùng để cập nhật lại cấu trúc một bảng
- Cú pháp

```ruby
change_table :<name>, <table_options> do |t|
    t.<type> :<col>, <field_options>
    t.<change>
end
```

- Cú pháp tương tự `create_table`
- Có hỗ trợ thêm một số Helpers

| Field Option  | Description  |
|---|---|
| `index` | Chỉ ra các Columns tạo Index cho bảng |
| `remove` | Chỉ ra các Columns bị xóa |
| `rename` | Chỉ ra Column cần đổi tên và tên mới của nó |

## add_column

- Tạo Column cho bảng
- Cú pháp:

```ruby
add_column :<table>, :<col>, :<type>, <options>
```

- Tạo Column `<ref_table>_id` cho bảng `<table>`, dùng để tham chiếu tới bảng `<ref_table>`
    - Mặc định: Kiểu bigint + Index
- Tuy nhiên, không tạo Foreign Key Constraint
    - Thêm bằng Option: `foreign_key: true`
    - Hoặc chỉ định cụ thể bảng: `foreign_key: {to_table: :<table>}`

## add_index

- Tạo Index Columns cho bảng
- Cú pháp:

```ruby
add_index :<table>, :<cols>, <options>
```

- Tạo index cho bảng `<table>`, từ một cột `:<col>` hoặc nhiều cột `[:<col>, :<col>]`
    - Mặc định: Kiểu bigint + IndexColumn
- Một số Options:
    - `unique`: `true` nếu muốn giá trị của các cột này là UNIQUE (kiểm tra tự động)


## add_reference

- Tạo Reference Column cho bảng
- Cú pháp:

```ruby
add_reference :<table>, :<ref_table>, <options>
```

- Tạo Column `<ref_table>_id` cho bảng `<table>`, dùng để tham chiếu tới bảng `<ref_table>`
    - Mặc định: Kiểu bigint + IndexColumn
- Tuy nhiên, không tạo Foreign Key Constraint
    - Thêm bằng Option: `foreign_key: true`
    - Hoặc chỉ định cụ thể bảng: `foreign_key: {to_table: :<table>}`

## add_foreign_key

- Tạo Foreign Key Column cho bảng
- Cú pháp:

```ruby
add_foreign_key :<table>, :<ref_table>, <options>
```

- Tạo Foreign Key Column `<ref_table>_id` cho bảng `<table>`, dùng để tham chiếu tới bảng `<ref_table>`
    - Mặc định: Kiểu bigint + IndexColumn
- Tạo cả Foreign Key Constraint `fk_rails_...`
- Một số Options:
    - `column`: Đặt tên lại cột Foreign Key
    - `primary_key`: Primary Key của `<ref_table>`
    - `on_delete`, `on_update`: Cách thức xử lý trên record tham chiếu khi record trên `<ref_table>` bị xóa: `:nullify, :cascade, :restrict`

# Seed and Fake Data

Seeding là quá trình sinh dữ liệu tự động cho DB. Seed thực tế là các lệnh Ruby dùng các ActiveRecord Methods để thay đổi dữ liệu trong DB. Các lệnh này được viết trong tệp `db/seeds.rb` và được chạy khi gọi `rails db:seed`

Dữ liệu cho Seedings có thể sinh tự động qua Gem **[Faker](https://www.rubydoc.info/gems/faker/)**


# References

## Column Type

| Column Type  |   |
|---|---|
| `:primary_key` | |
| `:string`, `:text` ||
| `:integer`, `:bigint`, `:binary` ||
| `:float`, `:decimal`, `:numeric` ||
| `:datetime`, `:time`, `:date` ||
| `:blob` ||
| `:boolean` ||

## Column Modifiers

| Column Modifiers | Description  |
|---|---|
| `index` | `true` nếu muốn tạo Index cho Column. Hoặc truyền vào một Hash với trường `unique: true` để tạo UNIQUE INDEX |
| `null` | `false` nếu không muốn Column nhận giá trị NULL. Mặc định `true` |
| `default` | Giá trị mặc định cho Column, dùng `nil` cho NULL. Không hỗ trợ giá trị động. VD: `Time.now` | 
| `limit` | Số ký tự tối đa đối với String, Số bytes tối đa đối với Text/Integer/Binary |
| `precision` | Độ chính xác cho Decimal/Numeric/DateTime/Time | 
| `scale` | Số chữ số sau dấu `.` đối với Numeric/Decimal |
| `collation` | Cách thức so sánh đối với String/Text, mặc định dùng lại từ bảng |
| `comment` | Mô tả về cột, được lưu cùng DB Metadata |

## Migration Commands

| Command  |  Description |
|---|---|
| `rails db:migrate` | Chạy lại các Migrations đang chờ (chưa được cập nhật vào Schema) |
| `rails db:rollback` | Khôi phục trạng thái trước khi chạy `n` Migration cuối cùng<br>- `STEP=n` |
| `rails db:seed` | Khởi tạo Seed |
| `rails db:drop` | Xóa DB |
| `rails db:setup` | Tạo DB + Chạy lại Schema (tạo lại các bảng) + Khởi tạo Seeds |
| `rails db:reset` | Xóa DB và Tạo lại DB + Chạy lại Schema (tạo lại các bảng) + Khởi tạo Seeds  |

