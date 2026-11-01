# Docker & Docker Compose — Windows / Ubuntu / macOS

> Mục tiêu: phát triển, build và kiểm thử container local; chuẩn bị image bàn giao đội vận hành. Không chạy các lệnh xóa dữ liệu trên máy đang phục vụ production.

## 1. Chọn phương án cài đặt

| OS | Khuyến nghị | Ghi chú |
|---|---|---|
| Windows 11 | WSL2 + Docker Desktop | Bật WSL2 và tích hợp distro Ubuntu trong Docker Desktop |
| Ubuntu | Docker Engine + Compose plugin | Dùng repository APT chính thức của Docker |
| macOS | Docker Desktop hoặc Colima + Docker CLI | Docker Desktop dễ bắt đầu; Colima là lựa chọn khác |

### Windows 11

PowerShell (Administrator):

```powershell
wsl --install
wsl --update
wsl --status
```

Khởi động lại nếu Windows yêu cầu. Cài Docker Desktop từ https://docs.docker.com/desktop/setup/install/windows-install/ rồi chọn **Use WSL 2 based engine** và bật **Settings > Resources > WSL Integration** cho Ubuntu. Kiểm tra từ PowerShell hoặc WSL:

```powershell
docker version
docker compose version
docker run --rm hello-world
```

Không cài Docker Engine lần thứ hai trong WSL nếu đã dùng Docker Desktop WSL integration, trừ khi chủ động quản trị hai daemon riêng.

### Ubuntu

Cài theo tài liệu Docker Engine chính thức: https://docs.docker.com/engine/install/ubuntu/ . Ví dụ quy trình APT cho bản Ubuntu được Docker hỗ trợ:

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources >/dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo docker run --rm hello-world
docker compose version
```

Nếu cần chạy `docker` không dùng `sudo`, có thể thêm user vào nhóm `docker` và đăng nhập lại. **Lưu ý: nhóm docker có quyền tương đương root**, chỉ áp dụng trên máy tin cậy. Tham khảo https://docs.docker.com/engine/install/linux-postinstall/ .

### macOS

```bash
brew install --cask docker
```

Mở Docker Desktop một lần để khởi tạo engine. Kiểm tra `docker version`, `docker compose version`. Nếu chọn Colima, xem https://github.com/abiosoft/colima; tránh chạy hai engine không cần thiết.

## 2. Docker Compose thực hành

Tạo `compose.yaml`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "127.0.0.1:8080:80"
    restart: unless-stopped
```

```bash
docker compose config
docker compose up -d
docker compose ps
docker compose logs -f --tail=100
docker compose down
```

Truy cập http://localhost:8080 . Dùng `docker compose down -v` **chỉ khi muốn xóa cả volume**.

## 3. Build và kiểm tra image

```bash
docker build -t my-service:dev .
docker image ls
docker history my-service:dev
docker inspect my-service:dev
docker run --rm -p 127.0.0.1:8000:8000 my-service:dev
```

Dùng `.dockerignore`, multi-stage build, image nền nhỏ, pin phiên bản phù hợp, chạy non-root, không COPY secret vào image. Với service xử lý file lớn: giới hạn upload, timeout, memory, CPU, concurrency; dùng thư mục tạm có quota và dọn file sau xử lý.

## 4. Công cụ bổ trợ

- **lazydocker:** giao diện TUI quản lý Docker: https://github.com/jesseduffield/lazydocker
- **dive:** phân tích layer image: https://github.com/wagoodman/dive
- **Buildx:** `docker buildx version`, build đa kiến trúc khi cần.
- **Trivy:** quét lỗ hổng image: https://trivy.dev/

Cài lazydocker theo hướng dẫn release chính thức của dự án hoặc package manager có sẵn; xác minh `lazydocker --version`. Không giả định mọi package manager đều có phiên bản mới nhất.

## 5. Debug & bảo trì

```bash
docker ps -a
docker logs --tail 100 CONTAINER
docker stats
docker system df
docker network ls
docker volume ls
```

`docker system prune` có thể xóa dữ liệu không còn được Docker tham chiếu; **xem kỹ trước khi chạy**, đặc biệt tránh `--volumes` trên máy có dữ liệu quan trọng.

## 6. Bàn giao cho Kubernetes

- Dockerfile + image tag cố định (tốt hơn là digest) + registry.
- README biến môi trường, secret (chỉ tên key, không commit giá trị), healthcheck.
- CPU/memory requests/limits, probes, timeout, graceful shutdown.
- Log stdout/stderr, metrics, hướng dẫn rollback.
- CI build/test/scan/push; đội Ops quyết định registry, namespace, ingress, secrets, storage và policy.

Nguồn: https://docs.docker.com/ ; https://docs.docker.com/compose/ .
