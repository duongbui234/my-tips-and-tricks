# Docker & System Cleanup Tips

Các lệnh dọn dẹp hệ thống và Docker để giải phóng dung lượng ổ cứng hiệu quả.

---

## 1. Dọn dẹp Docker (Docker Cleanup)

Docker thường chiếm rất nhiều dung lượng sau một thời gian sử dụng do các image, container và volume cũ.

### Xóa mọi thứ không sử dụng (Prune)

Lệnh này sẽ xóa các container đã dừng, network không dùng, image chưa gắn tag (dangling), và build cache.

```bash
docker system prune -a --volumes
```

### Xóa thủ công theo loại

- **Xóa tất cả container đã dừng:**
  ```bash
  docker container prune
  ```
- **Xóa các image không dùng (dangling):**
  ```bash
  docker image prune
  ```
- **Xóa các volume không còn gắn với container nào:**
  ```bash
  docker volume prune
  ```

---

## 2. Dọn dẹp hệ thống Ubuntu/Linux

### Dọn dẹp Cache của Package Manager (APT)

```bash
sudo apt-get autoremove   # Xóa các gói phụ thuộc không còn dùng
sudo apt-get autoclean    # Xóa các file cài đặt cũ trong cache
sudo apt-get clean        # Xóa toàn bộ file lưu tạm trong /var/cache/apt/archives/
```

### Kiểm tra dung lượng thư mục lớn nhất

Tìm 10 thư mục chiếm nhiều dung lượng nhất trong thư mục hiện tại:

```bash
du -ah . | sort -rh | head -n 10
```

---

## 3. Aliases khuyên dùng (Thêm vào .zshrc)

```bash
alias dclean="docker system prune -f"
alias dcleanall="docker system prune -a --volumes -f"
alias sysclean="sudo apt-get autoremove && sudo apt-get autoclean"
```
