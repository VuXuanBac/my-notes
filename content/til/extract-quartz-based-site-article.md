---
  title: Extract Quartz-based sites' articles
  draft: false
  date: 2024-03-06
  tags:
    - quartz
    - tool
    - til
---
#### ⚡ Vấn đề

> 🐶 **Làm thế nào để lấy được thông tin các bài viết trên các website sử dụng Quartz?** 🤔

🎈Đơn giản nhất, ta có thể xem danh sách các bài viết dựa trên Tags. Tuy nhiên, chất lượng của cách này hoàn toàn phụ thuộc vào người viết, hoàn toàn có thể có những bài viết không được gắn tag nào cả.

Sau khi tìm hiểu về mã nguồn của Quartz, bắt đầu từ chức năng Search của nó, với kỳ vọng thấy được một cách thức tổ chức bài viết, mình thu được một số kết quả khá thú vị:

🔎 Trước hết, Quartz sử dụng [**FlexSearch**](https://github.com/nextapps-de/flexsearch), một thư viện JS hỗ trợ tìm kiếm nội dung trên các websites với tốc độ rất ấn tượng. Ngoài ra, nó cũng trả kết quả dựa trên một thuật toán chấm điểm gọi là [**contexture index**](https://github.com/nextapps-de/flexsearch#contextual) - giúp kết quả trả về có mức độ tương thích tốt hơn, phù hợp với ngữ cảnh hơn.

FlexSearch tổ chức dữ liệu theo 3 đối tượng: `Index`, `Worker` và `Document`, trong đó 
- `Index` và `Worker` lưu trữ dữ liệu theo **id-content**
- `Document` lưu trữ dữ liệu theo các trường (**fields**) dưới dạng JSON.
- Ở đây, Quartz sử dụng `Document`.

Bắt đầu đọc code từ việc tìm kiếm tên thư viện, mình thấy tệp *search.inline.ts* là sử dụng thư viện này, ở đây:
- Kết quả tìm kiếm được lưu theo kiểu **`Item`** với 5 trường (line 6-10)
```js
interface Item {
  id: number
  slug: FullSlug
  title: string
  content: string
  tags: string[]
}
```
- Cấu trúc tìm kiếm sử dụng **`FlexSearch.Document`** với 3 trường dùng để tìm kiếm là `content`, `title` và `tags`
```js
let index = new FlexSearch.Document<Item>({
  charset: "latin:extra",
  encode: encoder,
  document: {
    id: "id",
    index: [
      {
        field: "title",
        tokenize: "forward",
      },
      {
        field: "content",
        tokenize: "forward",
      },
      {
        field: "tags",
        tokenize: "forward",
      },
    ],
  },
})
```
- Lần theo biến `index`, thấy có hai hàm `searchAsync` và `addAsync` được gọi.
- Khi `searchAsync`
  - Có thể search theo **tags**, bằng việc prepend `#`
  - Search đồng thời từ cả **title** và **content**, trong đó **title** được ưu tiên hơn (phần code phía sau)
```js
async function onType(e: HTMLElementEventMap["input"]) {
    if (!searchLayout || !index) return
    currentSearchTerm = (e.target as HTMLInputElement).value
    searchLayout.classList.toggle("display-results", currentSearchTerm !== "")
    searchType = currentSearchTerm.startsWith("#") ? "tags" : "basic"

    let searchResults: FlexSearch.SimpleDocumentSearchResultSetUnit[]
    if (searchType === "tags") {
      searchResults = await index.searchAsync({
        query: currentSearchTerm.substring(1),
        limit: numSearchResults,
        index: ["tags"],
      })
    } else if (searchType === "basic") {
      searchResults = await index.searchAsync({
        query: currentSearchTerm,
        limit: numSearchResults,
        index: ["title", "content"],
      })
    }
    //...
}
```
- Khi `addAsync`, dữ liệu `data` được dùng để thêm vào `FlexSearch.Document` object
```js
// Line 447 -> 468
async function fillDocument(data: { [key: FullSlug]: ContentDetails }) {
  // ....
  for (const [slug, fileData] of Object.entries<ContentDetails>(data)) {
    promises.push(
      index.addAsync(id++, { /*...*/ }),
    )
  }

  return await Promise.all(promises)
}

document.addEventListener("nav", async (e: CustomEventMap["nav"]) => {
    const data = await fetchData
    // ....
    await fillDocument(data) // Line 444
})
```
- Tiếp tục tìm kiếm theo `fetchData`, thấy nó được chạy trong một Script khi render pages
```js
// renderPage.tsx
export function pageResources(
  baseDir: FullSlug | RelativeURL,
  staticResources: StaticResources,
): StaticResources {
  const contentIndexPath = joinSegments(baseDir, "static/contentIndex.json")
  const contentIndexScript = `const fetchData = fetch("${contentIndexPath}").then(data => data.json())`
  // ....
}
```
- Ở đây, mọi thứ gần như đã rõ ràng, tệp chứa danh sách các bài viết nằm ở một đường dẫn kiểu **/static/contentIndex.json**

> [!note]
> Mất công lần mò là thế, cuối cùng, mình phát hiện ra trên [Document của Quartz](https://quartz.jzhao.xyz/plugins/ContentIndex) có thông tin này 😑

- Ngoài ra, nếu như website enable sitemap và/hoặc RSS, có thể xem danh sách bài viết qua **sitemap.xml** và **index.xml**.

```js
// quartz.config.ts
const config: QuartzConfig = {
  // ...
  plugins: {
    // ...
    emitters: [
      // ...
      Plugin.ContentIndex({
        enableSiteMap: true,
        enableRSS: true,
      })
    ],
  },
}
```

#### 📬 Tóm lại

> [!important]
> - Bạn có thể duyệt các bài viết trong một Quartz website theo đường dẫn **`https://your-site/static/contentIndex.json`**
> - Nếu website có hỗ trợ RSS, có thể đọc **`https://your-site/index.xml`**
> - Nếu website có hỗ trợ sitemap, có thể đọc **`https://your-site/sitemap.xml`**