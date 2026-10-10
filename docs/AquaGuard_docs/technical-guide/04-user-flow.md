# 4. User flow — người dùng thao tác thế nào và ba vai trò liên hệ ra sao

Tài liệu này mô tả **luồng đi thực tế của người dùng**, đặc biệt là hành trình
của một yêu cầu cứu hộ từ lúc người dân bấm nút cho tới lúc đóng nhiệm vụ, và
cách ba vai trò giao thoa với nhau.

---

## 4.1. Mối quan hệ giữa ba vai trò

```
        ┌──────────────┐        gửi yêu cầu        ┌──────────────┐
        │   NGƯỜI DÂN  │ ────────────────────────► │  QUẢN TRỊ    │
        │              │ ◄──────────────────────── │              │
        └──────┬───────┘   thông báo, cảnh báo lũ  └──────┬───────┘
               │                                          │
               │  vị trí trực tiếp,                       │  giao yêu cầu cho đội,
               │  gọi thoại, theo dõi                     │  quản lý đội & vai trò
               │                                          │
               │            ┌──────────────┐              │
               └──────────► │   CỨU HỘ     │ ◄────────────┘
                            │  (theo đội)  │
                            └──────────────┘
                     nhận nhiệm vụ · tới hiện trường
                     · trả lại hoặc hoàn tất
```

Ba mối quan hệ chính:

- **Người dân ↔ Cứu hộ**: quan hệ trực tiếp nhất — vị trí thời gian thực, tuyến
  đường và cuộc gọi thoại chỉ tồn tại giữa đúng hai người của một nhiệm vụ.
- **Quản trị → Cứu hộ**: quản trị *giao việc* nhưng **không thực hiện** nhiệm
  vụ; quản trị cũng quyết định ai là cứu hộ (đổi vai trò tài khoản).
- **Quản trị → Người dân**: quan hệ một chiều dạng phát tin — thông báo hệ thống
  và cảnh báo vùng ngập.

Một chi tiết quan trọng: **quản trị không phải là nút cổ chai bắt buộc**. Một
đội cứu hộ có thể tự nhận thẳng yêu cầu chưa ai phụ trách. Vai trò của quản trị
là điều phối chủ động khi cần ưu tiên hoặc cân đối tải giữa các đội.

---

## 4.2. Luồng bắt đầu: tạo tài khoản và vào việc

```
Người dùng mở web
   │
   ├─ Đăng ký ────► nhập số điện thoại +84 · mật khẩu · tên
   │                 │
   │                 ├─ chọn "Người dân"  → tạo tài khoản ngay
   │                 └─ chọn "Cứu hộ" / "Quản trị" → phải nhập MÃ VAI TRÒ
   │                        │
   │                        └─ sai mã → từ chối tạo tài khoản
   │
   ├─ Đăng nhập bằng số điện thoại + mật khẩu
   │
   └─ Hoặc đăng nhập bằng Google → lần đầu: chọn vai trò
   │
   ▼
Vào ứng dụng, được đưa tới màn hình mặc định theo vai trò:
   • Người dân → Trang chủ cá nhân
   • Cứu hộ   → Nhiệm vụ của đội
   • Quản trị → Bảng điều khiển
   │
   ▼
Ngay khi đăng nhập, nền ứng dụng tự động:
   • xin quyền vị trí và bắt đầu phát vị trí (để được coi là "đang trực tuyến")
   • mở kênh nhận cuộc gọi (để chuông reo ở bất kỳ trang nào)
   • bắt đầu hỏi thông báo định kỳ
   • với người dùng mới: chạy tour hướng dẫn từng bước
```

### Quên mật khẩu

```
Nhập số điện thoại → nhận mã OTP qua SMS → nhập mã (đúng thì mở phiên đặt lại
10 phút) → đặt mật khẩu mới → đăng nhập lại
```
Mỗi số chỉ xin được mã mới sau 60 giây; mã sai hoặc hết hạn thì phải xin lại.

---

## 4.3. Luồng chính: một yêu cầu cứu hộ từ đầu đến cuối

### Sơ đồ tổng thể

```
NGƯỜI DÂN                  QUẢN TRỊ                   CỨU HỘ (đội)
─────────                  ────────                   ────────────
 gửi SOS
 (GPS + ảnh + mức khẩn)
     │
     ├──────────────────────────────────────────────────────┐
     ▼                          ▼                           ▼
 [chờ xử lý] ──────────► thấy trong hàng đợi         thấy trong danh sách
     │                    điều phối                   yêu cầu chung
     │                          │                           │
     │                          │ giao cho một đội          │ (hoặc tự nhận thẳng
     │                          ▼                           │  yêu cầu chưa ai nhận)
     │                    [đã phân đội] ─────────────────►  │
     │                                                      │
     │                                    nhóm trưởng/phó bấm "Bắt đầu"
     │                                                      ▼
     │ nhận thông báo + email                       [đang thực hiện]
     │ "Đội cứu hộ đang đến!"                               │
     ▼                                                      │
 nút "Xem theo dõi" hiện ra ◄──── vị trí hai chiều ────────► bản đồ theo dõi
     │                            + tuyến đường + gọi thoại  │
     │                                                      │
     │                                        ┌─────────────┴─────────────┐
     │                                        ▼                           ▼
     │                                  bấm "Hoàn tất"              bấm "Trả lại"
     │                                  (phải xác nhận)                   │
     │                                        ▼                           ▼
     │                                  [đã hoàn tất]              quay về [chờ xử lý]
     ▼                                        │                    (ghi nhận ai đã trả)
 nhận thông báo + email hoàn tất ◄────────────┤
 người thân cũng được báo "đã an toàn" ◄──────┘
```

### Bước 1 — Người dân gửi yêu cầu

1. Mở trang SOS. Ứng dụng lặng lẽ lấy sẵn toạ độ để không phải chờ khi mở biểu mẫu.
2. Bấm nút gửi yêu cầu → biểu mẫu mở ra với **địa chỉ đã được tự điền** từ toạ độ.
3. Người dùng mô tả tình huống, chọn mức khẩn cấp, đính tối đa 5 ảnh.
4. Gửi → yêu cầu vào trạng thái **chờ xử lý** và xuất hiện đồng thời ở:
   - danh sách yêu cầu của chính người đó,
   - màn hình điều phối của quản trị,
   - danh sách yêu cầu chung của các đội cứu hộ,
   - và dưới dạng điểm trên bản đồ lũ.
5. Nếu người dùng từ chối quyền vị trí, vẫn gửi được yêu cầu nhưng mất phần dẫn
   đường và theo dõi — hệ thống hướng dẫn bật lại quyền.

### Bước 2 — Đưa yêu cầu tới một đội

Có **hai con đường**, không loại trừ nhau:

**(a) Quản trị chủ động giao.** Quản trị mở màn hình điều phối, lọc theo thành
phố / nhóm tuổi / giới tính nếu cần, sắp xếp theo ưu tiên (mặc định: chưa xử lý
trước → khẩn cấp cao trước → mới hơn trước), chọn một yêu cầu, xem chi tiết và
ảnh, rồi chọn một đội trong danh sách các đội đang hoạt động để giao. Yêu cầu
chuyển sang trạng thái **đã phân đội** và gắn với nhóm trưởng của đội đó.

**(b) Đội tự nhận.** Ở danh sách yêu cầu chung, nhóm trưởng hoặc nhóm phó của
một đội có thể nhận thẳng một yêu cầu **chưa ai phụ trách**. Yêu cầu đi thẳng
sang **đang thực hiện**, bỏ qua bước phân công.

Trong cả hai đường, hệ thống chặn xung đột: một yêu cầu đã được đội khác nhận
thì thao tác nhận của người đến sau bị từ chối kèm thông báo rõ ràng.

### Bước 3 — Đội bắt đầu nhiệm vụ

1. Nhóm trưởng/nhóm phó mở tab *chờ bắt đầu* trong màn hình nhiệm vụ của đội —
   những yêu cầu vừa được giao có nhãn **MỚI**.
2. Bấm "Bắt đầu nhiệm vụ". Hệ thống lấy nhanh toạ độ hiện tại của đội làm điểm
   xuất phát (không chờ quá lâu; độ chính xác sẽ được cải thiện ngay sau đó
   bằng luồng vị trí liên tục).
3. Yêu cầu chuyển sang **đang thực hiện**; bản đồ theo dõi mở ra ngay.
4. Người dân nhận **thông báo trong ứng dụng + email**: "Đội cứu hộ đang đến",
   kèm tên người phụ trách. Nút "Xem theo dõi" xuất hiện ở phía họ chỉ sau vài giây.

Nếu người bấm không phải nhóm trưởng/nhóm phó, hoặc chưa thuộc đội nào, hệ thống
từ chối và giải thích lý do (chưa có đội → mời sang trang đội; không đủ cấp bậc
→ báo chỉ nhóm trưởng/phó mới nhận được).

### Bước 4 — Trong lúc di chuyển

Hai bên cùng mở bản đồ theo dõi và bước vào một "phòng" chung:

- Vị trí của mỗi bên chảy liên tục sang bên kia.
- Bản đồ vẽ **tuyến đường đi thực tế** cùng quãng đường và thời gian ước tính.
- Cả hai thấy thông tin của nhau (tên, đội, số điện thoại) và ảnh hiện trường.
- Một nút **gọi thoại** cho phép nói chuyện trực tiếp. Chuông reo ở mọi trang
  của phía bên kia, kể cả khi họ chưa mở bản đồ. Nếu bên kia đang offline hoàn
  toàn, người gọi được báo ngay thay vì chờ.
- Quản trị cũng có thể mở bản đồ theo dõi của bất kỳ nhiệm vụ nào để giám sát.

### Bước 5 — Kết thúc

**Trường hợp thuận lợi — hoàn tất:**

1. Đội bấm "Hoàn tất" → hộp thoại yêu cầu **xác nhận** (chống bấm nhầm).
2. Yêu cầu chuyển sang **đã hoàn tất**, ghi lại mốc thời gian, phòng theo dõi
   được báo kết thúc và tự đóng ở cả hai phía.
3. Người dân nhận thông báo + email "Yêu cầu cứu hộ đã hoàn tất".
4. **Toàn bộ người thân đã kết nối** của người đó cũng nhận thông báo + email:
   "Người thân của bạn đã được cứu hộ an toàn".

**Trường hợp không tiếp cận được — trả lại:**

1. Đội bấm "Trả lại nhiệm vụ".
2. Yêu cầu quay về **chờ xử lý**, gỡ liên kết với đội, và ghi nhận **ai đã trả
   lại vào lúc nào**.
3. Người dân và các đội khác đều nhìn thấy ghi chú "đã được trả lại bởi …",
   nên biết yêu cầu vẫn còn hiệu lực chứ không bị bỏ quên.
4. Yêu cầu quay lại đúng vị trí ưu tiên trong hàng đợi và có thể được giao hoặc
   nhận lại từ đầu.

**Ngoại lệ — quản trị đóng thay:** trong tình huống đặc biệt (đội mất liên lạc,
xử lý ngoài hệ thống…), quản trị có thể đóng một nhiệm vụ đang thực hiện.

### Ai được làm gì ở từng trạng thái

| Trạng thái | Người dân | Cứu hộ | Quản trị |
| --- | --- | --- | --- |
| **Chờ xử lý** | Xem, chờ | Nhận trực tiếp (nếu chưa ai phụ trách, và là trưởng/phó) | Giao cho một đội |
| **Đã phân đội** | Xem, biết đội nào phụ trách | Đội được giao bấm bắt đầu | Xem, theo dõi |
| **Đang thực hiện** | Mở bản đồ theo dõi, gọi thoại | Theo dõi, gọi, hoàn tất, trả lại | Theo dõi, đóng thay |
| **Đã hoàn tất** | Xem lại lịch sử | Xem trong tab đã hoàn tất | Xem, thống kê, xuất dữ liệu |

---

## 4.4. Luồng thành lập và vận hành đội cứu hộ

Đây là **điều kiện tiên quyết** để một người cứu hộ làm được việc.

```
Người cứu hộ mới đăng nhập
   │
   ▼
Màn hình nhiệm vụ báo: "Bạn chưa thuộc đội nào" → mời sang trang Đội
   │
   ├── Cách A: TỰ TẠO ĐỘI
   │      nhập tên + mô tả → trở thành NHÓM TRƯỞNG
   │           │
   │           ▼
   │      mời thành viên bằng số điện thoại
   │      (chỉ mời được tài khoản cứu hộ chưa thuộc đội nào;
   │       có sẵn danh sách gợi ý người chưa có đội)
   │           │
   │           ▼
   │      người được mời thấy lời mời → chấp nhận / từ chối
   │
   └── Cách B: ĐƯỢC MỜI
          nhận lời mời trong ứng dụng → chấp nhận → thành THÀNH VIÊN
   │
   ▼
Trong đội:
   NHÓM TRƯỞNG  → mời người, thăng/giáng nhóm phó, loại thành viên,
                   sửa tên & mô tả đội, giải tán đội, NHẬN NHIỆM VỤ
   NHÓM PHÓ     → mời người, NHẬN NHIỆM VỤ
   THÀNH VIÊN   → xem toàn bộ nhiệm vụ của đội để phối hợp,
                   nhưng KHÔNG đổi được trạng thái nhiệm vụ; có thể tự rời đội
```

Mọi thành viên trong đội **thấy chung một danh sách nhiệm vụ** — đó là ý nghĩa
của việc điều phối theo đội thay vì theo cá nhân. Trang đội cũng hiển thị số
nhiệm vụ đang chạy, số đã hoàn tất và quy mô đội.

Một người chỉ thuộc **một đội đang hoạt động** tại một thời điểm; hệ thống từ
chối mời người đã có đội.

---

## 4.5. Luồng người thân (chỉ giữa những người dùng với nhau)

```
Người A                                     Người B
   │ tìm B bằng số điện thoại
   │ gửi lời mời + ghi quan hệ (bố, mẹ, anh…)
   └──────────────────────────────────────►  nhận thông báo
                                               │
                          ┌────────────────────┴──────────────────┐
                          ▼                                       ▼
                     chấp nhận                                từ chối
                          │                                  (lời mời bị xoá)
                          ▼
   A nhận thông báo "đã kết nối"
                          │
                          ▼
   Từ đây hai bên nhìn thấy của nhau:
     • trạng thái an toàn (An toàn / Nguy hiểm / Bị thương)
     • ghi chú sức khoẻ
     • vị trí gần nhất trên bản đồ, kèm thời điểm cập nhật
   Và khi một bên được cứu hộ thành công,
   bên kia tự động nhận thông báo + email "đã an toàn".
```

Người dùng cũng có thể **gỡ kết nối** bất cứ lúc nào từ trang cài đặt.

Việc **tự cập nhật trạng thái an toàn** là thao tác một chạm ngay trên trang chủ
— đây là cách nhanh nhất để trấn an người thân mà không cần gửi yêu cầu cứu hộ.

---

## 4.6. Luồng phát cảnh báo và thông báo của quản trị

### Cảnh báo vùng ngập

```
Quản trị mở trang gửi thông báo → chuyển sang phần biên tập bản đồ lũ
   │
   ▼
Bấm lên vị trí trên bản đồ → điền tên vùng, mức độ (an toàn → nguy kịch),
mực nước → chọn có phát cảnh báo hay không → lưu
   │
   ├─► Vùng ngập hiện NGAY trên bản đồ của tất cả người đang mở ứng dụng
   │
   └─► Nếu chọn phát cảnh báo:
         • mọi người dùng đang hoạt động nhận thông báo trong ứng dụng
         • ai có email thì nhận thêm email (gửi rải để không quá hạn mức)
```

Quản trị cũng có thể **sửa hoặc xoá** vùng ngập cũ khi nước rút.

### Thông báo thủ công

```
Chọn đối tượng:
   • TẤT CẢ người dùng → chỉ thông báo trong ứng dụng (tránh spam hộp thư)
   • MỘT người cụ thể (có ô tìm kiếm) → thông báo trong ứng dụng + email
Nhập tiêu đề + nội dung → gửi → hệ thống báo lại số người đã nhận
```

---

## 4.7. Luồng quản trị người dùng và vai trò

```
Quản trị mở Bảng điều khiển → tab Quản lý người dùng
   │
   ├─ Danh sách tách theo vai trò, sắp xếp theo tên hoặc số điện thoại
   │
   ├─ Đổi vai trò ngay trong bảng
   │     → người bị đổi sẽ thấy menu và quyền thay đổi ở lần thao tác kế tiếp
   │       (màn hình đang mở mà không còn quyền sẽ tự chuyển về mặc định)
   │
   └─ Xoá tài khoản: chọn một hoặc nhiều dòng rồi xoá hàng loạt
         → KHÔNG thể chọn/xoá chính tài khoản của mình
         → dữ liệu lịch sử liên quan được xử lý an toàn, không vỡ tham chiếu
```

Ngoài ra quản trị xem được danh sách các đội cứu hộ kèm nhóm trưởng và thành viên.

---

## 4.8. Luồng thống kê và xuất dữ liệu

```
Quản trị mở Thống kê hệ thống
   │
   ├─ Chế độ BIỂU ĐỒ
   │    • tổng người dùng, đăng ký mới 7 ngày, đường tăng trưởng 30 ngày,
   │      phân bố vai trò
   │    • xu hướng yêu cầu 30 ngày, phân bố mức khẩn cấp và trạng thái
   │    • thời gian phản hồi nhanh nhất / chậm nhất / trung bình, tỉ lệ giải quyết
   │
   └─ Chế độ XUẤT DỮ LIỆU
        chọn khoảng ngày → bật/tắt ẩn danh → tải từng tập hoặc tải tất cả
        (ẩn danh: bỏ tên/điện thoại/email/mô tả/IP, ngày sinh chỉ còn năm,
         toạ độ làm tròn ~1 km, nhưng vẫn giữ mã số để ghép nối được các tệp)
```

---

## 4.9. Các tình huống ngoài luồng chuẩn

| Tình huống | Hệ thống xử lý thế nào |
| --- | --- |
| Người cứu hộ chưa vào đội | Màn hình nhiệm vụ hiện lời mời sang trang đội, không cho nhận việc |
| Người cứu hộ là thành viên thường | Vẫn xem đầy đủ nhiệm vụ của đội, nhưng nút đổi trạng thái bị ẩn; nếu cố gọi vẫn bị máy chủ từ chối kèm lý do |
| Hai đội cùng bấm nhận một yêu cầu | Người đến sau nhận thông báo yêu cầu đã được đội khác nhận |
| Người dân từ chối quyền vị trí | Vẫn gửi được yêu cầu; mất phần theo dõi và dẫn đường, kèm hướng dẫn bật lại quyền |
| Người được gọi không online | Người gọi nhận báo "không liên lạc được" ngay, không chờ vô ích |
| Đang gọi mà một bên rớt mạng hoàn toàn | Cuộc gọi được kết thúc và bên còn lại được báo |
| Đóng bản đồ theo dõi giữa cuộc gọi | Cuộc gọi **không** bị ngắt, vì kênh cuộc gọi là kênh riêng luôn mở |
| Mất kết nối thời gian thực | Kết nối tự thử lại sau vài giây; trong lúc đó các màn hình vẫn cập nhật nhờ vòng hỏi lại định kỳ |
| Máy chủ vừa "ngủ dậy" | Yêu cầu đầu tiên chậm hơn bình thường; dịch vụ giám sát ping định kỳ để hạn chế tình huống này |
| Yêu cầu bị bỏ quên trong hàng đợi | Cách sắp xếp mặc định luôn đẩy yêu cầu chưa xử lý và mức khẩn cấp cao lên trên; quản trị thấy ngay ở màn hình điều phối |

---

## 4.10. Tóm tắt một chu trình hoàn chỉnh

1. Người dân đăng ký, kết nối người thân, bật quyền vị trí.
2. Lũ về — người dân báo trạng thái "Nguy hiểm" và **gửi yêu cầu cứu hộ** kèm
   ảnh, toạ độ và mức khẩn cấp.
3. Quản trị thấy yêu cầu ở đầu hàng đợi (chưa xử lý + mức cao), **giao cho một
   đội** phù hợp; hoặc một đội **tự nhận** trước.
4. Nhóm trưởng/nhóm phó **bắt đầu nhiệm vụ** → người dân được báo "đội đang đến".
5. Hai bên **theo dõi vị trí của nhau theo thời gian thực**, có tuyến đường và
   **gọi thoại** khi cần.
6. Đội **hoàn tất** (có xác nhận) → người dân và **toàn bộ người thân** được báo
   đã an toàn.
7. Toàn bộ mốc thời gian được ghi lại, đi vào **thống kê thời gian phản hồi** và
   có thể **xuất ra CSV** để báo cáo.
