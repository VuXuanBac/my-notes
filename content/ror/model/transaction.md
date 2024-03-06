---
title: Transaction
draft: false
tags:
  - ror
  - database
---

Hệ thống MultiUsers khi mà nhiều Users có thể đồng thời tương tác với DB, trong khi một yêu cầu cần một chuỗi thao tác DB để hoàn thành, việc xung đột giữa các thao tác có thể gây mất tính nhất quán về dữ liệu.

Một số biểu hiện của không nhất quán dữ liệu:
- Dirty Read: Đọc dữ liệu đã sửa nhưng chưa lưu trữ lâu dài (commit)
- Nonrepeatable Read: Hai lần đọc dữ liệu trong cùng TX thu được kết quả khác nhau
- Phantoms

Transaction là một chương trình thực thi bao đóng một nhóm các lệnh CSDL, đảm bảo chúng được thực thi như một lệnh đơn duy nhất, tức là DBMS khi thực thi một Transaction cần đảm bảo DB khi kết thúc Transaction phải ở một trong hai trạng thái:
- Toàn bộ các thao tác hoàn thành thành công và những tác động đã được ghi nhận lâu dài vào DB. 
- Các thao tác của Transaction không ảnh hưởng đến dữ liệu đang có trong DB và các Transaction khác. 

Một số điều kiện làm Transaction thất bại:
- Phương tiện: Phần cứng (VD: Disk), phần mềm, mạng
- Hệ thống: Chia cho 0, Overflow, Interrupt (User), Abort (Hệ thống kiểm soát đa tiến trình)
- Điều kiện: Các điều kiện, exceptions trong Transaction

Trong trường hợp Transaction chỉ chứa các lệnh truy vấn thì nó được gọi là **read-only transaction**

# DB Transaction

## Properties

- ACID
  - **Atomicity**: TX là đơn vị xử lý cơ bản - tất cả hoặc không là gì cả.
  - **Consistency preservation**: Nhất quán dữ liệu dù thành công hay thất bại - Trạng thái DB là xác định.
  - **Isolation**: Độc lập, không ảnh hưởng đến TXs khác. Có 4 levels
  - **Durability**: Đảm bảo các thay đổi trong TX phải hoàn tất vào DB (lưu trữ lâu dài)

## Buffers

Nhìn chung, các thao tác với CSDL gồm 2 thao tác Đọc và Ghi. Hai thao tác này thực hiện luân chuyển dữ liệu từ Disk sang Buffers (Main Memory - RAM) và ngược lại. Thao tác trên dữ liệu được thực hiện trên Buffers.

Thông thường, các DBMS sẽ duy trì một số Buffers nhất định, và khi mà chúng đã được dùng hết thì một số chiến lược thay thế dữ liệu sẽ được dùng (tùy thuộc vào DBMS)

## State

Để phục vụ mục đích khôi phục trạng thái DB, hệ thống cần sử dụng một số lệnh:
- **BEGIN_TRANSACTION**: Đánh dấu vị trí bắt đầu một TX
- **END_TRANSACTION**: Đánh dấu vị trí kết thúc một TX
- **COMMIT_TRANSACTION**: Báo hiệu trạng thái kết thúc thành công của Transaction. Từ đây, các thay đổi sẽ được lưu trữ lâu dài trong DB và không thể hủy (undone).
- **ROLLBACK/ABORT**: Báo hiệu trạng thái kết thúc thất bại của Transaction. Từ đây, các thay đổi được tạo ra trong Transaction cần được hủy (undone)
- **READ/WRITE**: Các chỉ thị cho thao tác thực hiện trong Transaction

![State Transition of Transaction Execution](assets/transaction/state_diagram.png)

- **Active** là trạng thái Transaction nhận liên tục các thao tác **READ, WRITE**
  - Khi nhận thao tác **ABORT** thì lập tức chuyển lên trạng thái FAILED
- **Partially Committed** là trạng thái kết thúc tạm thời của Transaction, khi các thay đổi **hoàn thành trong Buffers**, lúc này các thủ tục kiểm tra sẽ được tiến hành

## Log

Để khôi phục được trạng thái trước DB về trước khi thực hiện Transaction, DBMS sử dụng Log File (cho các thao tác thay đổi).
- Log File được lưu vào Disk và định kỳ sẽ được Backup
- Có thể sử dụng Buffers để lưu tạm các log entries, và lưu vào Disk vào thời điểm thích hợp (VD: Buffer đầy, ngay trước TX hoàn thành)

Các lệnh trong TX sẽ tương ứng một entry trong Log File và mỗi TX sẽ được sinh một mã định danh `T`:
- `start_transaction, T`:
- `write_item, T, X, old_value, new_value`: Một số giao thức sẽ không lưu `new_value`
- `read_item, T, X`: Có thể không cần thiết trừ khi cần Auditing.
- `commit, T`
- `abort, T`

## Commit Point

**Commit point** là thời điểm mà các thao tác trên DB đã được thực thi thành công và được ghi Log. Ngay sau Commit point, Transaction sẽ đạt trạng thái **COMMITTED**, lúc này `commit` entry mới được ghi Log.

Khi có lỗi, hệ thống sẽ duyệt ngược Log File và tìm kiếm **toàn bộ** TXs có lệnh `start_transaction` mà chưa có `commit` và thực hiện Undoing (rollback)

## SQL Transaction


Mỗi lệnh SQL được coi là một đơn vị thực thi (tất cả hoặc không là gì cả). Mặc định, mỗi lệnh cũng là một TX (với cấu hình `autocommit = 1`)

Một Transaction có thể được tạo ngầm định hoặc chủ động.
- Chế độ ngầm định chỉ yêu cầu có **COMMIT** hoặc **ROLLBACK** để chỉ định vị trí kết thúc chuỗi lệnh
- Chế độ chủ động cho phép xác định vị trí bắt đầu và kết thúc của TX.

SAVEPOINT là một tính năng cho phép Rollback lại một vị trí (trước một lệnh) - khôi phục một phần TX

Mỗi TX có một số đặc trưng (cấu hình qua lệnh **SET TRANSACTION**):
- Access Mode: 
  - Read-Write: Cho phép TX có các thao tác đọc/ghi
  - Read-Only:
- Name:
- Isolation Level:

```sql
-- MySQL
-----------------------------
CREATE TABLE customer (a INT, b CHAR (20), INDEX (a));
-- Do a transaction with autocommit turned on.
START TRANSACTION;
INSERT INTO customer VALUES (10, 'Heikki');
COMMIT;

-----------------------------
-- Do another transaction with autocommit turned off.
SET autocommit=0;
INSERT INTO customer VALUES (15, 'John');
INSERT INTO customer VALUES (20, 'Paul');
DELETE FROM customer WHERE b = 'Heikki';
-- Now we undo those last 2 inserts and the delete.
ROLLBACK;

-----------------------------
SELECT * FROM customer;

+------+--------+
| a    | b      |
+------+--------+
|   10 | Heikki |
+------+--------+
```

# `transaction`

Để tạo một transaction, ta có thể gọi trên Model Class hoặc Model instance bất kỳ, tuy nhiên bên trong transaction đó, ta có thể thực hiện lệnh trên bất kỳ object nào.

VD: Khi một **Order** được chấp nhận, ta cần:
- Cập nhật trạng thái của **Order** thành `accepted`
- Cập nhật số lượng của các vật phẩm **Product** trong **Order** đó.

```ruby
class Order < ApplicationRecord
    def accept
        transaction do 
            self.status = :accepted
            self.items.each do |product|
                ret = product.update_amount_with -1
                raise ActiveRecord::Rollback unless ret
            end
        end
    end
end
```

# Rollback

Bất kể Exception nào phát sinh trong quá trình thực hiện Transaction đều kích hoạt sự kiện **ROLLBACK** - khôi phục trạng thái.

Các Exceptions này sẽ không được xử lý bởi Rails và được lan lên theo các lời gọi, tức là cần được bắt ở vị trí phù hợp. Riêng Exception `ActiveRecord::Rollback` sẽ tự động được xử lý, vì Exception này kích hoạt sự kiện ROLLBACK.

# Nested Transaction

Khi các Transactions lồng nhau, các lệnh DB bên trong Transactions con sẽ được coi như **nhúng trực tiếp vào Transaction cha** (hầu hết các DB đều không hỗ trợ Nested Transaction).

```ruby
User.transaction do
  User.create(username: 'Kotori')
  
  User.transaction do
    User.create(username: 'Nemu')
    raise ActiveRecord::Rollback
  end
end
```

```ruby
User.transaction do
  User.create(username: 'Kotori')
  User.create(username: 'Nemu')

  User.transaction do
    raise ActiveRecord::Rollback
  end
end
```

Ở đây, khi dùng Nested Transactions, hai lệnh `create` đều giống như cùng xuất hiện trong Transaction cha. Lệnh `raise ActiveRecord::Rollback` sẽ được bắt bởi Transaction con và biến mất trong Transaction cha. Khi đó, hai lệnh `create` sẽ vẫn được thực thi.

Để tạo Nested Transaction thực sự (Transaction con độc lập), cần thêm option `requires_new: true` cho Transaction con.

# Active Record Queue

Một Model instance trước khi được cập nhật vào DB (theo một số thao tác `save`, `update`, `destroy`) sẽ trải qua một quá trình, toàn bộ quá trình này được đóng gói vào một Transaction, bao gồm các thao tác:
- Validations
- Callbacks
- DB Operations

Trong suốt quá trình này, nếu ở bất cứ lệnh nào sinh lỗi, **không có lệnh nào phía sau được thực hiện** và **Transaction sẽ được ROLLBACK**.

Riêng hai lỗi `ActiveRecord::Rollback` và `ActiveRecord::RecordInvalid` sẽ được xử lý trực tiếp mà không đẩy lên cho các lời gọi trước.