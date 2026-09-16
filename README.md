# VNITUBE
Phần mềm Download video miễn phí từ youtube, facebook.. trên windows - VNITUBE PRO. Mình tự code bằng Python kết hợp FFmpeg

**Các trang web được hỗ trợ:**

YouTube 

Tiktok

Vimeo

Facebook

DailyMotion


Download

https://forumviet.com/threads/phan-mem-download-video-mien-phi-tu-youtube-facebook-tren-windows-vnitube-pro.5555/

# Kế hoạch phát triển phần mềm VNITube (Tương tự MassTube cho Windows)

Xây dựng ứng dụng desktop **VNITube** dành riêng cho Windows, lấy cảm hứng từ công cụ huyền thoại **MassTube**: thiết kế gọn gàng, khởi động tức thì, tải nhanh chỉ với một cú nhấp chuột, tiêu thụ cực ít tài nguyên (CPU & RAM), hỗ trợ đầy đủ độ phân giải từ 240p đến 8K 60fps, chuyển đổi đa định dạng (MP4, WebM, FLV, 3GP, MP3, WAV), vượt rào giới hạn độ tuổi không cần đăng nhập, tích hợp quản lý lịch sử và cập nhật thuật toán trích xuất liên tục.

---

## 1. Phân tích Yêu cầu & Kiến trúc Công nghệ

### 1.1 Lựa chọn công nghệ đáp ứng tiêu chí "Tiêu thụ tài nguyên thấp (CPU & RAM)"
- **Giao diện & Ứng dụng Desktop**: **Python 3.13 + PyQt6** (bản build Qt6 C++ native).
  - *Tại sao không dùng Electron?* Electron tiêu thụ từ 150MB - 300MB RAM chỉ để hiển thị giao diện. Với **PyQt6**, ứng dụng chỉ tiêu thụ khoảng **45MB - 65MB RAM**, phản hồi ngay lập tức, cực kỳ nhẹ và tương thích tốt trên Windows 10/11.
- **Lõi trích xuất & tải video (Core Engine)**: **yt-dlp (Python API trực tiếp)**.
  - Tích hợp trực tiếp vào luồng xử lý (in-process `yt_dlp.YoutubeDL`), không cần gọi qua tiến trình con console, giúp trích xuất siêu tốc và nhận dữ liệu tiến trình (progress hook) mượt mà không tốn CPU.
  - Hỗ trợ đầy đủ YouTube, Vevo, Vimeo, Facebook, Dailymotion và hơn 1000 trang web khác.
- **Xử lý Audio/Video & Chuyển đổi định dạng**: **FFmpeg tích hợp (qua `imageio-ffmpeg`)**.
  - Đóng gói sẵn file `ffmpeg.exe` tĩnh 64-bit, người dùng không cần cài đặt thêm bất kỳ phần mềm hay thiết lập biến môi trường nào. Hỗ trợ ghép luồng DASH video 8K/4K + audio và xuất FLV, MP4, WebM, 3GP, MP3, WAV.
- **Đa luồng (Multi-threading)**: Sử dụng `QThread` / `QThreadPool` tách biệt hoàn toàn giữa UI và tiến trình tải, đảm bảo giao diện 60fps mượt mà, không giật lag khi tải tệp dung lượng lớn.

---

## 2. Các Tính Năng Chi Tiết của VNITube

### 2.1 Trích xuất & Tải 1-Click (Fast One-Click Download)
- Tự động nhận diện liên kết video từ Clipboard khi người dùng sao chép liên kết hoặc bấm nút "Dán liên kết (Paste URL)".
- Phân tích thông tin video nhanh chóng: hiển thị Ảnh thu nhỏ (Thumbnail), Tiêu đề, Kênh/Tác giả, Thời lượng, Lượt xem.
- Hiển thị bảng/danh sách tùy chọn chất lượng trực quan:
  - **Video**: 8K (4320p 60fps), 4K (2160p 60fps), 2K (1440p), Full HD (1080p 60fps), HD (720p 60fps), 480p, 360p, 240p, 144p.
  - **Định dạng container**: MP4, WebM, FLV, 3GP.
  - **Âm thanh**: MP3 (320kbps, 192kbps, 128kbps), WAV (Lossless), M4A.
- Nút **"Tải ngay"** hoặc nhấp đúp vào hàng chất lượng để tải ngay tức thì.
- Hàng đợi tải xuống hiển thị: Thanh tiến trình %, Tốc độ (MB/s), Dung lượng đã tải / Tổng dung lượng, Thời gian còn lại (ETA), nút Hủy/Tạm dừng.

### 2.2 Vượt rào giới hạn độ tuổi không cần đăng nhập (Age-Restriction Bypass)
- Sử dụng cơ chế mô phỏng Client chuyên sâu của YouTube (`player_client: ['android', 'ios', 'mweb']` kết hợp cấu hình extractor args tối ưu).
- YouTube cho phép client Android/iOS truy cập stream nội dung giới hạn độ tuổi mà không yêu cầu OAuth/Cookies tài khoản Google.
- Đồng thời cung cấp tùy chọn nhập cookie trình duyệt (Chrome, Edge, Firefox, Brave) cho trường hợp video riêng tư đặc biệt.

### 2.3 Quản lý Lịch sử Tải xuống (Download History Manager)
- Lưu trữ lịch sử tải xuống cục bộ (Title, Thumbnail, Định dạng/Độ phân giải, Dung lượng, Đường dẫn file, Ngày tải, Link gốc).
- Tìm kiếm và lọc lịch sử nhanh.
- **Phát video trực tiếp**: Mở bằng trình phát mặc định của Windows hoặc trình phát bên ngoài tùy chỉnh.
- **Mở thư mục**: Mở Windows Explorer và trỏ chính xác vào file đã tải (`explorer /select, file_path`).
- **Xóa lịch sử**: Cho phép xóa bản ghi trong danh sách hoặc xóa kèm file thực tế trên ổ cứng.
- **Xuất / Nhập lịch sử**: Hỗ trợ Xuất ra file JSON / CSV và Nhập lại từ file JSON.

### 2.4 Hỗ trợ Trình phát Video bên ngoài (External Players)
- Tùy chọn trong Cài đặt:
  - Trình phát mặc định của Windows (Default Player).
  - Trình phát tùy chọn: Cho phép duyệt đường dẫn file `.exe` (VLC Media Player, MPC-HC, PotPlayer, KMPlayer, v.v.).

### 2.5 Cấu hình Proxy
- Hỗ trợ các giao thức mạng: **HTTP, HTTPS, SOCKS5**.
- Nhập Host, Port, Username, Password.
- Nút "Kiểm tra Proxy" để xác thực kết nối internet trước khi tải.

### 2.6 Cơ chế Cập nhật Thường xuyên (Engine Updater)
- Tích hợp chức năng kiểm tra và cập nhật lõi `yt-dlp` trực tiếp trong ứng dụng:
  - Hiển thị phiên bản hiện tại và phiên bản mới nhất từ kho lưu trữ.
  - 1-click cập nhật thuật toán trích xuất mới nhất mà không cần cài đặt lại toàn bộ phần mềm, giúp liên tục thích ứng với các thay đổi mã nguồn từ YouTube.

---

## 3. Cấu trúc Thư mục Dự án

```
d:\Project\Vnitube\
├── .venv/                         # Virtual environment Python (quản lý cô lập)
├── vnitube/
│   ├── __init__.py
│   ├── config.py                  # Quản lý cài đặt (đường dẫn lưu, proxy, player, bypass, v.v.)
│   ├── core/
│   │   ├── __init__.py
│   │   ├── extractor.py           # Phân tích URL, bóc tách danh sách format 8K/4K/1080p/MP3/WAV...
│   │   ├── downloader.py          # Luồng tải nền (QThread), progress hooks, ghép luồng FFmpeg
│   │   ├── history_manager.py     # Quản lý lịch sử (JSON storage, CRUD, xuất CSV/JSON, nhập JSON)
│   │   └── updater.py             # Kiểm tra & cập nhật yt-dlp core engine
│   ├── ui/
│   │   ├── __init__.py
│   │   ├── main_window.py         # Cửa sổ chính với Navigation Bar (Tải xuống, Lịch sử, Cài đặt, Thông tin)
│   │   ├── download_panel.py      # Giao diện tải: URL input, clipboard paste, video info card, format list, active tasks
│   │   ├── history_panel.py       # Bảng lịch sử tải xuống, bộ lọc, các nút phát video, mở thư mục, xuất/nhập
│   │   ├── settings_panel.py      # Cài đặt thư mục tải, proxy, trình phát ngoại, cập nhật yt-dlp, cấu hình bypass
│   │   ├── theme.py               # Giao diện Dark Mode cao cấp (QSS phong cách MassTube hiện đại, bo tròn tinh tế)
│   │   └── widgets/
│   │       ├── __init__.py
│   │       ├── video_card.py      # Thẻ hiển thị thông tin video & thumbnail
│   │       └── task_item.py       # Widget thanh tiến trình từng video đang tải
│   └── utils/
│       ├── __init__.py
│       ├── player.py              # Xử lý mở video bằng player ngoài hoặc Windows default
│       └── helpers.py             # Format bytes, duration, sanitize filename, clipboard
├── main.py                        # Điểm khởi chạy ứng dụng
├── run.bat                        # Script khởi chạy nhanh 1-click cho Windows
├── requirements.txt               # Danh sách dependencies
└── README.md                      # Hướng dẫn sử dụng chi tiết bằng tiếng Việt
```

---

## 4. Kế hoạch Triển khai (Proposed Implementation Steps)

### Bước 1: Khởi tạo Môi trường Ảo & Cài đặt Thư viện
- Tạo môi trường ảo `.venv` theo tiêu chuẩn skill `managing-python-dependencies`.
- Cài đặt các thư viện: `PyQt6`, `yt-dlp`, `imageio-ffmpeg`, `requests`.
- Đóng băng phiên bản vào `requirements.txt`.

### Bước 2: Xây dựng Module Lõi (Core Engine)
- Xây dựng `config.py`: Tự động tải/lưu cấu hình JSON (thư mục lưu mặc định là thư mục Downloads của Windows, proxy, cấu hình player, định dạng ưa thích).
- Xây dựng `extractor.py`:
  - Trích xuất metadata nhanh (không tải trước stream).
  - Tách và phân loại rõ ràng: 8K, 4K, 2K, 1080p, 720p, 480p, 360p, 240p, MP3, WAV.
  - Tích hợp client spoofing (Android, iOS) để tự động vượt rào độ tuổi không cần đăng nhập.
  - Hỗ trợ các trang: YouTube, Vevo, Vimeo, Facebook, DailyMotion và hàng trăm website khác.
- Xây dựng `downloader.py`:
  - `QThread` chạy ngầm, gửi tín hiệu tiến trình theo thời gian thực (tỷ lệ %, tốc độ KB/s hoặc MB/s, ETA, kích thước).
  - Tích hợp FFmpeg từ `imageio-ffmpeg` để hợp nhất (mux) âm thanh và hình ảnh chất lượng cao hoặc chuyển mã sang FLV, MP4, WebM, 3GP, MP3, WAV.
- Xây dựng `history_manager.py`:
  - Quản lý danh sách tải xuống, lưu vào `history.json`.
  - Hàm xuất lịch sử ra JSON và CSV; hàm nhập lịch sử từ JSON.
- Xây dựng `updater.py`:
  - Kiểm tra phiên bản yt-dlp qua GitHub/PyPI API.
  - Cơ chế cập nhật pip ngầm cho core engine.

### Bước 3: Thiết kế Giao diện Đồ họa Hiện đại (Modern UI)
- Tạo `theme.py` với bảng màu Dark Mode sang trọng (phối màu graphite tối, neon blue, accent red đặc trưng của video platforms).
- `main_window.py`: Khung cửa sổ chính với thanh điều hướng (Sidebar/Tab):
  - **📥 Tải xuống (Downloader)**: Nhập link, nút dán clipboard, preview card, bảng chọn độ phân giải/định dạng, danh sách tải đang chạy.
  - **📜 Lịch sử (History)**: Danh mục file đã tải, phát video bằng player ngoài, mở thư mục, tìm kiếm, xuất/nhập lịch sử.
  - **⚙️ Cài đặt (Settings)**: Thư mục lưu mặc định, proxy mạng, chọn trình phát video ngoài, cập nhật yt-dlp, tùy chọn bypass.
  - **ℹ️ Giới thiệu (About)**: Thông tin ứng dụng VNITube, phiên bản, trạng thái FFmpeg và Core engine.

### Bước 4: Kiểm thử & Xác minh (Verification Plan)
1. **Kiểm tra khởi chạy ứng dụng**: Khởi chạy `python main.py`, kiểm tra UI hiển thị đúng, mượt mà, bộ nhớ RAM chỉ ~50-60MB.
2. **Kiểm tra trích xuất link**: Thử nghiệm với các link YouTube, Vimeo, Facebook, DailyMotion để kiểm tra danh sách độ phân giải từ 240p đến 4K/8K và MP3/WAV.
3. **Kiểm tra tải video & trích xuất MP3/WAV**: Tải mẫu và xác nhận file tạo ra có âm thanh + hình ảnh chuẩn, đọc được bằng trình phát video.
4. **Kiểm tra vượt độ tuổi (Age Bypass)**: Thử nghiệm với các video có cờ age-restriction trên YouTube qua client android/ios.
5. **Kiểm tra quản lý lịch sử**: Xem, xóa, mở file, mở thư mục, xuất CSV/JSON và nhập lại.
6. **Kiểm tra cấu hình Proxy**: Nhập proxy và kiểm tra kết nối.
7. **Tạo file `run.bat`**: Đảm bảo người dùng chỉ cần nhấp đúp vào `run.bat` trên Windows là mở được ngay ứng dụng.

---

## 5. Xác nhận & Phản hồi

Bạn có đồng ý với kế hoạch triển khai trên không? Nếu có bất kỳ điều chỉnh nào về tính năng hoặc giao diện, vui lòng cho tôi biết để cập nhật trước khi tiến hành viết mã.
