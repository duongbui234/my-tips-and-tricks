# Ubuntu CLI & Clipboard Tips & Tricks

Tài liệu tối ưu hóa quy trình làm việc với Hình ảnh, Clipboard và AI Coding Agent trên Ubuntu Linux.

---

## 1. Cài đặt bộ công cụ cốt lõi

Mở terminal tiêu chuẩn của Ubuntu và chạy lệnh sau:

```bash
sudo apt-get update
sudo apt-get install gnome-screenshot xclip -y
```

---

## 2. Hướng dẫn gán phím tắt tùy chỉnh (Ctrl + Shift + A)

Thực hiện gán phím tắt để chụp ảnh màn hình và tự động ép định dạng dữ liệu ảnh thô (Raw Image Data) vào clipboard:

### Bước 2.1: Mở cài đặt bàn phím của Ubuntu

1. Nhấn phím `Super` (phím Windows) để mở thanh tìm kiếm.
2. Gõ tìm **Settings** (Cài đặt) và nhấn Enter.
3. Ở thanh thực đơn bên trái, chọn mục **Keyboard** (Bàn phím).

### Bước 2.2: Truy cập khu vực phím tắt tùy chỉnh

1. Cuộn xuống dưới cùng tìm đến phần _Keyboard Shortcuts_.
2. Nhấp chuột vào dòng **View and Customize Shortcuts** (Xem và tùy chỉnh phím tắt).
3. Một hộp thoại hiện lên, cuộn xuống dưới cùng và chọn **Custom Shortcuts** (Phím tắt tùy chỉnh).

### Bước 2.3: Thêm lệnh và gán phím tắt

1. Bấm vào nút dấu cộng **`+`** (hoặc nút _Add Shortcut_).
2. Thiết lập chính xác các thông tin sau:

- **Name**: `Chụp ảnh màn hình vào Clipboard`
- **Command**:

```bash
bash -c "gnome-screenshot -a -f /tmp/screenshot.png && xclip -selection clipboard -t image/png -i /tmp/screenshot.png"
```

- **Shortcut**: Nhấn tổ hợp phím `Ctrl + Shift + A`

---

## 3. Mục đích repo

Dự án này là một repo để lưu những tip & trick hay dùng trên Ubuntu và macOS.
