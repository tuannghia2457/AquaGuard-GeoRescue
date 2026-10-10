# 6. Điều phối cứu hộ tự động — Hỏi & Đáp

Tài liệu này giải thích cơ chế điều phối tự động của AquaGuard dưới dạng hỏi
đáp, dùng cho việc trình bày và bảo vệ. Mọi con số đều lấy từ mã nguồn thật:
`backend/config/dispatch.js` (tham số) và `backend/services/dispatch.js` (logic).

---

## 6.1. Câu hỏi nền tảng

### Hỏi: Trước khi có tính năng này, việc phân công diễn ra thế nào?

Hoàn toàn thủ công và bị động. Người dân gửi SOS, bản ghi nằm ở trạng thái
`pending` và **không ai được báo**. Đội cứu hộ phải tự mở bảng điều khiển, tự
nhìn thấy, tự bấm nhận. Quản trị viên muốn phân công thì phải tự chọn đội trong
danh sách. Không có bất kỳ tiêu chí nào — ai nhìn thấy trước thì nhận, kể cả khi
họ ở xa hơn một đội khác đang rảnh ngay gần đó.

Với thiên tai, độ trễ đó là thứ đắt nhất.

### Hỏi: Cơ chế điều phối hiện tại hoạt động như thế nào?

**Giao thẳng** cho người phù hợp nhất. Hệ thống tự chọn và tự giao, không hỏi ý
kiến, không có bước chờ chấp nhận.

```
Người dân bấm SOS
      │
      ▼
POST /api/sos ──► trả 201 NGAY (không chờ điều phối)
      │
      └─► Điều phối chạy nền
              │
              ▼
      ① Xác định toạ độ nạn nhân
              │
              ▼
      ② Quét cứu hộ viên trong bán kính 5 km
              │
              ▼
      ③ Lọc: đang trực · còn hoạt động · có đội · chưa từng được giao ca này
              │
              ├── rỗng ──► nới 10 km ──► nới 20 km ──► hết: báo quản trị viên
              │
              ▼
      ④ Chấm điểm bốn tiêu chí, chọn người cao điểm nhất
              │
              ▼
      ⑤ GIAO NGAY: gán assigned_to, chuyển trạng thái sang 'assigned'
              │
              ▼
      ⑥ Báo hai phía: cứu hộ viên (hộp thoại) + người dân ("đội đang đến")
```

Toàn bộ diễn ra trong khoảng một giây, và người dân **không phải chờ** — phản
hồi HTTP đã trả về từ trước khi thuật toán chạy.

### Hỏi: Thuật toán chọn cứu hộ viên dựa trên gì? Công thức cụ thể?

Bốn tiêu chí, mỗi tiêu chí quy về thang **0 đến 1** (1 là tốt nhất), rồi nhân
trọng số và cộng lại:

```
điểm = 100 × ( w₁·gần + w₂·rảnh + w₃·tin_cậy + w₄·mới )
```

| Tiêu chí | Công thức | Ý nghĩa |
|---|---|---|
| **gần** (proximity) | `1 − min(khoảng_cách / bán_kính, 1)` | Ngay tại chỗ = 1, ở rìa bán kính = 0 |
| **rảnh** (availability) | `1 / (1 + số_ca_đang_làm)` | 0 ca = 1,00 · 1 ca = 0,50 · 2 ca = 0,33 |
| **tin cậy** (reliability) | `1 − tỉ_lệ_bỏ_ca` | Hay bỏ ca thì bị hạ điểm |
| **mới** (freshness) | `1 − min(tuổi_GPS / 60s, 1)` | Toạ độ càng cũ càng kém tin |

Trọng số **thay đổi theo độ khẩn cấp** của yêu cầu:

| Độ khẩn cấp | gần | rảnh | tin cậy | mới |
|---|---|---|---|---|
| Nguy kịch (`critical`) | **0,70** | 0,10 | 0,15 | 0,05 |
| Cao (`high`) | 0,60 | 0,18 | 0,17 | 0,05 |
| Trung bình (`medium`) | 0,50 | 0,25 | 0,20 | 0,05 |
| Thấp (`low`) | 0,40 | **0,35** | 0,20 | 0,05 |

Mỗi hàng cộng lại đúng bằng 1,00.

### Hỏi: Vì sao phải quy về thang 0–1 mà không cộng thẳng các giá trị?

Vì **đơn vị khác nhau thì trọng số vô nghĩa**. "3 km" và "2 nhiệm vụ" là hai
đại lượng không so sánh được: cộng thẳng lại thì kết quả phụ thuộc vào việc ta
đo bằng km hay bằng mét, chứ không phản ánh ý đồ thiết kế.

Chuẩn hoá về cùng một thang khiến trọng số trở thành thứ **đọc được và giải
thích được**: "0,70 cho khoảng cách" nghĩa là khoảng cách chiếm 70% quyết định.
Muốn thuật toán cư xử khác đi thì chỉnh trọng số, không phải viết lại công thức.

### Hỏi: Vì sao trọng số lại thay đổi theo độ khẩn cấp?

Vì mục tiêu của hai loại ca là khác nhau.

Ca **nguy kịch** — người đang bị nước cuốn — thì thứ duy nhất đáng kể là *đến
nhanh*. Kể cả khi người gần nhất đang bận hai ca khác, vẫn nên giao cho họ, vì
vài phút chênh lệch có thể là ranh giới sinh tử. Nên khoảng cách chiếm 0,70.

Ca **mức thấp** — hỗ trợ di dời đồ đạc chẳng hạn — thì không cần vội. Lúc này
mối lo lớn hơn là *vắt kiệt vài người chăm chỉ* trong khi những người khác rảnh.
Nên trọng số cân bằng tải tăng lên 0,35, gần ngang với khoảng cách.

Nói cách khác: một công thức, nhưng bốn cách ưu tiên tuỳ tình huống.

### Hỏi: Một cứu hộ viên phải thoả điều kiện gì mới được xét?

Năm điều kiện lọc, thực hiện bằng **một câu SQL duy nhất**:

1. `role = 'rescuer'` và `is_active = TRUE`
2. **`duty_status = 'on'`** — đang bật ca trực
3. Thuộc một đội cứu hộ đang hoạt động (`join_status = 'active'`)
4. Không phải chính người gửi yêu cầu
5. **Chưa từng được giao** đúng yêu cầu này (tránh giao lại người vừa bỏ ca)

Chỉ khi qua đủ năm cửa mới được chấm điểm.

### Hỏi: Bán kính quét là bao nhiêu, và vì sao lại có bậc thang?

Ba bậc: **5 km → 10 km → 20 km**.

Thuật toán thử bán kính nhỏ trước. Chỉ khi trong bán kính đó **không còn ứng
viên mới nào** mới nới rộng ra. Điều này quan trọng vì bước lọc đã loại những
người từng được giao ca này — nên khi 5 km "cạn người", việc nới rộng diễn ra
**tự động**, không cần logic riêng.

Vì sao không quét thẳng 20 km ngay từ đầu? Vì làm vậy sẽ đánh mất ưu thế của
người ở gần: một đội cách 18 km đang rảnh có thể vượt điểm một đội cách 2 km
đang bận, dù trong lũ thì 16 km chênh lệch là rất nhiều thời gian.

### Hỏi: Nếu không tìm được ai thì sao?

Yêu cầu **không bao giờ rơi vào im lặng**. Hệ thống gắn cờ
`dispatch_status = 'no_candidate'`, giữ nguyên trạng thái `pending`, và gửi
thông báo cho **toàn bộ quản trị viên** đang hoạt động với lý do cụ thể:

| Lý do | Nghĩa |
|---|---|
| `no_candidate` | Không có đội nào đang trực gần khu vực |
| `no_location` | Yêu cầu không có toạ độ GPS |
| `max_attempts` | Đã giao 5 lượt nhưng đều không thành |

Lúc này việc chuyển từ "máy lo" sang "người lo". Quản trị viên có thể phân công
tay, hoặc gọi `POST /api/dispatch/requests/:id/retry` để chạy lại điều phối sau
khi đã huy động thêm đội lên trực.

---

## 6.2. Câu hỏi kỹ thuật

### Hỏi: Hệ thống biết cứu hộ viên đang ở đâu bằng cách nào?

Gộp **hai nguồn**, ưu tiên nguồn tươi hơn:

| Nguồn | Đặc điểm | Vai trò |
|---|---|---|
| **Redis GEO** | Vị trí trực tiếp, hạn 60 giây | Ưu tiên khi có |
| **`user_locations`** | Vị trí cuối cùng, bền vững | Dự phòng, nhận toạ độ ≤ 60 phút |

Đây là một bài học rút ra khi kiểm thử thật: ban đầu hệ thống **chỉ** dùng Redis,
coi "có mặt trong Redis" là điều kiện bắt buộc. Nhưng trình duyệt **bóp thời gian
chạy của thẻ nền** và đóng băng hẳn khi điện thoại khoá màn hình — mà việc bơm
GPS lại dựa vào `setInterval`. Hậu quả: một cứu hộ viên đang trực, đang cầm máy,
vẫn "biến mất" khỏi danh sách chỉ vì họ chuyển sang thẻ khác.

Cách sửa dựa trên một nhận định về bản chất: **tín hiệu sẵn sàng thật là
`duty_status`** — do chính cứu hộ viên bật, bền vững, không phụ thuộc trình
duyệt. Vị trí trực tiếp chỉ nên quyết định *toạ độ nào chính xác hơn*, chứ không
có quyền loại một người đã tự khai báo là đang trực. Toạ độ cũ vẫn bị trừ điểm
qua tiêu chí "mới", nhưng không bị loại.

Song song đó, nhịp tim WebSocket 30 giây cũng được sửa để **ghi lại vị trí vào
Redis**. WebSocket không bị trình duyệt bóp như bộ đếm thời gian, nên kết nối
còn sống thì hiện diện còn sống.

### Hỏi: Vì sao dùng Redis GEO mà không tính khoảng cách bằng SQL?

Ba lý do:

1. **Tốc độ.** `GEOSEARCH` của Redis dùng cấu trúc geohash, trả kết quả đã sắp
   xếp kèm khoảng cách trong khoảng một mili-giây. Tính Haversine bằng SQL phải
   quét toàn bảng và tính lượng giác cho từng dòng.
2. **Sẵn có.** Hệ thống đã dùng Redis làm kho vị trí nóng cho bản đồ trực tiếp
   từ trước; hàm `nearbyUsers()` đã tồn tại, chỉ chưa được dùng ở đâu.
3. **Đúng ngữ nghĩa.** Khoá Redis có thời hạn 60 giây — nghĩa là chỉ những người
   thực sự đang gửi vị trí mới nằm trong tập, không cần lọc thêm.

Nhánh Haversine trong SQL vẫn tồn tại như nguồn phụ, cho trường hợp Redis chưa
được cấu hình hoặc kết nối hỏng.

### Hỏi: Vì sao đo đường chim bay mà không phải thời gian di chuyển thực tế?

Đây là lựa chọn có chủ đích, không phải cắt giảm.

Trong lũ lụt, **dữ liệu định tuyến đường bộ trở nên sai lệch**: đường bị ngập bị
API định tuyến coi là thông thoáng, trong khi thuyền cứu hộ lại đi thẳng qua chỗ
mà bản đồ coi là không có đường. Một ETA "chính xác" tính trên giả định đường sá
bình thường có thể sai hơn cả đường chim bay.

Cộng thêm: gọi API định tuyến cho từng ứng viên làm tăng độ trễ và phụ thuộc vào
dịch vụ ngoài — hai thứ không nên có trong đường xử lý khẩn cấp.

Nếu về sau muốn nâng cấp, hướng hợp lý là *lọc thô bằng đường chim bay rồi tính
ETA cho 3 người dẫn đầu*, chứ không phải thay thế hoàn toàn.

### Hỏi: Làm sao chống được tình huống hai người cùng nhận một yêu cầu?

Ba lớp bảo vệ, từ ngoài vào trong:

1. **Khoá trong tiến trình** — tập `inFlight` chặn hai luồng cùng điều phối một
   yêu cầu (nhịp watchdog và phản hồi người dùng có thể chạm nhau).
2. **Điều kiện trong câu lệnh cập nhật** — mọi thao tác giao việc đều mang
   `WHERE id = ? AND status = 'pending'`. Nếu ai đó vừa giành mất, `rowCount`
   bằng 0 và luồng hiện tại lặng lẽ nhường.
3. **Chỉ mục duy nhất trong cơ sở dữ liệu** — `idx_rdo_one_active_assignment`
   bảo đảm mỗi yêu cầu chỉ có đúng **một** phân công đang hiệu lực, kể cả khi
   hai tiến trình chạy song song.

Lớp 3 là chốt chặn cuối và không thể vượt qua, vì nó do chính cơ sở dữ liệu
cưỡng chế.

### Hỏi: Giao nhầm cho người vừa tắt máy thì sao?

Đây là rủi ro lớn nhất của việc giao thẳng, và là cái giá phải trả khi bỏ bước
hỏi trước. Giải pháp là một **watchdog** chạy mỗi 15 giây, thu hồi những ca chết
cứng.

Nó chỉ thu hồi khi thoả **cả hai** dấu hiệu:

1. Quá **120 giây** mà yêu cầu vẫn ở trạng thái `assigned` — người được giao
   chưa bấm bắt đầu, tức chưa chắc họ đã thấy
2. Cứu hộ viên **không còn hiện diện** trong Redis — thực sự đã ngoại tuyến

Vì sao phải cả hai? Chỉ điều kiện (1) thì oan cho người đang lái xe chưa kịp
bấm. Chỉ điều kiện (2) thì oan cho người vừa mất sóng ba giây.

Khi thu hồi, ca quay về `pending` và được giao cho người kế — người vừa bị thu
hồi tự động bị loại vì đã nằm trong lịch sử của ca này.

### Hỏi: Vì sao không chờ điều phối xong rồi mới trả kết quả cho người dân?

Vì **nạn nhân đang trong lũ**. Nguyên tắc là phản hồi HTTP phải trả về ngay khi
yêu cầu đã được ghi an toàn vào cơ sở dữ liệu; việc quét, chấm điểm và gửi thông
báo chạy nền theo kiểu "bắn rồi quên".

Điều này cũng có nghĩa: **thất bại của điều phối không được phép làm hỏng việc
tạo yêu cầu**. Toàn bộ lỗi trong luồng điều phối đều bị nuốt và ghi nhật ký —
đúng cùng một nguyên tắc mà hệ thống đã áp dụng cho việc gửi email.

### Hỏi: Máy chủ ngủ hoặc khởi động lại giữa chừng thì trạng thái có mất không?

Không. Đây là ràng buộc thực tế: backend chạy trên gói miễn phí của Render và
**ngủ sau khoảng 15 phút không có lưu lượng**.

Vì vậy mọi trạng thái điều phối nằm trong **cơ sở dữ liệu**, không nằm trong bộ
nhớ tiến trình. Cụ thể, hạn chót của watchdog được suy ra từ cột `created_at`
của bản ghi phân công chứ không phải từ một `setTimeout` đang treo — bởi mọi
`setTimeout` sẽ bốc hơi khi tiến trình ngủ.

Khi máy chủ thức dậy, nhịp quét đầu tiên xử lý bù toàn bộ những gì đã quá hạn.

---

## 6.3. Câu hỏi về lựa chọn thiết kế

### Hỏi: Vì sao chọn giao thẳng thay vì mời rồi chờ chấp nhận?

Đây là một quyết định đã được cân nhắc và **thay đổi trong quá trình phát triển**.

Phiên bản đầu dùng cơ chế mời tuần tự kiểu gọi xe công nghệ: mời người tốt nhất,
chờ 45 giây, không phản hồi thì chuyển người kế. Sau đó chuyển sang giao thẳng.

| | Mời + chờ nhận | Giao thẳng (hiện tại) |
|---|---|---|
| Độ trễ | Tối đa 45 giây mỗi lượt | Tức thì |
| Quyền của cứu hộ viên | Được từ chối | Không |
| Rủi ro chính | Không ai nhận, ca trôi qua nhiều lượt | Giao trúng người mất tích |
| Lưới an toàn cần có | Đồng hồ hết hạn | Watchdog thu hồi |

Lý do chuyển: trong cứu hộ thiên tai, **quyền từ chối không thực sự phù hợp**.
Cứu hộ viên đã chủ động bật ca trực, tức đã cam kết sẵn sàng — thêm một bước hỏi
lại chỉ làm chậm, trong khi tình huống không cho phép mặc cả.

Đổi lại phải chấp nhận rủi ro mới và xử lý nó bằng watchdog. Không có lựa chọn
nào miễn phí; điều quan trọng là biết mình đang đánh đổi cái gì.

### Hỏi: Giao cho cá nhân hay cho cả đội?

**Giao cho cá nhân, nhưng ghi sổ theo đội.** Cột `assigned_to` là một cứu hộ
viên cụ thể, còn `assigned_group_id` là đội của người đó.

Lý do là một khập khiễng có thật trong dữ liệu: **vị trí GPS là của từng người,
còn đội thì không có toạ độ**. Năm thành viên một đội có thể đang ở năm nơi khác
nhau, nên "đội này cách nạn nhân bao xa" là câu hỏi không có đáp án rõ ràng.

Ghi kèm mã đội giữ được lợi ích của mô hình đội: đồng đội vẫn nhìn thấy nhiệm vụ
của nhau qua `GET /api/sos/team`, và toàn bộ giao diện đội **không phải sửa gì**.

Đi kèm quyết định này là việc **bỏ ràng buộc chỉ trưởng/phó nhóm mới được nhận
nhiệm vụ** — vì nếu người gần nhất là thành viên thường mà không có quyền nhận
thì thuật toán trở nên vô nghĩa.

### Hỏi: `duty_status` khác gì với "đang trực tuyến"?

Đây là hai khái niệm dễ nhầm nhưng khác nhau về bản chất:

| | Trực tuyến (presence) | Ca trực (`duty_status`) |
|---|---|---|
| Ai quyết định | Trình duyệt, tự động | Cứu hộ viên, chủ động bấm |
| Ý nghĩa | "Ứng dụng đang mở" | "Tôi sẵn sàng nhận nhiệm vụ" |
| Độ bền | Hết hạn sau 60 giây | Lưu trong cơ sở dữ liệu |
| Vai trò | Cung cấp toạ độ chính xác | **Điều kiện bắt buộc để được xét** |

Mở ứng dụng không có nghĩa là sẵn sàng đi cứu hộ — người ta có thể đang ăn cơm,
đang ngủ, hoặc chỉ xem tin tức lũ lụt. Tách hai khái niệm này giúp hệ thống
không làm phiền người không sẵn sàng, đồng thời không bỏ sót người đang sẵn sàng
mà trình duyệt tình cờ ngưng gửi tín hiệu.

Mặc định là **tắt** — hệ thống không tự ý coi ai đó đang trực.

### Hỏi: Đo tải công việc theo cá nhân hay theo đội?

Theo **cá nhân** — đếm số yêu cầu mà chính người đó đang giữ ở trạng thái
`assigned` hoặc `in_progress`.

Vì việc giao là cho cá nhân, nên đo tải theo đội sẽ dẫn tới nghịch lý: một người
đang rảnh bị trừ điểm chỉ vì đồng đội bận, còn một người đang ôm ba ca vẫn được
chọn nếu đội của họ ít việc.

Công thức `1 / (1 + số_ca)` giảm dần chứ **không cắt cứng** — nghĩa là một ca
nguy kịch vẫn có thể được giao cho người đang bận, miễn là họ gần hơn hẳn.

### Hỏi: Độ tin cậy được đo bằng gì?

Bằng **tỉ lệ bỏ ca** trong 7 ngày gần nhất: số ca được giao rồi bị thả lại, chia
cho tổng số ca được giao.

Không phải kết cục nào cũng bị tính:

| Kết cục | Tính vào tỉ lệ bỏ ca? |
|---|---|
| `released` — tự bỏ ca | Có |
| `reassigned` — watchdog thu hồi vì mất tích | Có |
| `superseded` — quản trị viên phân công đè lên | **Không** (không phải lỗi của họ) |
| `completed` — hoàn tất | **Không** |

Có một ngưỡng bảo vệ: dưới **3 lần** được giao thì coi như hoàn toàn tin cậy.
Nếu không, một người mới vào bị thu hồi đúng ca đầu tiên sẽ mang tỉ lệ bỏ ca
100% và gần như không bao giờ được giao việc nữa — một cái bẫy dữ liệu thưa
kinh điển.

Tiêu chí này ban đầu đo *tỉ lệ từ chối lời mời*. Khi chuyển sang giao thẳng,
khái niệm "từ chối" biến mất, nên nó được chuyển sang đo tỉ lệ bỏ ca — giữ
nguyên ý nghĩa "người này có đáng tin không", chỉ đổi cách quan sát.

### Hỏi: Quản trị viên còn can thiệp được không?

Có, và luôn được ưu tiên hơn máy. Khi quản trị viên phân công tay, yêu cầu được
đánh dấu `dispatch_status = 'manual'` và thuật toán **rút lui hẳn** khỏi yêu cầu
đó.

Điều này khả thi nhờ một quyết định về cấu trúc dữ liệu: quá trình điều phối
được ghi ở cột **song song** `dispatch_status`, không lẫn vào vòng đời
`pending → in_progress → resolved` vốn có. Nhờ vậy quản trị viên luôn nhìn thấy
bức tranh thật và giành quyền được ở bất kỳ thời điểm nào.

Ngoài ra `GET /api/dispatch/requests/:id/trail` cho phép xem lại **dấu vết điều
phối**: đã giao cho ai, khoảng cách bao nhiêu, điểm bao nhiêu, kết cục ra sao.
Đây là thứ trả lời được câu hỏi "vì sao hệ thống lại chọn người này".

---

## 6.4. Hạn chế và hướng phát triển

### Hỏi: Hệ thống hiện còn hạn chế gì?

Bốn điểm, nêu thẳng:

1. **Khoảng cách là đường chim bay.** Ở đô thị khô ráo, đường bộ có thể dài hơn
   đáng kể so với đường thẳng. Đã giải thích ở trên vì sao đây là lựa chọn hợp
   lý *trong bối cảnh lũ lụt*, nhưng nó vẫn là một xấp xỉ.

2. **Không có mô hình năng lực đội.** Hệ thống chưa phân biệt đội có thuyền với
   đội chỉ đi bộ, hay ca cần y tế với ca cần di dời. Mọi cứu hộ viên được coi là
   thay thế được cho nhau.

3. **Chưa có giới hạn cứng số ca.** Tải chỉ trừ điểm chứ không loại. Về lý
   thuyết một người rất gần có thể bị dồn nhiều ca cùng lúc trong đợt cao điểm.

4. **Chỉ chạy trên một tiến trình.** Khoá `inFlight` nằm trong bộ nhớ, nên nếu
   nhân rộng ra nhiều tiến trình thì phải chuyển sang khoá phân tán. Hai lớp bảo
   vệ còn lại (điều kiện cập nhật và chỉ mục duy nhất) vẫn đúng, nên đây là vấn
   đề hiệu năng chứ không phải vấn đề đúng đắn.

### Hỏi: Hướng phát triển tiếp theo?

- **Năng lực và chuyên môn**: gắn thẻ cho đội (thuyền, y tế, cứu hộ trên cao) và
  cho yêu cầu, đưa mức độ khớp vào công thức chấm điểm.
- **Thời gian di chuyển thực**: lọc thô bằng đường chim bay rồi tính ETA cho ba
  ứng viên dẫn đầu, kết hợp dữ liệu vùng ngập nếu có.
- **Điều phối theo lô**: khi nhiều yêu cầu đến cùng lúc, giải bài toán ghép cặp
  tối ưu toàn cục thay vì xử lý lần lượt từng yêu cầu.
- **Học từ dữ liệu**: dùng thời gian hoàn thành thực tế để tự hiệu chỉnh trọng
  số, thay vì đặt tay như hiện nay.

### Hỏi: Có thể tắt tính năng này không?

Có. Đặt biến môi trường `DISPATCH_ENABLED=false` là hệ thống quay về đúng hành
vi cũ — yêu cầu nằm ở `pending` chờ quản trị viên phân công tay. Không cần gỡ
mã, không cần hoàn tác cơ sở dữ liệu.

Đây là một yêu cầu bắt buộc với tính năng chạm vào đường xử lý khẩn cấp: phải
luôn có đường lui an toàn.

---

## 6.5. Tóm tắt tham số

| Tham số | Giá trị | Biến môi trường |
|---|---|---|
| Bậc thang bán kính | 5 → 10 → 20 km | `DISPATCH_RADIUS_LADDER` |
| Số lượt giao tối đa | 5 | `DISPATCH_MAX_ATTEMPTS` |
| Ngưỡng watchdog thu hồi | 120 giây | `DISPATCH_STALE_ASSIGNMENT` |
| Chu kỳ watchdog | 15 giây | `DISPATCH_SWEEP_INTERVAL_MS` |
| Hạn vị trí trực tiếp (Redis) | 60 giây | — |
| Hạn vị trí dự phòng | 60 phút | `DISPATCH_FALLBACK_MAX_AGE_MIN` |
| Ngưỡng có ý nghĩa của lịch sử | 3 lần | `DISPATCH_MIN_OFFERS_HISTORY` |
| Cửa sổ lịch sử | 7 ngày | `DISPATCH_HISTORY_WINDOW_DAYS` |
| Trạng thái sau khi giao | `assigned` | `DISPATCH_ASSIGNED_STATUS` |
| Công tắc tổng | bật | `DISPATCH_ENABLED` |

## 6.6. Các tệp liên quan

| Tệp | Vai trò |
|---|---|
| `backend/config/dispatch.js` | Toàn bộ tham số và trọng số |
| `backend/services/dispatch.js` | Thuật toán: lọc, chấm điểm, giao, watchdog |
| `backend/routes/dispatch.js` | API: ca trực, dấu vết, chạy lại |
| `backend/migrations/011_dispatch.sql` | Bảng phân công, cột ca trực |
| `backend/migrations/012_direct_assign.sql` | Chuyển sang chế độ giao thẳng |
| `frontend/src/contexts/DispatchContext.jsx` | Nhận thông báo qua WebSocket |
| `frontend/src/components/dispatch/` | Hộp thoại phân công, công tắc ca trực |
| `backend/scripts/seed-dispatch-test.js` | Dựng dữ liệu giả để kiểm thử |
