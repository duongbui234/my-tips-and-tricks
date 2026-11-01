# Ubuntu — Zsh Developer Terminal

## Tổng quan công cụ

| Công cụ | Vai trò | Khi nào sử dụng |
|---|---|---|
| Zsh | Shell tương tác | Terminal dev hằng ngày |
| Oh My Zsh | Framework quản lý cấu hình và plugin Zsh | Completion, alias, plugin |
| Powerlevel10k / Starship | Prompt hiển thị trạng thái shell/Git | Chọn **một** prompt |
| zsh-autosuggestions | Gợi ý lệnh theo lịch sử | Gõ lệnh nhanh |
| zsh-syntax-highlighting | Tô màu lệnh hợp lệ/không hợp lệ | Phát hiện lỗi cú pháp sớm |
| fzf | Tìm kiếm mờ tương tác | History, file, branch |
| zoxide | Điều hướng thư mục theo tần suất | Chuyển giữa dự án |
| ripgrep (`rg`) | Tìm chuỗi trong code | Tìm symbol/config |
| fd | Tìm file theo tên | Duyệt source tree |
| bat | Xem file có tô màu cú pháp | Đọc config/code |
| eza | Liệt kê file và thư mục | Thay thế `ls` khi cần |
| lazygit | Git TUI | Stage, commit, review |
| git-delta (`delta`) | Renderer cho Git diff | So sánh code |
| tmux | Quản lý nhiều phiên terminal và session | Chạy nhiều tác vụ trong một SSH |
| direnv | Nạp biến môi trường theo thư mục | Thiết lập env dự án, cần cho phép `direnv allow` |
| uv | Python package/project manager | Virtualenv, dependency |
| gh | GitHub CLI | PR, issue, auth |
| btop | Theo dõi CPU, RAM, process | Chẩn đoán hiệu năng |
| jq / yq | Xử lý JSON / YAML | API, Kubernetes manifest |


## Mục tiêu
Terminal phục vụ Git, Docker, SSH, Python, Node.js, Codex/Claude Code và Kubernetes. Dùng Zsh + **một** prompt (Powerlevel10k hoặc Starship), fzf, zoxide, ripgrep, fd, bat, eza, lazygit, delta, tmux, direnv, uv, gh, btop, jq, yq. Nếu Ubuntu đã có Oh My Zsh/Powerlevel10k, giữ nguyên và không cài thêm prompt thứ hai.

## 1. Kiểm tra hệ thống

```bash
lsb_release -a
printf 'Shell: %s\n' "$SHELL"
command -v apt brew zsh git curl
for cmd in fzf zoxide rg fd fdfind bat batcat eza lazygit delta tmux direnv uv gh btop jq yq; do
  command -v "$cmd" || true
done
```

## 2. Sao lưu cấu hình

```bash
stamp=$(date +%Y%m%d-%H%M%S)
[ -f ~/.zshrc ] && cp ~/.zshrc ~/.zshrc.backup.$stamp
[ -f ~/.bashrc ] && cp ~/.bashrc ~/.bashrc.backup.$stamp
```

## 3. Cài gói APT

```bash
sudo apt update
sudo apt install -y zsh git curl unzip build-essential \
  fzf ripgrep fd-find bat tmux direnv btop jq \
  zsh-autosuggestions zsh-syntax-highlighting
```

Trên Ubuntu, tên lệnh thường là `fdfind` và `batcat`. Dùng alias không xung đột trong Zsh nếu muốn:

```zsh
alias fd='fdfind'
alias bat='batcat'
```

Các gói `zoxide`, `eza`, `lazygit`, `git-delta`, `yq`, `gh` có thể khác nhau tùy phiên bản Ubuntu/repository. **Không giả định mọi gói đều có trong APT**. Kiểm tra trước:

```bash
for pkg in zoxide eza lazygit git-delta yq gh; do
  echo "=== $pkg ==="
  apt-cache policy "$pkg" | head -5
done
```

Chỉ cài những gói có Candidate hợp lệ:

```bash
# Ví dụ: sudo apt install -y zoxide eza lazygit git-delta yq gh
```

Nếu không có, theo hướng dẫn chính thức của từng dự án; ưu tiên repo chính thức hoặc binary phát hành đã xác minh, không chạy ngẫu nhiên script bên thứ ba với sudo.

- zoxide: https://github.com/ajeetdsouza/zoxide
- eza: https://github.com/eza-community/eza
- lazygit: https://github.com/jesseduffield/lazygit
- delta: https://github.com/dandavison/delta
- GitHub CLI: https://cli.github.com/
- yq: https://github.com/mikefarah/yq
- uv: https://docs.astral.sh/uv/getting-started/installation/

## 4. Shell và prompt
Nếu chưa có Oh My Zsh, có thể chọn **Zsh + Starship** (https://starship.rs/) hoặc **Oh My Zsh + Powerlevel10k** (https://github.com/romkatv/powerlevel10k). Không cần cả hai. Chưa nên đổi shell mặc định (`chsh`) cho tới khi kiểm tra Zsh hoạt động và plugin ổn định. Trên server SSH/headless, giữ Bash cũng hoàn toàn phù hợp.

## 5. Tích hợp plugin vào Zsh
Mở `nano ~/.zshrc`, thêm **một lần**, không trùng với cấu hình sẵn có:

```zsh
# >>> developer-terminal >>>
command -v zoxide >/dev/null && eval "$(zoxide init zsh)"
command -v fzf >/dev/null && source <(fzf --zsh)
command -v direnv >/dev/null && eval "$(direnv hook zsh)"
# Nếu không dùng Oh My Zsh plugin tương ứng, bật hai dòng sau:
# source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
# source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
# <<< developer-terminal <<<
```

`fzf --zsh` cần phiên bản fzf đủ mới. Nếu phiên bản APT báo không hỗ trợ, dùng hướng dẫn tích hợp của phiên bản đang cài; không giữ một dòng gây lỗi khởi động. Kiểm tra vị trí file plugin bằng `dpkg -L zsh-autosuggestions` và `dpkg -L zsh-syntax-highlighting` trước khi bỏ dấu `#`. Syntax highlighting nên nạp gần cuối `.zshrc`.

```bash
zsh -n ~/.zshrc
zsh
```

Nếu muốn đổi shell mặc định sau khi xác nhận ổn định:

```bash
chsh -s "$(command -v zsh)"
```

Đăng xuất và đăng nhập lại để áp dụng.

## 6. Công cụ bổ sung cho dev
- Python: `uv` (cài theo tài liệu chính thức; kiểm tra `uv --version`).
- Node.js: chọn **một** trong fnm, nvm hoặc mise; không init nhiều manager cùng lúc.
- Docker: lazydocker (https://github.com/jesseduffield/lazydocker).
- Kubernetes: kubectl, k9s, kubectx/kubens; chỉ cài khi cần và dùng đúng version tương thích cluster.
- JSON/YAML: jq, yq; chú ý có nhiều chương trình tên `yq` khác nhau, hướng dẫn này ưu tiên **Mike Farah yq**.

## 7. Kiểm tra sau cài

```bash
for cmd in zsh fzf zoxide rg fdfind batcat eza lazygit delta tmux direnv uv gh btop jq yq; do
  command -v "$cmd" || echo "CHƯA CÓ: $cmd"
done
```

Thử `fzf`, `rg 'TODO' .`, `btop` (nhấn `q` để thoát), `lazygit` trong Git repo. `direnv allow` chỉ dùng cho `.envrc` đã xem và tin tưởng.

## 8. Rollback
Nếu cấu hình Zsh bị lỗi: chạy `zsh -f`, sau đó khôi phục bản backup `.zshrc` hoặc xóa khối `developer-terminal`. Rollback cấu hình không tự gỡ các gói APT. Muốn gỡ gói, kiểm tra trước bằng `apt show <tên>` và `sudo apt remove <tên>`; không dùng `autoremove` mù quáng trên máy dev.