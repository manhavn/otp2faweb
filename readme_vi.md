# OTP 2FA Web

Ứng dụng web Svelte/Vite để tạo mã OTP 2FA, đọc QR code, tạo nội dung `otpauth://` có thể chỉnh sửa và xuất file tag/QR để tải xuống.

## Tính Năng

- Nhập nội dung `otpauth://` hoặc secret Base32 để tạo mã OTP 6 số.
- Hỗ trợ TOTP theo thời gian và HOTP theo bộ đếm.
- Với TOTP, mã OTP tự cập nhật theo chu kỳ 30 giây.
- Với HOTP, có thể nhập bộ đếm hoặc tăng nhanh bộ đếm bằng nút `+1`.
- Thanh thời gian giảm dần theo thời gian còn lại của mã hiện tại khi dùng TOTP.
- Upload ảnh QR code để tự đọc nội dung, bao gồm ảnh QR SVG.
- Quét QR code bằng camera thiết bị.
- Hiển thị key trích xuất và hỗ trợ copy key.
- Cho phép nhập hoặc sửa nhà cung cấp và tài khoản ngay cả khi nội dung nguồn không có metadata.
- Tạo nội dung `otpauth://totp/...` hoặc `otpauth://hotp/...` mới từ loại OTP đang chọn, key trích xuất, nhà cung cấp, tài khoản và HOTP counter.
- Copy nội dung `otpauth://` mới.
- Tải nội dung `otpauth://` mới dưới dạng file `.txt`, tên file dựa trên nhà cung cấp và tài khoản.
- Xuất nội dung `otpauth://` mới thành QR code `.svg` hoặc `.png`, tên file dựa trên nhà cung cấp và tài khoản.
- Copy mã OTP hiện tại.
- Thêm, xóa và kéo thả sắp xếp tags.
- Preview nội dung tags theo định dạng `tag1|tag2|tag3`.
- Tạo nhanh tags và filename từ nhà cung cấp có thể sửa, tài khoản có thể sửa và key trích xuất bằng nút `Generate content`.
- Nhập filename và tải xuống file chứa nội dung preview.
- Tự chuẩn hóa filename, ví dụ `otp|GoogleLogin|user@gmail.com.txt` thành `otp-GoogleLogin-user-gmail.com.txt`.

## Cài Đặt

```bash
npm install
```

## Chạy Development

```bash
npm run dev
```

Sau đó mở URL Vite hiển thị trong terminal, thường là:

```text
http://localhost:5173
```

## Kiểm Tra

```bash
npm run check
```

## Build Production

```bash
npm run build
```

## Preview Build

```bash
npm run preview
```

## Cách Dùng

1. Dán secret Base32 hoặc URI `otpauth://` vào ô `Nội dung`.
2. Hoặc chọn ảnh QR code bằng nút chọn ảnh.
3. Hoặc bấm `Camera` để quét QR trực tiếp từ thiết bị.
4. Chọn loại OTP: `Theo thời gian` cho TOTP hoặc `Theo bộ đếm` cho HOTP.
5. Nếu dùng HOTP, nhập giá trị `Bộ đếm` hoặc bấm `+1` để tăng counter.
6. Xem key trích xuất và mã OTP hiện tại.
7. Sửa `Nhà cung cấp` và `Tài khoản` trong phần `Nội dung otpauth mới` nếu cần.
8. Copy nội dung `otpauth://` mới, tải TXT hoặc xuất QR SVG/PNG.
9. Với phần tạo file, nhập tag rồi bấm `Add` hoặc nhấn `Enter`.
10. Hoặc bấm `Generate content` để tự tạo tags theo thứ tự `Nhà cung cấp|Tài khoản|Key trích xuất` bằng các giá trị đang sửa trong phần `Nội dung otpauth mới`.
11. Kéo thả tags để đổi thứ tự, nội dung preview sẽ tự cập nhật.
12. Nhập tên file ở ô `Filename` và bấm `Download`.

## Nội Dung OTPAuth Mới

Phần `Nội dung otpauth mới` tạo URI OTPAuth mới từ trạng thái hiện tại của form:

- Loại OTP: `totp` hoặc `hotp`.
- Secret: key Base32 đang được trích xuất.
- Label: `Nhà cung cấp` và `Tài khoản` có thể chỉnh sửa.
- Query params: `secret`, `digits`, thêm `period` cho TOTP hoặc `counter` cho HOTP.

Bạn có thể xuất URI mới này thành:

- Text: `[nhà-cung-cấp-tài-khoản].txt`.
- QR SVG: `[nhà-cung-cấp-tài-khoản].svg`.
- QR PNG: `[nhà-cung-cấp-tài-khoản].png`.

Nếu nhà cung cấp và tài khoản đều trống, app dùng fallback `otp-auth` cho tên file export.

## Generate Content

Nút `Generate content` tự tạo nội dung tag tải xuống từ dữ liệu OTPAuth có thể chỉnh sửa:

- Tag 1: `Nhà cung cấp` đang sửa nếu có.
- Tag 2: `Tài khoản` đang sửa nếu có.
- Tag 3: `Key trích xuất`.

Filename được tạo theo mẫu:

```text
otp-tags[-nhà-cung-cấp][-tài-khoản].txt
```

Ví dụ:

```text
otp-tags-Google-user-gmail.com.txt
```

## Lưu Ý

- Tất cả xử lý OTP, QR và tạo file chạy trên trình duyệt.
- Secret/key không được gửi ra server.
- Tính năng camera cần chạy trên `localhost` hoặc HTTPS để trình duyệt cho phép truy cập camera.
- Web Crypto API cần môi trường browser hiện đại.

## Công Nghệ

- Svelte 5
- Vite
- TypeScript
- Web Crypto API
- jsQR
- qrcode-generator
