# 1. Tổng quan sản phẩm, tính năng và cách hoạt động

## 1.1. AquaGuard là gì

AquaGuard là một nền tảng web hỗ trợ **ứng phó và điều phối cứu hộ trong tình
huống lũ lụt**, tập trung cho bối cảnh Việt Nam (mặc định lấy Đà Nẵng làm trung
tâm bản đồ).

Vấn đề mà nền tảng giải quyết: khi lũ về, người dân bị cô lập rất khó mô tả
chính xác mình đang ở đâu, đội cứu hộ không biết ai cần giúp trước, còn cơ quan
điều phối thì không có bức tranh tổng thể theo thời gian thực. AquaGuard gom cả
ba phía đó vào một màn hình chung:

- **Người dân** bấm một nút để gửi yêu cầu cứu hộ kèm toạ độ GPS, ảnh hiện
  trường và mức độ khẩn cấp; đồng thời báo tình trạng an toàn cho người thân.
- **Đội cứu hộ** thấy hàng đợi yêu cầu, nhận nhiệm vụ theo đội, dẫn đường tới
  hiện trường và gọi thoại trực tiếp cho nạn nhân.
- **Quản trị viên** theo dõi toàn hệ thống, phân yêu cầu cho các đội, phát cảnh
  báo vùng lũ, và xuất dữ liệu để báo cáo.

Toàn bộ giao diện có **song ngữ Việt – Anh** và chế độ sáng/tối, hoạt động tốt
trên cả máy tính lẫn điện thoại (bản mobile có thanh điều hướng dưới đáy).

---

## 1.2. Ba vai trò trong hệ thống

| Vai trò | Là ai | Mục tiêu chính |
| --- | --- | --- |
| **Người dân (citizen)** | Người dùng phổ thông, mặc định khi đăng ký | Gửi yêu cầu cứu hộ, theo dõi vùng ngập, giữ liên lạc với người thân |
| **Cứu hộ (rescuer)** | Thành viên đội cứu hộ | Nhận và thực hiện nhiệm vụ cứu hộ theo đội |
| **Quản trị (admin)** | Người điều phối / vận hành hệ thống | Điều phối yêu cầu, quản lý người dùng và đội, cảnh báo cộng đồng, thống kê |

Vai trò được chọn ngay khi đăng ký. Để tránh việc ai cũng tự nhận là cứu hộ hay
quản trị, hai vai trò này yêu cầu nhập thêm **mã vai trò** do đơn vị vận hành
cấp; sai mã thì không tạo được tài khoản. Quản trị viên vẫn có thể nâng/hạ vai
trò của bất kỳ tài khoản nào sau đó.

Một tài khoản chỉ có **một** vai trò tại một thời điểm, và vai trò quyết định
toàn bộ những gì người đó nhìn thấy trong ứng dụng: menu, trang được phép mở,
và các hành động được phép thực hiện.

---

## 1.3. Tính năng chung cho mọi người dùng

### Tài khoản và đăng nhập

- Đăng ký bằng **số điện thoại Việt Nam** (dạng +84…) kèm mật khẩu tối thiểu 6
  ký tự; có thể khai thêm tên hiển thị, giới tính, ngày sinh và email.
- Đăng nhập bằng số điện thoại + mật khẩu. Phiên đăng nhập được giữ trong trình
  duyệt nên mở lại tab là vào thẳng ứng dụng, không cần đăng nhập lại.
- Có thêm lựa chọn **đăng nhập bằng Google**; lần đầu vào, người dùng được hỏi
  chọn vai trò trước khi bắt đầu.
- **Quên mật khẩu**: người dùng nhập số điện thoại → hệ thống gửi mã OTP qua
  SMS → nhập đúng mã sẽ mở một phiên đặt lại mật khẩu ngắn hạn (10 phút) → đặt
  mật khẩu mới rồi đăng nhập lại. Mỗi số chỉ được xin mã mới sau 60 giây, và
  toàn bộ luồng có giới hạn số lần thử để chống dò.
- Tài khoản bị khoá sẽ không đăng nhập được, kèm thông báo rõ ràng.

### Hồ sơ và cài đặt

Trang cài đặt gồm các phần: hồ sơ cá nhân (tên, ảnh đại diện, liên hệ khẩn cấp,
địa chỉ…), **người thân** (chỉ người dân), giao diện sáng/tối, và ngôn ngữ.

### Thông báo

Mọi sự kiện quan trọng đều sinh ra **thông báo trong ứng dụng** (chuông thông
báo có đếm số chưa đọc, tự làm mới nền và hiện toast khi có tin mới), và những
sự kiện quan trọng nhất còn được **gửi kèm email** nếu người dùng có khai email:
chào mừng khi đăng ký, khi có đội nhận yêu cầu, khi yêu cầu hoàn tất, khi người
thân được cứu an toàn, khi quản trị gửi thông báo riêng, và khi có cảnh báo
vùng lũ mới.

Email luôn được gửi theo kiểu "bắn đi rồi quên": nếu dịch vụ email lỗi thì hành
động chính (đăng ký, nhận nhiệm vụ, hoàn tất cứu hộ…) vẫn thành công bình thường.

### Trợ lý AI

Một cửa sổ chat nổi ở góc màn hình trả lời các câu hỏi về ngập lụt, sơ tán, chỗ
trú ẩn, cách gửi yêu cầu cứu hộ… Trợ lý trả lời theo đúng ngôn ngữ người dùng
đang gõ, luôn nhắc số khẩn cấp 113/114/115, và có sẵn bộ câu trả lời dự phòng
khi dịch vụ AI không phản hồi.

### Hướng dẫn sử dụng lần đầu

Người dùng mới được dẫn qua một **tour tương tác**: hệ thống tự chuyển trang, mở
đúng hộp thoại và trỏ vào từng nút một (bản đồ, lớp thời tiết, nút gửi SOS, ô mô
tả, mức khẩn cấp, đính ảnh…). Tour ghi nhớ trạng thái nên chỉ chạy một lần.

---

## 1.4. Tính năng cho người dân

### Trang chủ cá nhân

Một màn hình duy nhất tổng hợp mọi thứ người dân cần trong lúc khẩn cấp:

- **Bảng trạng thái an toàn**: bấm một nút để tự báo mình đang *An toàn / Nguy
  hiểm / Bị thương*. Trạng thái này hiển thị ngay cho những người thân đã kết nối.
- **Băng-rôn yêu cầu đang mở**: nếu đang có yêu cầu cứu hộ chưa xong, nó luôn nổi
  lên đầu trang kèm trạng thái hiện tại.
- **Bảng an toàn gia đình**: danh sách người thân kèm trạng thái, ghi chú sức
  khoẻ, vị trí gần nhất và thời điểm cập nhật.
- **Lời mời kết nối đang chờ**, **thao tác nhanh**, và **lịch sử yêu cầu cứu hộ**.

### Gửi yêu cầu cứu hộ (SOS)

Luồng người dùng:

1. Người dân mở trang SOS và bấm nút gửi yêu cầu. Ngay khi vào trang, ứng dụng
   đã âm thầm "làm nóng" GPS để lúc mở biểu mẫu là có sẵn toạ độ, không phải chờ.
2. Biểu mẫu tự điền **địa chỉ** bằng cách tra ngược từ toạ độ GPS; người dùng
   vẫn sửa được nếu muốn.
3. Người dùng mô tả tình huống, chọn **mức khẩn cấp** (thấp / trung bình / cao /
   nguy kịch) và đính kèm **tối đa 5 ảnh** (mỗi ảnh tối đa 5 MB).
4. Gửi. Ảnh được đẩy lên dịch vụ lưu trữ ảnh, yêu cầu được ghi nhận ở trạng thái
   *chờ xử lý*, và mọi thay đổi trạng thái từ đây về sau đều được ghi vào nhật ký.
5. Trang tự làm mới nhanh (vài giây một lần) khi còn yêu cầu đang mở, nên người
   dân thấy gần như tức thì lúc có đội nhận việc.
6. Khi đội cứu hộ bắt đầu di chuyển, nút **"Xem theo dõi"** hiện ra: mở bản đồ
   thời gian thực thấy vị trí mình, vị trí đội cứu hộ, tuyến đường, khoảng cách
   còn lại — và nút **gọi thoại** cho người đang tới cứu.

Nếu một đội nhận rồi trả lại nhiệm vụ, yêu cầu quay về hàng đợi và người dân
được thấy dòng ghi chú "đã được trả lại bởi …" để hiểu chuyện gì đang xảy ra.

### Kết nối người thân

Người dùng tìm người thân **bằng số điện thoại**, gửi lời mời kèm quan hệ (bố,
mẹ, anh…). Người kia nhận thông báo và chấp nhận hoặc từ chối. Khi đã kết nối,
hai bên nhìn thấy trạng thái an toàn, ghi chú sức khoẻ và vị trí gần nhất của
nhau trên bản đồ; và khi một người được cứu hộ thành công, tất cả người thân
đều nhận thông báo (kèm email) báo rằng người đó đã an toàn.

### Bản đồ lũ trực tiếp

Bản đồ là nơi hội tụ nhiều lớp dữ liệu cùng lúc:

- **Vùng ngập** do quản trị viên đánh dấu, phân theo 5 mức (an toàn → thấp →
  trung bình → nặng → nguy kịch) kèm mực nước.
- **Lớp thời tiết** chồng lên bản đồ: mưa, gió, mây, nhiệt độ, áp suất, sóng.
- **Vị trí của tôi**, **vị trí người thân** (viền màu theo trạng thái an toàn).
- **Các yêu cầu cứu hộ đang mở** và **các đội cứu hộ đang trực tuyến**, di
  chuyển gần như thời gian thực.
- **Tuyến đường** và khoảng cách giữa hai điểm khi đang theo dõi một nhiệm vụ.

---

## 1.5. Tính năng cho đội cứu hộ

### Đội cứu hộ (điều kiện tiên quyết)

Một người cứu hộ **chưa vào đội thì chưa làm gì được**: hệ thống chặn ngay ở
màn hình nhiệm vụ và mời họ sang trang đội.

- Bất kỳ người cứu hộ nào cũng có thể **tạo một đội mới**; người tạo trở thành
  **nhóm trưởng**.
- Nhóm trưởng/nhóm phó **mời thành viên bằng số điện thoại** (chỉ mời được tài
  khoản có vai trò cứu hộ, và người đó phải chưa thuộc đội nào). Trang đội có
  cả danh sách gợi ý những người cứu hộ chưa có đội để mời nhanh.
- Người được mời thấy lời mời trong ứng dụng và **chấp nhận hoặc từ chối**.
- Trong đội có ba cấp: **nhóm trưởng, nhóm phó, thành viên**. Nhóm trưởng có
  thể thăng/giáng nhóm phó, loại thành viên, đổi tên/mô tả đội, hoặc giải tán
  đội; thành viên có thể tự rời đội.
- Trang đội cũng hiện thống kê: số nhiệm vụ đang chạy, số đã hoàn tất, quy mô đội.

**Quy tắc quan trọng**: chỉ **nhóm trưởng và nhóm phó** mới được bấm nhận nhiệm
vụ. Thành viên thường vẫn thấy đầy đủ thông tin nhiệm vụ của đội để phối hợp,
nhưng không đổi được trạng thái.

### Hàng đợi và thực hiện nhiệm vụ

Người cứu hộ có hai màn hình làm việc:

- **Nhiệm vụ của đội** — chỉ những yêu cầu đã được giao cho đội mình, chia theo
  ba tab: *chờ bắt đầu*, *đang thực hiện*, *đã hoàn tất*. Đây là màn hình mặc
  định khi đăng nhập.
- **Danh sách yêu cầu** — toàn bộ yêu cầu trong hệ thống, có bộ lọc theo thành
  phố, nhóm tuổi, giới tính của người gặp nạn và nhiều kiểu sắp xếp. Từ đây đội
  có thể **nhận thẳng một yêu cầu chưa ai phụ trách** mà không cần chờ quản trị
  phân công.

Khi bấm nhận nhiệm vụ, hệ thống lấy nhanh toạ độ của đội để đặt điểm xuất phát,
chuyển yêu cầu sang *đang thực hiện*, mở luôn bản đồ theo dõi, đồng thời báo cho
người dân rằng "đội cứu hộ đang đến".

Trên bản đồ theo dõi, đội thấy vị trí nạn nhân cập nhật liên tục, tuyến đường
và khoảng cách, thông tin liên hệ, ảnh hiện trường, và có thể **gọi thoại** cho
nạn nhân.

Kết thúc, đội có hai lựa chọn:

- **Hoàn tất** — phải xác nhận thêm một bước ở hộp thoại (tránh bấm nhầm một cái
  là đóng nhiệm vụ). Sau khi hoàn tất, người dân và toàn bộ người thân của họ
  đều nhận thông báo.
- **Trả lại** — nếu không tiếp cận được, nhiệm vụ quay về hàng đợi cho đội khác,
  có ghi nhận ai đã trả và vào lúc nào.

### Trung tâm giám sát

Một màn hình mô phỏng hạ tầng quan trắc: **cảm biến IoT** (mực nước, lượng mưa,
lưu lượng) và **drone giám sát** với các khung phát hiện vùng ngập / người gặp
nạn. Dữ liệu ở đây hiện là **dữ liệu mô phỏng**, dùng để trình diễn hướng phát
triển tiếp theo chứ chưa nối vào thiết bị thật.

---

## 1.6. Tính năng cho quản trị viên

### Bảng điều khiển và quản lý người dùng

- Các chỉ số tổng quan của hệ thống.
- Danh sách người dùng tách theo vai trò, sắp xếp theo tên hoặc số điện thoại,
  **đổi vai trò** trực tiếp trong bảng, **xoá một hoặc nhiều tài khoản** cùng lúc
  (có chọn hàng loạt). Quản trị viên **không thể tự xoá tài khoản của chính mình**.
- Danh sách các **đội cứu hộ** kèm nhóm trưởng và thành viên.

### Điều phối yêu cầu cứu hộ

Đây là màn hình vận hành chính:

- Bốn thẻ thống kê (tổng / chờ xử lý / đang thực hiện / đã xong) đồng thời là
  bộ lọc nhanh.
- **Lọc** theo thành phố (suy ra từ toạ độ hoặc mô tả địa chỉ), nhóm tuổi và
  giới tính của người gặp nạn; **sắp xếp** theo ưu tiên, mới nhất, cũ nhất hoặc
  tuổi.
- Chọn một yêu cầu → xem đầy đủ chi tiết, ảnh, toạ độ → **giao cho một đội cứu
  hộ** trong danh sách các đội đang hoạt động.
- Có thể **mở bản đồ theo dõi** của bất kỳ nhiệm vụ nào đang chạy, và **đóng
  nhiệm vụ** thay cho đội trong tình huống cần thiết.

### Gửi thông báo và cảnh báo vùng lũ

Trang này có hai phần:

- **Gửi thông báo**: chọn gửi cho *tất cả người dùng* hoặc *một người cụ thể*
  (có ô tìm kiếm), nhập tiêu đề và nội dung. Gửi cho một người thì kèm email;
  gửi cho tất cả thì chỉ thông báo trong ứng dụng để tránh spam hộp thư.
- **Biên tập bản đồ lũ**: bấm lên bản đồ để tạo một vùng ngập mới (tên, mức độ,
  mực nước), sửa hoặc xoá vùng cũ. Khi tạo vùng mới, quản trị có thể chọn **phát
  cảnh báo ngay**: toàn bộ người dùng nhận thông báo trong ứng dụng, còn những
  ai có email thì nhận thêm email (gửi rải đều để không vượt hạn mức dịch vụ).
  Vùng ngập vừa tạo hiện ngay trên bản đồ của mọi người.

### Thống kê hệ thống

Các biểu đồ và chỉ số: tổng người dùng và số đăng ký mới 7 ngày, đường tăng
trưởng người dùng 30 ngày, phân bố vai trò; xu hướng yêu cầu cứu hộ 30 ngày,
phân bố theo mức khẩn cấp và theo trạng thái; **thời gian phản hồi** nhanh nhất,
chậm nhất, trung bình và **tỉ lệ giải quyết**.

### Xuất dữ liệu

Xuất ra CSV theo từng tập dữ liệu (người dùng, yêu cầu cứu hộ, vị trí, đội,
thành viên đội, nhật ký thay đổi trạng thái, nhật ký hệ thống, bản tổng hợp
thống kê, và vùng ngập), có **lọc theo khoảng ngày** và tuỳ chọn **ẩn danh**:
khi bật, tên/điện thoại/email/mô tả tự do/địa chỉ IP bị lược bỏ, ngày sinh chỉ
còn năm, toạ độ được làm tròn về mức ~1 km, nhưng vẫn giữ mã số để các tệp ghép
nối được với nhau. Tệp xuất có sẵn dấu nhận dạng UTF-8 để mở bằng Excel không
lỗi tiếng Việt.

---

## 1.7. Hệ thống hoạt động ra sao — các cơ chế và "thuật toán" chính

### a) Vòng đời một yêu cầu cứu hộ

```
   [Người dân gửi]
          │
          ▼
     chờ xử lý ───────────────► (đội tự nhận trực tiếp) ───┐
          │                                                │
          │ (quản trị giao cho một đội)                     │
          ▼                                                 ▼
     đã phân đội ──(đội bấm bắt đầu)──────────────►  đang thực hiện
          ▲                                                 │
          └───────────(đội trả lại nhiệm vụ)─────────────────┤
                                                            │
                                                (hoàn tất, có xác nhận)
                                                            ▼
                                                        đã hoàn tất
```

Mỗi bước chuyển đều có điều kiện chặt: chỉ giao được yêu cầu **đang chờ**; chỉ
đội được giao (hoặc đội nhận yêu cầu chưa ai phụ trách) mới bắt đầu được; chỉ
đội đang phụ trách mới trả lại hoặc hoàn tất được (quản trị là ngoại lệ duy
nhất). Mọi lần đổi trạng thái đều ghi vào **nhật ký kiểm toán** kèm người thực
hiện và thời điểm — đây cũng là nguồn để tính thời gian phản hồi trong thống kê.

### b) Xếp hạng ưu tiên hàng đợi

Chế độ sắp xếp mặc định không phải "mới nhất trước" mà là **theo mức nguy cấp**,
so sánh lần lượt theo ba tiêu chí:

1. **Trạng thái**: chưa ai xử lý xếp trên đang thực hiện, đang thực hiện xếp
   trên đã xong.
2. **Mức khẩn cấp**: nguy kịch → cao → trung bình → thấp.
3. **Thời gian tạo**: yêu cầu mới lên trước.

Ngoài ra còn có các chế độ sắp xếp theo thời gian và theo tuổi người gặp nạn
(để ưu tiên trẻ nhỏ hoặc người già khi cần).

### c) Định vị liên tục và khái niệm "đang trực tuyến"

Đây là phần tinh tế nhất của hệ thống. Có **hai luồng vị trí khác nhau**:

- **Luồng hiện diện (luôn bật)**: chỉ cần đang đăng nhập, thiết bị liên tục gửi
  toạ độ về máy chủ. Toạ độ này chỉ được ghi vào **bộ nhớ nóng** với thời hạn
  sống **60 giây** — nghĩa là "có bản ghi = người này đang online và vị trí còn
  tươi". Nhờ vậy bản đồ hiển thị được các đội cứu hộ đang di chuyển mà không
  làm cơ sở dữ liệu chính bị ghi liên tục.
- **Luồng theo dõi nhiệm vụ**: khi mở bản đồ theo dõi của một yêu cầu, hai bên
  (người dân và đội cứu hộ) vào chung một "phòng" và trao đổi toạ độ trực tiếp
  cho nhau qua kết nối thời gian thực.

Để tiết kiệm tài nguyên, toạ độ chỉ được ghi vào bộ nhớ nóng khi **di chuyển đủ
xa (khoảng vài mét)** hoặc **đã quá 2 giây** kể từ lần ghi trước. Còn cơ sở dữ
liệu bền vững thì chỉ lưu **điểm đầu và điểm cuối** của mỗi phiên theo dõi —
đủ để biết "vị trí biết được gần nhất" kể cả khi máy chủ khởi động lại.

Khi hiển thị danh sách yêu cầu, hệ thống **đắp vị trí trực tiếp lên trên** toạ
độ đã lưu: yêu cầu nào còn đang mở thì lấy vị trí mới nhất của người gặp nạn và
của đội cứu hộ, thay vì vị trí lúc gửi yêu cầu.

### d) Tìm đội gần nhất

Vị trí của những người đang trực tuyến còn được lưu vào một cấu trúc dữ liệu
không gian theo vai trò, cho phép hỏi "những đội cứu hộ nào đang online trong
bán kính X km quanh điểm này", trả về **sắp xếp theo khoảng cách tăng dần**.
Kết quả được đối chiếu lại với bản ghi hiện diện để loại những người đã offline.

### e) Dẫn đường, khoảng cách và địa chỉ

- **Tuyến đường** giữa đội cứu hộ và nạn nhân được tính bằng dịch vụ định tuyến
  đường bộ mở, trả về đường đi thực tế cùng quãng đường và thời gian ước tính —
  không phải đường chim bay.
- **Địa chỉ và thành phố** được suy ra từ toạ độ bằng dịch vụ tra cứu địa danh
  mở, và được **ghi nhớ tạm** theo toạ độ làm tròn để không gọi lại nhiều lần
  cho cùng một điểm.

### f) Cập nhật gần thời gian thực

Hệ thống kết hợp hai kỹ thuật: **kênh thời gian thực** (cho vị trí, sự kiện bắt
đầu/kết thúc theo dõi và tín hiệu cuộc gọi) và **hỏi lại định kỳ** cho phần còn
lại, với nhịp được chọn theo mức độ khẩn của từng màn hình:

| Màn hình | Nhịp làm mới |
| --- | --- |
| Yêu cầu của người dân (khi còn yêu cầu đang mở) | ~4 giây |
| Yêu cầu của người dân (khi rảnh) | ~15 giây |
| Vị trí đội cứu hộ trên bản đồ | ~4 giây |
| Yêu cầu cứu hộ trên bản đồ | ~5 giây |
| Hàng đợi nhiệm vụ của đội | ~10 giây |
| Chuông thông báo | ~25 giây |
| Vị trí người thân | ~60 giây |

### g) Gọi thoại giữa hai bên của một nhiệm vụ

Cuộc gọi chỉ được phép giữa **đúng hai người của một nhiệm vụ đang hoạt động**:
người gửi yêu cầu và người của đội đang phụ trách. Máy chủ kiểm tra điều kiện
này một lần lúc bắt đầu cuộc gọi, sau đó chỉ đóng vai trò **trạm chuyển tiếp tín
hiệu**; âm thanh đi thẳng giữa hai thiết bị chứ không đi qua máy chủ. Chuông
reo được ở **mọi trang** của ứng dụng, kể cả khi chưa mở bản đồ theo dõi, và
người dùng có thể để một cuộc gọi ở nhiều tab mà vẫn nhận được. Nếu người kia
không online, người gọi nhận thông báo "không liên lạc được" thay vì chờ vô ích.
Phiên bản hiện tại **chỉ hỗ trợ gọi thoại**; phần hình đã có sẵn khung nhưng
chưa bật.

### h) Bảo vệ và chống lạm dụng

- **Giới hạn tần suất** theo địa chỉ mạng cho các điểm nhạy cảm: đăng nhập (10
  lần / 15 phút), đăng ký (5 / 15 phút), quên mật khẩu (5 / 15 phút), quản trị
  gửi thông báo (20 / phút), phát cảnh báo lũ (10 / phút), xuất dữ liệu (60 /
  phút).
- **Kiểm tra quyền hai lớp**: giao diện ẩn những gì vai trò không được dùng, và
  máy chủ vẫn kiểm tra lại quyền ở mọi yêu cầu — ẩn nút không đồng nghĩa với
  chặn được.
- Mật khẩu được băm trước khi lưu; phiên đăng nhập hết hạn sau 7 ngày.
- Ảnh tải lên bị giới hạn kiểu tệp (chỉ ảnh), dung lượng (5 MB) và số lượng (5 ảnh).

---

## 1.8. Bảng phân quyền

### Theo trang / màn hình

| Màn hình | Người dân | Cứu hộ | Quản trị |
| --- | :---: | :---: | :---: |
| Trang chủ cá nhân | ✅ | — | — |
| Bản đồ lũ trực tiếp | ✅ | ✅ | — |
| Gửi yêu cầu cứu hộ (SOS) | ✅ | — | — |
| Danh sách yêu cầu (toàn hệ thống) | — | ✅ | — |
| Nhiệm vụ của đội | — | ✅ | — |
| Đội cứu hộ | — | ✅ | — |
| Trung tâm giám sát | — | ✅ | ✅ |
| Hướng dẫn an toàn | ✅ | — | — |
| Bảng điều khiển quản trị | — | — | ✅ |
| Điều phối yêu cầu cứu hộ | — | — | ✅ |
| Quản lý người dùng | — | — | ✅ |
| Quản lý đội cứu hộ | — | — | ✅ |
| Gửi thông báo & biên tập vùng lũ | — | — | ✅ |
| Thống kê & xuất dữ liệu | — | — | ✅ |
| Cài đặt | ✅ | ✅ | ✅ |
| Trợ lý AI | ✅ | ✅ | ✅ |

> Ghi chú: hai trang **Tin tức & Cảnh báo** và **Giới thiệu** đã có sẵn trong mã
> nguồn nhưng hiện **không được gắn vào menu của vai trò nào**, nên chưa truy
> cập được từ giao diện. Tab **người thân** trong Cài đặt chỉ hiện với người dân.

### Theo hành động

| Hành động | Người dân | Cứu hộ | Quản trị |
| --- | :---: | :---: | :---: |
| Tạo yêu cầu cứu hộ | ✅ | — | — |
| Xem yêu cầu của chính mình | ✅ | — | — |
| Xem toàn bộ yêu cầu | ✅¹ | ✅ | ✅ |
| Giao yêu cầu cho một đội | — | — | ✅ |
| Nhận / bắt đầu nhiệm vụ | — | ✅² | — |
| Trả lại nhiệm vụ | — | ✅² | — |
| Hoàn tất nhiệm vụ | — | ✅² | ✅ |
| Mở bản đồ theo dõi | ✅³ | ✅ | ✅ |
| Gọi thoại | ✅³ | ✅³ | — |
| Tạo đội / mời thành viên | — | ✅⁴ | — |
| Đổi vai trò thành viên đội | — | ✅⁴ | — |
| Cập nhật trạng thái an toàn | ✅ | ✅ | ✅ |
| Kết nối người thân | ✅ | ✅ | ✅ |
| Đổi vai trò người dùng | — | — | ✅ |
| Xoá tài khoản người dùng | — | — | ✅⁵ |
| Gửi thông báo hàng loạt | — | — | ✅ |
| Tạo / sửa / xoá vùng ngập | — | — | ✅ |
| Xem thống kê hệ thống | — | — | ✅ |
| Xuất dữ liệu CSV | — | — | ✅ |

¹ Về mặt kỹ thuật người dân gọi được danh sách chung (bản đồ cần dữ liệu này),
nhưng giao diện chỉ cho họ thao tác trên yêu cầu của chính mình.
² Chỉ **nhóm trưởng / nhóm phó** của một đội đang hoạt động.
³ Chỉ với nhiệm vụ mà mình là một trong hai bên liên quan.
⁴ Chỉ nhóm trưởng (một số thao tác cho phép cả nhóm phó).
⁵ Không được xoá chính tài khoản của mình.

---

## 1.9. Giới hạn hiện tại

- **Trung tâm giám sát** (cảm biến, drone) chạy trên dữ liệu mô phỏng.
- Bản tin trong trang Tin tức là nội dung tĩnh, và trang này hiện chưa mở cho
  vai trò nào.
- Cuộc gọi mới hỗ trợ **âm thanh**; và vì trình duyệt yêu cầu môi trường bảo
  mật, tính năng chỉ hoạt động trên HTTPS (hoặc localhost khi phát triển).
- Vùng ngập được quản trị đánh dấu thủ công, chưa lấy tự động từ trạm quan trắc.
- Hệ thống chưa có bộ kiểm thử tự động; kiểm chứng thay đổi bằng cách chạy và
  thao tác thật trên ứng dụng.
