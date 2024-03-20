---
title: Ruby on Rails
draft: false
date: 2024-03-04
tags:
  - ror
  - toc
---

Rails xây dựng ứng dụng web theo mô hình MVC với hai nguyên tắc:

- **DRY**: Don't Repeat Yourself
- **Convention over Configuration**: Lập trình theo những quy ước, mọi thứ sẽ vận hành mà không cần cấu hình

# TOC

Giống như các kiến trúc MVC khác, ứng dụng web trên Rails cũng bao gồm Model-View-Controller

Hãy bắt đầu với Model:

- Để sinh các Database Tables, ta sinh các [migrations](ror/model/migration.md)
- Rails sử dụng thư viện **[Active Record](ror/model/active_record.md)** như một ORM.
- Các chức năng [validations](ror/model/validation.md), [associations](ror/model/association.md) và _[Callbacks](ror/model/callback.md)_ cũng định nghĩa ngay trong Model Class.
- Trên Model Class, ta có thể thực hiện các [truy vấn, tạo các **scopes**,...](ror/model/query.md)
- Ta cũng thường sử dụng [Transaction](ror/model/transaction.md)

Về Controller:

- Trước hết, cần hiểu cơ chế [routing](ror/controller/routing.md), sau đó là [parameters](ror/controller/parameter.md)
- Cũng cần hiểu về [session, cookie và **flash**](ror/controller/session_cookie_flash.md)
- Một số chức năng trong các Actions có thể đưa vào _[Filters](ror/controller/filter.md)_

Với View:

- Hiểu về [cách Actions trong Controller trả về một View](ror/view/action_view.md)
- Hiểu về cách nhúng Ruby code trong View - [ERB](ror/other/odds_ends.md#erb)
- Tổ chức Views thông qua việc chia nhỏ thành [Layouts, Templates và Partials](ror/view/layout_template_partial.md)
- Các chức năng chia sẻ trong View có thể nhóm vào Module gọi là _[Helpers](ror/view/helper.md)_

Ngoài ra:

- Tích hợp việc gửi email cho ứng dụng với [Action Mailer](ror/other/action-mailer.md)
- Cách thức tổ chức tài nguyên trong Rails 7 với [Asset Pipeline](ror/other/asset_pipeline.md)
- Thư viện hỗ trợ xử lý với JavaScript - [Hotwire - Turbo Rails](ror/other/hotwire.md)
- Kiến trúc ứng dụng với [Rack, Puma, Engine...](ror/other/rack_puma.md)

Một số chức năng khác cũng nên biết:

- Quản lý tệp gửi từ người dùng với [Active Storage](ror/other/active_storage.md)
- Quản lý và tạo các Background Job với [Active Job - Sidekiq, Whenever](ror/other/active_job.md)
- Thêm nữa: [Concern](ror/other/concern.md), [I18n](ror/other/odds_ends.md#i18n), [XSS & CSRF](ror/other/odds_ends.md), [debugging](ror/other/debug.md)

Một số [Gem](tags/gem) và chức năng nâng cao:

- Tích hợp chức năng đa ngôn ngữ và cultural với [Gem I18n](ror/gem/i18n.md)
- Để phân trang cho việc hiển thị danh sách, có thể sử dụng [Gem Pagy](ror/gem/pagy.md)
- [Gem Ransack](ror/gem/ransack.md) giúp dễ dàng chức năng tìm kiếm và sắp xếp cho ứng dụng
- Hãy thử tạo [cơ chế xác thực - Authentication cơ bản](ror/other/authentication.md), sau đó triển khai lại với [Gem Devise](ror/gem/devise.md)
- Về phân quyền - Authorization, có thể sử dụng [Gem CanCanCan](ror/gem/cancancan.md)

Về [Unit Testing](tags/unit-testing)

- Hãy bắt đầu với những khái niệm cơ bản và cơ chế [tích hợp sẵn trong Rails](ror/unit-test/built_in.md)
- Một Framework được sử dụng phổ biến cho việc Testing là [RSpec](ror/unit-test/rspec.md) với _[Matchers](ror/unit-test/matcher.md)_, _[Mocks và Stubs](ror/unit-test/mock_stub.md)_
- Sử dụng [Gem FactoryBot](ror/unit-test/factory_bot.md) để sinh dữ liệu cho quá trình testing.

Về [API](tags/api)

- Ta có thể tạo các Controllers tương tự như với ứng dụng Web, song thay vì trả về View và Redirect, ta trả về một JSON Response.
- Hoặc có thể sử dụng [Gem Grape](ror/gem/grape.md)
