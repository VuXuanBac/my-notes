---
  title: Deploy your rails app to render.com
  draft: false
  date: 2024-03-06
  tags: 
    - deploy
    - ror
    - til
---

> 🦝 Sau khi đã có một ứng dụng chạy ổn định, tại sao không thử chạy nó trên nền tảng web thực sự ? 😉

🎈[Render](https://render.com/) cung cấp nền tảng để deploy web app, từ **static sites** đến **dynamic web apps** và hỗ trợ nhiều **frameworks** khác nhau như RoR (Ruby), Express (Node.js), Django (Python),.... Ngoài ra, Render cũng cung cấp các services khác như: Background workers, Cron jobs, Data stores (PostgreSQL, Redis).

Đặc biệt, Render cũng cung cấp các gói miễn phí để thử nghiệm một ứng dụng nhỏ.

Mình đã thử nghiệm chạy một ứng dụng web trên Render với một DB server và một RoR web app.

#### 🐘 PostgreSQL

Render hỗ trợ tốt nhất với PostgreSQL. Việc tạo một PostgreSQL instance tương đối dễ dàng, như [hướng dẫn](https://docs.render.com/databases)

Render DB cung cấp hai cách thức kết nối là

- **Internal URL**: Kết nối từ các services chạy trong cùng phạm vi địa lý với DB Server.
- **External URL**: Kết nối từ vị trí địa lý bất kỳ.

Rõ ràng, nếu kết nối là Internal thì thời gian trễ truy cập càng nhỏ.

Ở đây, ứng dụng web sẽ chạy cùng server với DB. Ta sẽ lưu lại địa chỉ Internal URL của DB vừa tạo.

> [!note]
> Gói Render Free sẽ xóa DB instance sau 90 ngày.

#### 🍅 RoR Web App

Ta sẽ tạo một RoR app sử dụng PostgreSQL.

```cmd
rails new <appname> --database=postgresql
```

Khi đã code và đẩy lên Github Repository, ta có thể tạo một Web service instance trên Render trỏ đến repository đó.

Chú ý một số cấu hình sau:

- Thêm hai biến môi trường:
  - **RAILS_MASTER_KEY**: Copy từ **_config/master.key_**.
  - **DATABASE_URL**: Internal URL của PostgreSQL DB
- Thêm lệnh **`bundle exec rails db:migrate`** vào Build Command của Web services.
  - Build Commands lúc này bao gồm:
    - `bundle install`
    - `bundle exec rails assets:precompile`
    - `bundle exec rails assets:clean`
    - `bundle exec rails db:migrate`
  - Thực tế, lệnh này nên gọi trong **Pre-deploy Command**, nhưng bản Render Free không cho phép cấu hình này.
