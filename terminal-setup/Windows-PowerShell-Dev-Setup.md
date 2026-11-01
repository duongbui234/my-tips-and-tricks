# Windows 11 — PowerShell 7 Developer Terminal

## Mục tiêu
Cài và cấu hình các công cụ: Oh My Posh, Terminal-Icons, PSReadLine, zoxide, fzf + PSFzf, ripgrep, fd, lazygit, posh-git, delta, GitHub CLI, bat, eza, fnm, uv. Giữ Windows PowerShell 5.1 để tương thích, sử dụng `pwsh` (PowerShell 7+) cho công việc mới.

## 1. Kiểm tra trước khi cài
Mở **PowerShell 7**, không phải Windows PowerShell 5.1:

```powershell
$PSVersionTable.PSVersion
Get-Command winget, git, pwsh -ErrorAction SilentlyContinue
$PROFILE
Test-Path $PROFILE
```

Yêu cầu WinGet hoạt động và có Internet. Kiểm tra từng công cụ:

```powershell
'oh-my-posh','zoxide','fzf','rg','fd','lazygit','delta','gh','bat','eza','fnm','uv' | ForEach-Object {
  $cmd = Get-Command $_ -ErrorAction SilentlyContinue
  "{0,-15} {1}" -f $_, $(if ($cmd) { $cmd.Source } else { 'CHƯA CÓ' })
}
Get-Module PSReadLine,Terminal-Icons,PSFzf,posh-git -ListAvailable | Select-Object Name,Version
```

## 2. Sao lưu cấu hình

```powershell
$backupDir = Join-Path $HOME ('.pwsh-backup-' + (Get-Date -Format 'yyyyMMdd-HHmmss'))
New-Item -ItemType Directory -Path $backupDir -Force | Out-Null
if (Test-Path $PROFILE) { Copy-Item $PROFILE (Join-Path $backupDir 'profile.ps1') }
Write-Host "Backup: $backupDir"
```

## 3. Cài CLI bằng WinGet
WinGet có thể yêu cầu chấp nhận điều khoản. Lệnh dưới đây bỏ qua công cụ đã tìm thấy trong PATH, không tự nâng cấp các bản đang dùng. Kiểm tra ID bằng `winget search <tên>` nếu kho gói thay đổi.

```powershell
$packages = @(
  @{cmd='oh-my-posh'; id='JanDeDobbeleer.OhMyPosh'},
  @{cmd='zoxide'; id='ajeetdsouza.zoxide'},
  @{cmd='fzf'; id='junegunn.fzf'},
  @{cmd='rg'; id='BurntSushi.ripgrep.MSVC'},
  @{cmd='fd'; id='sharkdp.fd'},
  @{cmd='lazygit'; id='JesseDuffield.lazygit'},
  @{cmd='delta'; id='dandavison.delta'},
  @{cmd='gh'; id='GitHub.cli'},
  @{cmd='bat'; id='sharkdp.bat'},
  @{cmd='eza'; id='eza-community.eza'},
  @{cmd='fnm'; id='Schniz.fnm'},
  @{cmd='uv'; id='astral-sh.uv'}
)
foreach ($p in $packages) {
  if (Get-Command $p.cmd -ErrorAction SilentlyContinue) { Write-Host "SKIP $($p.cmd)"; continue }
  winget install --id $p.id --exact --source winget
  if ($LASTEXITCODE -ne 0) { Write-Warning "Kiểm tra lại gói: $($p.id)" }
}
```

Đóng và mở lại PowerShell 7 để cập nhật PATH. Nếu một ID không tồn tại, tìm bằng `winget search` thay vì chạy lặp vô hạn.

## 4. Cài PowerShell modules

```powershell
$modules = 'Terminal-Icons','PSFzf','posh-git'
foreach ($m in $modules) {
  if (-not (Get-Module -ListAvailable -Name $m)) {
    Install-Module -Name $m -Scope CurrentUser -Repository PSGallery
  } else { Write-Host "SKIP $m" }
}
# PSReadLine thường có sẵn trong PowerShell 7
Get-Module PSReadLine -ListAvailable
```

Nếu PSGallery yêu cầu xác nhận nguồn, chỉ chấp nhận khi bạn tin tưởng nguồn. Không cần cài lại PSReadLine khi đã có.

## 5. Cấu hình profile an toàn

```powershell
New-Item -ItemType File -Path $PROFILE -Force | Out-Null
notepad $PROFILE
```

Thêm đoạn này **một lần** vào cuối profile; giữ nguyên cấu hình cũ. Nếu bạn đã cấu hình Oh My Posh/PSFzf/PSReadLine, gộp thủ công thay vì nạp trùng:

```powershell
# >>> developer-terminal >>>
Import-Module Terminal-Icons -ErrorAction SilentlyContinue
Import-Module posh-git -ErrorAction SilentlyContinue
Import-Module PSFzf -ErrorAction SilentlyContinue
if (Get-Module PSReadLine) {
  Set-PSReadLineOption -PredictionSource History -ErrorAction SilentlyContinue
  Set-PSReadLineOption -PredictionViewStyle ListView -ErrorAction SilentlyContinue
}
if (Get-Command oh-my-posh -ErrorAction SilentlyContinue) {
  oh-my-posh init pwsh | Invoke-Expression
}
if (Get-Command zoxide -ErrorAction SilentlyContinue) {
  zoxide init powershell | Out-String | Invoke-Expression
}
# <<< developer-terminal <<<
```

Để PowerShell 7 mở tại thư mục home **chỉ trong Windows Terminal**, đặt Profiles → PowerShell → Starting directory thành `%USERPROFILE%`; không cần thêm `Set-Location $HOME` vào profile vì sẽ phá thư mục làm việc khi mở từ dự án.

## 6. Font và kiểm tra
Cài Nerd Font (ví dụ JetBrainsMono Nerd Font) từ nguồn chính thức và chọn trong Windows Terminal → PowerShell → Appearance. Sau đó:

```powershell
$PSVersionTable.PSVersion
Get-Command oh-my-posh,zoxide,fzf,rg,fd,lazygit,delta,gh,bat,eza,fnm,uv -ErrorAction SilentlyContinue
Get-Module Terminal-Icons,PSReadLine,PSFzf,posh-git
```

Thử `z <tên-thư-mục-đã-truy-cập>`, `fzf`, `lazygit` (trong Git repo), `gh auth login` (nếu cần). `fnm` chỉ cần thêm init khi quyết định dùng nó để quản lý Node; không chạy song song với nvm/mise.

## 7. Rollback
Mở `pwsh -NoProfile` nếu profile bị lỗi. Mở `$PROFILE`, xóa đúng khối giữa `# >>> developer-terminal >>>` và `# <<< developer-terminal <<<`, hoặc khôi phục file profile từ thư mục backup ở bước 2. Việc rollback cấu hình **không tự gỡ CLI/module**. Muốn gỡ riêng: `winget uninstall --id <ID>` hoặc `Uninstall-Module <Tên>` sau khi kiểm tra phụ thuộc.

## Tài liệu chính thức
- https://learn.microsoft.com/powershell/
- https://learn.microsoft.com/windows/package-manager/winget/
- https://ohmyposh.dev/
- https://github.com/PowerShell/PSReadLine
- https://github.com/kelleyma49/PSFzf