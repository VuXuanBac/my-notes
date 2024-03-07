---
title: Odds and Ends
draft: false
date: 2024-03-04
tags:
  - ror
  - odds-and-ends
---

## ERB

- ERB (Embedded Ruby) là một template hỗ trợ nhúng code Ruby bên trong, nhằm tạo nội dung động cho trang web.
- Nhúng Ruby code bằng cú pháp:
    - `<%= ... %>`: Nội dung bên trong được thực thi và kết quả trả về được chuyển về String và gán lại vào tệp.
    - `<% ... %>`: Nội dung được thực thi nhưng kết quả trả về không được gán lại.

## YAML

YAML (YAML Ain's Markup Language) là một module có sẵn trong thư viện chuẩn của Rails.
Nó được dùng để định nghĩa các cấu hình cho DB, Test Data, Locales,...

Một số đặc điểm:
- Phân cấp dựa trên indentation
- Hướng đến mức thân thiện người dùng

## `count, size, length`

- `length`: Lấy số lượng từ danh sách records đã load vào memory. VD: `User.all`
- `count`: Lấy số lượng bằng truy vấn trực tiếp vào DB. Có thể xác định thêm Filters
- `size`: Kết hợp cả hai: Tự động truy vấn nếu chưa load danh sách records vào memory (`count`), và đọc trực tiếp nếu đã load (`length`)

## TailwindCSS

Cấu hình
- Cấu hình tự động thêm TailwindCSS khi tạo project: `rails new <project> --css tailwind`
- Cấu hình thêm thủ công trên project đã có:
    - `./bin/bundle add tailwindcss-rails`
    - `./bin/rails tailwindcss:install`     [Sinh _./config/tailwind.config.js_ và một số cấu hình]
    - Cập nhật đường dẫn tới các Templates sử dụng Tailwind. VD:

    ```css
    /** _./config/tailwind.config.js_ */
    /** @type {import('tailwindcss').Config} */
    module.exports = {
      content: [
        './public/*.html',
        './app/helpers/**/*.rb',
        './app/javascript/**/*.js',
        './app/views/**/*',
      ],
      theme: {
        extend: {},
      },
      plugins: [],
    }
    ```

Chú ý: TailwindCSS tự động tải **esbuild** như một Bundler

# XSS

XSS là kỹ thuật tấn công chèn mã độc vào mã nguồn ứng dụng (thường là JS), thường gây ra do Server không kiểm tra dữ liệu người dùng nhập vào. 
   - VD: Kẻ tấn công nhúng đoạn script khi nhập bình luận cho bài viết, Server không kiểm tra mà lưu lại đoạn script này dẫn đến phản hồi từ Server cho các người dùng trong tương lai sẽ chứa đoạn mã script không mong muốn.
   - Rails phòng tránh bằng cách tự động **Escape dữ liệu nhúng trong HTML trả về cho người dùng**. VD: `<script>` => `&lt;script&gt;`. 
   - Ngoài ra, Rails cũng hỗ trợ method `csp_meta_tag` giúp tạo một `meta` Tag chỉ định dùng **Content Security Policy** cho trang web, một trong những chính sách là chỉ thực thi script từ cùng nguồn hoặc từ các whitelist, cấm các inline script

# CSRF

CSRF là kỹ thuật tấn công đánh lừa người dùng thực thi một hành động không mong muốn trên một ứng dụng mà người dùng đã xác thực (đăng nhập) trước đó, thường qua Cookie. 
  - VD: Kẻ tấn công tạo một trang web chứa một đường dẫn thực hiện một yêu cầu tới địa chỉ ứng dụng khác, khi người dùng nhấn vào (hoặc có thể thực hiện ngay khi người dùng truy cập vào trang) thì yêu cầu cũng như Cookie của người dùng cho ứng dụng đó cũng sẽ được gửi.
  - Rails phòng tránh bằng cách sinh CSRF Token, nhúng nó vào HTML `meta` Tag qua `<%= csrf_meta_tags %>` (AJAX) hoặc thêm `protect_from_forgery with: :exception` (Form) vào trong ApplicationController. Khi đó, Token sẽ gửi kèm các requests và kiểm tra so khớp tại Server. Như vậy, kẻ tấn công cần đoán đúng Token.
