# Cài đặt k3s

Tài liệu này mô tả cách cài đặt **k3s (lightweight Kubernetes)** trên **một máy duy nhất**, phục vụ cho:
- Lab / học Kubernetes
- Xây dựng lakehouse (MinIO, Trino, Iceberg, Spark job)
- Không dùng cho production

Mục tiêu chính:
- Nhẹ
- Ổn định
- Dễ cài lại
- Phù hợp máy cấu hình thấp (8GB RAM)

---

## 1. Điều kiện tiên quyết

### Phần cứng
- RAM: 8GB
- Disk trống: ≥ 40GB (khuyến nghị SSD)
- CPU: ≥ 4 cores (2 cores vẫn chạy được)

### Phần mềm
- OS: Ubuntu 20.04 / 22.04 (hoặc Debian tương đương)
- Quyền `sudo`

---

## 2. Chuẩn bị hệ thống

### 2.1. Tắt swap (BẮT BUỘC)

Kubernetes yêu cầu tắt swap để quản lý memory chính xác.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
````

Kiểm tra:

```bash
free -h
```

Swap phải bằng `0`.

---

### 2.2. Bật kernel modules cần thiết

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Persist sau reboot:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

---

### 2.3. Cấu hình sysctl cho networking

```bash
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## 3. Cài đặt k3s (single-node, tối giản)

Cài k3s với các thành phần **tối thiểu**, phù hợp máy 8GB:

```bash
curl -sfL https://get.k3s.io | sh -s - \
  --disable traefik \
  --disable servicelb \
  --disable metrics-server
```

### Giải thích

* `traefik`: ingress controller (chưa cần cho lab)
* `servicelb`: load balancer giả (không cần với 1 node)
* `metrics-server`: tốn RAM, chỉ bật khi cần monitoring

### 3.1. Cài containerd
```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
---

## 4. Kiểm tra trạng thái k3s

### 4.1. Kiểm tra service

```bash
sudo systemctl status k3s
```

Trạng thái mong đợi:

```
Active: active (running)
```

---

### 4.2. Cấu hình kubectl cho user thường

```bash
sudo mkdir -p /etc/rancher/k3s
cat <<'EOF' | sudo tee /etc/rancher/k3s/config.yaml
write-kubeconfig-mode: "0644"
EOF
```

---

### 4.3. Kiểm tra cluster

```bash
kubectl get nodes
kubectl get pods -A
```

Kết quả mong đợi:

* Node: `Ready`
* Namespace `kube-system` chỉ có:

  * `coredns`
  * `local-path-provisioner`

---

## 5. Storage mặc định

k3s cung cấp sẵn `local-path` StorageClass (phù hợp single-node).

Kiểm tra:

```bash
kubectl get sc
```

Kết quả:

```
local-path   (default)
```

Storage này sẽ được dùng cho:

* MinIO
* Postgres
* Hive Metastore
* Các service stateful trong lab

---

## 6. Tài nguyên sử dụng (tham khảo)

Trạng thái idle sau khi cài:

* k3s: ~600MB – 1GB RAM
* OS + cache: ~1–1.5GB RAM

Còn lại ~5GB RAM cho workload (đủ cho lakehouse lab nếu giới hạn tài nguyên đúng cách).

---

## 7. Gỡ k3s (khi cần cài lại)

```bash
sudo /usr/local/bin/k3s-uninstall.sh
```

Xoá kubeconfig local:

```bash
rm -rf ~/.kube
```

---

## 8. Bước tiếp theo

Sau khi k3s sẵn sàng, thứ tự triển khai khuyến nghị:

1. Namespace + Storage
2. MinIO (object storage)
3. Iceberg catalog (REST hoặc Hive Metastore)
4. Trino
5. Spark job (chạy khi cần)
6. Airflow / Monitoring (chỉ bật khi cần)

---

## Ghi chú

* Không chạy quá nhiều service cùng lúc trên máy 8GB
* Luôn set `resources.requests` và `resources.limits`
* Spark nên chạy dạng job (ephemeral), không chạy cluster thường trực
