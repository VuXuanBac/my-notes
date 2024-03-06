---
title: Active Storage
draft: false
tags:
  - ror
  - extension
---

Khi một Model có một số thuộc tính liên quan đến tệp tin, ta có thể sử dụng bộ công cụ **Active Storage**, với các chức năng
- **Upload File** tới một số dịch vụ bộ nhớ đám mây Amazon S3, Google Cloud Storage, Microsoft Azure Storage
    - Trên môi trường development sẽ sử dụng Local Disk.
- Gắn các tệp cho Active Record Object.
- Xử lý metadata.

Một số GEM Requirements khi sử dụng chức năng đặc biệt của Active Storage
- `image_processing`, `libvips`, `ImageMagick`: Image Analysis + Transformation.
- `ffmpeg`: Video Preview + Analysis
- `poppler`, `muPDF`: PDF Preview

# Setup

```cmd
bin/rails active_storage:install
bin/rails db:migrate
```

Hai lệnh này sẽ cấu hình ban đầu cho Active Storage: Tạo 3 bảng (tên bắt đầu bằng **active_storage_**)
- **_blobs**: Chứa dữ liệu về Filename, Content Type,...
- **_attachments**: Join Table giữa Model và Blob.
- **_variant_records**: Lưu các Variants (các tệp với kích thước khác nhau)

Ta có thể cấu hình trong tệp **config/storage.yml** về vị trí lưu trữ dữ liệu của Active Storage - tương ứng với các dịch vụ. VD: Khai báo 3 dịch vụ, trong đó `local` và `test` lưu trữ trong bộ nhớ.

```yaml
local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

amazon:
  service: S3
  access_key_id: ""
  secret_access_key: ""
  bucket: ""
  region: "" # e.g. 'us-east-1'
```

Sau đó, trong các tệp cấu hình môi trường (Development, Production), ta chỉ định dịch vụ sử dụng. VD:

```ruby
# config/environments/development.rb
config.active_storage.service = :local

# config/environments/production.rb
config.active_storage.service = :amazon

# config/environments/test.rb
config.active_storage.service = :test
```

# Record Handling

## Attach Files to Record

Ta có thể tạo liên kết giữa Model và Blob, tức là xác định một tệp làm thuộc tính thông qua

- `has_one_attached`: Association 1-1.
- `has_many_attached`: Association 1-1.

Sau đó, ta có thể gán một tệp cho thuộc tính qua instance method `.attach`. Tệp này có thể:
- Upload từ Client (qua `<input type="file">`) lên Server qua HTTP Request.
- IO từ Server
- URL

Với trường hợp đầu, ta có thể truyền trực tiếp `params[:<attachment>]` cho đối số

Với hai trường hợp còn lại, ta truyền đối số với các Options sau
- `io`: [Required] Sử dụng `File.open` để lấy nội dung.
- `filename`: [Required] Tên tệp
- `content_type`: [Optional] MIME Type
    - Nếu không xác định, tự suy ra từ nội dung
    - Mặc định là **application/octet-stream**

## Remove File to Record

Để loại bỏ một tệp ra khỏi bản ghi, ta sử dụng
- `purge`
- `purge_later`: Loại bỏ trong nền, với hỗ trợ của Active Job

Hai methods này sẽ xóa bản ghi trong Blob và xóa tệp trên dịch vụ.

## Validation

