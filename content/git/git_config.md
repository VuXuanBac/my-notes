---
title: Git Config
draft: false
date: 2024-03-05
tags:
  - git
  - system
  - tool
---

[Documentation](https://git-scm.com/docs/git-config)

Để cấu hình cho Git, ta có thể sử dụng lệnh `git config` hoặc lưu các cấu hình vào một tệp. Các cấu hình xuất hiện sau sẽ ghi đè các cấu hình định nghĩa trước.

Các cấu hình cho Git có thể:
- Hệ thống: 
	- **`git config --system`**
	- `/etc/gitconfig`
- Toàn cục (người dùng):
	- **`git config --global`**
	- `~/.gitconfig`
- Cục bộ (project, repository):
	- **`git config --local`**
	- `$GIT_DIR/config`
- Worktree:
	- **`git config --worktree`**
	- `$GIT_DIR/config.worktree`
- Hoặc sử dụng option **`-c`** để thêm cấu hình ở mức lệnh.

Hiển thị danh sách các configurations áp dụng trong phạm vi hiện tại: `git config --list --show-origin`

# Some configurations

Các key cấu hình được tạo thành từ `<section>.<key>`

- `core.editor`: Editor sử dụng cho một số trường hợp như commit message.
- `alias.<name>`: Tạo alias cho các lệnh trong git.
- `commit.template`: Đường dẫn đến tệp dùng làm template cho commit message

# Config from Files
[Reference](https://git-scm.com/docs/git-config#_syntax)

- Comment bắt đầu bằng `#` hoặc `;`
- Cú pháp cho config section: `[section "subsection"]`
- `include` và `includeIf` là hai sections đặc biệt dùng để nhúng một config file khác vào file hiện tại.
	- Sử dụng key `path` để chỉ định đường dẫn tới config file cần nhúng.
	- Một số điều kiện cho `includeIf`:
		- `gitdir:<pattern>`: True nếu project hiện tại match với pattern.
		- `hasconfig:remote.*.url:<pattern>`: True nếu remote url của project match với pattern

```
# ~/.gitconfig

[core]
        editor = "nano -w"
[alias]
        amend = "commit --amend"
        summary = "log --oneline --all --graph --abbrev-commit"
[user]
        email = personal.email@gmail.com
        name = my-name
[includeIf "gitdir:~/b/"]
		path = ~/.gitconfig-personal
[includeIf "gitdir:~/w/"]
		path = ~/.gitconfig-work

# ~/.gitconfig-personal
[user]
        email = personal.email@gmail.com
        name = my-name

# ~/.gitconfig-work
[user]
        email = work.email@domain.com
        name = staff-name
```

