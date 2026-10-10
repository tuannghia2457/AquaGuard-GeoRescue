# 2. Công nghệ sử dụng và hạ tầng triển khai

## 2.1. Tóm tắt nhanh

AquaGuard là một **monorepo** gồm ba phần: giao diện web, máy chủ API, và cấu
hình hạ tầng. Hệ thống không tự vận hành máy chủ vật lý — mọi thành phần đều
chạy trên **dịch vụ được quản lý (managed service)** với gói miễn phí hoặc chi
phí thấp, mỗi thành phần nằm ở một nhà cung cấp phù hợp nhất với đặc thù của nó.

```
                     Người dùng (trình duyệt / điện thoại)
                                     │
                        HTTPS + WebSocket (WSS)
                                     │
          ┌──────────────────────────┴──────────────────────────┐
          │                                                     │
   Giao diện web (SPA)                            Máy chủ API + WebSocket
   React 19 + Vite                                Node.js 20 + Express
   ▸ Vercel — aquaguard.vn                        ▸ Render — aquaguard-api.onrender.com
          │                                                     │
          │                             ┌───────────────────────┼───────────────────────┐
          │                             │                       │                       │
          │                      PostgreSQL (Neon)        Redis (Upstash)         Dịch vụ ngoài
          │                      dữ liệu chính            vị trí trực tiếp        Resend · Twilio
          │                                                (TTL 60 giây)          Cloudinary · Cloudflare TURN
          │
   Dịch vụ gọi trực tiếp từ trình duyệt:
   Firebase (đăng nhập Google + vùng ngập) · Groq (chatbot) · Windy (thời tiết)
   OpenStreetMap (nền bản đồ) · Nominatim (tra địa chỉ) · OSRM (định tuyến)
```

---

## 2.2. Công nghệ phía giao diện (frontend)

| Thành phần | Công nghệ | Vai trò |
| --- | --- | --- |
| Thư viện giao diện | **React 19** | Toàn bộ giao diện dạng component |
| Công cụ build | **Vite 6** | Máy chủ phát triển siêu nhanh + đóng gói bản production |
| CSS | **Tailwind CSS v4** (qua plugin Vite) | Toàn bộ styling theo utility class, hỗ trợ sáng/tối |
| Điều hướng | **React Router DOM 7** | Định tuyến ở mức trình duyệt |
| Bản đồ | **Leaflet 1.9 + React-Leaflet 5** | Bản đồ, marker, đường đi, vùng ngập |
| Biểu đồ | **Recharts 3** | Biểu đồ trong trang thống kê |
| Hướng dẫn | **react-joyride 3** | Tour giới thiệu cho người dùng mới |
| Đăng nhập Google & dữ liệu vùng ngập | **Firebase 12** (Auth + Firestore) | Đăng nhập Google tuỳ chọn; lưu vùng ngập do quản trị vẽ |

Đặc điểm: dự án dùng **JavaScript thuần (không TypeScript)**, module dạng ESM,
không cấu hình linter, không có framework kiểm thử. Yêu cầu Node.js ≥ 20.

## 2.3. Công nghệ phía máy chủ (backend)

| Thành phần | Công nghệ | Vai trò |
| --- | --- | --- |
| Nền tảng | **Node.js 20** | Môi trường chạy |
| Web framework | **Express 4** | REST API |
| Thời gian thực | **ws** (WebSocket thuần) | Vị trí trực tiếp + tín hiệu cuộc gọi, dùng chung cổng với HTTP |
| Cơ sở dữ liệu | **node-postgres (`pg`)** | Truy vấn **SQL thuần**, không dùng ORM |
| Bộ nhớ nóng | **ioredis** | Vị trí trực tuyến, hiện diện, truy vấn theo bán kính |
| Xác thực | **jsonwebtoken** + **bcrypt** | Token phiên 7 ngày, băm mật khẩu |
| Email | **resend** | Email giao dịch |
| SMS OTP | **twilio** (Verify) | Mã xác thực khi quên mật khẩu |
| Ảnh | **cloudinary** + **multer** | Nhận ảnh trong bộ nhớ rồi đẩy lên kho ảnh |

Backend viết theo **CommonJS**, logic nghiệp vụ nằm trực tiếp trong các route
xử lý (không có lớp service/model riêng).

---

## 2.4. Nơi triển khai từng phần

| Phần | Dịch vụ | Địa chỉ / định danh | Ghi chú vận hành |
| --- | --- | --- | --- |
| **Giao diện web** | **Vercel** | `aquaguard.vn`, `www.aquaguard.vn` và tên miền `*.vercel.app` mặc định | Tự động triển khai mỗi khi đẩy mã lên nhánh `main`. Có cấu hình chuyển hướng mọi đường dẫn về trang gốc (kiểu ứng dụng một trang) |
| **API + WebSocket** | **Render** | `aquaguard-api.onrender.com` | Một tiến trình Node duy nhất phục vụ cả REST lẫn WebSocket. Gói miễn phí **tự ngủ sau ~15 phút không có lưu lượng** |
| **Cơ sở dữ liệu** | **Neon** (PostgreSQL serverless) | Kết nối qua chuỗi kết nối trong biến môi trường | Mã nguồn tự bật SSL khi máy chủ không phải localhost |
| **Bộ nhớ nóng** | **Upstash** (Redis serverless) | Kết nối `rediss://…` | **Tuỳ chọn** — không cấu hình thì hệ thống tự chuyển sang dùng PostgreSQL cho vị trí |
| **Email** | **Resend** | Tên miền gửi `aquaguard.vn` (đã xác thực DNS) | Người gửi: `AquaGuard <no-reply@aquaguard.vn>` |
| **Giữ máy chủ thức** | **UptimeRobot** | Gọi `/api/health` mỗi 5 phút | Chống việc gói miễn phí của Render ngủ đông, tránh cho người dùng thật phải chờ máy chủ khởi động lại |
| **SMS OTP** | **Twilio Verify** | Dịch vụ Verify | Gửi và kiểm mã OTP, không tự lưu mã |
| **Lưu trữ ảnh** | **Cloudinary** | Thư mục ảnh SOS | Tự tối ưu chất lượng và định dạng ảnh |
| **Đăng nhập Google + vùng ngập** | **Firebase** (Auth + Firestore) | Dự án Firebase | Vùng ngập do quản trị vẽ được đồng bộ thời gian thực về mọi máy đang mở bản đồ |
| **Trợ lý AI** | **Groq** | Mô hình Llama 3.3 70B | Gọi trực tiếp từ trình duyệt; có bộ trả lời dự phòng khi lỗi |
| **Chuyển tiếp cuộc gọi** | **Cloudflare TURN** (hoặc TURN tĩnh) + STUN công cộng | Thông tin đăng nhập được máy chủ cấp phát ngắn hạn | Không cấu hình vẫn chạy được với STUN trong các mạng thông thường |
| **Nền bản đồ & tiện ích bản đồ** | OpenStreetMap (ô bản đồ), **Nominatim** (tra địa chỉ), **OSRM** (định tuyến), **Windy** (lớp thời tiết) | Dịch vụ công cộng miễn phí | Gọi trực tiếp từ trình duyệt |

**Nguyên tắc về bí mật cấu hình**: mọi khoá và mật khẩu đều nằm ở **biến môi
trường trên từng nền tảng** (Render cho backend, Vercel cho frontend), **không
bao giờ nằm trong mã nguồn**. Thay đổi biến môi trường bắt buộc phải triển khai
lại thì mới có hiệu lực.

**Về CORS**: danh sách tên miền được phép gọi API được khai báo cứng trong mã
backend (kèm khả năng bổ sung một tên miền qua biến môi trường). Thêm một tên
miền frontend mới thì phải cập nhật danh sách này.

---

## 2.5. Biến môi trường

Mỗi gói có tệp cấu hình riêng, đều **không được đưa vào Git** (có sẵn tệp mẫu).

### Backend

| Biến | Dùng để |
| --- | --- |
| `DATABASE_URL` | Chuỗi kết nối PostgreSQL |
| `JWT_SECRET` | Khoá ký token phiên đăng nhập |
| `REDIS_URL` | Kết nối Redis (bỏ trống = tắt bộ nhớ nóng) |
| `PORT` | Cổng máy chủ (mặc định 5001) |
| `CLOUDINARY_URL` | Kho ảnh |
| `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_VERIFY_SERVICE_SID` | SMS OTP |
| `RESEND_API_KEY`, `EMAIL_FROM` | Email giao dịch |
| `FRONTEND_URL` | Bổ sung một tên miền vào danh sách CORS |
| `ROLE_PASSWORD` | Mã vai trò khi đăng ký tài khoản cứu hộ / quản trị |
| `CF_TURN_KEY_ID`, `CF_TURN_API_TOKEN` **hoặc** `TURN_URLS`, `TURN_USERNAME`, `TURN_CREDENTIAL` | Máy chủ chuyển tiếp cho cuộc gọi (tuỳ chọn) |
| `NODE_ENV` | Bật chế độ production (ảnh hưởng SSL và CORS khi phát triển) |

### Frontend

Tất cả biến của frontend đều mang tiền tố `VITE_` và **được đóng gói vào mã chạy
trên trình duyệt** — tuyệt đối không đặt bí mật thật vào đây.

| Biến | Dùng để |
| --- | --- |
| `VITE_API_BASE_URL` | Địa chỉ gốc của API |
| `VITE_WS_URL` | Địa chỉ WebSocket |
| `VITE_FIREBASE_*` (6 biến) | Cấu hình Firebase phía client |
| `VITE_GROQ_API_KEY` | Trợ lý AI |
| `VITE_WINDY_API_KEY` | Lớp thời tiết trên bản đồ |
| `VITE_OWM_API_KEY` | Ô bản đồ thời tiết bổ sung |
| `VITE_GOOGLE_MAPS_API_KEY` | Tra địa chỉ (phương án phụ) |

---

## 2.6. Môi trường phát triển

### Cách 1 — Docker Compose (khuyến nghị, dựng cả hệ thống)

Tệp compose nằm trong thư mục hạ tầng (**không phải ở gốc kho mã**). Bốn dịch
vụ được dựng lên:

| Dịch vụ | Ảnh | Cổng trên máy |
| --- | --- | --- |
| PostgreSQL | `postgres:16-alpine` | **5433** (tránh đụng PostgreSQL cài sẵn) |
| Redis | `redis:7-alpine` | 6379 |
| Backend | build từ mã nguồn, chạy chế độ tự nạp lại | 5001 |
| Frontend | build từ mã nguồn, chạy Vite | 5173 |

PostgreSQL **tự chạy tệp khởi tạo lược đồ trong lần khởi động đầu tiên**, nên
chỉ cần dựng lên là đã có đủ bảng và dữ liệu mẫu. Backend và frontend đều gắn
mã nguồn từ máy thật vào container để sửa file là thấy đổi ngay.

Có sẵn một Makefile với các lệnh tắt (bật/tắt/dựng lại, xem log từng dịch vụ,
mở shell, mở psql, xoá và tạo lại cơ sở dữ liệu, chạy một tệp migration).
Lưu ý: các lệnh Makefile phải chạy **từ trong thư mục hạ tầng**.

**Bẫy thường gặp**: khi thêm một thư viện mới cho backend, cài trên máy thật là
chưa đủ — container giữ thư mục thư viện riêng của nó, phải dựng lại container
kèm làm mới ổ đĩa ẩn danh. Đổi tệp biến môi trường của backend cũng cần tạo lại
container chứ khởi động lại không nạp lại được.

### Cách 2 — chạy trực tiếp từng gói

Cài thư viện rồi chạy backend ở cổng 5001 và frontend ở cổng 5173. Cách này cần
tự trỏ cơ sở dữ liệu tới một PostgreSQL sẵn có (hoặc Neon).

### Kiểm chứng thay đổi

Dự án **không có bộ kiểm thử tự động và không có linter**. "Kiểm chứng" nghĩa là
chạy thật và thao tác qua luồng; riêng với thay đổi giao diện thì có thể chạy
lệnh build của frontend để chắc chắn mã biên dịch được.

---

## 2.7. Quản lý cơ sở dữ liệu

- Tệp lược đồ trong thư mục hạ tầng là **nguồn chân lý duy nhất** cho cấu trúc
  cơ sở dữ liệu; nó cũng chính là tệp mà container PostgreSQL tự chạy lần đầu.
- Thư mục migration của backend chứa các tệp SQL **áp dụng thủ công** — dự án
  không dùng công cụ chạy migration tự động. Vì vậy mỗi thay đổi lược đồ phải
  được phản ánh **cả trong tệp migration lẫn trong tệp lược đồ gốc**, nếu không
  thì một môi trường Docker dựng mới sẽ thiếu cột.

---

## 2.8. Đặc thù vận hành cần biết

- **Máy chủ ngủ đông**: gói miễn phí Render tắt tiến trình sau khoảng 15 phút
  không có lưu lượng; lần gọi kế tiếp phải chờ khởi động lại. UptimeRobot ping
  định kỳ chính là để tránh việc này.
- **Redis là tuỳ chọn**: nếu Redis chết hoặc không cấu hình, hệ thống **không
  sập** — phần vị trí trực tiếp giảm chất lượng và quay về đọc vị trí lưu trong
  PostgreSQL.
- **Email không bao giờ chặn nghiệp vụ**: mọi lệnh gửi email đều chạy nền và tự
  nuốt lỗi, nên dịch vụ email trục trặc cũng không làm hỏng việc đăng ký hay
  hoàn tất cứu hộ. Email cảnh báo lũ hàng loạt được gửi rải (khoảng 3 thư/giây)
  để không vượt hạn mức nhà cung cấp.
- **Trạng thái trong bộ nhớ tiến trình**: các phòng theo dõi, sổ đăng ký kết nối
  và bộ đếm giới hạn tần suất đều nằm trong RAM của tiến trình. Hệ quả: khởi
  động lại máy chủ sẽ reset chúng, và kiến trúc hiện tại phù hợp với **một
  tiến trình duy nhất** — muốn chạy nhiều bản sao thì cần chuyển các phần này
  sang một kho dùng chung.
- **Yêu cầu HTTPS cho cuộc gọi**: trình duyệt chỉ cho phép truy cập micro trong
  ngữ cảnh bảo mật, nên tính năng gọi chỉ hoạt động trên tên miền HTTPS hoặc
  localhost.
