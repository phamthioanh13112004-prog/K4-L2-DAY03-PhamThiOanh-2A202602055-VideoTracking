# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Phạm Thị Oanh — 2A202602055
Clip: `clip_01`, `clip_02`


## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |
| xe đang đỗ, không di chuyển (vẫn cần track) | biển báo giao thông, cột điện |

Bổ sung của nhóm:
- **Xe quá nhỏ hoặc quá xa:** chỉ bắt đầu track khi xác định rõ đó là xe bốn bánh — không đặt ngưỡng pixel cố định, nhưng nếu bbox nhỏ hơn khoảng 15×15px thì cần đặc biệt thận trọng. Nếu không chắc, chờ thêm 2–3 frame.
- **Xe bị cắt bởi rìa ảnh ngay từ frame 1:** bắt track ngay frame đó, bbox chạm rìa, không đợi đến khi xe vào hẳn.
- **Xe trên biển quảng cáo, trong gương chiếu hậu xe khác:** không gán, đây là ảnh/phản chiếu, không phải đối tượng thật trong cảnh.


## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | **giữ nguyên ID** nếu bị che **dưới 25 frame** (≈ 2 giây ở 12.5 fps) | Trong ngưỡng 2 giây, vẫn đủ chắc đây là cùng một đối tượng dựa trên motion continuity |
| Xe bị che hoàn toàn nhưng không rời khung, che **dưới 25 frame** | **giữ nguyên ID**; bật Occluded tại các frame bị che hoàn toàn | Xe chưa ra khỏi scene; chỉ bị khuất tạm thời |
| Xe bị che lâu hơn 25 frame | **tạo track mới** | Quá 2 giây không đủ chắc chắn đây vẫn là cùng xe |
| Xe rời khung hình rồi quay lại (dù chỉ 1 frame) | **track mới** | Xe đã ra khỏi khung là kết thúc track đó; lần xuất hiện sau là đối tượng mới trong scene |
| Hai xe cắt nhau / chồng lên nhau | **giữ nguyên ID từng xe**; kiểm tra frame-by-frame quanh điểm giao | ID phải đi theo đúng đối tượng; không được đổi ID chỉ vì hai bbox chồng nhau |
| Xe bị che hoàn toàn bởi xe khác (ví dụ xe đi sau xe tải) | Nếu xe đó **không rời khung**: giữ ID, bật Occluded. Nếu xe đó **đã rời khung** qua bên kia xe tải: bấm Outside tại frame cuối còn thấy, track mới khi hiện lại | Phân biệt "bị che trong khung" vs "đã ra ngoài khung dù mắt không nhìn thấy" |
| Xe đứng yên nhiều frame liền (ví dụ đang đỗ) | **giữ nguyên track và ID**; không tự xóa hay sửa chỉ vì bbox ít thay đổi | Xe đỗ vẫn là `vehicle`; validator cảnh báo bbox đứng im là để nhắc kiểm tra, không phải ra lệnh xóa |
| Merge hai track nhầm | Dùng chức năng **Merge** trong CVAT (phím `M`): click bbox track 1 → click bbox track 2 → bấm `M` lần nữa để chốt | Tuyệt đối không xóa rồi vẽ lại để tránh mất interpolation đã làm |


## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa ảnh (x=0 hoặc x=width); **không đoán** phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm **phần nhìn thấy được**, bật Occluded nếu phần bị che đáng kể (> 30% diện tích ước tính) |
| Xe bị che hoàn toàn | không vẽ bbox; bật **Outside** tại frame đầu tiên xe vắng mặt hoàn toàn |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên **xác định chắc chắn** đó là xe bốn bánh; nếu nghi ngờ, ghi vào Ca mơ hồ |
| Xe đang đỗ, không di chuyển | vẫn gán `vehicle`, giữ track trong toàn bộ thời gian xe còn trong khung |
| Keyframe đặt dày ở đâu | đặt dày hơn khi: xe rẽ, phanh, tăng tốc, đổi hướng, bị che một phần, bbox có nguy cơ trôi; khi xe đi thẳng đều tốc độ ổn định có thể thưa hơn |
| Xe đang quay đầu / đổi làn | đặt keyframe ở đầu, giữa và cuối đoạn quay; interpolation giữa ba keyframe này thường đủ chính xác |
| Kiểm tra interpolation drift | tua ngược về **giữa hai keyframe xa nhất** và kiểm bbox — đây là chỗ lỗi trốn lâu nhất |
| Sau khi bấm Outside | kiểm tra frame liền sau: bbox không được tiếp tục xuất hiện. Nếu vẫn còn bbox ở frame sau Outside, track chưa thực sự đóng đúng |


## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1 — Xe đỗ bbox gần như đứng im

- Clip / frame / ID: `clip_01 / frame 1–15 / ID 1`
- Tình huống: bbox của track 1 gần như đứng im suốt 15 frame đầu (validator báo cảnh báo "bbox gần như đứng im").
- Quyết định: **giữ nguyên track và ID**. Kiểm tra video xác nhận xe thực sự đang đỗ, không phải quên bấm Outside.
- Lý do: xe đỗ vẫn là `vehicle` cần track. Cảnh báo validator chỉ nhắc kiểm tra, không phải lệnh xóa. Bbox đứng im ≠ annotation sai.
- Bài học: luôn kiểm tra video trực tiếp trước khi sửa theo cảnh báo tự động.

### Ca 2 — Xe đỗ trong clip_02 (track 2 và track 3)

- Clip / frame / ID: `clip_02 / frame 1–15 / ID 2` và `clip_02 / frame 1–15 / ID 3`
- Tình huống: cả hai track có bbox gần như đứng im từ frame 1 đến frame 15 — cùng pattern với Ca 1.
- Quyết định: **giữ nguyên** sau khi xác nhận bằng video rằng đây là các xe đang đỗ trong khung.
- Lý do: tình huống xe đỗ xuất hiện nhiều ở đầu clip khi camera mới đến cảnh. Luật nhất quán: chỉ sửa khi xác nhận đã quên Outside hoặc bbox lệch ra khỏi xe.

### Ca 3 — Track rất ngắn (3 frame)

- Clip / frame / ID: `clip_02 / frame 1–3 / ID 4`
- Tình huống: track 4 chỉ xuất hiện trong 3 frame đầu rồi biến mất (validator cảnh báo "track quá ngắn").
- Quyết định: kiểm tra ba frame đó và các frame kế tiếp. Nếu xe rời khung sau frame 3 và không quay lại — **giữ nguyên track**, bấm Outside tại frame 3. Nếu xe thực ra vẫn còn trong khung nhưng track bị mất — cần kéo dài track hoặc tạo lại.
- Lý do: track ngắn không tự động là lỗi. Xe đi qua nhanh hoặc chỉ xuất hiện ở rìa ảnh trong vài frame là hoàn toàn hợp lệ.
- Bài học: luôn dựa vào hình ảnh thực tế để quyết định, không sửa chỉ để làm validator hết cảnh báo.

### Ca 4 — Hai xe đi gần nhau, nguy cơ đổi ID

- Clip / frame / ID: `clip_01 / frame 50–80 / ID 3 và ID 5` *(ước tính — kiểm tra lại trên video)*
- Tình huống: hai xe đi song song, bbox gần chồng lên nhau trong một số frame. Interpolation có thể trôi sang xe bên cạnh.
- Quyết định: đặt keyframe dày hơn tại đoạn hai xe gần nhau nhất (khoảng mỗi 3–5 frame); kiểm tra frame-by-frame quanh điểm gần nhau để đảm bảo mỗi bbox bám đúng xe.
- Lý do: CVAT interpolation tuyến tính giữa hai keyframe — nếu hai keyframe cách nhau xa và xe thay đổi vị trí đột ngột, bbox trung gian sẽ trôi. Đặt keyframe dày là cách duy nhất tránh drift.


## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

*(Cập nhật sau khi nhận gold từ Lab Coach và hoàn thành peer review)*

**Những điểm đã biết cần theo dõi:**

- `Kiểm tra kỹ track 1 (clip_01) và track 2, 3 (clip_02) — các xe đứng yên ở đầu clip.` Cần xác nhận Outside đã được bấm đúng khi xe rời khung, không phải chỉ khi bbox ngừng di chuyển.
- `Kiểm tra track 4 (clip_02, 3 frame) — xác nhận đây là xe thật vào khung ngắn, không phải artifact hay nhầm.`
- `Kiểm tra các đoạn interpolation dài (> 15 frame giữa hai keyframe) để phát hiện bbox drift tại đoạn xe thay đổi hướng.`
- `Sau peer review và gold: cập nhật bảng finding, điền closure (fixed / not-a-defect / needs-review) và ghi lại rule nào cần làm rõ thêm.`

**Template finding để điền sau khi có kết quả:**

| Finding | Frame | ID | Loại lỗi | Đã sửa thế nào | Closure |
| --- | --- | --- | --- | --- | --- |
| *(từ gold eval)* | — | — | — | — | — |
| *(từ peer review)* | — | — | — | — | — |

---

*Cập nhật lần cuối: 16/09/2026 — Phạm Thị Oanh*
