# AI Coding CLI — Codex, Claude Code, MCP & agents

> Hướng dẫn workflow và kiểm tra, không lưu API key/token vào repository. Tính năng, tên model, giới hạn và cách đăng nhập có thể thay đổi theo phiên bản hoặc gói tài khoản.

## 1. Chuẩn bị môi trường

- Git và GitHub CLI (`git --version`, `gh --version`).
- Node.js LTS và npm (`node -v`, `npm -v`), nên dùng **một** trình quản lý version Node (fnm, nvm hoặc mise).
- Python và uv nếu dự án cần.
- VS Code hoặc IDE phù hợp, terminal có shell completion, ripgrep, fzf.
- Tạo Git branch trước khi yêu cầu agent sửa nhiều file.

Windows: ưu tiên WSL2 Ubuntu cho workflow shell Linux; PowerShell 7 cũng được nếu CLI hỗ trợ. Ubuntu/macOS: chạy trong terminal thường.

## 2. Codex CLI

Nguồn chính thức: https://github.com/openai/codex và https://developers.openai.com/codex/cli/

```bash
npm install -g @openai/codex
codex --version
codex
```

Đăng nhập theo hướng dẫn của CLI (ChatGPT hoặc phương thức API được hỗ trợ). Không ghi token vào `.md`, Git hay shell history. Kiểm tra các lệnh hiện có bằng `codex --help` thay vì giả định mọi bản giống nhau.

Session:

```bash
codex resume --help
codex resume --last
```

`resume` tiếp tục session đã lưu; **không có nghĩa là tự chạy mãi mãi hoặc tự tiếp tục sau khi máy tắt**. Trước khi dùng chế độ tự động, kiểm tra quyền thực thi, sandbox và chính sách approval.

## 3. Claude Code

Nguồn chính thức: https://docs.anthropic.com/en/docs/claude-code/overview và https://code.claude.com/docs/

Cài theo phương thức được tài liệu chính thức hiện tại khuyến nghị cho OS của bạn; kiểm tra `claude --version`, `claude --help`, sau đó chạy `claude` trong thư mục dự án. Tránh copy các lệnh cài không rõ nguồn. Các lựa chọn đăng nhập và billing phụ thuộc tài khoản.

## 4. Prompt triển khai công việc

Mẫu yêu cầu agent:

```text
Mục tiêu: [tính năng cụ thể].
Bối cảnh: [repo, module, ngôn ngữ, runtime].
Giới hạn: không sửa file ngoài phạm vi; không đụng production; không lộ secret.
Bước 1: đọc cấu trúc, luồng dữ liệu, tests hiện có.
Bước 2: đưa kế hoạch và giả định cần xác nhận.
Bước 3: triển khai thay đổi nhỏ, có kiểm thử.
Bước 4: chạy lint/test/build và báo lỗi còn lại.
Bước 5: tóm tắt file đã đổi, rủi ro và cách rollback.
```

Với tác vụ reverse engineering: phân biệt dữ kiện từ source/binary/log với suy đoán; không tuyên bố đã tái hiện 1:1 khi chưa kiểm chứng.

## 5. MCP servers

MCP (Model Context Protocol) kết nối agent với nguồn dữ liệu/công cụ. Nguồn: https://modelcontextprotocol.io/

Quy trình:
1. Chọn MCP server đáng tin cậy, đọc danh sách tools và phạm vi dữ liệu.
2. Cài và cấu hình bằng **cú pháp chính thức của đúng CLI/version**; không copy cấu hình của công cụ khác.
3. Cấp quyền tối thiểu (read-only trước); kiểm tra kết nối bằng thao tác đọc.
4. Chỉ cho phép thao tác ghi, tạo PR, deploy khi cần và đã xác minh target.
5. Không để token, password, kubeconfig, SSH private key trong repo hoặc prompt công khai.

## 6. Workflow với GitHub

```bash
git status
git switch -c feat/my-change
gh auth status
gh repo view
git diff --check
git diff
```

Sau khi review thay đổi và chạy test, commit rồi tạo PR qua `gh pr create` khi phù hợp. Agent không nên tự merge, push vào protected branch hoặc deploy production nếu chưa được duyệt.

## 7. Kỷ luật khi dùng nhiều agents

- Mỗi agent một phạm vi file/module rõ ràng; tránh cùng sửa một file.
- Có người hoặc agent tổng hợp chịu trách nhiệm tích hợp.
- Ghi lại quyết định kiến trúc trong README/ADR.
- Yêu cầu bằng chứng: đường dẫn file, test, command và output.
- Giới hạn số vòng lặp và chi phí; theo dõi context, usage và timeout.
- Không dùng `--dangerously-...` hoặc chế độ bỏ approval chỉ vì muốn nhanh.

## 8. Khắc phục lỗi

- CLI không nhận lệnh: kiểm tra `PATH`, Node version, shell restart.
- Đăng nhập lỗi: kiểm tra phương thức xác thực, proxy, thời gian hệ thống.
- Agent không thấy repo: kiểm tra `pwd`, `git status`, quyền filesystem.
- MCP không kết nối: kiểm tra executable, config path, environment, quyền.
- Session bị gián đoạn: xem tính năng resume của CLI, không mặc định có autosave vô hạn.

Luôn kiểm tra tài liệu chính thức trước khi áp dụng các tùy chọn mới.
