# Gắn đăng nhập Clerk vào website HTML

Tài liệu này dành cho trợ lý AI (Claude Code, Codex, Antigravity hoặc AI khác làm việc được với file trên máy). Người dùng đã có website HTML chạy trên tên miền riêng, đã cài Clerk ở chế độ chạy thật (Production) cho tên miền đó, và gửi kèm đoạn mã Clerk của họ. Việc của bạn: gắn cửa đăng nhập vào website trong thư mục đang mở rồi cập nhật website lên mạng. Người dùng thường không biết kỹ thuật: làm hết phần kỹ thuật, chỉ hỏi khi thật sự cần, báo cáo bằng lời thường.

Đọc nguyên văn toàn bộ tài liệu này trước khi làm. Nếu công cụ đọc web của bạn chỉ trả về bản tóm tắt, hãy tải nguyên file về (ví dụ bằng `curl`) rồi đọc, vì đoạn mã ở Bước 3 phải chép đúng từng ký tự.

Kết quả cần đạt: mở tên miền của website thì người chưa đăng nhập chỉ thấy khung đăng nhập của Clerk; đăng nhập xong thấy toàn bộ trang, có nút tròn ở góc trên bên phải để đăng xuất.

## Bước 1. Kiểm tra đoạn mã người dùng gửi

1. Đoạn mã đúng có 2 thẻ `<script>`, một thẻ chứa `data-clerk-publishable-key="pk_..."`. Thiếu thẻ đó thì dừng, nhờ người dùng sao chép lại ở Clerk: Configure, API keys, khung Quick copy, chọn JavaScript, bấm nút sao chép.
2. Khoá phải bắt đầu bằng `pk_live_` (mã chạy thật). Phần sau `pk_live_` là chuỗi base64 của `clerk.<tên miền>$`: giải mã ra để biết tên miền Clerk đã khai, rồi so với tên miền của website (xem file `CNAME` trong thư mục nếu có, hoặc hỏi người dùng). Hai tên miền phải trùng nhau, hoặc website là tên miền con của tên miền đó.
3. Khoá bắt đầu bằng `pk_test_` là mã chạy thử: chỉ chạy trên máy, không đăng nhập được trên tên miền thật. Dừng và nhờ người dùng chuyển Clerk sang Production (nút ghi Development ở thanh trên cùng, chọn Production) rồi sao chép lại mã.
4. Nếu người dùng gửi nhầm khoá bắt đầu bằng `sk_` (khoá bí mật): không dùng, không ghi vào file nào, khuyên họ vào Clerk tạo khoá bí mật mới và xoá khoá cũ vì khoá đã lộ.

## Bước 2. Tìm website và sao lưu

1. Tìm mọi file `.html` trong thư mục đang mở và các thư mục con. Bỏ qua thư mục thư viện: `node_modules`, `vendor`, `.git`, `dist`, `build`.
2. Nếu thư mục là dự án dựng bằng công cụ (có `package.json` dùng React, Next.js, Vue, Astro...) thì dừng: hướng dẫn này chỉ dành cho website HTML thường. Báo người dùng và gợi ý dùng hướng dẫn Clerk riêng cho công cụ đó.
3. Ghi lại danh sách file sẽ sửa để báo cuối việc. Không cần dừng chờ người dùng duyệt danh sách, họ đã nhờ bạn làm.
4. Sao lưu: chép cả thư mục website sang một thư mục **cùng cấp, nằm ngoài** thư mục website, tên là tên thư mục cũ thêm `-ban-goc` (ví dụ `landing-page-ban-goc`). Đã có thư mục sao lưu thì không chép đè. Không đặt bản sao lưu bên trong thư mục website, vì nó sẽ bị đưa lên mạng cùng website.

## Bước 3. Gắn mã vào từng trang

Trong mỗi file `.html`, dán 2 đoạn sau ngay trước dòng `</head>`, đúng thứ tự. Không sửa gì khác trong file.

- **Đoạn 1**: đoạn mã Clerk của người dùng, giữ nguyên từng ký tự.
- **Đoạn 2**: đoạn khoá trang dưới đây, giữ nguyên.

Nếu file đã có `id="clerk-gate"` (từng gắn trước đó) thì thay Đoạn 1 cũ bằng mã mới, không dán chồng thêm một lần nữa.

```html
<style id="clerk-gate">body { visibility:hidden; }</style>
<script>
window.addEventListener('load', async () => {
  const gate = document.getElementById('clerk-gate');
  const wrap = document.createElement('div');
  const box = document.createElement('div');
  wrap.appendChild(box);
  try {
    await Clerk.load({
      ui: { ClerkUI: window.__internal_ClerkUICtor },
      afterSignOutUrl: window.location.origin + window.location.pathname
    });
    if (Clerk.isSignedIn) {
      wrap.style.cssText = 'position:fixed;top:16px;right:16px;z-index:99999';
      document.body.appendChild(wrap);
      Clerk.mountUserButton(box);
    } else {
      Array.from(document.body.children).forEach((el) => (el.style.display = 'none'));
      wrap.style.cssText = 'position:fixed;inset:0;overflow:auto;display:flex;align-items:center;justify-content:center;z-index:99999';
      document.body.appendChild(wrap);
      Clerk.mountSignIn(box, {
        forceRedirectUrl: window.location.origin + window.location.pathname
      });
    }
  } catch (e) {
    Array.from(document.body.children).forEach((el) => (el.style.display = 'none'));
    box.textContent = 'Không tải được Clerk: ' + e.message;
    document.body.appendChild(wrap);
  }
  gate.remove();
});
</script>
```

Ghi file bằng mã hoá UTF-8 để chữ tiếng Việt trong trang không bị lỗi, và giữ nguyên kiểu xuống dòng sẵn có của file.

## Bước 4. Đếm lại

Đếm số file có `id="clerk-gate"`. Con số phải bằng số file đã liệt kê ở Bước 2. Lệch thì sửa cho đủ rồi đếm lại.

## Bước 5. Cập nhật website lên mạng

Người dùng đã nhờ bạn gắn đăng nhập vào website đang chạy, nên cập nhật lên mạng là một phần của việc này.

1. Cập nhật bằng đúng cách website đang được xuất bản:
   - Thư mục là kho git nối với GitHub (GitHub Pages): chỉ `git add` đúng các file `.html` đã sửa, commit với lời ghi "Gắn đăng nhập Clerk", rồi push lên nhánh đang xuất bản.
   - Website xuất bản bằng cách khác (Netlify, Vercel, hosting tải file lên...): làm theo đúng cách đó nếu bạn có công cụ; không có thì hướng dẫn người dùng tải lại thư mục website lên như họ vẫn làm.
   - Không rõ website đang xuất bản bằng cách nào thì hỏi người dùng một câu.
2. Chờ nơi lưu trữ cập nhật xong (GitHub Pages thường mất 1 đến 3 phút), rồi tải trang chủ trên tên miền về kiểm tra có `id="clerk-gate"` chưa. Nếu bạn có công cụ điều khiển trình duyệt, mở tên miền lên xem khung đăng nhập của Clerk có hiện ra không.

## Bước 6. Báo cho người dùng

Viết bằng lời thường, không dùng từ kỹ thuật, gồm:

1. Đã gắn đăng nhập vào bao nhiêu trang (ví dụ "5 trên 5 trang") và đã cập nhật lên mạng.
2. Bản gốc đã sao lưu ở thư mục nào.
3. Tên miền để mở thử, và dặn mở bằng **cửa sổ ẩn danh** (Chrome: Ctrl + Shift + N).
4. Điều họ phải thấy: chưa đăng nhập thì chỉ thấy khung đăng nhập; đăng nhập bằng tài khoản đã tạo trong Clerk (mục Users, chế độ Production) thì thấy toàn trang.
5. Gặp lỗi thì làm gì:
   - Vẫn thấy trang cũ không có khung đăng nhập: chờ thêm vài phút rồi tải lại trang, nơi lưu trữ chưa cập nhật xong.
   - Báo "Không tải được Clerk": tên miền chưa xác minh xong trong Clerk (mục Domains), hoặc mã đang dùng không khớp tên miền.
   - Không đăng nhập được: tài khoản phải được tạo trong Clerk ở chế độ Production, đúng ứng dụng với mã đang dùng.

## Giới hạn phải nói thật với người dùng

Cách này khoá giao diện, chưa phải khoá dữ liệu. Người rành máy vẫn đọc được nội dung trong file của trang. Không đặt thông tin mật (số điện thoại khách, giá nội bộ, mật khẩu) lên website chỉ dựa vào lớp đăng nhập này.

## Những việc không làm

- Không đưa lên mạng file nào ngoài các trang `.html` đã sửa.
- Không xoá file nào của người dùng.
- Không ghi khoá bí mật `sk_` vào bất kỳ đâu.
