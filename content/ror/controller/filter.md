---
title: Filters
draft: false
tags:
  - ror
  - controller
---

Filters là các methods được chạy tại một thời điểm nhất định trong vòng đời của một Controller Action, có thể là trước (before) khi chạy, sau (after) khi chạy hoặc cả trước và sau (around) khi chạy Action.

Các Filters được đăng ký trong một Controller thì có thể dùng lại trong các Controllers kế thừa nó, tức là nếu thiết lập trong `ApplicationController` thì mọi Controllers của ứng dụng đều có thể áp dụng.

# Before Filter

Một Filters cần chạy trước một hoặc một số Actions (Before Filter) được đăng ký qua method `before_action`.
- Sử dụng `only`, theo sau là mảng các Symbol đại diện cho các Actions, để chỉ định cụ thể các Actions **được** áp dụng Filter.
- Sử dụng `except`, theo sau là mảng các Symbol đại diện cho các Actions, để chỉ định cụ thể các Actions **không được** áp dụng Filter.

Before Filters nếu chứa các lệnh `render` hay `redirect_to` thì các Filters cũng như Actions phía sau sẽ không được thực thi.

Ta có thể sử dụng method `skip_before_action` để một Before Filters (được kế thừa) không áp dụng cho các Actions bên trong Controller hiện tại. Ta có thể xác định thêm Option `only` hoặc `except`.

# After Filter

Tương tự, ta đăng ký một After Filter thông qua method `after_action` (và các Options `only`, `except`), các Filters này sẽ được chạy sau khi thực thi thành công một Action gắn với nó.

Đặc điểm của các Filters này là có thể tương tác với Response trả về từ Action, và trước khi gửi về Client.

Chú ý là After Filters chỉ được chạy khi mà Action thực thi thành công mà không có lỗi

# Around Filter

Tương tự, ta đăng ký một Around Filter thông qua method `around_action` (và các Options `only`, `except`), các Filters này dùng để bao lấy một Action, các đoạn mã của nó sẽ có phần chạy trước và và có phần chạy sau một Action gắn với nó.

Around Filters sử dụng method `yield` để chỉ định vị trí thực thi của Action trong đoạn mã của nó, hoặc bản thân nó có thể tự trả về Response mà không thông qua Action.

# Hooks into Filters

Ta có thể sử dụng `prepend_before_action`, `append_before_action`,... để tạo các Filters được gọi trước/sau toàn bộ các Filters đã định nghĩa.