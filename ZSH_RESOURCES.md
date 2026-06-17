# Zsh Aliases & Useful Functions

Tổng hợp các phím tắt (aliases) và hàm (functions) hữu ích để tối ưu hóa thao tác trên Terminal (Zsh).

---

## 1. Các phím tắt (Aliases) cơ bản

Thêm các dòng sau vào file `~/.zshrc`:

```bash
# Điều hướng và hệ thống
alias cls="clear"
alias ll="ls -lah"           # list chi tiết kèm hidden file
alias ..="cd .."             # lùi 1 cấp
alias ...="cd ../.."         # lùi 2 cấp
alias reload="source ~/.zshrc"   # reload config nhanh

# Git shortcuts
alias gs="git status"
alias gp="git pull"
alias gc="git commit -m"
```

---

## 2. Các hàm quản lý Port (Port Management)

Các hàm hỗ trợ kiểm tra và giải phóng port đang bị chiếm dụng.

### Hiển thị process đang chiếm port

```bash
showport() {
    lsof -i :"$1"
}
```

### Kill process đang chiếm port

```bash
killport() {
    local pid=$(lsof -ti :"$1")

    if [ -z "$pid" ]; then
        echo "Không tìm thấy process nào sử dụng port $1"
        return 1
    fi

    echo "Killing PID: $pid (port $1)"
    kill -9 $pid
}
```

---

## 3. Cách cài đặt

1. Mở file cấu hình zsh: `nano ~/.zshrc` hoặc `code ~/.zshrc`.
2. Dán nội dung trên vào cuối file.
3. Lưu lại và chạy lệnh: `source ~/.zshrc` (hoặc dùng alias `reload` vừa tạo).
