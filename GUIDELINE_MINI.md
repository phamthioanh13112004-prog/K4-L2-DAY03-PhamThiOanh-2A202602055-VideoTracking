# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Phạm Thị Oanh
Clip: `clip_01`, `clip_02`


## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `Không có`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** | Theo quy tắc lab: 25 frame tương đương khoảng 2 giây ở 12.5 fps; vẫn xem là cùng một đối tượng |
| Xe bị che lâu hơn ngưỡng trên | tạo track mới | Khi bị che quá lâu, không đủ chắc chắn để đảm bảo đó vẫn là cùng một đối tượng |
| Xe rời khung hình rồi quay lại | **track mới** | Xe đã rời khỏi khung hình nên khi xuất hiện lại cần xem là một lần xuất hiện mới |
| Hai xe cắt nhau / chồng lên nhau | giữ nguyên ID của từng xe và kiểm tra frame-by-frame quanh đoạn giao nhau | Tránh đổi ID khi hai xe đi gần hoặc che nhau; ID phải đi theo đúng đối tượng |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**, giữ cùng ID và bật Occluded nếu phù hợp |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định chắc chắn đó là xe bốn bánh; không đặt một ngưỡng kích thước pixel cố định |
| Xe đang đỗ, không di chuyển | vẫn gán `vehicle` và giữ track trong thời gian xe còn xuất hiện trong khung hình |
| Keyframe đặt dày ở đâu | đặt dày hơn khi xe rẽ, phanh, thay đổi hướng, bị che hoặc bbox có nguy cơ trôi; khi xe đi thẳng đều có thể đặt thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_02 / frame 1–15 / ID 2`
- Tình huống: bbox của track 2 gần như đứng im trong 15 frame đầu.
- Quyết định: giữ nguyên ID và bbox nếu kiểm tra video xác nhận xe thực sự đứng yên; không tự đổi ID chỉ vì bbox ít thay đổi.
- Lý do: validator cảnh báo bbox gần như đứng im, nhưng xe đang đỗ vẫn thuộc lớp `vehicle` và cần được track trong thời gian còn ở trong khung hình.

### Ca 2

- Clip / frame / ID: `clip_02 / frame 1–15 / ID 3`
- Tình huống: bbox của track 3 gần như đứng im trong 15 frame đầu.
- Quyết định: kiểm tra trực tiếp video; nếu xe thực sự đứng yên thì giữ nguyên track và ID.
- Lý do: bbox đứng yên không tự động có nghĩa là gán nhãn sai. Chỉ cần sửa nếu xác nhận đã quên Outside hoặc bbox không còn tương ứng với xe.

### Ca 3

- Clip / frame / ID: `clip_02 / frame 1–3 / ID 4`
- Tình huống: track 4 chỉ xuất hiện trong 3 frame.
- Quyết định: kiểm tra ba frame đầu và các frame kế tiếp để xác định xe thực sự chỉ xuất hiện trong khoảng này hay track bị tạo nhầm.
- Lý do: validator cảnh báo track quá ngắn; cần dựa vào hình ảnh thực tế để quyết định giữ hay xóa track, không sửa chỉ để làm validator hết cảnh báo.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Hiện tại chưa thực hiện bước chấm với gold và kiểm chéo, nên chưa có finding thực tế để ghi nhận. Sau khi thực hiện peer review/gold comparison, cần cập nhật lại các quy tắc nếu phát hiện điểm chưa rõ.

- `Cần kiểm tra kỹ các track có bbox gần như đứng im để phân biệt xe thực sự đứng yên với trường hợp quên Outside.`
- `Cần kiểm tra các đoạn track bị đứt quãng để phân biệt trường hợp xe bị che khuất với trường hợp xe rời khung rồi quay lại; trường hợp sau phải tạo track mới.`
---

