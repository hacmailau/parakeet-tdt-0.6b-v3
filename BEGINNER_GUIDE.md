# Hướng dẫn Codebase: Parakeet TDT Transcription App for Beginners

Chào mừng! Tài liệu này sẽ giúp bạn hiểu, cài đặt và chạy ứng dụng chuyển đổi giọng nói thành văn bản (Speech-to-Text) sử dụng mô hình Parakeet TDT của NVIDIA.

## 1. Tổng quan Dự án

Dự án này là một ứng dụng web đơn giản (được xây dựng bằng **Gradio**) cho phép người dùng tải lên tệp âm thanh hoặc ghi âm trực tiếp, sau đó sử dụng trí tuệ nhân tạo (**AI**) để chuyển đổi âm thanh đó thành văn bản.

**Công nghệ chính:**
- **Python:** Ngôn ngữ lập trình chính.
- **Gradio:** Thư viện giúp tạo giao diện web (UI) nhanh chóng cho các mô hình AI.
- **NVIDIA NeMo:** Bộ công cụ (Toolkit) để làm việc với các mô hình AI đàm thoại.
- **Parakeet TDT 0.6b:** Mô hình AI cụ thể được sử dụng để nhận dạng giọng nói đa ngôn ngữ.

## 2. Cấu trúc Thư mục

```
parakeet-tdt-0.6b-v3/
├── app.py                # <--- Tệp chính chứa toàn bộ mã nguồn của ứng dụng
├── requirements.txt      # Danh sách các thư viện Python cần cài đặt
├── packages.txt          # Danh sách các gói hệ thống (thường dùng cho Docker/Spaces)
├── README.md             # Giới thiệu chung về dự án
└── data/                 # Thư mục chứa dữ liệu mẫu (ví dụ: file mp3)
```

## 3. Phân tích Mã nguồn (`app.py`)

Tệp `app.py` là nơi mọi thứ diễn ra. Chúng ta hãy chia nhỏ nó ra:

### A. Khởi tạo và Tải Mô hình (Dòng 1-19)
```python
from nemo.collections.asr.models import ASRModel
# ... import các thư viện khác
model = ASRModel.from_pretrained(model_name="nvidia/parakeet-tdt-0.6b-v3")
model.eval()
```
- Đoạn này tải mô hình AI từ Internet về máy. `model.eval()` chuyển mô hình sang chế độ "dự đoán" (không học nữa).

### B. Xử lý Âm thanh (Hàm `get_audio_segment`)
Hàm này dùng thư viện `pydub` để cắt một đoạn âm thanh nhỏ từ file lớn, giúp tính năng "click để nghe lại" hoạt động.

### C. Chức năng Chính (Hàm `get_transcripts_and_raw_times`)
Đây là "trái tim" của ứng dụng, thực hiện các bước:
1.  **Nhận file âm thanh:** Từ người dùng upload hoặc microphone.
2.  **Chuẩn hóa:** Chuyển đổi âm thanh về dạng chuẩn (16000Hz, Mono) để mô hình AI hiểu được.
3.  **Xử lý file dài:** Nếu file > 8 phút, nó bật chế độ tối ưu bộ nhớ.
4.  **Transcribe (Dịch):** Gọi lệnh `model.transcribe()` để AI làm việc.
5.  **Xuất kết quả:** Tạo file CSV và SRT (phụ đề) để người dùng tải về.

### D. Giao diện (Gradio Interface)
Đoạn cuối file thiết lập giao diện người dùng:
- Có 2 tab: "Audio File" (Tải file) và "Microphone" (Ghi âm).
- Nút "Transcribe" kích hoạt hàm xử lý ở trên.
- Bảng kết quả hiển thị thời gian và nội dung văn bản.

## 4. Hướng dẫn Cài đặt & Chạy

Để chạy được dự án này trên máy cá nhân, bạn cần thực hiện các bước sau:

### Yêu cầu:
- **Python 3.8+** đã được cài đặt.
- **FFmpeg:** Cần thiết để xử lý âm thanh (pydub).
  - *Windows:* Tải và thêm vào PATH.
  - *Mac:* `brew install ffmpeg`
  - *Linux:* `sudo apt install ffmpeg`
- **GPU (Card màn hình):** Khuyến nghị dùng NVIDIA GPU để chạy nhanh. Nếu dùng CPU sẽ rất chậm.

### Bước 1: Cài đặt thư viện
Mở terminal (CMD/PowerShell) tại thư mục dự án và chạy:

```bash
# Cài đặt PyTorch trước (truy cập pytorch.org để lấy lệnh cài đúng với cấu hình máy bạn)
# Ví dụ cho Windows có CUDA 11.8:
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# Cài đặt các thư viện khác từ requirements.txt
pip install -r requirements.txt
```

*Lưu ý: Việc cài đặt `nemo_toolkit` và `Cython` có thể hơi phức tạp trên Windows và yêu cầu C++ Build Tools.*

### Bước 2: Chạy ứng dụng
Sau khi cài xong, chạy lệnh:

```bash
python app.py
```

### Bước 3: Sử dụng
1.  Trình duyệt sẽ tự mở địa chỉ (thường là `http://127.0.0.1:7860`).
2.  Chọn tab **Audio File**, tải file mẫu trong thư mục `data/` lên.
3.  Nhấn **Transcribe Uploaded File**.
4.  Chờ AI xử lý và xem kết quả bên dưới.

## 5. Các lỗi thường gặp

1.  **CUDA Out of Memory:**
    - *Nguyên nhân:* GPU của bạn không đủ bộ nhớ VRAM.
    - *Khắc phục:* Thử file âm thanh ngắn hơn hoặc khởi động lại máy để giải phóng VRAM.

2.  **ModuleNotFoundError (pydub, gradio...):**
    - *Nguyên nhân:* Chưa cài đủ thư viện.
    - *Khắc phục:* Chạy lại `pip install -r requirements.txt`.

3.  **FileNotFoundError (liên quan đến FFmpeg):**
    - *Nguyên nhân:* Máy chưa cài hoặc chưa nhận FFmpeg.
    - *Khắc phục:* Cài đặt FFmpeg và kiểm tra bằng lệnh `ffmpeg -version`.

---
*Chúc bạn thành công với dự án Speech-to-Text đầu tiên của mình!*
