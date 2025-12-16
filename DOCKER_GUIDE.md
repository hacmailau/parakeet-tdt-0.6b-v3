# Hướng dẫn Chạy Ứng dụng Parakeet TDT với Docker

Tài liệu này sẽ hướng dẫn bạn từng bước cách đóng gói và chạy ứng dụng này bên trong một "container" Docker. Docker giúp bạn chạy ứng dụng trên bất kỳ máy nào mà không cần lo lắng về việc cài đặt môi trường Python hay các thư viện phức tạp.

## 1. Yêu cầu Cần thiết

Trước khi bắt đầu, hãy đảm bảo bạn đã cài đặt:

1.  **Docker Desktop:** Tải và cài đặt tại [docker.com](https://www.docker.com/products/docker-desktop/).
2.  **NVIDIA Container Toolkit:** (Chỉ dành cho máy thật có GPU NVIDIA) Giúp Docker sử dụng được card màn hình của bạn.
    - Xem hướng dẫn cài đặt tại [đây](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html).
    - Nếu bạn dùng Windows, hãy đảm bảo Docker Desktop được cài đặt cùng với **WSL2** và đã bật tính năng GPU support.

## 2. Dockerfile là gì?

Chúng ta đã tạo sẵn 2 file quan trọng trong thư mục dự án:

1.  `Dockerfile`: Giống như một công thức nấu ăn, nó chỉ cho Docker cách tạo ra môi trường làm việc:
    - Bắt đầu từ hệ điều hành có sẵn NVIDIA CUDA (giúp chạy AI nhanh).
    - Cài đặt thêm `ffmpeg` (xử lý âm thanh).
    - Cài đặt `python` và các thư viện cần thiết.
    - Chạy ứng dụng `app.py`.

2.  `.dockerignore`: Danh sách các file mà Docker nên bỏ qua (như các file rác, file ẩn) để container nhẹ hơn.

## 3. Các bước Thực hiện

Thực hiện các lệnh sau trong Terminal (CMD hoặc PowerShell) tại thư mục dự án.

### Bước 1: Xây dựng (Build) Docker Image

Lệnh này sẽ đọc `Dockerfile`, tải các phần cần thiết và tạo ra một "Image" (bản đóng gói) của ứng dụng. Quá trình này có thể mất 5-10 phút tùy tốc độ mạng vì phải tải image CUDA lớn (~2-3GB).

```bash
docker build -t parakeet-app .
```
- `-t parakeet-app`: Đặt tên cho image là `parakeet-app` để dễ gọi sau này.
- `.`: Dấu chấm ở cuối rất quan trọng, báo cho Docker tìm file trong thư mục hiện tại.

### Bước 2: Chạy (Run) Container

Sau khi build xong, hãy chạy lệnh sau để khởi động ứng dụng:

```bash
docker run --gpus all -p 7860:7860 parakeet-app
```

Giải thích lệnh:
- `docker run`: Lệnh chạy container.
- `--gpus all`: **Quan trọng!** Cho phép container sử dụng tất cả GPU NVIDIA của máy.
- `-p 7860:7860`: Nối cổng 7860 của container với cổng 7860 của máy thật.
- `parakeet-app`: Tên của image chúng ta vừa tạo.

### Bước 3: Sử dụng

Truy cập vào địa chỉ sau trên trình duyệt:

- **Nếu chạy trên máy cá nhân:** [http://localhost:7860](http://localhost:7860)
- **Nếu chạy trên server khác (LAN/VPN):** `http://<IP_CUA_SERVER>:7860`
  - Ví dụ: `http://10.192.214.115:7860`
  - *Lưu ý: Hãy đảm bảo Firewall của server đã mở cổng 7860.*

Bạn sẽ thấy giao diện ứng dụng giống hệt như khi chạy trực tiếp bằng Python, nhưng giờ nó đang chạy gọn gàng trong Docker!

## 4. Các Lệnh Hữu ích Khác

- **Xem các container đang chạy:**
  ```bash
  docker ps
  ```

- **Dừng container:**
  Lấy ID của container từ lệnh trên (ví dụ `a1b2c3d4`), sau đó chạy:
  ```bash
  docker stop a1b2c3d4
  ```
  Hoặc nếu đang chạy ở terminal, chỉ cần nhấn `Ctrl+C`.

- **Xóa image cũ (để giải phóng ổ cứng):**
  ```bash
  docker rmi parakeet-app
  ```

## 5. Khắc phục sự cố

- **Lỗi: `could not select device driver ... with capabilities: [[gpu]]`**
  - *Nguyên nhân:* Bạn chưa cài hoặc chưa cấu hình đúng NVIDIA Container Toolkit.
  - *Khắc phục:* Kiểm tra lại cài đặt Docker Desktop và Nvidia Driver. Trên Windows, hãy chắc chắn bạn đang dùng WSL2 backend.

- **Không truy cập được localhost:7860**
  - Hãy kiểm tra xem bạn đã thêm tham số `-p 7860:7860` chưa.
  - Kiểm tra log trong terminal xem ứng dụng có báo lỗi gì không.

---
*Chúc mừng! Bạn đã biết cách đóng gói ứng dụng AI bằng Docker.*
