# OTP 2FA Web

Ứng dụng web Svelte/Vite để tạo mã TOTP 2FA, đọc QR code và tạo file nội dung từ danh sách tags.

## Tính Năng

- Nhập nội dung `otpauth://` hoặc secret Base32 để tạo mã OTP 6 số.
- Mã OTP tự cập nhật theo chu kỳ 30 giây.
- Thanh thời gian giảm dần theo thời gian còn lại của mã hiện tại.
- Upload ảnh QR code để tự đọc nội dung.
- Quét QR code bằng camera thiết bị.
- Hiển thị key trích xuất và hỗ trợ copy key.
- Copy mã OTP hiện tại.
- Thêm, xóa và kéo thả sắp xếp tags.
- Preview nội dung tags theo định dạng `tag1|tag2|tag3`.
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
4. Xem key trích xuất và mã OTP hiện tại.
5. Với phần tạo file, nhập tag rồi bấm `Add` hoặc nhấn `Enter`.
6. Kéo thả tags để đổi thứ tự, nội dung preview sẽ tự cập nhật.
7. Nhập tên file ở ô `Filename` và bấm `Download`.

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
