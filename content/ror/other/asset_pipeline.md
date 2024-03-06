---
title: Asset Pipeline
draft: false
tags:
  - ror
  - intro
---

Khi ứng dụng cần sử dụng các tài nguyên như JS, CSS, Images, Fonts,... ta cần phải đặt chúng ở một nơi nào đó, nơi đó phải vừa dễ dàng truy cập từ ứng dụng, vừa phục vụ Client hiệu quả.

# Overview

Asset Pipeline trong Rails phục vụ mục đích đó, với một số chức năng:
- **Asset Organization**: Hỗ trợ tổ chức các tài nguyên trong cấu trúc thư mục ứng dụng. 
  - **app/assets** Các tài nguyên của riêng ứng dụng
  - **lib/assets** Các tài nguyên nằm trong thư viện (dùng cho các ứng dụng liên quan)
  - **vendor/assets** Các tài nguyên từ bên thứ ba
- **Pre-Compilation**: Biên dịch lại các tệp để đảm bảo phù hợp với Browser. -> Có thể dùng nhiều ngôn ngữ khác nhau: SASS, JSX,...
  - _Preprocessor Engines_: Các công cụ tiền xử lý cho các tài nguyên, trước khi được kết hợp theo Manifest File. VD: Các tệp .scss được xử lý bởi bộ Sass để chuyển về .css
- **Concatenation**: Nối nhiều tệp thành một tệp để trả về cho Client -> Giảm số Requests
  - _Manifest File_: Mô tả cách thức kết hợp các tài nguyên nằm ở các vị trí khác nhau thành một tệp tài nguyên.
- **Minification**: Nén tệp tin (.min.) để giảm kích thước -> Giảm tải
- **Fingerprinting**: Hỗ trợ quản lý thay đổi để phục vụ Caching.
  - Mặc định, các Browsers, CDNs, ISPs,... triển khai sẵn cơ chế Caching. Fingerprint giúp cho biết khi nào có thể dùng cache, khi nào nên tạo request để reload
  - Fingerprint thường là nhúng mã băm nội dung của tệp vào tên tệp.
  - Tức là khi tệp thay đổi, tên tệp cũng thay đổi, từ đó tránh Caching không mong muốn.

Có một số Gem cung cấp một phần hoặc toàn bộ các chức năng này:
- **Sprockets**: Cung cấp toàn bộ, và đã có từ lâu
- **Propshaft**: Dự kiến là Gem mặc định của Rails 8
- **ImportMap**: Hỗ trợ quản lý các JS Packages.
- **CSSBundling**: CSS Bundler cho các thư viện như TailwindCSS, Bootstrap,...
- **JSBundling**: JS Bundler với các công cụ như esbuild, rollup.js, Webpack,...

Với Rails 7, Asset Pipeline được tạo từ các Gem `importmaps-rails`, `sprockets` và `sprockets-rails`.

**Tuy nhiên, các Gem này không hỗ trợ chức năng Transpiling, tức là không thể xử lý các tệp dạng SASS, Babel, TypeScript, JSX, Tailwind,...**

# Sprockets

Cách đơn giản nhất để cung cấp tài nguyên cho ứng dụng là để nó vào thư mục _public/_, mọi thứ trong đó có thể truy cập tự do giống như duyệt thư mục. Tuy nhiên, việc tổ chức như vậy là không hiệu quả, ta cần một cách tự động nhúng các tài nguyên ở các vị trí khác nhau trong thư mục projects vào trong _public/_ khi cần.

**Sprockets** là công cụ dùng để xử lý các tài nguyên cung cấp bởi ứng dụng, đóng gói, xử lý và cuối cùng là **thêm chúng vào thư mục _public/assets/_**, với các chức năng Fingerprinting, Minification, SourceMap,...

Tất cả các tài nguyên được quản lý bởi Sprockets luôn phải truy xuất thông qua **đường dẫn tương đối** so với **Load Path** (đường dẫn của thư mục chứa tài nguyên cấu hình trong Sprockets). VD: `Load Path = [app/assets/javascripts,...]` thì tệp `app/assets/javascripts/models/project.js` được truy xuất qua `models/project.js`

Sprockets sử dụng File Extension để biết được nên sử dụng công cụ nào để xử lý một tệp. Với các tệp có thể được Transpiled hay Compiled bởi Sprockets thì đường dẫn truy xuất cần xác định Extensions kết quả (không phải Extensions của tệp trên ổ đĩa)

Xem thêm:
- [How Sprockets Works](https://github.com/rails/sprockets/blob/main/guides/how_sprockets_works.md)
- [sprockets](https://github.com/rails/sprockets): Ruby Gem
- [sprockets-rails](https://github.com/rails/sprockets-rails): Mở rộng `sprockets` cho Rails framework

## Load Path - Where to Find

Để chỉ thị cho Sprockets biết sự tồn tại của các tài nguyên, ta thêm đường dẫn của các tài nguyên hoặc thư mục chứa chúng vào **Rails.application.config.assets.paths** trong tệp **config/initializers/assets.rb**.

VD: Chỉ thị Sprockets biết có tài nguyên nằm tại _vendor/theme/_
```ruby
# config/initializers/assets.rb
Rails.application.config.assets.paths << Rails.root.join("vendor", "theme")
```

## Directives - Which needs Processing

Các chỉ thị (directives) dùng để hướng dẫn Sprockets các tệp cần được xử lý, xác định sự phụ thuộc giữa các tài nguyên đã được xác định ở Load Path.

Sau quá trình này, các tệp kết quả sẽ được cung cấp công khai trong _public/assets_

### Path in Directives

Đường dẫn xác định trong các chỉ thị có thể là
- Đường dẫn tương đối so với tệp chứa các chỉ thị này. 
    - Luôn bắt đầu bằng `./` hoặc `../`
- Đường dẫn tới tài nguyên đã được thêm vào **Load Path** (đường dẫn Logic)
    - Không bắt đầu bằng `./` hoặc `../`

### Directives

Các chỉ thị luôn được đặt trong phần Comment của các tệp và phần comment này phải ở đầu tệp.

- **Các chỉ thị dạng `link...` thể hiện tệp hiện tại phụ thuộc vào các tài nguyên, và do đó các tài nguyên này luôn được compile thành các tệp riêng**.
  - `= link` nạp một tệp xác định
  - `= link_directory` nạp toàn bộ thư mục hiện tại (không bao gồm thư mục con)
    - Ta có thể xác định thêm phần Extension hoặc Content-Type. VD: `link_directory ../my_stylesheets .css` để chỉ lọc các tệp có định dạng `.css` trong `../my_stylesheets`
  - `= link_tree` nạp toàn bộ thư mục hiện tại và thư mục con.
- **Các chỉ thị dạng `require...` dùng để nhúng nội dung của tài nguyên khác vào tệp chứa chỉ thị**
  - `= require` nạp một tệp tài nguyên.
  - `= require_self` nạp nội dung của tệp hiện tại (không bao gồm phần comment - chứa các chỉ thị) tại vị trí của chỉ thị này.
  - `= require_directory` nạp toàn bộ các tệp có cùng định dạng với tệp hiện tại trong một thư mục xác định (không bao gồm thư mục con của nó)
  - `= require_tree` tương tự `= require_directory` nhưng duyệt cả toàn bộ thư mục con.
- Các chỉ thị cho một tệp cần sử dụng **Đường dẫn Logic + Xác định cả extension**
- Các chỉ thị cho một thư mục cần sử dụng **Đường dẫn Tương đối**

### Index File - Directives for whole Directory

Với các thư mục, ta có thể sử dụng Index File (cũng chứa các chỉ thị) để thông báo Sprockets cách thức xử lý các tệp bên trong. 

VD: Với cấu trúc thư mục

```
my_library
├── alpha.js
├── beta.js
├── jquery.js
```

Ta có thể tạo tệp `my_library/index.js` với nội dung như sau:

```js
//= require ./jquery.js
//= require ./alpha.js
//= require ./beta.js
```

Chú ý: **Ở đây, ta có thể xác định thứ tự xử lý cho Sprockets, thay vì dùng `require_tree .` sẽ xử lý tệp theo thứ tự bảng chữ cái, gây kết quả không mong muốn khi các tệp phụ thuộc nhau**

Với Index File, Sprockets sẽ biên dịch thư mục thành `my_library.js`

### CSS Bundling

Các tệp CSS có thể đóng gói lại trong tệp _app/assets/stylesheets/application.css_. Mặc định nó chứa hai chỉ thị là
- `= require_tree .`: Đóng gói CSS từ trong thư mục hiện tại: _app/assets/stylesheets_
- `= require_self`: Đóng gói CSS từ chính tệp này và nối vào cuối tệp kết quả.

## Asset Helper - Retrieve

Các tài nguyên sau khi được thêm vào Load Path và Compiled, tức là các tài nguyên có sẵn trong **public/assets**, ta có thể truy xuất theo đường dẫn URL: VD: `<img src="/assets/path/to/image.jpg"/>`

Hoặc đơn giản hơn, ta có thể sử dụng các Helpers cung cấp sẵn bởi Rails (`ActionView::Helpers::AssetUrlHelper` và `ActionView::Helpers::AssetTagHelper`) như:

`asset_path` (hoặc `path_to_asset`) mặc định sử dụng `compute_asset_path` (được triển khai bởi Asset Pipeline Gem) để tính toán đường dẫn tới tài nguyên theo **đường dẫn Logic**. Ngoại trừ:
- Đường dẫn bắt đầu bằng `/` luôn là đường dẫn URL.
- Đường dẫn có dạng URL

Một số Options:
- `skip_pipeline`: `true` nếu muốn sử dụng đường dẫn URL
- `extname`: Phần Extension xác định thủ công

Các Helpers khác sẽ triển khai `asset_path` và đường dẫn URL kết quả có Prefix như sau
- `audio_path`, `audio_tag`: Prefix `/audios`
- `video_path`, `video_tag`: Prefix `/videos`
- `font_path`: Prefix `/fonts`
- `image_path`, `image_tag`: Prefix `/assets`
- `javascript_path`, `javascript_include_tag`: Prefix `/assets`
- `stylesheet_path`, `stylesheet_link_tag`: Prefix `/assets`

## Precompile

Các tài nguyên tự động được xử lý (compile) trong môi trường Development.

Trong môi trường Production (khi deploy), các tài nguyên đó nên được compile sẵn, để chỉ thị Sprockets biết tệp nào cần được precompile, ta sử dụng lệnh:

```ruby
# config/initializers/assets.rb
Rails.application.config.assets.precompile << %w(application.js application.css)
```

hoặc từ phiên bản Sprockets 4, ta có thể sử dụng cú pháp thuận tiện hơn với Manifest File:

Manifest file **app/assets/config/manifest.js** là tệp cấu hình mặc định, xác định các tài nguyên cần cung cấp công khai của ứng dụng, do đó trong tệp này ta xác định các `link` directives.

Nội dung mặc định của nó như sau:

```js
// app/assets/config/manifest.js

//= link_tree ../images
//= link_directory ../stylesheets .css
//= link_tree ../../javascript .js
//= link_tree ../../../vendor/javascript .js
```
- Precompile toàn bộ các tệp trong thư mục _app/assets/images_ và các thư mục con của nó.
- Precompile toàn bộ các tệp `.css` trong thư mục _app/assets/stylesheets_ (không bao gồm trong thư mục con).
- Precompile toàn bộ các tệp `.js` trong thư mục _app/javascript_ và _vendor/javascript_ (cả thư mục con)

Chú ý: Gem `sprockets-rails` sẽ đưa ra cảnh báo khi một tài nguyên được truy xuất (VD: qua Helpers) mà không được thêm vào Precompile List: Lỗi `Sprockets::Rails::Helper::AssetNotPrecompiled`. Ta có thể tắt tính năng này trong tệp _config/environments/development.rb_

```ruby
config.assets.check_precompiled_asset = false
```

# ImportMap

JS Module có thể nhúng vào JS Module khác sử dụng `import` và theo sau là đường dẫn tương đối (so với tệp hiện tại) hoặc tuyệt đối tới Module.

ImportMap là tiện ích mà Browser triển khai nhằm hỗ trợ khai báo các định danh cho đường dẫn tới Module. Cụ thể, ImportMap có định dạng JSON, chứa các ánh xạ từ định danh cho Module tới đường dẫn trỏ tới nó.

ImportMap được nhúng vào bên trong `<script type="importmap"></script>`

Sau khi khai báo ImportMap cho ứng dụng, thay vì sử dụng đường dẫn, ta có thể sử dụng định danh để `import` JS Module.

**Chú ý, định danh có thể ở dạng đường dẫn**.

**importmaps-rails** là một Gem hỗ trợ tích hợp ImportMap cho Rails.

Cụ thể, Gem này hoạt động như sau:
- Các ánh xạ được khai báo trong _config/importmap.rb_
  - Tự động nạp khi có thay đổi (development), song nếu xóa ánh xạ thì cần chạy lại server.
- Cung cấp Helper `javascript_importmap_tags` để sinh `<script type="importmap"></script>` với nội dung là các ánh xạ khai báo.
- Tự động nhúng JS Entry Point cho ứng dụng (là JS Module chứa các `import` tới các Modules khác)
  - Mặc định là _app/javascript/application.js_
  - Nội dung sinh là: `<script type="module">import "application"</script>`
  - **Chú ý: Định danh "application" được ánh xạ tới _app/javascript/application.js_**
- Tự động nhúng **Es-module-shims**: `<script src="/assets/es-module-shims.min" async="async" data-turbo-track="reload"></script>`
  - Để hỗ trợ một số Browsers (phiên bản cũ) hiểu ImportMaps

Như vậy, khi muốn nhúng một JS Module cho ứng dụng, ta cần:
- B1: Khai báo ánh xạ trong _config/importmap.rb_
- B2: Import nó cho ứng dụng (sử dụng định danh)
    - Trong _app/javascript/application.js_ hoặc các tệp đã được import vào đây.
    - Hoặc trực tiếp trên HTML/ERB: `<%= javascript_import_module_tag <module_identifier> %>`

Chú ý, ta có thể bỏ qua B1, khi đó, trong B2, sử dụng đường dẫn tương đối của Module. Tuy nhiên, cách này có rủi ro vì tài nguyên được sắp xếp khác nhau giữa Development (Project Folder) và Production (_public/assets_)

## Local Modules

Các ánh xạ trong _config/importmap.rb_ được tạo từ các hàm `pin` và `pin_all_from`.

### pin_all_from

Hàm này chỉ thị nạp toàn bộ các Module bên trong một **thư mục** (và các thư mục con)

Cú pháp: `pin_all_from(dir, under: nil, to: nil, preload: false)`

- `dir` xác định **đường dẫn tới thư mục trong cấu trúc Project**, là tương đối so với `Rails.root`.
- `under` xác định **định danh chung** cho toàn bộ các Module trong thư mục
  - Từ đó, ta có thể truy cập đến từng Module theo dạng đường dẫn.
  - _Có thể dùng chung một định danh cho nhiều thư mục_
- `to` xác định **đường dẫn Logic - đường dẫn tới Module sau khi được xử lý bởi Asset Pipeline** (tức là đường dẫn tương đối so với _public/assets_ - Path tương ứng trong `Rails.application.config.assets.paths`)
-  `preload`: Luôn nạp Module này, bất kể có được import hay không.
  - Các Modules này sẽ được nhúng trực tiếp trong HTML dưới `<link rel="modulepreload" href="...">`

Trừ khi `dir` đủ xác định giá trị cho `to` và `under`, nếu không ta cần xác định ít nhất một trong hai Options này.

VD: Ta có thư mục UI Template và để nó trong thư mục
**vendor/theme/
├── css
├── fonts
├── images
├── js
├── scss
└── vendors**

```ruby
# config/initializers/assets.rb
Rails.application.config.assets.paths << Rails.root.join("vendor", "theme")

# config/importmap.rb
....
pin_all_from "vendor/theme/js", under: "@js", to: "js"
pin_all_from "vendor/theme/vendors", under: "@vendor", to: "vendors"
pin_all_from "vendor/theme/vendors/chart.js", under: "@vendor", to: "vendors/chart.js"
pin_all_from "vendor/theme/vendors/progressbar.js", under: "@vendor", to: "vendors/progressbar.js"
```

### pin

Hàm này chỉ thị nạp một Module cụ thể

Cú pháp: `pin(name, to: nil, preload: false)`

- `name` xác định **định danh** cho Module
- `to` xác định **đường dẫn Logic - đường dẫn tới Module sau khi được xử lý bởi Asset Pipeline** (tức là đường dẫn tương đối so với _public/assets_ - Path tương ứng trong `Rails.application.config.assets.paths`)
-  `preload`: Luôn nạp Module này, bất kể có được import hay không.
  - Các Modules này sẽ được nhúng trực tiếp trong HTML dưới `<link rel="modulepreload" href="...">`

Trừ khi `name` đủ xác định giá trị cho `to`, nếu không ta cần xác định giá trị cho Option này.

## CDN Modules

**importmap-rails** thực tế hữu dụng hơn trong việc nhúng các JS Packages từ CDN hoặc Package Repositories như JSPM (mặc định), UNPKG, JsDelivr

Để thêm một Package vào ứng dụng, ta chạy lệnh `./bin/importmap pin [name]`

Lệnh này thực hiện tạo một ánh xạ từ định danh Module `[name]` tới đường dẫn CDN của Module đó (đồng thời cũng tạo các ánh xạ phụ thuộc của Module đó)

Ta sử dụng option `--from` để chỉ định Package Repositories sử dụng. VD: `unpkg`, `jsdelivr` (mặc định là `jspm`)

Ngoài ra, ta có thể thêm option `--download` cho lệnh để chỉ thị việc tải các Modules đó về ứng dụng. Các Modules tải về sẽ được đặt trong thư mục **vendor/javascript** (thư mục mặc định được thêm vào Sprockets Precompile List)

  - VD: `bin/importmap pin react react-dom`
- Từ đó ta có thể sử dụng thư viện bằng lệnh `import` trong `application.js`
  - VD: `import React from "react"`

# JS Management

## JS Bundling

Khi không muốn dùng ImportMaps mà muốn chức năng Bundling, ta có thể sử dụng một số Bundler như: Bun, esbuild, webpack, rollup.js,... ta chỉ định option `--javascript` khi tạo Project

VD: `rails new [name] --javascript=bun` [Bun]

Khi chạy ứng dụng với Bundling, ta cần chạy `bin/dev` để đảm bảo code mới nhất

- Javascript Runtime:
  - Với esbuild, rollup.js hoặc Webpack: NodeJS hoặc Yarn
  - Với Bun: Bun chính là một Runtime + Bundler