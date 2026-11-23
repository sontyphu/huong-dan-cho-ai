# Gắn đăng nhập Clerk vào website HTML

Tài liệu này dành cho trợ lý AI (Claude Code, Codex, Antigravity hoặc AI khác làm việc được với file trên máy). Người dùng đã gửi kèm đoạn mã Clerk của họ và nhờ bạn gắn cửa đăng nhập vào website trong thư mục đang mở. Người dùng thường không biết kỹ thuật: làm hết phần kỹ thuật, chỉ hỏi khi thật sự cần, báo cáo bằng lời thường.

Đọc nguyên văn toàn bộ tài liệu này trước khi làm. Nếu công cụ đọc web của bạn chỉ trả về bản tóm tắt, hãy tải nguyên file về (ví dụ bằng `curl`) rồi đọc, vì đoạn mã ở Bước 3 phải chép đúng từng ký tự.

Kết quả cần đạt: mở website trên máy qua địa chỉ localhost thì người chưa đăng nhập chỉ thấy khung đăng nhập của Clerk; đăng nhập xong thấy toàn bộ trang, có nút tròn ở góc trên bên phải để đăng xuất.

## Bước 1. Kiểm tra đoạn mã người dùng gửi

1. Đoạn mã đúng có 2 thẻ `<script>`, một thẻ chứa `data-clerk-publishable-key="pk_..."`. Thiếu thẻ đó thì dừng, nhờ người dùng sao chép lại ở Clerk: Configure, API keys, khung Quick copy, chọn JavaScript, bấm nút sao chép.
2. Khoá bắt đầu bằng `pk_test_`: dùng được trên localhost. Khoá bắt đầu bằng `pk_live_`: chỉ chạy trên tên miền đã khai với Clerk; vẫn gắn được nhưng báo trước rằng mở trên localhost sẽ không đăng nhập được.
3. Nếu người dùng gửi nhầm khoá bắt đầu bằng `sk_` (khoá bí mật): không dùng, không ghi vào file nào, khuyên họ vào Clerk tạo khoá bí mật mới và xoá khoá cũ vì khoá đã lộ.

## Bước 2. Tìm website và sao lưu

1. Tìm mọi file `.html` trong thư mục đang mở và các thư mục con. Bỏ qua thư mục thư viện: `node_modules`, `vendor`, `.git`, `dist`, `build`.
2. Nếu thư mục là dự án dựng bằng công cụ (có `package.json` dùng React, Next.js, Vue, Astro...) thì dừng: hướng dẫn này chỉ dành cho website HTML thường. Báo người dùng và gợi ý dùng hướng dẫn Clerk riêng cho công cụ đó.
3. Ghi lại danh sách file sẽ sửa để báo cuối việc. Không cần dừng chờ người dùng duyệt danh sách, họ đã nhờ bạn làm.
4. Sao lưu: chép cả thư mục website sang một thư mục cùng cấp, tên là tên thư mục cũ thêm `-ban-goc` (ví dụ `landing-page-ban-goc`). Đã có thư mục sao lưu thì không chép đè.

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

## Bước 5. Mở website trên máy

1. Không mở file trực tiếp kiểu `file://`, Clerk không chạy được như vậy.
2. Chạy một máy chủ tĩnh tại thư mục website, dùng công cụ có sẵn trên máy, ví dụ `npx --yes http-server -p 8080 -c-1` hoặc `python -m http.server 8080`. Cổng 8080 bận thì đổi cổng khác.
3. Mở thử địa chỉ, chắc chắn trang trả về được. Nếu bạn có công cụ điều khiển trình duyệt, mở trang lên xem khung đăng nhập của Clerk có hiện ra không.

## Bước 6. Báo cho người dùng

Viết bằng lời thường, không dùng từ kỹ thuật, gồm:

1. Đã gắn đăng nhập vào bao nhiêu trang (ví dụ "5 trên 5 trang").
2. Bản gốc đã sao lưu ở thư mục nào.
3. Đường link để mở, ví dụ `http://localhost:8080`, và dặn mở bằng **cửa sổ ẩn danh** (Chrome: Ctrl + Shift + N).
4. Điều họ phải thấy: chưa đăng nhập thì chỉ thấy khung đăng nhập; đăng nhập bằng tài khoản đã tạo trong Clerk (mục Users) thì thấy toàn trang và nút tròn ở góc phải, bấm vào có Sign out để đăng xuất.
5. Gặp lỗi thì làm gì:
   - Trang trắng hoặc báo "Không tải được Clerk": kiểm lại đoạn mã đã dán đủ chưa, hoặc mạng có chặn trang clerk.accounts.dev không.
   - Không đăng nhập được: tài khoản phải được tạo trong Clerk, đúng ứng dụng và đúng chế độ (Development hay Production) với mã đang dùng.

## Giới hạn phải nói thật với người dùng

Cách này khoá giao diện, chưa phải khoá dữ liệu. Người rành máy vẫn đọc được nội dung trong file của trang. Không đặt thông tin mật (số điện thoại khách, giá nội bộ, mật khẩu) lên website chỉ dựa vào lớp đăng nhập này.

## Những việc không làm

- Không đưa website lên mạng, không đẩy lên GitHub, không gửi gì ra ngoài khi người dùng chưa đồng ý.
- Không xoá file nào của người dùng.
- Không ghi khoá bí mật `sk_` vào bất kỳ đâu.
