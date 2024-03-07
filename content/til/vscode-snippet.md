---
  title: Visual Studio Code Snippets
  draft: false
  date: 2024-03-07
  tags: 
    - vscode
    - tool
    - til
  description: How to create a snippets in Visual Studio Code
  aliases:
---

> 🐺 Nếu quá chán với việc lặp lại các nội dung khi viết trên Visual Studio Code, hãy thử Snippets? 😙

🎈 **Snippets** là khuôn mẫu cho một đoạn text tự động được chèn vào editor khi có lệnh.

> [!note]
> Ta có thể sử dụng lệnh `Ctrl + Space` để chèn một Snippet vào Visual Studio Code.
> Ngoài ra, nếu cấu hình `"editor.tabCompletion": "on"` thì có thể chèn Snippets bằng `Tab`

🔎 Trong VSCode, một Snippets được định nghĩa với các thông tin:
- **name**: Tên hiển thị
- **description**: Mô tả hiển thị cùng với Tooltip
- **prefix**: Chỉ thị kích hoạt Snippets (kết hợp với lệnh)
    - Đoạn text hoặc mảng các text.
- **body**: Nội dung (chính là template cần chèn) sẽ thay thế vị trí **prefix**
    - Đoạn text hoặc mảng các text (mỗi phần tử tương ứng một dòng)

🚀 Để tạo một snippet

- Nhấn `Ctrl + Shift + P` để mở Command Palette
- Tìm kiếm **Configure User Snippets**
- Lựa chọn một ngôn ngữ, định dạng tệp hoặc chọn Global Snippets để cấu hình chung.

> [!note]
> Với Global Snippets, ta cần chỉ định trường **`scope`** với giá trị là tên các ngôn ngữ mà Snippets này tác động đến, hoặc bỏ trống nếu muốn áp dụng cho mọi định dạng tệp.
> VD: "scope": "javascript,typescript"

- Định nghĩa các trường cho Snippet
- Đặc biệt, với **body**, ta có thể sử dụng một số kỹ thuật sau để phản ánh nội dung cần chèn:

⬇ **Tabstop**: Sử dụng các chỉ số dạng `$1, $2, ${1:...},...` để chỉ thứ tự di chuyển Tab giữa các vị trí cần người dùng nhập nội dung trong Template. 

> [!info]
> Nếu có nhiều vị trí có chỉ số Tabstop giống nhau, nội dung người dùng nhập sẽ tự động chèn cho tất cả các vị trí đó.

💬**Placeholder**: Nội dung mặc định cho phần nội dung người dùng cần điền. VD: `${1:foo}`

🎲**Choice**: Nội dung cần điền được gợi ý sẵn một số giá trị, người dùng chỉ cần chọn. VD: `${1|one,two,three|}` 

🍪**Variable**: Một số giá trị đặc biệt được đặt tên. [Xem thêm](https://code.visualstudio.com/docs/editor/userdefinedsnippets#_variables).

🔀**Transform**: Thay đổi giá trị điền (thường là từ **variables**) sử dụng Regex.

#### 📖Ví dụ

Để phục vụ viết notes, mình tạo hai Snippets cho Markdown

🔖Thứ nhất, một snippets để tự động chèn các highlight notes (Github hỗ trợ hiển thị)

```json
"Github Highlight Note": {
  "prefix": ">",
  "description": "Github syntax for markdown  highlight note",
  "body": [
  	"> [!${1|note,important,tip,info,caution,warning|}]",
  	"> ${2:Content of the note}"
  ]
}
```

Kết quả khi nhập `>` và nhấn `Ctrl + Space`, nội dung chèn là:

```
> [!note] # Một Dropdown hiển thị danh sách Options
> Content of the note
```

🔖Thứ hai, một snippets với cú pháp phức tạp hơn giúp mình tự động viết Markdown Frontmatter cho Quartz note.

```
"Quartz Front Matter": {
  "prefix": "blog",
  "description": "Markdown Frontmatter that the  page's properties",
  "body": [
    "---",
    "  title:  ${1:${TM_FILENAME_BASE/([^-\\.]+)([-\\.]|$)/${1:/capitalize}${2:+ }/g}}",
    "  draft: ${2|true,false|}",
    "  date: $CURRENT_YEAR-$CURRENT_MONTH-{3:$CURRENT_DATE}",
    "  tags:",
    "    - ${4:Tags for this note}",
    "  description: ${5:Description of the page use for link previews}",
    "  aliases: ${6: Other names for this note. This is a list of strings}",
    "---",
  ]
}
```

Kết quả khi chèn là:

```
---
  title: Vscode snippet     # Tên tệp đã được xử lý
  draft: true
  date: 2024-03-07
  tags:
    - Tags for this note
  description: Description of the page used for link previews
  aliases:  Other names for this note. This is a list of strings
---
```

✏ **Giải thích một chút**
- Giá trị cho phần **title** mình sử dụng **variable** `TM_FILENAME_BASE` là tên tệp hiện tại (không bao gồm extension)
- Sau đó mình sử dụng Regex để **transform**, cụ thể là thay thế các ký tự `-` hoặc `.` trong tên tệp bằng ký tự `space`, và Capitalize các từ trong tên tệp.
- Phần **date** mình đang để tự động lấy ngày hiện tại, và chỉ tự động thay đổi (tạo Tabstop) phần ngày
- Cũng có thể cấu hình thêm để phần **tags** tự động lấy giá trị trong Clipboard (biến `$CLIPBOARD`) hoặc `$CURRENT_WORD` để chèn. 

#### 🪝Tham khảo

- [Document](https://code.visualstudio.com/docs/editor/userdefinedsnippets)
