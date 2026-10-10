# 3. Kiến trúc hệ thống

## 3.1. Bức tranh tổng thể

AquaGuard theo mô hình **ba tầng kinh điển** — máy khách → tầng ứng dụng không
trạng thái → tầng dữ liệu — được mở rộng thêm **một kênh thời gian thực riêng**
cho việc theo dõi vị trí và tín hiệu cuộc gọi.

```
┌──────────────────────────── TẦNG GIAO DIỆN ────────────────────────────┐
│  React SPA (Vercel)                                                    │
│  ├─ Cây provider: Ngôn ngữ → Xác thực → Vị trí trực tiếp → Toast        │
│  │                → Cuộc gọi → Thông báo                               │
│  ├─ Một route duy nhất được bảo vệ, điều hướng nội bộ theo "trang"      │
│  └─ Gọi thẳng vài dịch vụ ngoài: Firebase, Groq, OSRM, Nominatim, Windy │
└────────────┬────────────────────────────────────┬──────────────────────┘
             │ REST (JSON, JWT ở header)          │ WebSocket (JWT ở query)
             ▼                                    ▼
┌──────────────────────────── TẦNG ỨNG DỤNG ─────────────────────────────┐
│  Một tiến trình Node (Render) — HTTP và WebSocket dùng chung một cổng   │
│                                                                        │
│  REST:  xác thực · SOS · gia đình · vị trí · thông báo · thống kê       │
│         · xuất dữ liệu · cấu hình cuộc gọi                             │
│  Chặn trước: CORS → giới hạn tần suất → xác thực token → kiểm vai trò   │
│  WebSocket: hiện diện vị trí · phòng theo dõi từng nhiệm vụ            │
│             · chuyển tiếp tín hiệu cuộc gọi                            │
└────────────┬───────────────────────────────┬───────────────────────────┘
             │                               │
             ▼                               ▼
   ┌───────────────────┐            ┌──────────────────┐
   │ PostgreSQL (Neon) │            │  Redis (Upstash) │
   │ nguồn dữ liệu gốc │            │  dữ liệu nóng    │
   │ 11 bảng           │            │  TTL 60 giây     │
   └───────────────────┘            └──────────────────┘
             │
             ▼
   Dịch vụ ngoài do backend gọi: Resend · Twilio · Cloudinary · Cloudflare TURN
```

### Nguyên tắc thiết kế

1. **Tầng ứng dụng không giữ phiên**: mỗi yêu cầu tự mang theo token, máy chủ
   không lưu session. Nhờ vậy có thể khởi động lại tiến trình bất cứ lúc nào.
   Trạng thái duy nhất trong bộ nhớ là tập kết nối thời gian thực đang mở —
   thứ vốn dĩ đã là tạm thời.
2. **Tách dữ liệu "nóng" và dữ liệu "bền"**: toạ độ thay đổi liên tục ghi vào
   Redis với thời hạn ngắn; PostgreSQL chỉ giữ các mốc quan trọng. Nhờ đó đường
   ghi nặng nhất của hệ thống không đè lên cơ sở dữ liệu quan hệ.
3. **Suy giảm êm**: Redis chết → quay về đọc PostgreSQL. Email lỗi → nghiệp vụ
   vẫn xong. Không có TURN → cuộc gọi vẫn chạy trên STUN trong đa số mạng.
4. **Quyền được kiểm ở cả hai đầu**: giao diện ẩn thứ không được phép, máy chủ
   kiểm tra lại độc lập.

---

## 3.2. Kiến trúc backend

### 3.2.1. Cấu trúc

```
backend/
├── index.js          điểm vào duy nhất: CORS, giới hạn tần suất, gắn router,
│                     tạo máy chủ HTTP + WebSocket dùng chung một cổng
├── db.js             một kết nối gộp PostgreSQL dùng chung toàn hệ thống
├── redisClient.js    một kết nối Redis dùng chung + các hàm tiện ích vị trí
├── middleware/
│   ├── auth.js       kiểm token, kiểm vai trò
│   └── rateLimit.js  giới hạn tần suất trong bộ nhớ
├── routes/           logic nghiệp vụ nằm trực tiếp trong handler, SQL thuần
│   ├── auth.js       tài khoản, hồ sơ, OTP, quản lý người dùng, đội cứu hộ
│   ├── sos.js        vòng đời yêu cầu cứu hộ
│   ├── family.js     kết nối người thân, trạng thái an toàn, vị trí
│   ├── locations.js  vị trí trực tiếp, tìm quanh đây
│   ├── notifications.js  thông báo trong ứng dụng, phát cảnh báo
│   ├── analytics.js  thống kê cho quản trị
│   ├── export.js     xuất CSV có ẩn danh
│   └── rtc.js        cấp máy chủ chuyển tiếp cho cuộc gọi
├── utils/            email, thông báo, tải ảnh
└── migrations/       các tệp SQL áp dụng thủ công
```

Một điểm cần nhớ: **lớp xác thực chỉ giải mã ra ba thông tin** — mã người dùng,
số điện thoại và vai trò. Mọi thứ khác (tên, email…) đều phải truy vấn lại từ
cơ sở dữ liệu, để tránh dữ liệu cũ nằm mãi trong token 7 ngày.

### 3.2.2. Bản đồ API

| Nhóm | Nội dung | Ai gọi được |
| --- | --- | --- |
| `/api/auth` | Đăng ký, đăng nhập, hồ sơ, OTP quên mật khẩu, danh sách người dùng, đổi vai trò, xoá tài khoản, toàn bộ quản lý đội cứu hộ và lời mời | Công khai (đăng ký/đăng nhập/OTP), còn lại theo vai trò |
| `/api/sos` | Tạo yêu cầu, yêu cầu của tôi, tất cả yêu cầu, yêu cầu của đội, thống kê, giao đội, nhận, trả lại, hoàn tất | Người dân tạo; cứu hộ nhận/trả/hoàn tất; quản trị giao/hoàn tất |
| `/api/family` | Tìm theo số điện thoại, gửi/chấp nhận/từ chối lời mời, danh sách người thân, cập nhật trạng thái an toàn và vị trí | Mọi người dùng đã đăng nhập |
| `/api/locations` | Vị trí trực tiếp theo vai trò, vị trí một người, tìm người gần đây | Mọi người dùng đã đăng nhập |
| `/api/notifications` | Danh sách, đánh dấu đã đọc, xoá; quản trị gửi thông báo và cảnh báo lũ | Người dùng cho phần của mình; quản trị cho phần phát tin |
| `/api/analytics` | Tổng quan, tăng trưởng người dùng, xu hướng cứu hộ | Quản trị |
| `/api/export` | Xuất từng tập dữ liệu ra CSV | Quản trị |
| `/api/rtc` | Danh sách máy chủ STUN/TURN cho cuộc gọi | Người dùng đã đăng nhập |
| `/api/health` | Kiểm tra sống | Công khai (dịch vụ giám sát dùng) |

Thứ tự đi qua các lớp chặn: **CORS → giới hạn tần suất (chỉ ở các điểm nhạy
cảm) → xác thực token → kiểm vai trò → handler**.

### 3.2.3. Kênh thời gian thực

Máy chủ WebSocket chạy **trên chính máy chủ HTTP** (cùng cổng). Máy khách gắn
token vào chuỗi truy vấn lúc kết nối; token sai là đóng kết nối ngay. Một người
dùng có thể có **nhiều kết nối song song** (nhiều tab, nhiều mục đích), nên máy
chủ giữ một sổ đăng ký ánh xạ người dùng → tập kết nối.

Có ba loại lưu lượng đi trên cùng kênh này:

**1. Hiện diện vị trí (luôn bật).** Chỉ cần đăng nhập là trình duyệt mở một kết
nối chuyên trách và bơm toạ độ đều đặn. Máy chủ ghi vào **Redis**: một bản ghi
vị trí có thời hạn 60 giây (còn bản ghi = còn online) và một tập dữ liệu không
gian theo vai trò để truy vấn "ai đang ở gần đây". **Luồng này không bao giờ
ghi vào PostgreSQL.** Để tiết kiệm hạn mức, chỉ ghi khi đã di chuyển đủ xa hoặc
đã quá 2 giây kể từ lần ghi trước.

**2. Phòng theo dõi từng nhiệm vụ.** Khi mở bản đồ theo dõi, máy khách mở một
kết nối ngắn hạn và tham gia một "phòng" theo mã yêu cầu. Toạ độ được phát trực
tiếp giữa hai bên trong phòng. **Chỉ luồng này mới ghi vào PostgreSQL**, và chỉ
ghi **điểm đầu phiên và điểm cuối phiên** — đủ để giữ lại "vị trí biết được gần
nhất" khi Redis hết hạn hoặc máy chủ khởi động lại. Phòng tự dọn khi không còn
ai.

Các mốc đổi trạng thái (bắt đầu theo dõi, kết thúc, bị trả lại) cũng được các
handler REST phát vào phòng, nên giao diện của cả hai bên cập nhật tức thì mà
không cần chờ vòng hỏi lại.

**3. Tín hiệu cuộc gọi.** Máy chủ chỉ làm nhiệm vụ chuyển tiếp: khi một bên bắt
đầu cuộc gọi, máy chủ kiểm tra **một lần** rằng người gọi và người nhận đúng là
hai bên của một nhiệm vụ đang hoạt động, ghi nhận phiên gọi đó, rồi từ đó chỉ
việc chuyển tiếp các gói tín hiệu qua lại mà không phải truy vấn lại cơ sở dữ
liệu. **Âm thanh không đi qua máy chủ** — hai thiết bị nối trực tiếp với nhau.
Khi một bên mất toàn bộ kết nối, phiên gọi được huỷ và bên kia được thông báo;
nhưng chỉ đóng bản đồ theo dõi thì cuộc gọi vẫn tiếp tục vì kết nối tín hiệu là
kết nối riêng, luôn mở.

Ngoài ra máy chủ có nhịp tim 30 giây để dọn các kết nối chết.

### 3.2.4. Mô hình dữ liệu

Mười một bảng, tất cả dùng khoá chính tự tăng và mốc thời gian có múi giờ.

| Bảng | Giữ cái gì | Quan hệ chính |
| --- | --- | --- |
| `users` | Danh tính trung tâm: đăng nhập, hồ sơ, vai trò, trạng thái an toàn, ghi chú sức khoẻ, khoá đặt lại mật khẩu | Được hầu hết bảng khác tham chiếu |
| `rescue_requests` | Yêu cầu cứu hộ: mô tả, toạ độ, ảnh, mức khẩn cấp, trạng thái, đội/người phụ trách, các mốc thời gian | → `users`, → `rescue_groups` |
| `rescue_request_logs` | Nhật ký mọi lần đổi trạng thái (ai, từ gì sang gì, ghi chú) | → `rescue_requests`, → `users` |
| `rescue_groups` | Đội cứu hộ: tên, mô tả, nhóm trưởng, trạng thái hoạt động | → `users` |
| `rescue_group_members` | Bảng nối N–N giữa người và đội, kèm cấp bậc (trưởng/phó/thành viên) và tình trạng tham gia | → `rescue_groups`, → `users` |
| `rescue_group_invites` | Lời mời vào đội và kết quả phản hồi | → `rescue_groups`, → `users` |
| `family_connections` | Liên kết người thân hai chiều kèm quan hệ và trạng thái duyệt | → `users` (hai đầu) |
| `notifications` | Thông báo trong ứng dụng, có cờ đã đọc và dữ liệu kèm theo dạng JSON | → `users` |
| `news_articles` | Bài viết do quản trị đăng (đã có bảng, giao diện chưa dùng) | → `users` |
| `audit_logs` | Nhật ký hành động dạng chỉ ghi thêm, có ảnh trước/sau và địa chỉ IP | → `users` |
| `user_locations` | Vị trí bền vững: **một dòng cho mỗi người**, luôn ghi đè | → `users` |

Điểm đáng chú ý về thiết kế:

- **Không phi chuẩn hoá tên**: các bảng chỉ lưu mã tham chiếu; tên hiển thị được
  lấy bằng phép nối bảng khi truy vấn, nên đổi tên ở một chỗ là đổi khắp nơi.
- **Xoá an toàn**: các khoá ngoại đặt hành vi *đặt về rỗng* hoặc *xoá theo* hợp
  lý, nên xoá một tài khoản không làm hỏng dữ liệu lịch sử.
- **Chỉ mục có điều kiện** cho các truy vấn nóng (yêu cầu đang chờ, thông báo
  chưa đọc, bài viết đã xuất bản).
- **Trigger tự cập nhật** cột thời điểm sửa đổi ở các bảng chính.
- Người dùng đăng ký bằng **số điện thoại**; **email là tuỳ chọn** — mọi tính
  năng email đều tự bỏ qua nếu người nhận không có email.

### 3.2.5. Chiến lược "nóng – bền"

| Loại dữ liệu | Nơi ghi | Vòng đời |
| --- | --- | --- |
| Toạ độ khi đang di chuyển | Redis | 60 giây, làm mới liên tục |
| Ai đang trực tuyến | Redis (chính sự tồn tại của bản ghi) | 60 giây |
| Truy vấn "gần đây" theo bán kính | Redis (cấu trúc không gian) | Đối chiếu lại với bản ghi hiện diện để loại người đã offline |
| Vị trí biết được gần nhất | PostgreSQL | Vĩnh viễn, ghi đè |
| Điểm đầu/cuối một phiên theo dõi | PostgreSQL | Vĩnh viễn |
| Mọi dữ liệu nghiệp vụ khác | PostgreSQL | Vĩnh viễn |

Khi trả về danh sách yêu cầu, backend **đắp vị trí Redis lên trên** toạ độ trong
cơ sở dữ liệu cho các yêu cầu còn đang mở, nên bản đồ luôn thấy vị trí mới nhất
mà vẫn không phải ghi liên tục xuống đĩa.

---

## 3.3. Kiến trúc frontend

### 3.3.1. Cấu trúc

```
frontend/src/
├── App.jsx           cây provider + khai báo route
├── contexts/         trạng thái toàn cục
│   ├── AuthContext        phiên đăng nhập, đăng ký, đăng nhập, đăng xuất
│   ├── LiveLocationContext kết nối vị trí luôn bật
│   ├── CallContext        kết nối tín hiệu cuộc gọi luôn bật + giao diện gọi
│   ├── NotificationContext danh sách thông báo, số chưa đọc, hỏi lại định kỳ
│   └── LanguageContext    chuyển Việt/Anh
├── config/
│   ├── rbac.js       khai báo tập trung: vai trò, menu, quyền vào từng trang
│   └── firebase.js   khởi tạo Firebase (chỉ khi cần)
├── pages/            màn hình, có thư mục con theo vai trò (admin/rescuer/citizen)
├── components/       bản đồ, cứu hộ, gọi, thông báo, bố cục, giám sát, chat…
├── hooks/            theo dõi cứu hộ qua WebSocket, điều khiển cuộc gọi
├── services/         lớp gọi API
├── translations/     tệp tiếng Việt và tiếng Anh
└── utils/            lưu phiên, đồng bộ GPS, chuẩn hoá số điện thoại, chuông…
```

### 3.3.2. Cách điều hướng

Ứng dụng cố tình dùng **rất ít route thật**: đăng nhập, quên mật khẩu, trang báo
không đủ quyền, và **một route gốc được bảo vệ**. Bên trong route gốc là một
"khung ứng dụng" chứa thanh bên (máy tính), thanh dưới (điện thoại) và vùng nội
dung; việc chuyển màn hình là **đổi trang đang hiển thị trong khung**, không
phải đổi URL.

Ưu điểm của lựa chọn này là các kết nối luôn-bật (vị trí, cuộc gọi, thông báo)
không bị ngắt khi người dùng đi lại giữa các màn hình. Đánh đổi là URL không
phản ánh màn hình đang xem (không chia sẻ được đường dẫn sâu).

### 3.3.3. Phân quyền phía giao diện

Toàn bộ menu, khả năng vào từng trang, nhãn và màu của vai trò được khai báo
**tập trung ở một tệp cấu hình duy nhất**. Từ đó suy ra:

- Thanh bên và thanh dưới chỉ dựng những mục vai trò hiện tại được thấy.
- Mỗi lần chuyển màn hình đều kiểm tra quyền; không đủ quyền thì tự đưa về màn
  hình mặc định của vai trò (người dân → trang chủ, cứu hộ → nhiệm vụ của đội,
  quản trị → bảng điều khiển).
- Nếu vai trò thay đổi giữa chừng (bị quản trị đổi), màn hình đang mở sẽ tự
  chuyển về mặc định.

Đây là lớp **tiện dụng**, không phải lớp bảo mật — bảo mật thật nằm ở máy chủ.

### 3.3.4. Trạng thái toàn cục và thứ tự lồng nhau

Thứ tự các provider có ý nghĩa: **ngôn ngữ → xác thực → vị trí trực tiếp →
thông báo nổi → cuộc gọi → thông báo**. Xác thực phải bọc ngoài vì hai kết nối
luôn-bật cần token; các provider cần hiển thị thông báo nổi phải nằm trong lớp
cung cấp thông báo nổi.

Về lưu phiên: phiên đăng nhập bằng số điện thoại được giữ ở phạm vi **tab**
(mỗi tab một phiên, tiện khi cần mở nhiều vai trò để thử), còn phiên Google
được giữ ở phạm vi trình duyệt. Khi khởi động lại, ứng dụng ưu tiên khôi phục
phiên số điện thoại trước, sau đó mới hỏi Firebase.

### 3.3.5. Đa ngôn ngữ

Mọi chuỗi hiển thị đều tra qua khoá dịch trong hai tệp Việt/Anh. Nguyên tắc bắt
buộc khi phát triển: **thêm chuỗi mới thì phải thêm vào cả hai tệp với cùng một
khoá**, không viết cứng chữ trong giao diện.

### 3.3.6. Truyền dữ liệu

- **REST** cho hầu hết dữ liệu, kèm token ở tiêu đề.
- **WebSocket** cho vị trí và tín hiệu cuộc gọi.
- **Hỏi lại định kỳ** cho các danh sách, với nhịp khác nhau theo mức khẩn (xem
  bảng nhịp ở tài liệu số 1).
- **Sự kiện trong trình duyệt** để các thành phần rời rạc nói chuyện với nhau:
  ví dụ khi một yêu cầu vừa được tạo, được nhận hay hoàn tất, một sự kiện được
  phát ra để các màn hình khác tự làm mới.

---

## 3.4. Một yêu cầu đi qua hệ thống như thế nào

Ví dụ: người dân gửi một yêu cầu cứu hộ kèm ảnh.

```
Trình duyệt
  │ 1. lấy GPS (đã làm nóng sẵn) → tra ngược ra địa chỉ (dịch vụ bản đồ mở)
  │ 2. gói mô tả + mức khẩn cấp + ảnh thành một biểu mẫu nhiều phần
  ▼
Máy chủ — lớp chặn
  │ 3. kiểm CORS → kiểm token → kiểm vai trò phải là người dân
  ▼
Máy chủ — xử lý
  │ 4. nhận ảnh vào bộ nhớ (giới hạn 5 ảnh, mỗi ảnh 5 MB, chỉ nhận ảnh)
  │ 5. đẩy song song lên kho ảnh, nhận về các đường dẫn
  │ 6. ghi một dòng yêu cầu mới ở trạng thái chờ xử lý
  │ 7. ghi một dòng nhật ký trạng thái
  ▼
Trả về cho trình duyệt
  │ 8. giao diện tự làm mới nhanh, phát một sự kiện nội bộ
  ▼
Các phía khác
  • Bản đồ của mọi người thấy điểm mới ở vòng hỏi lại kế tiếp
  • Màn hình điều phối của quản trị và danh sách của đội cứu hộ thấy yêu cầu mới
  • Khi có đội nhận: thông báo trong ứng dụng + email tới người dân,
    phòng theo dõi được mở, và vị trí bắt đầu chảy qua kênh thời gian thực
```

---

## 3.5. Bảo mật

| Lớp | Cách làm |
| --- | --- |
| Mật khẩu | Băm bằng bcrypt trước khi lưu; tối thiểu 6 ký tự |
| Phiên | Token ký số, hạn 7 ngày, chỉ chứa mã người dùng / số điện thoại / vai trò |
| Phân quyền | Kiểm ở giao diện (ẩn menu) **và** kiểm lại ở máy chủ trên từng điểm cuối |
| Quyền theo dữ liệu | Nhiều thao tác còn ràng buộc ngay trong câu lệnh SQL (ví dụ chỉ cập nhật được yêu cầu đang do chính mình phụ trách), nên đoán mã yêu cầu cũng không chiếm được nhiệm vụ của người khác |
| Chống dò | Giới hạn tần suất theo địa chỉ mạng ở đăng nhập, đăng ký, quên mật khẩu, phát thông báo và xuất dữ liệu; riêng OTP còn giới hạn theo từng số điện thoại |
| Nâng vai trò | Đăng ký tài khoản cứu hộ/quản trị cần mã vai trò do đơn vị vận hành cấp |
| CORS | Danh sách tên miền được phép khai báo tường minh; chế độ phát triển nới thêm cho địa chỉ mạng nội bộ |
| Tải tệp | Chỉ nhận ảnh, giới hạn dung lượng và số lượng, không ghi xuống đĩa máy chủ |
| Cuộc gọi | Thông tin đăng nhập máy chủ chuyển tiếp được cấp phát phía máy chủ, không nhúng vào mã trình duyệt; chỉ hai bên của nhiệm vụ đang hoạt động mới gọi được nhau |
| Dữ liệu cá nhân | Chức năng xuất có chế độ ẩn danh (bỏ tên/điện thoại/email/mô tả/IP, ngày sinh còn năm, toạ độ làm tròn ~1 km) |
| Bí mật cấu hình | Nằm ở biến môi trường trên nền tảng, không có trong kho mã |

**Điểm cần lưu ý**: mọi biến của frontend đều lộ ra trong gói mã tải về trình
duyệt. Vì vậy các khoá dịch vụ gọi trực tiếp từ trình duyệt (trợ lý AI, bản đồ
thời tiết) nên là khoá có giới hạn tên miền hoặc hạn mức, không phải khoá đặc quyền.

---

## 3.6. Khả năng mở rộng và các đánh đổi đã chọn

| Chủ đề | Hiện trạng | Hệ quả / hướng đi |
| --- | --- | --- |
| Số bản sao máy chủ | Trạng thái phòng theo dõi, sổ kết nối và bộ đếm giới hạn nằm trong RAM tiến trình | Phù hợp **một tiến trình**; muốn chạy nhiều bản sao cần chuyển các phần này sang kho dùng chung |
| Cập nhật dữ liệu | Chủ yếu hỏi lại định kỳ, chỉ vị trí và cuộc gọi dùng kênh đẩy | Đơn giản, dễ đoán tải; nếu số người dùng tăng mạnh thì nên đẩy thêm sự kiện qua kênh thời gian thực |
| Truy vấn | SQL thuần, không ORM | Kiểm soát tốt hiệu năng nhưng phải tự giữ kỷ luật khi lược đồ đổi |
| Migration | Áp dụng thủ công, đồng thời phải cập nhật tệp lược đồ gốc | Rủi ro lệch giữa môi trường nếu quên một bên |
| Kiểm thử | Chưa có bộ kiểm thử tự động | Kiểm chứng bằng chạy thật; nên bổ sung kiểm thử cho phần vòng đời yêu cầu và phân quyền trước tiên |
| Vùng ngập | Lưu ở dịch vụ ngoài (Firestore) chứ không cùng cơ sở dữ liệu chính | Được cập nhật thời gian thực miễn phí, nhưng dữ liệu bản đồ nằm tách khỏi phần còn lại và không tham gia được vào các truy vấn/thống kê SQL |
