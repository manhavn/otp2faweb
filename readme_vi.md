# OTP 2FA Web

Ứng dụng web Svelte/Vite để tạo mã OTP 2FA, đọc QR code và tạo file nội dung từ danh sách tags.

## Tính Năng

- Nhập nội dung `otpauth://` hoặc secret Base32 để tạo mã OTP 6 số.
- Hỗ trợ TOTP theo thời gian và HOTP theo bộ đếm.
- Với TOTP, mã OTP tự cập nhật theo chu kỳ 30 giây.
- Với HOTP, có thể nhập bộ đếm hoặc tăng nhanh bộ đếm bằng nút `+1`.
- Thanh thời gian giảm dần theo thời gian còn lại của mã hiện tại khi dùng TOTP.
- Upload ảnh QR code để tự đọc nội dung.
- Quét QR code bằng camera thiết bị.
- Hiển thị key trích xuất và hỗ trợ copy key.
- Hiển thị nhà cung cấp, tài khoản và hỗ trợ copy từng trường nếu URI có metadata.
- Copy mã OTP hiện tại.
- Thêm, xóa và kéo thả sắp xếp tags.
- Preview nội dung tags theo định dạng `tag1|tag2|tag3`.
- Tạo nhanh tags và filename từ nhà cung cấp, tài khoản, key trích xuất bằng nút `Generate content`.
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
6. Xem key trích xuất, nhà cung cấp, tài khoản và mã OTP hiện tại.
7. Với phần tạo file, nhập tag rồi bấm `Add` hoặc nhấn `Enter`.
8. Hoặc bấm `Generate content` để tự tạo tags theo thứ tự `Nhà cung cấp|Tài khoản|Key trích xuất`.
9. Kéo thả tags để đổi thứ tự, nội dung preview sẽ tự cập nhật.
10. Nhập tên file ở ô `Filename` và bấm `Download`.

## Generate Content

Nút `Generate content` tự tạo nội dung tải xuống từ dữ liệu OTP đang có:

- Tag 1: `Nhà cung cấp` nếu có.
- Tag 2: `Tài khoản` nếu có.
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
