---
title: Query
draft: false
tags:
  - ror
  - model
---

Dưới đây là một số hỗ trợ của Active Record khi muốn thực hiện truy vấn vào DB. 

Các methods sau đều là Class Method, các Bang Method sẽ sinh lỗi khi **không tìm thấy** (RecordNotFound)

Các methods này đều luôn trả về một `ActiveRecord::Relation` hoặc một instance hoặc `nil`.

Ta có thể tạo Chain Call trên các methods trả về `ActiveRecord::Relation`. Với Chain Call, truy vấn chỉ được thực hiện trong DB khi kết thúc chuỗi.

# SELECT

Các methods sau sử dụng `SELECT *` để truy vấn

|Method|Description|Exception|
|--|--|--|
|`take`|Lấy N bản ghi đầu|`take!`|
|`first`|Tương tự `take`, nhưng bị ảnh hưởng bởi Default Scope||
|`find`|Tìm kiếm một/nhiều bản ghi dựa trên PID|Có|
|`find_by`|Tìm kiếm một bản ghi dựa trên nhiều trường|`find_by!`|

Ta có thể sử dụng `limit`, `offset` để thực hiện `SELECT *` với các Options `LIMIT`, `OFFSET` tương ứng.

## SELECT Fields

Các lệnh SELECT phía trên thực hiện nạp toàn bộ các trường trong bản ghi. Ta sử dụng `select` để lọc ra một số trường nhất định.

Đối số của nó là danh sách các trường cần lấy.

Kết quả trả về là mảng các instance, song mỗi instance đều chưa khởi tạo hết các thuộc tính (việc truy cập vào các thuộc tính chưa nạp sẽ gây lỗi).

```ruby
User.select([:id, :name])
# => [#<User id: 1, name: "Oscar">, #<User id: 2, name: "Oscar">, #<User id: 3, name: "Foo">]
```

## Pluck

`pluck` hoạt động tương tự `select`, chỉ khác là nó kết thúc Chain Call, trả về một mảng.

Ta có thể dùng `pick` khi chỉ muốn lấy phần tử đầu tiên trong kết quả của `pluck`.

Ta có thể dùng `ids` để Pluck toàn bộ PID.

## Existence

Khi muốn kiểm tra một/nhiều bản ghi có tồn tại hay không ta có thể sử dụng 
- `exists?`
- `any?`
- `many?`

Các lệnh này cũng thực hiện truy vấn nhưng sẽ trả về Boolean dựa trên số lượng bản ghi trả về.

## Batch Query

Khi số lượng bản ghi cần đọc quá lớn, ta cần sử dụng các Batch Queries để đảm bảo chỉ nạp một phần bộ bản ghi vào bộ nhớ.

- `find_each`: Query một Batch và cung cấp **từng phần tử** cho Block.
- `find_in_batches`: Query một Batch và cung cấp **cả Batch** cho Block.

Khi chưa hết các bản ghi, các methods này cũng tự động nạp một Batch mới. Khi sử dụng hai methods này không nên sử dụng ORDER cho tập hợp.

Một số Options:
- `start`: Method này nạp theo thứ tự tăng PID, Option này xác định phần tử bắt đầu nạp (bỏ qua các bản ghi trước)
- `finish`
- `order`: [`find_each` Only] Thứ tự nạp bản ghi theo PID tăng hay giảm

# WHERE

`where` giúp lọc các bản ghi theo điều kiện. 

Đối số có thể là

**1. String**: Một lệnh SQL. [Nên tránh dùng]

```ruby
Book.where("title = 'Introduction to Algorithms'")
```

**2. Template + Params**: Một String xác định mẫu điều kiện, sử dụng Placeholder là ký tự `?` hoặc một key, các đối số phía sau của `where` sẽ được truyền vào các Placeholder này.

- Sử dụng Placeholder `?`

```ruby
Book.where("title = ? AND out_of_print = ?", params[:title], false)
```

- Sử dụng Key

```ruby
Book.where("created_at >= :start_date AND created_at <= :end_date",
  { start_date: params[:start_date], end_date: params[:end_date] })
```

**3. Hash**: Sử dụng trực tiếp Hash (mà không cần Template) với key là tên trường dữ liệu và value có thể là:
- _Scalar_: Sử dụng `=`
- _Range_: Sử dụng `BETWEEN...END`
    - Có thể không cần xác định điểm đầu hoặc cuối của Range -> Sử dụng `>=` hoặc `<=`
- _Array_: Sử dụng `IN`

## Join WHERE

Ta có thể sử dụng các methods `not, or, and` để nối hai mệnh đề điều kiện.

VD:
```ruby
### NOT ####
Customer.where.not(orders_count: [1, 3, 5])
# SELECT * FROM customers WHERE (customers.orders_count NOT IN (1,3,5))

### OR ####
Customer.where(last_name: 'Smith').or(Customer.where(orders_count: [1, 3, 5]))
# SELECT * FROM customers WHERE customers.last_name = 'Smith' AND customers.orders_count IN (1,3,5)

### AND ####
Customer.where(last_name: 'Smith').where(orders_count: [1, 3, 5])
# Customer.where(last_name: 'Smith').where(orders_count: [1, 3, 5])

Customer.where(id: [1, 2]).and(Customer.where(id: [2, 3]))
# SELECT * FROM customers WHERE (customers.id IN (1, 2) AND customers.id IN (2, 3))
```

# ORDER

`order` giúp sắp xếp thứ tự bản ghi lấy ra.

Đối số là một Hash với
- key là trường dữ liệu cần sắp xếp
- value là `:desc, :asc` thể hiện thứ tự giảm (`:desc`) hoặc tăng (`:asc`). Mặc định là tăng

# GROUP BY

Để tạo mệnh đề `GROUP BY` ta sử dụng method `group`

Đối số là các trường thuộc tính cần nhóm.

Kết quả trả về là một mảng, với mỗi nhóm chỉ lấy phần tử đầu tiên (dựa trên PID)

## HAVING

Ta có thể xác định điều kiện lọc trên nhóm bằng method `having`, với đối số tương tự `where`, nếu muốn dùng SQL Aggregation Method (như SUM,...) thì nên sử dụng Template + Params

# Scope

Ta có thể tạo các Query Methods dựa trên các methods trên, các methods tự tạo này được gọi là Scope.

Các Scopes luôn trả về một `ActiveRecord::Relation`. Điều này giúp tạo Chain Call giống như các methods trên. Chính điều này cũng khiến Scope khác với một việc định nghĩa một Class Method.

Scope được đăng ký cho Model qua `scope`, với đối số là tên của Scope (sẽ được dùng để gọi thực thi Scope) và một PROC để xác định chuỗi các lệnh truy vấn cho Scope.

Các Scopes cũng có thể nhận vào đối số.

```ruby
class Book < ApplicationRecord
  scope :out_of_print, -> { where(out_of_print: true) }
  scope :out_of_print_and_expensive, -> { out_of_print.where("price > 500") }

  # With Arguments
  scope :costs_more_than, ->(amount) { where("price > ?", amount) }
end
```

Ta có thể xác định điều kiện `if...else` bên trong PROC định nghĩa Scope. Lúc này, nếu điều kiện `false`, thay vì trả về `nil`, Scope sẽ sử dụng `all` để luôn trả về một `ActiveRecord::Relation`

## default_scope

`default_scope` giúp đăng ký các Scope được gọi tự động mỗi khi thực hiện truy vấn trên Model.

# More and Advanced

## Overriding

Sau khi định nghĩa một chuỗi lệnh, ta có thể hủy hoặc định nghĩa lại một số thao tác thông qua:
- `unscope`: Hủy một thao tác, đối số là tên method. Không xác định đối số sẽ hủy toàn bộ Scope đang áp dụng.
- `only`: Chỉ giữ lại một số thao tác, đối số là  tên của các methods.
- `reselect`: Chọn lại trường cho `select`
- `reorder`: Chọn lại trường/thứ tự cho `order`

## Readonly, Locking

Ta có thể sử dụng `readonly` để đảm bảo các bản ghi nạp lên chỉ có thể đọc - các thao tác cập nhật sẽ gây lỗi.

Để tránh tình trạng Race Condition khi cập nhật trong môi trường đa người dùng, Rails cung cấp hai cơ chế:
- **Optimistic Locking**: [Mức ứng dụng] Sử dụng `lock_version` column, mỗi lần cập nhật bản ghi sẽ tăng cột này, đảm bảo các giao dịch có `lock_version` nhỏ hơn sẽ không được chấp nhận.
    - Khi đó, cần bắt Exception và xử lý
- **Pessimistic Locking**: [Mức DB] Sử dụng cơ chế bảo vệ của DBMS
    - Sử dụng `lock` trên instance để khóa bản ghi - ngăn chặn mọi thao tác cập nhật lên bản ghi đã khóa.
    - Dùng trong một giao dịch để tránh Deadlock. Kết thúc giao dịch, bản ghi sẽ được mở.
    - Ta cũng có thể sử dụng `with_lock` và định nghĩa một giao dịch cho Block.

## Eager Loading

Khi làm việc với Association ta thường làm việc với dữ liệu lớn, do sự liên kết giữa các bảng. VD: Khi nạp một Book, ta cũng thường phải nạp các Authors.

Khi nạp qua Associations như vậy, Rails sử dụng cơ chế Lazy Loading - chỉ nạp (Query vào DB để lấy dữ liệu lên Memory) khi sử dụng đến. Điều này có thể dẫn đến vấn đề về hiệu năng gọi là **N+1 Queries**. VD:

```ruby
books = Book.limit(10)

books.each do |book|
  puts book.author.last_name
end
```

Ở đây, ta muốn nạp 10 Books và mỗi Book lại cần nạp Author của nó (để truy xuất `last_name`). Khi xem xét dưới dạng SQL Script sinh ra:
- 1 Query cho 10 Books: `SELECT * FROM book LIMIT 10`
- Với mỗi Book, thực hiện nạp Author qua Association.

Rõ ràng cách này là không hiệu quả, vì sau khi nạp Book, ta cũng xác định ngay các Author IDs.

Để giải quyết vấn đề **N+1 Queries** này, ta có một số cách sau:

### preload

`preload` chỉ thị rằng, khi nạp các bản ghi trên một Model, ta đồng thời nạp các bản ghi của các Models khác (xác định trong đối số) dựa trên Association giữa chúng. VD:

```ruby
Book.preload(:author)
```

sẽ thực hiện hai truy vấn là:
- `SELECT * FROM Book`
- `SELECT * FROM Author WHERE Author.book_id IN (1,2,3,...,N)`

### includes

`includes` hỗ trợ tốt hơn `preload`, khi cho phép xác định điều kiện.

- Nếu không đi cùng `where` thì sẽ tương đương `preload`.
- Nếu đi cùng `where` (trên Model liên kết), lệnh truy vấn sẽ dùng `LEFT OUT JOIN` để nối các Model.

### eager_load

`eager_load` sẽ luôn nối các bảng sử dụng `LEFT OUT JOIN`

## Enum

Enum giúp định nghĩa một thuộc tính cho Model, với giá trị chỉ nhận một trong các giá trị xác định trước. Khi ánh xạ vào một trường trong DB, giá trị này sẽ được chuyển về `integer`, và ngược lại.

Như vậy, với Enum, giá trị của thuộc tính Integer sẽ có ý nghĩa cụ thể.

```ruby
class Order < ApplicationRecord
  enum :status, [:shipped, :being_packaged, :complete, :cancelled]
end
```

Việc định nghĩa Enum cũng đồng thời định nghĩa một số methods và scopes. VD:
- `Order.shipped`: Scope lọc ra các bản ghi `status = shipped`
- `Order.first.shipped?`: Method kiểm tra giá trị cho `status` của bản ghi có phải là `shipped` không.
- `Order.first.shipped!`: Method cập nhật giá trị cho `status` là `shipped` - 0 trong DB.

## Aggregations

Các methods sau dùng để tính toán trên các bản ghi thu được
- `count`
- `sum`, `average`
- `maximum`, `minimum`