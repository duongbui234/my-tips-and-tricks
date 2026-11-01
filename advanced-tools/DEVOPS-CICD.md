# DevOps CI/CD — GitHub Actions, GitLab CI, Jenkins, SSH

## Tổng quan công cụ và khái niệm

| Công cụ / khái niệm | Vai trò | Khi nào sử dụng |
|---|---|---|
| GitHub Actions | CI/CD tích hợp GitHub | Test/build/deploy theo push, PR |
| GitLab CI/CD | CI/CD tích hợp GitLab | Pipeline theo `.gitlab-ci.yml` |
| GitLab Runner | Agent thực thi job GitLab CI | Chạy pipeline |
| Jenkins | Automation server CI/CD | Pipeline tùy biến, self-host |
| Jenkinsfile | Pipeline as Code cho Jenkins | Version control pipeline |
| SSH | Kết nối/điều khiển máy từ xa an toàn | Truy cập server, Git |
| GitHub CLI (`gh`) | Quản lý GitHub từ terminal | PR, release, workflow |
| Docker / Buildx | Build image ứng dụng | Artifact container |
| Container Registry | Lưu image đã build | Phân phối cho cluster |
| Kubernetes | Triển khai và quản lý workload | CD trên cluster |
| Helm | Quản lý release Kubernetes | Upgrade/rollback |
| Secrets manager | Lưu và cấp thông tin bí mật | Token, password, credentials |
| Artifact / SBOM | Đầu ra build / danh mục thành phần phần mềm | Truy vết và kiểm toán |
| Monitoring / logging | Theo dõi sức khỏe và lỗi hệ thống | Vận hành sau deploy |


## 1. Phân biệt thành phần

- **CI:** lint, test, build, security scan trên mỗi push/PR.
- **CD:** xuất bản artifact/image; triển khai staging/production theo approval và policy.
- **Registry:** nơi lưu image; không dùng tag `latest` làm định danh duy nhất khi deploy.
- **Secrets:** quản lý qua CI secret store hoặc secret manager, không commit vào Git.

## 2. GitHub Actions mẫu cho dự án Python

Tạo `.github/workflows/ci.yml` trong repository ứng dụng:

```yaml
name: CI
on:
  push:
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: python -m pip install --upgrade pip
      - run: pip install -r requirements.txt
      - run: python -m pytest
```

Chỉnh Python version, dependency và lệnh test theo dự án; pin action bằng commit SHA khi yêu cầu supply-chain chặt. Tài liệu: https://docs.github.com/en/actions .

## 3. GitLab CI mẫu

Tạo `.gitlab-ci.yml`:

```yaml
stages: [test]
python-test:
  stage: test
  image: python:3.12-slim
  script:
    - pip install -r requirements.txt
    - python -m pytest
```

Cần GitLab Runner phù hợp; cấu hình variable/secrets trong GitLab. Tài liệu: https://docs.gitlab.com/ci/ .

## 4. Jenkins

- Jenkins là automation server, cần máy chủ/agent và vận hành, update, backup.
- Khuyến nghị Jenkinsfile dạng Pipeline-as-Code.
- Dùng credentials store, không hardcode token trong Jenkinsfile.
- Tách controller và build agents; giới hạn quyền deploy.
- Tài liệu: https://www.jenkins.io/doc/book/pipeline/

## 5. SSH và an toàn

```bash
ssh -V
ssh-keygen -t ed25519 -C "dev-machine"
ssh -T git@github.com
```

Không ghi đè khóa cũ khi `ssh-keygen` hỏi đường dẫn. Kiểm tra fingerprint host theo nguồn đáng tin cậy trước khi chấp nhận. **Không chia sẻ private key**; chỉ public key có đuôi `.pub` được đưa lên GitHub/server.

## 6. Pipeline bàn giao image cho Ops

1. PR → lint/test + security scan.
2. Merge → build Docker image; gắn tag bằng commit SHA.
3. Push image lên registry được cấp quyền.
4. Lưu image digest, SBOM, kết quả scan, changelog.
5. Deploy staging qua Helm/manifests và smoke test.
6. Production cần approval, rollout strategy và rollback.
7. Quan sát metrics/log và xác nhận release thành công.

Đối với service convert Office sang PDF: thêm test fixture DOC/DOCX/PPT/PPTX/XLS/XLSX, kiểm thử IPA/Khmer, giới hạn file 100 MiB, timeout, memory, concurrency và không lộ tài liệu người dùng trong log.

## 7. Checklist trước khi triển khai

- `docker compose config`, `docker build`, unit/integration tests.
- Image scan và license review.
- Health probes, requests/limits, logging.
- Secret injection và RBAC tối thiểu.
- Database migration/backward compatibility nếu có.
- Backup, rollback và người chịu trách nhiệm vận hành.
