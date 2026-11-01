# Kubernetes local & công cụ CLI

> Dành cho Windows (WSL2), Ubuntu và macOS. Thực hành local trước khi bàn giao đội Ops triển khai cluster thực tế.

## Tổng quan công cụ và khái niệm

| Công cụ / khái niệm | Vai trò | Khi nào sử dụng |
|---|---|---|
| Kubernetes | Điều phối container trên cluster | Triển khai và vận hành service |
| kubectl | Tương tác Kubernetes API | Apply, get, describe, logs |
| k9s | TUI quan sát Pod, Deployment, log | Debug cluster nhanh |
| Helm | Package manager cho Kubernetes | Đóng gói, nâng cấp ứng dụng |
| kubectx / kubens | Chuyển context / namespace | Làm việc nhiều cluster |
| kind | Cluster Kubernetes local bằng Docker | Test manifest, CI |
| k3d | Chạy k3s cluster trong Docker | Lab local nhẹ |
| stern | Theo dõi log nhiều Pod | Debug microservices |
| Pod | Đơn vị chạy container nhỏ nhất trong K8s | Hiểu trạng thái workload |
| Deployment | Quản lý replicas và rollout Pod | Deploy ứng dụng stateless |
| Service | Định danh và kết nối mạng tới Pod | Truy cập nội bộ |
| Ingress / Gateway | Định tuyến traffic HTTP(S) vào cluster | Public API/web |
| ConfigMap / Secret | Cấu hình thường / dữ liệu nhạy cảm | Cấp cấu hình cho Pod |
| HPA | Tự điều chỉnh số Pod theo metric | Autoscaling |
| PV / PVC | Cấp phát và yêu cầu lưu trữ | Dữ liệu bền vững |


## 1. Bộ công cụ

| Công cụ | Vai trò |
|---|---|
| kubectl | Tương tác Kubernetes API |
| k9s | TUI quan sát Pod, Deployment, log |
| Helm | Package manager cho Kubernetes |
| kubectx / kubens | Chuyển context / namespace |
| kind hoặc k3d | Tạo cluster local bằng container |
| stern (tùy chọn) | Theo dõi log nhiều Pod |

## 2. Cài đặt theo OS

**Windows 11:** cài WSL2 Ubuntu, Docker Desktop WSL integration, rồi cài các CLI trong WSL theo hướng dẫn Linux. Có thể dùng WinGet cho kubectl/Helm nếu thích chạy từ PowerShell; tránh trộn kubeconfig Windows và WSL không chủ ý.

**macOS (Homebrew):**

```bash
brew install kubectl helm k9s kubectx kind k3d
kubectl version --client
helm version
k9s version
```

**Ubuntu:** cài từng CLI theo hướng dẫn chính thức, không dùng lệnh tải binary không kiểm tra nguồn:

- kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
- Helm: https://helm.sh/docs/intro/install/
- k9s: https://k9scli.io/
- kubectx/kubens: https://github.com/ahmetb/kubectx
- kind: https://kind.sigs.k8s.io/docs/user/quick-start/
- k3d: https://k3d.io/

Kiểm tra version tương thích: `kubectl` nên nằm trong khoảng lệch tối đa một minor version so với control plane (theo chính sách version skew của Kubernetes).

## 3. Tạo cluster local miễn phí

Yêu cầu Docker daemon đã chạy. Chọn **một** trong hai:

```bash
# kind
kind create cluster --name dev
kubectl config current-context
kubectl get nodes
```

Hoặc:

```bash
# k3d
k3d cluster create dev
kubectl get nodes
```

Đừng chạy cả hai nếu không cần, để tiết kiệm RAM. Xóa cluster local khi hoàn tất: `kind delete cluster --name dev` hoặc `k3d cluster delete dev`.

## 4. Deploy ứng dụng demo

```bash
kubectl create namespace playground
kubectl -n playground create deployment web --image=nginx:alpine
kubectl -n playground expose deployment web --port=80
kubectl -n playground rollout status deployment/web
kubectl -n playground get deploy,pods,svc
kubectl -n playground port-forward svc/web 8080:80
```

Truy cập http://localhost:8080 . Dừng port-forward bằng `Ctrl+C`.

```bash
kubectl -n playground describe deploy web
kubectl -n playground logs deploy/web --tail=100
kubectl -n playground get events --sort-by=.lastTimestamp
kubectl -n playground delete namespace playground
```

## 5. Helm và k9s

```bash
helm list -A
helm lint ./chart
helm template demo ./chart
k9s
```

Chỉ chạy `helm install/upgrade` khi đã hiểu chart và namespace đích. Trong k9s, nhấn `?` để xem phím tắt, `:q` để thoát.

## 6. An toàn context

```bash
kubectl config get-contexts
kubectl config current-context
kubectl config view --minify
kubectl get ns
```

**Trước khi apply/delete/scale, xác nhận đúng cluster và namespace.** Kubeconfig có thể chứa thông tin nhạy cảm; không commit `~/.kube/config` vào Git.

## 7. Checklist gần production

- Manifest hoặc Helm chart, resource requests/limits.
- Readiness/liveness/startup probes; rolling update và rollback.
- ConfigMap, Secret, ServiceAccount/RBAC tối thiểu.
- Service, Ingress/Gateway, TLS, NetworkPolicy.
- PersistentVolume nếu cần; log/metrics/tracing; autoscaling phù hợp.
- Quy trình CI/CD và quy tắc quản lý image tag/digest.

Tài liệu: https://kubernetes.io/docs/home/ ; https://helm.sh/docs/ .
