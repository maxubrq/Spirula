# Spirula — Typesetting Engine · Chốt thiết kế & Tài nguyên

> Tài liệu tổng hợp từ quá trình thảo luận. Phần I là những gì đã **chốt**
> (kèm những chỗ cố ý **hoãn** và những chỗ phải **giữ cửa**). Phần II là
> danh sách tài nguyên, mỗi mục gắn với đúng quyết định đã ra — không phải
> thư mục liệt kê cho đủ.

---

## Tên dự án

- **Tên thật:** **Spirula** — package/repo/CLI: `spirula`.
- **Codename:** **φ / ∅** — một âm ("phi"), hai ký hiệu. Đọc là φ (tỉ lệ vàng,
  cái *nên có mặt* — sự hài hòa), viết là ∅ (rỗng, cái *nên vắng mặt* — ma sát,
  công cụ tự xóa mình). *Present the φ, absent the ∅.*

**Vì sao Spirula:** loài mực biển sâu có vỏ trong xoắn thành đường **xoắn ốc
logarit** — cùng họ hình học với φ (codename). Nhỏ, khiêm tốn, sống tầng sâu,
chỉ nổi lên khi cần — cộng hưởng với tinh thần ∅. Vỏ xoắn = **logo có sẵn**,
gói cả ba lớp: con vật + φ + dòng chảy của text.

**Đã kiểm tra namespace (còn trống sạch, tính đến 2026-07):**

| Registry | Trạng thái |
|----------|-----------|
| crates.io | 🟢 trống (404) |
| npm | 🟢 trống (404) |
| PyPI | 🟢 trống (404) |
| GitHub | 🟢 không repo nào tên đúng `spirula` (chỉ vài `spirulae` không liên quan) |

*Ghi chú lịch sử lựa chọn:* các ứng viên trước bị loại vì va chạm namespace —
`argonaut` (đã có trên crates.io/npm/PyPI + repo cùng tên đang active),
`diatom` (là một ngôn ngữ lập trình viết bằng Rust — va chạm chí mạng với
domain), `kelp` (kín mọi registry). `Zer0phi` bị loại vì `0`-giả phản tín hiệu
craft, phát âm mơ hồ, và cố đóng gói codename thành tên-thật (hai vai khác nhau).

---

## Phần I — Chốt thiết kế

### 0. Nguyên lý xương sống

**Single source, many projections.**

Khi một biểu diễn bị đòi phục vụ nhiều chủ nhân mâu thuẫn (tác giả gõ tay,
máy biên dịch, `git diff`, mắt người đọc), đừng thỏa hiệp trong một biểu
diễn. Đẻ ra nhiều biểu diễn chuyên trách, **buộc đúng một cái làm nguồn sự
thật**, các cái còn lại là *hàm phái sinh một chiều* của nó.

Đây là điểm phân biệt cốt lõi với LaTeX: LaTeX bắt *một* chuỗi text gánh tất
cả — nên nó mù, nó trộn nội dung với trình bày, nó đau. Mọi quyết định bên
dưới đều là hệ quả của nguyên lý này.

### 1. Định vị chiến lược (qua khung SFIM)

- **T1 (Threshold) = cái bánh xe đã tốt → KHÔNG làm lại.** Shaping chữ,
  đọc/subset font, ghi PDF, ngắt dòng không vỡ. Wire thư viện vào, đi tiếp.
  Đổ công vào đây là lãng phí thuần túy.
- **Nhưng: rất nhiều T1 của LaTeX đang *gãy*** (lỗi khó hiểu, biên dịch chậm,
  cú pháp khó học, cài đặt hàng GB). Đây là khe hở để chen vào — và là chỗ
  Typst đã thắng.
- **T2 (Proportional) = chỗ cạnh tranh:** tốc độ biên dịch (nên dẫn đầu — đây
  là điểm đau lớn nhất của LaTeX), độ phủ font/script, template.
- **T3 (Delight):** biến T1-gãy-của-đối-thủ thành bất ngờ — lỗi báo tử tế,
  chạy ngay trên web, live preview. Nhớ: T3 decay xuống T1 khi bị copy.
- **T4 (Symbolic):** khoa học dùng LaTeX **vì bản sắc bộ lạc**, không phải vì
  công thức đẹp. "Tôi viết bằng LaTeX" là tuyên bố danh tính. Look nhận diện
  được, khiến người ta tự hào khoe ra = T4.
- **T5 (Transformative):** nằm ở **khả năng lập trình tài liệu** + hạ tầng
  plain-text (diff/version/tái lập). Với khoa học, T5 này là **bất khả đảo** —
  cả ngành đã dựng pipeline sản xuất tri thức trên nền này.
- **T6 (Paradigmatic):** nhận ra, đừng thiết kế. Cơ hội T6 của ta: xóa bỏ tiền
  đề "phải chọn giữa markup và WYSIWYG".

> **Kết luận chiến lược:** dồn toàn bộ tài năng vào **layout engine + thiết kế
> ngôn ngữ + trải nghiệm biên tập**. Mọi thứ khác là T1 — wire vào rồi quên.

### 2. Kiến trúc ba tầng

Ba tầng về **khái niệm**, nhưng đường nứt của diff chỉ có **hai** (ngữ nghĩa
vs hình thức), và **cấu trúc đứng cùng phe nội dung**:

| Tầng | Nội dung | Nằm ở đâu | Vào diff nào |
|------|----------|-----------|--------------|
| Nội dung + cấu trúc + inline-style cơ bản | prose, heading, nhấn mạnh mang nghĩa; table/image/diagram (component, có default theo template) | **file tài liệu** | diff "nội dung" |
| Component rich media | mỗi cái là object riêng, style default theo template | **file tài liệu** (khai báo) | diff "nội dung" |
| Layout / trình bày thuần | căn lề, khoảng cách, màu, kích thước | **file layout** tách riêng, import kiểu CSS | diff "trình bày" |

**Vì sao đúng:** sửa cấu trúc *là hành vi mang nghĩa* (thêm section, đổ dữ
liệu vào bảng) → phải nằm trong diff nội dung. Chỉ **hình thức thuần** mới
externalize sang file layout. Số tầng khái niệm (3) không cần khớp số file (2).

**Phần thưởng đạt được:** sửa một câu + nudge một cái bảng → hai file đổi,
**mỗi diff kể đúng một câu chuyện**. Hai ý định không còn bị hàn dính vào một
hunk như `\vspace` của LaTeX. Diff khớp *đơn vị suy nghĩ* của tác giả, không
khớp đơn vị lưu trữ.

### 3. Ràng buộc cứng (KHÓA — không mở lại)

1. **Plain-text là nguồn sự thật DUY NHẤT và ĐẦY ĐỦ.** Mọi GUI/preview chỉ là
   *ống nhòm hai chiều* nhìn vào text đó, không bao giờ là định dạng lưu trữ
   song song. Không được có nguồn sự thật thứ hai.
2. **Text phải là thứ con người tác giả được bằng tay.** Ống nhòm ghi ngược
   phải sinh ra đúng loại text một người sẽ tự gõ — sạch, đặt đúng chỗ, không
   khối auto-gen "do not edit".
3. **Diff phải khớp ý định**, tách bạch: `git diff` trả lời riêng rẽ *"nội
   dung đổi gì?"* và *"vẻ ngoài đổi gì?"* mà không câu nào làm nhiễu câu kia.
4. **Ranh giới inline:** mang nghĩa thì cho inline (bold/italic/code/link);
   ngoại hình thuần (màu, cỡ chữ, khoảng cách) **cấm** inline. Đây là đường
   `<em>` vs `<i>` của web.
5. **Editor gánh nửa nghịch lý mà format không gánh nổi:** render nội tuyến
   *cả trình bày lẫn nội dung-bị-mã-hóa* (bảng, mermaid, toán nở ra tại chỗ),
   xóa nhu cầu split-screen mà Typst phải chịu.
6. **Format thiết kế *cho* editor ngay từ móng** — cú pháp phải để editor ghi
   ngược ý-định-trình-bày vào đúng tầng một cách sạch và không mơ hồ.

### 4. Cố ý HOÃN (chưa làm — chờ người dùng thật đòi)

Theo test: *nếu chỉ là code trong renderer → hoãn được; nếu in dấu vào cú
pháp con người gõ → phải giữ cửa.*

- **Cascade / multi-level scoping.** Bây giờ: **một luật mặc định + id cho
  ngoại lệ**, hết. Cái đau cascade là đau của *quy mô* — không tới cho tới khi
  có hàng nghìn người dùng, và tới lúc đó mới có thông tin thật để giải đúng.
  Cascade là **cơ chế renderer**, nhét vào lúc nào cũng được.
- **Structural diff (so cây thay vì so dòng).** Là phiên bản triệt để nhất của
  "diff khớp ý định" — nhưng kiến trúc ba-file đã cho ta điều đó *rẻ hơn*.
  Diff là công cụ đứng ngoài, thay lúc nào cũng được. Ghi vào lề "nếu tách-file
  không đủ" (dự là sẽ đủ rất lâu).
- **Round-trip tinh vi + suy-ra-ý-định (luật hay instance khi bấm "center").**
  Bài toán ngược không có nghiệm duy nhất — để sau, giải bằng *gu* khi có dữ
  liệu dùng thật.

### 5. Phải GIỮ CỬA từ đầu (rẻ bây giờ, đắt kinh hoàng nếu vá sau)

Vì đây là **format**, không phải renderer — nó có tính vĩnh viễn (đổi cú pháp
sau khi có người dùng ≈ tự sát, xem 40 năm LaTeX):

1. **Ranh giới cái-gì-được-inline** (đừng lỡ cho màu-chữ vào inline).
2. **Sự tồn tại của đường nứt document-file / layout-file** (gộp thì dễ, tách
   sau thì phá vỡ mọi file đã viết).
3. **File layout trỏ về document được *bằng cách nào đó*** (đừng thiết kế cách
   trỏ cho tinh vi — chỉ cần đừng làm nó *bất khả* trỏ). Editor tự đúc handle,
   con người không gõ id bằng tay → rò rỉ vào diff nội dung bị bịt.

### 6. Ý tưởng thêm đã ghim: file output-ở-giữa

- Một file **generated, read-only, một chiều** — sinh từ source, biểu diễn
  đã-giải-mã (bảng, mermaid → text tường minh) để **xem diff nội dung mà không
  phải render trong đầu**.
- **Không phải nguồn sự thật thứ hai** vì read-only + một chiều = nó là *hàm
  của* source, không phải đối thủ. Con quỷ hai-nguồn không thức dậy.
- **Giới hạn thật:** chỉ giải mù cho vật *có dạng text tường minh hơn dạng khai
  báo* (bảng, cây, tô-pô đơn giản). Vật mà *mọi* dạng text đều mù (bố cục không
  gian thật) → chỉ editor-render-nội-tuyến cứu được. Hai cơ chế **bổ sung**
  nhau, không thay nhau.

### 7. Những chỗ "có gu hay lộn xộn" — quyết định sản phẩm, không có nghiệm toán

Đây là nơi sản phẩm thắng bằng gu hoặc chết bằng sự tùy tiện:

- Đường biên chính xác **nội dung / cấu trúc / trình bày**.
- Vùng xám "structure có default trình bày" kiểu `*nghiêng*` (`<em>` vs `<i>`).
- Phổ **nhỏ-thì-render-nội-tuyến / lớn-thì-bung-khung-riêng** — theo *kích
  thước* hay theo *bản chất vật*?
- (Khi tới lúc) mặc định **luật vs instance**, và **biểu diễn thứ tự ưu tiên
  cho con người ĐỌC được**, không suy ra ngầm từ hình dạng selector.

---

## Phần II — Tài nguyên

Ký hiệu: 🆓 = miễn phí/đọc online · 📖 = sách · 📄 = paper/spec · 💻 = mã nguồn.

### A. Nền tảng triết lý & lịch sử — *hiểu vì sao paradigm này tồn tại*

- 📖 **Donald Knuth — *The TeXbook*** & bộ *Computers & Typesetting* (5 tập).
  Nguồn gốc mô hình "boxes and glue", luật spacing toán (Appendix G), thuật
  toán gạch nối Liang (Appendix H). Đây là kinh thánh của cả lĩnh vực. *Volume
  B: TeX: The Program* cho thấy engine thật viết ra sao.
- 📖 **Donald Knuth — *Digital Typography*** (CSLI, 1999). Tuyển tập essay,
  dễ tiêu hơn TeXbook, cho bối cảnh "tại sao".
- 📖 **Leslie Lamport — *LaTeX: A Document Preparation System***. Đọc để hiểu
  *giấc mơ tách nội dung khỏi trình bày* (tác giả khai báo `\section`, class lo
  hình thức) — và tự rút ra vì sao nó **rò rỉ** (`\vspace`, hack tay). Chính
  chỗ rò rỉ đó là vết thương ta đang chữa.
- 🆓 **Lịch sử từ "markup":** tra "markup language etymology / manuscript
  markup / compositor typesetting". Markup nguyên thủy là ghi chú **biên tập
  viên viết cho thợ sắp chữ** — ngôn ngữ markup máy tính đảo ngược vai đó, bắt
  tác giả tự làm việc của compositor. Cái editor bạn định xây = tái lập
  compositor, lần này là một lớp phần mềm.

### B. Layout engine — TRÁI TIM, chỗ đổ tài năng

- 📄🆓 **Knuth & Plass (1981), "Breaking Paragraphs into Lines"**, *Software:
  Practice and Experience*, 11(11):1119–1184. Thuật toán ngắt dòng tối ưu toàn
  cục bằng quy hoạch động — nền tảng của mọi thứ đẹp. Bản PDF free:
  `https://gwern.net/doc/design/typography/tex/1981-knuth.pdf`
  - 💻 Cài đặt để đọc code: `robertknight/tex-linebreak` (JS),
    `baskerville/paragraph-breaker` (Rust).
- 📄🆓 **Michael Plass (1981), luận án "Optimal Pagination Techniques"** —
  ngắt *trang* (khác ngắt dòng, khó hơn). Và **"Knuth–Plass Revisited"** (ACM
  DocEng 2015/2016, Frank Mittelbach) cho hướng ngắt trang tối ưu hiện đại.
- 📄🆓 **Unicode UAX #14 — Line Breaking Algorithm**:
  `https://www.unicode.org/reports/tr14/` — *chỗ nào trong chuỗi được phép
  ngắt*. Bắt buộc.
- 📄🆓 **Unicode UAX #9 — Bidirectional Algorithm** (`/tr9/`) và **UAX #29 —
  Text Segmentation** (`/tr29/`, grapheme/word/sentence). Bắt buộc nếu muốn
  vượt tiếng Anh — mà bạn viết tiếng Việt, nên cần.
- 📄🆓 **Frank Liang (1983), "Word Hy-phen-a-tion by Com-pu-ter"**, PhD thesis
  Stanford — thuật toán gạch nối theo pattern (dùng trong TeX tới nay).

### C. Text shaping & font — CÁI BÁNH XE, tuyệt đối đừng làm lại

- 💻🆓 **HarfBuzz** (`https://harfbuzz.github.io`) — shaping engine chuẩn công
  nghiệp. Đây là thứ bạn *dùng*, không phải viết lại.
- 💻🆓 **FreeType** (`https://freetype.org`) — rasterize + đọc font.
- **Stack Rust (nếu chọn Rust — khuyến nghị, xem mục E):**
  - 💻 **cosmic-text** (`github.com/pop-os/cosmic-text`) — gộp cả stack
    (fontdb + shaping + swash + layout/bidi) vào một crate. *Điểm khởi đầu
    nhanh nhất.* (bản gần đây shaping qua `harfrust`.)
  - 💻 **rustybuzz** (`github.com/harfbuzz/rustybuzz`) — port HarfBuzz sang
    Rust thuần, khớp HarfBuzz v10.x, pass gần hết test suite. Track record tốt.
  - 💻 **harfrust** — mới hơn, gần HarfBuzz hơn; cân nhắc cho dự án mới.
  - 💻 **swash** — outline + rasterize, hỗ trợ ligature + color emoji (COLRv1).
  - 💻 **ttf-parser**, **fontdb** — parse & tìm font.
  - ⚠️ Lưu ý từ hệ sinh thái: edge case chữ phức tạp (Arabic ligature động,
    Indic) chưa bằng bản C lâu đời — kiểm chứng trước nếu cần script phức tạp.

### D. Dàn công thức toán — engine con có luật riêng

- 📄🆓 **OpenType MATH table spec** (Microsoft Typography) — bảng chuẩn hóa
  metric toán trong font (thay dần luật Appendix G thủ công của TeX).
- 📄🆓 **MathML Core** (`https://www.w3.org/TR/mathml-core/`) — mô hình cây
  của biểu thức toán, cách trình duyệt render. Hữu ích cho mô hình dữ liệu.
- Tham chiếu cài đặt: cách Typst xử lý toán (`crates/typst-layout`, phần math)
  và dự án **MathJax/KaTeX** (luật spacing đọc được trong code).

### E. Thiết kế ngôn ngữ, parser & biên dịch incremental

- 📖🆓 **Robert Nystrom — *Crafting Interpreters***
  (`https://craftinginterpreters.com`, đọc free online). Lexer → parser → AST
  → tree-walk & bytecode. Nền tảng cho tầng frontend của bạn (T5 "mô tả quy
  luật"). Bắt đầu từ đây.
- 💻🆓 **Typst — mã nguồn + tài liệu kiến trúc.** *Tài nguyên học quan trọng
  nhất trong cả danh sách.* Đọc kỹ:
  - Kiến trúc:
    `https://github.com/typst/typst/blob/main/docs/dev/architecture.md`
    — 4 pha: **parsing → evaluation → layout → export**, tách bạch *đánh giá
    source* khỏi *layout vật lý*. Đúng mô hình bạn cần.
  - `crates/typst` (ngôn ngữ + thư viện), `crates/typst-eval` (interpreter),
    `crates/typst-layout` (layout engine), `crates/typst-pdf` (exporter),
    `crates/typst-ide` (IDE/editor support).
  - Mô hình **set rules vs show rules** — cách Typst tách cấu hình khỏi nội
    dung; nghiên cứu kỹ vì nó gần với đường nứt content/presentation của bạn.
  - "content tree" — biểu diễn *có cấu trúc, không phụ thuộc thứ tự*, tốt hơn
    markup thô để xử lý tiếp. Đây là ý tưởng cốt lõi để mượn.
- 💻🆓 **comemo** (`github.com/typst/comemo`) — framework memoization cho
  **incremental compilation** (biên dịch lại nhanh). Đây là bí quyết tốc độ
  (T2) của Typst.
- 💻🆓 **Tree-sitter** (`https://tree-sitter.github.io`) — **parsing
  incremental** cho editor: cây cú pháp cập nhật khi gõ từng phím. *Cực kỳ liên
  quan* với "format thiết kế cho editor từ móng" (ràng buộc #6).
- 📖 **SICP** hoặc **PLAI (Programming Languages: Application and
  Interpretation, 🆓 online)** nếu muốn nền lý thuyết ngôn ngữ sâu hơn.

### F. Mô hình tài liệu & tách nội dung / trình bày

- 💻🆓 **Pandoc** (`https://pandoc.org`) — nghiên cứu **document AST** của nó
  (`Text.Pandoc.Definition`): một mô hình tài liệu trung gian mà *nhiều* cú
  pháp map vào và *nhiều* output sinh ra từ đó. Đây chính là "single source,
  many projections" đã có sẵn — học cách họ mô hình hóa Block vs Inline (khớp
  ranh giới inline/block của bạn).
- 💻🆓 **Djot** (`https://djot.net`, John MacFarlane — tác giả Pandoc &
  CommonMark). Markdown "thế hệ sau", cú pháp chặt hơn, ít nhập nhằng hơn.
  Đọc để thấy *một markup nhẹ được thiết kế lại cho đàng hoàng* trông thế nào.
- 📄🆓 **CommonMark spec** (`https://spec.commonmark.org`) — bài học về việc
  *đặc tả một markup nhẹ cho chặt chẽ* (Markdown gốc mơ hồ tới mức đau).
- 📄🆓 **WHATWG HTML — semantic elements**, đặc biệt tranh luận **`<em>` (ý
  định/nhấn mạnh) vs `<i>` (hình thức/nghiêng)**. Đây *đúng* là ranh giới xám
  ở ràng buộc #4 và mục 7 — web đã cãi 20 năm, đọc để không cãi lại từ đầu.
- 💻🆓 **Scribble** (Racket, `https://docs.racket-lang.org/scribble/`) —
  **tài liệu-là-chương-trình** làm nghiêm túc: prose trộn code trong một ngôn
  ngữ. Tham chiếu tốt cho tinh thần T5 (lập trình được như *siêu năng lực
  opt-in*).
- 📄 **DocBook** & **DITA** — mô hình *structured content* công nghiệp (tách
  triệt để nội dung/trình bày). Nặng nề, nhưng cho thấy đầu cực đoan của phổ.

### G. Cascade & styling — HỌC TỪ NỖI ĐAU CSS (dù đang hoãn cascade)

Đọc để **giữ cửa đúng cách** và không lặp lại `!important`:

- 📄🆓 **CSS-Tricks — "Cascade Layers Guide"**
  (`https://css-tricks.com/css-cascade-layers/`). Bài học vàng: cascade gốc
  *"dựa vào heuristic thay vì cho quyền điều khiển trực tiếp"* và *"trộn lẫn
  việc chọn phần tử với việc định thứ tự ưu tiên"*. **Đây chính xác là cái sai
  bạn phải tránh** — ưu tiên phải HIỆN & ĐỌC ĐƯỢC, không tính ngầm từ hình dạng
  selector.
- 📄🆓 **MDN — `@layer`** và **MDN — Specificity**
  (`developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@layer`). Cách
  làm thứ tự ưu tiên *tường minh* (khai báo `@layer a, b, c` — thứ tự = ưu
  tiên, bỏ qua specificity). Mô hình cho "danh sách có thứ tự con người liếc
  qua là biết ai thắng".
- 🆓 **Miriam Suzanne** (tác giả spec cascade layers, đề xuất 2019) — tìm bài
  viết/talk của cô ấy về *vì sao* cascade cần layers.
- 🆓 **ITCSS / BEM** — các phương pháp *kỷ luật* cascade *trước khi* có
  `@layer`. Cho thấy người ta đau thế nào khi ưu tiên không tường minh.

### H. Editor: round-trip, direct manipulation, immediate feedback

- 🎥🆓 **Bret Victor — "Inventing on Principle" (2012)**. Nguyên lý:
  *"Creators need an immediate connection to what they create."* Nền tảng
  triết lý cho ống nhòm hai chiều & render nội tuyến. Transcript:
  `https://jamesclear.com/great-speeches/inventing-on-principle-by-bret-victor`
  (video tìm trên `worrydream.com` — trang của chính Victor).
- 🆓 **Bret Victor — các essay khác** ("Magic Ink", "Media for Thinking the
  Unthinkable") và dự án **Dynamicland** — cho tầm nhìn xa hơn về direct
  manipulation.
- 📄🆓 **Projectional / structured editing** — tra "projectional editor",
  **JetBrains MPS**, và tranh luận "structured editing vs text editing". Đây là
  chính xác câu hỏi "editor xen giữa người và text thô" của bạn; đọc cả *phê
  bình* projectional editing (vì sao nhiều cái thất bại) để tránh bẫy.
- 📄🆓 **Language Server Protocol** (`https://microsoft.github.io/language-server-protocol/`)
  — chuẩn để tách "trí tuệ ngôn ngữ" khỏi editor. Nếu format-cho-editor, LSP là
  cách bạn cấp tính năng cho nhiều editor mà không viết lại.

### I. Đồng bộ hai chiều & diff (cho tương lai — round-trip, cộng tác)

- 📄🆓 **CRDTs — Yjs** (`github.com/yjs/yjs`) & **Automerge**
  (`https://automerge.org`). Nếu muốn cộng tác thời gian thực + hợp nhất chỉnh
  sửa mà giữ plain-text làm nguồn. Đọc paper gốc CRDT nếu cần nền lý thuyết.
- 📄🆓 **Structural / tree diff** — tra "tree diff algorithm", "GumTree AST
  diff", "difftastic" (`github.com/Wilfred/difftastic`, diff hiểu cú pháp). Đây
  là hiện thực của "diff khớp ý định" nếu tách-file không đủ (đang HOÃN).
- 🆓 **Pro Git — chương "Git Internals"** (`git-scm.com/book`) — hiểu Git lưu
  & diff thế nào, để format của bạn *thân thiện với Git* (dòng ổn định, diff
  gọn).

### J. Backend / output — T1, wire vào rồi quên

- 📄 **PDF spec (ISO 32000)** — chỉ cần đủ để dùng thư viện; đừng ghi PDF từ
  số 0. Rust: `pdf-writer` (chính Typst dùng). SVG output đơn giản hơn, tốt để
  prototype.

### K. Gu typography — để có "look" đáng khoe (T4)

- 📖 **Robert Bringhurst — *The Elements of Typographic Style***. Kinh điển về
  gu chữ. Cái "chữ ký thị giác" khiến tác giả tự hào (T4) đến từ đây.
- 🆓 **Practical Typography** (Matthew Butterick, `practicaltypography.com`,
  đọc free) — gu typography ứng dụng, ngắn gọn, thực dụng.

---

### Lộ trình đề xuất (thứ tự đọc/làm)

1. **Prototype hiểu pipeline** (vài tuần): đọc *Crafting Interpreters* (E) +
   kiến trúc Typst (E). Làm markup tối giản, dùng `cosmic-text` (C) đo/shape
   chữ, ngắt dòng tham lam trước, render ra SVG (J). Mục tiêu: chạy hết pipeline
   một lần, hiểu từng tầng.
2. **Nâng layout** (B): thay ngắt dòng tham lam bằng Knuth–Plass. Đây là lúc
   chất lượng bản in bắt đầu vượt WYSIWYG.
3. **Thiết kế ngôn ngữ dưới ràng buộc round-trip** (E + F): chốt cú pháp với
   ba tầng + ranh giới inline/block (giữ cửa #4, #5). Tham chiếu Pandoc AST,
   Djot, set/show rules của Typst.
4. **Editor-là-ống-nhòm** (H): render nội tuyến, immediate feedback. Đây là
   con hào phòng thủ thật.
5. Cascade, structural diff, round-trip tinh vi: **chờ người dùng thật** (mục
   I.4, I.5 phần I).
