# macOS — iTerm2 + Oh My Zsh + Powerlevel10k Developer Terminal

## Tổng quan công cụ

| Công cụ | Vai trò | Khi nào sử dụng |
|---|---|---|
| iTerm2 | Ứng dụng terminal nhiều pane, profile | Terminal chính trên macOS |
| Homebrew | Trình quản lý package | Cài, nâng cấp CLI |
| Zsh | Shell mặc định thường dùng trên macOS | Chạy lệnh, scripting |
| Oh My Zsh | Framework plugin và cấu hình Zsh | Completion, alias |
| Powerlevel10k | Prompt nhanh, tùy biến | Hiển thị Git, path; giữ cấu hình hiện tại |
| zsh-autosuggestions | Gợi ý lệnh theo history | Nhập lệnh nhanh |
| zsh-syntax-highlighting | Tô màu cú pháp lệnh | Giảm gõ sai |
| fzf | Tìm kiếm mờ | File, history, Git |
| zoxide | Điều hướng thư mục thông minh | Chuyển workspace |
| fd / rg | Tìm file / tìm nội dung | Khám phá code |
| eza / bat | Liệt kê file / đọc file đẹp hơn | Duyệt project |
| lazygit / git-delta | Git TUI / Git diff renderer | Review và commit |
| tmux | Multiplexer cho terminal | Giữ phiên làm việc |
| direnv | Nạp biến môi trường theo project | Env riêng mỗi repo |
| btop | Theo dõi tài nguyên | CPU, RAM, process |
| jq / yq | Xử lý JSON / YAML | API và manifest |


## Môi trường đã xác nhận
- iTerm2 **3.7.6** (ứng dụng hiển thị là `iTerm.app`, Homebrew cask `iterm2`).
- Shell `/bin/zsh`; đang dùng **Oh My Zsh + Powerlevel10k**. **Không cài Starship chồng lên Powerlevel10k**.
- Đã có trước: `rg`, `uv`, `gh`, `jq`.
- Theo log Homebrew, đã cài thành công: `fzf`, `zoxide`, `fd`, `eza`, `bat`, `lazygit`, `git-delta`, `tmux`, `direnv`, `btop`, `yq`, `zsh-autosuggestions`, `zsh-syntax-highlighting`.

## 1. Kiểm tra

```zsh
brew --version
printf 'Shell: %s\n' "$SHELL"
for cmd in fzf zoxide rg fd eza bat lazygit delta tmux direnv uv gh btop jq yq; do
  if command -v "$cmd" >/dev/null; then echo "✓ $cmd"; else echo "✗ $cmd"; fi
done
sed -n '/^plugins=(/,/^)/p' ~/.zshrc
```

## 2. Cài mới trên một máy macOS khác (bỏ qua nếu đã cài)
Cài Homebrew từ https://brew.sh nếu chưa có. Homebrew tự xử lý các formula đã cài; không cần chạy lại trên máy hiện tại.

```zsh
brew install fzf zoxide ripgrep fd eza bat lazygit git-delta tmux direnv btop jq yq uv gh zsh-autosuggestions zsh-syntax-highlighting
```

Có thể cài thêm `lazydocker`, `k9s`, `kubectx`, `just`, `hyperfine`, `dust`, `duf` khi cần DevOps. Chỉ chọn **một** trình quản lý Node chính (fnm, nvm hoặc mise).

## 3. Sao lưu trước khi sửa

```zsh
stamp=$(date +%Y%m%d-%H%M%S)
cp ~/.zshrc ~/.zshrc.backup.$stamp
[[ -f ~/.p10k.zsh ]] && cp ~/.p10k.zsh ~/.p10k.zsh.backup.$stamp
```

## 4. Kiểm tra plugin để tránh nạp trùng

```zsh
sed -n '/^plugins=(/,/^)/p' ~/.zshrc
grep -nE 'zoxide init|fzf --zsh|direnv hook|zsh-autosuggestions|zsh-syntax-highlighting' ~/.zshrc
```

Nếu `zsh-autosuggestions` hoặc `zsh-syntax-highlighting` đã được khai báo trong `plugins=(...)` và có plugin tương ứng ở `$ZSH_CUSTOM/plugins`, **không source thêm bản Homebrew**. Nếu chưa có, dùng hai dòng `source` dưới đây. Để syntax highlighting ở gần cuối cấu hình, sau Oh My Zsh.

## 5. Tích hợp vào ~/.zshrc

```zsh
nano ~/.zshrc
```

Thêm **từng dòng chưa tồn tại** ở cuối file (sau `source $ZSH/oh-my-zsh.sh`, giữ nguyên `ZSH_THEME="powerlevel10k/powerlevel10k"` và `source ~/.p10k.zsh`):

```zsh
# >>> developer-terminal >>>
command -v zoxide >/dev/null && eval "$(zoxide init zsh)"
command -v fzf >/dev/null && source <(fzf --zsh)
command -v direnv >/dev/null && eval "$(direnv hook zsh)"
# Chỉ bật nếu CHƯA được Oh My Zsh hoặc cấu hình khác nạp:
# source "$(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh"
# source "$(brew --prefix)/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh"
# <<< developer-terminal <<<
```

Bỏ dấu `#` ở hai dòng cuối **chỉ khi chưa có plugin tương ứng**. Nếu dùng Oh My Zsh plugin thay vì Homebrew, đảm bảo plugin đã được cài tại thư mục mà Oh My Zsh tìm kiếm. Không source cùng một plugin hai lần.

```zsh
zsh -n ~/.zshrc && exec zsh
```

## 6. Tùy chỉnh tùy chọn

```zsh
p10k configure           # đổi giao diện prompt, không cần Starship
fzf                     # tìm kiếm fuzzy
z ~/Downloads            # di chuyển bằng zoxide sau khi đã truy cập thư mục
fd '*.md'               # tìm file
rg 'TODO' .              # tìm nội dung
lazygit                  # chạy trong Git repo
btop                     # nhấn q để thoát
```

Để có icon đẹp, chọn **Nerd Font** trong iTerm2 → Settings → Profiles → Text. Không ghi đè `ls`, `cd`, `cat` mặc định nếu chưa thực sự cần.

`direnv` chỉ tự áp dụng `.envrc` khi bạn chủ động chạy `direnv allow` trong thư mục tin cậy; không chạy `allow` với repo lạ.

## 7. Rollback
Nếu shell khởi động lỗi: `zsh -f` để vào Zsh không nạp cấu hình. Khôi phục file backup `cp ~/.zshrc.backup.<timestamp> ~/.zshrc`, hoặc xóa khối `developer-terminal` vừa thêm. `exec zsh` để kiểm tra lại. Rollback này **không gỡ** các công cụ Homebrew.

## Tài liệu chính thức
- https://iterm2.com/
- https://ohmyz.sh/
- https://github.com/romkatv/powerlevel10k
- https://github.com/junegunn/fzf
- https://github.com/ajeetdsouza/zoxide
- https://direnv.net/