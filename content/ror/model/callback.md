---
title: Callback
draft: false
tags:
  - ror
  - model
---

Callback là các methods gắn vào vòng đời của một Model instance. Với Callback, ta có thể thêm các logic xử lý sẽ được thực thi mỗi khi instance được **create**, **save**, **update**, **delete**, **validate** hoặc **load**.

Để đăng ký một Callback cho Model, ta sử dụng các Macros và truyền vào một METHOD/BLOCK/PROC

# Order

Khi một thao tác cập nhật (create, update, destroy) diễn ra, Model instance trải qua một chuỗi các trạng thái, tương ứng với đó là các Callbacks có thể gắn vào, chuỗi này tạo thành một Transaction. Các Callbacks trong Transaction luôn diễn ra theo một thứ tự xác định. 

  - **Create**:
    - before_validation
    - after_validation
    - before_save
    - around_save
    - _before\_create_
    - _around\_create_
    - _after\_create_
    - after_save
    - after_commit/after_rollback
  - **Update**:
    - before_validation
    - after_validation
    - before_save
    - around_save
    - _before\_update_
    - _around\_update_
    - _after\_update_
    - after_save
    - after_commit/after_rollback
  - **Destroy**:
    - before_destroy
    - around_destroy
    - after_destroy
    - after_commit/after_rollback

Bất kể Callback nào trong Transaction cũng có thể sinh Exception `throw :abort` để hủy Transaction, tức là:
- Không thực hiện tiếp các Callbacks phía sau
- Rollback: Các thao tác cập nhật được khôi phục lại

# Callback Trigger

Chỉ một số thao tác cập nhật cho phép thực thi Callback:
- create, create!
- destroy, destroy!, destroy_all, destroy_by
- save, save!, save(validate: false)
- toggle!
- touch
- update_attribute, update, update!
- valid?

# Conditional Callback

Ta có thể xác định điều kiện cần kiểm tra trước khi gọi Callback tương ứng, tức là Callback chỉ được thực thi nếu như điều kiện trả về `true`.

Để xác định một/nhiều điều kiện gắn với một Callback, ta sử dụng options: `:if` và `:unless`. Đối số của chúng là **Proc**, **Method Name** hoặc **Array của các giá trị trên**.

Method dùng để kiểm tra điều kiện sẽ không nhận vào đối số (thay vào đó có thể sử dụng trực tiếp các attributes và methods từ Model). Đối số của Proc sẽ là object đại diện cho Model instance.

```ruby
class Comment < ApplicationRecord
  before_save :filter_content,
    if: [:subject_to_parental_control?, Proc.new { untrusted_author? }]
end
```

# Transaction Callback

Với Transaction, có hai Callbacks được tạo riêng cho nó là:
- `after_commit`: Transaction thành công
- `after_rollback`: Transaction hoàn thành việc Rollback

Hai Callbacks này được gọi sau `after_save` và `after_destroy` và chỉ được gọi khi mà **toàn bộ** (trong trường hợp Transaction thực hiện nhiều thao tác trên nhiều object) các cập nhật đã được thực hiện hoặc hủy, tức là DB đã ở trạng thái ổn định.

Hai callbacks này cũng sẽ được gọi trên **từng object tham gia vào Transaction**.