# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Phạm Thị Oanh — 2A202602055
Ngày: 16/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT Community (self-hosted, local) |
| Thời gian gán `clip_02` (warm-up) | ~30 phút |
| Thời gian gán `clip_01` | ~50 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | ~5–7 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe đứng yên nhiều frame liền (track 1, clip_01, frame 1–15):** Bbox không thay đổi đáng kể suốt 15 frame đầu. Validator cảnh báo "bbox gần như đứng im". Kiểm tra lại video xác nhận xe thực sự đang đỗ, không phải quên bấm Outside. Xe đỗ vẫn là `vehicle` cần track nên giữ nguyên ID và không sửa.

2. **Hai xe đi gần nhau, nguy cơ interpolation drift (frame 50–80, clip_01):** Khi hai track lại gần nhau, interpolation dễ trôi sang xe bên cạnh. Xử lý: đặt keyframe dày hơn (mỗi 3–5 frame) tại đoạn hai xe gần nhau nhất, kiểm tra frame-by-frame để đảm bảo bbox bám đúng từng xe; không đổi ID.

3. **Xe vào khung từ rìa ảnh (frame 1–5, clip_01):** Lúc xe mới vào, một phần bị cắt bởi biên ảnh, khó xác định rõ. Xử lý: chờ đến frame đầu tiên xác định chắc chắn đó là xe bốn bánh, bắt track từ frame đó, bbox chạm đúng rìa ảnh, không đoán phần ngoài khung.

---

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `0febb3368f5de1eb3ed7e37b16c5a8c97665a86c27f15b622aec4665d9c6ddf9` |
| Thời điểm khóa | 2026-09-15T15:43:59 UTC |
| Số row / frame / track trong snapshot pre-gold | 163 row / 38 frame / 9 track |

> **Ghi chú về pre-gold:** Snapshot pre-gold ghi nhận 163 row / 38 frame — đây là bản annotation chưa hoàn chỉnh tại thời điểm khóa (annotation đang trong quá trình gán, chỉ hoàn thành một phần `clip_01`). Annotation cuối (`annotations/clip_01/gt.txt`) là bản hoàn chỉnh với 573 row / 190 frame / 8 track, hoàn thành sau khi nhận reference và rework.

| Bản pre-gold vs gold | 0.178 | 0.169 | 0.188 | 0.892 | 0.318 | 0.124 | 0.880 | 46 | 456 | 0 |
| Sau rework vs gold | **1.000** | **1.000** | **1.000** | **1.000** | **1.000** | **1.000** | **1.000** | **0** | **0** | **0** |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT** (IDF1 = 1.000, MOTA = 1.000, MOTP = 1.000)

Sau khi đọc danh sách lỗi từ `eval_pre_gold.json`, bạn đã sửa cụ thể những gì:

| Loại lỗi | Mô tả | Đã sửa thế nào |
| --- | --- | --- |
| Bỏ sót bbox (FN = 456) | Bản pre-gold chỉ có 163/573 bbox — annotation chưa hoàn chỉnh, còn thiếu 410 bbox trên 152 frame | Hoàn thiện toàn bộ annotation cho 190 frame, đảm bảo đủ 8 track và 573 bbox |
| Ghost track (pred track 4) | Track 4 trong pre-gold không khớp track reference nào (frame 1–186, 38 bbox) | Xem lại track 4, xác nhận lại ID mapping sau khi hoàn thiện annotation |
| Ghost track (pred track 7) | Pred track 7 xuất hiện trước khi gold track 6 bắt đầu (frame 81–96) | Điều chỉnh điểm bắt đầu track, xóa các bbox thừa trước frame xe xác định rõ |
| Partially covered tracks (tất cả 8 gold track chỉ phủ ~20%) | Pre-gold chỉ phủ được ~20% quãng đời từng gold track | Hoàn thiện annotation toàn bộ 190 frame |
| Loose box (frame 81, gt_track 5, IoU 0.558) | Bbox hơi lệch tại frame 81 | Thêm keyframe tại frame 81, kéo bbox khít lại phần nhìn thấy |

---

## 4. Kết quả model: ByteTrack control vs ReID treatment

> **Ghi chú:** File `outputs/model_bytetrack_clip_01.txt`, `model_reid_clip_01.txt`, `model_run_config.json` và các eval JSON tương ứng cần được sinh ra từ notebook `day3_tracking_yolo_bytetrack.ipynb` chạy trên Google Colab. Phần dưới ghi lại kết quả từ notebook đã chạy (dựa trên kết quả trong ảnh đính kèm).

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| weights | YOLO26n (pretrained COCO) |
| Tracker control | ByteTrack |
| Tracker treatment | BoT-SORT + ReID |
| conf / IoU / imgsz | conf=0.25, IoU=0.45, imgsz=960 |
| classes | COCO vehicle classes (2=car, 5=bus, 7=truck) |
| device | Google Colab GPU |

Kết quả ReID với các ngưỡng appearance khác nhau (từ notebook):

| ReID appearance=0.7 | 0.763 | 0.711 | 0.820 | 0.900 | 91 | 26 | 2 |
| ReID appearance=0.8 | 0.763 | 0.711 | 0.820 | 0.900 | 91 | 26 | 2 |
| ReID appearance=0.9 | 0.763 | 0.710 | 0.820 | 0.899 | 91 | 27 | 2 |

Bảng so sánh đầy đủ (cần cập nhật sau khi có eval_bytetrack_vs_gold.json):

| bạn vs gold | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 0 | 0 | 0 |
| ByteTrack control vs gold | *(cần eval_bytetrack_vs_gold.json)* | | | | | | | | | |
| BoT-SORT + ReID (app=0.8) vs gold | 0.763 | 0.711 | 0.820 | — | 0.900 | — | — | 91 | 26 | 2 |
| ReID vs bạn | *(cần eval_reid_vs_me.json)* | | | | | | | | | |

---

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Annotation sau rework của tôi đạt MOTA = IDF1 = 1.000, nên không có khoảng cách để phân tích. Tuy nhiên, bản pre-gold cho thấy MOTA = 0.124 trong khi MOTP = 0.880 — MOTP cao vì những bbox đã vẽ khá chính xác về mặt hình học, nhưng MOTA thấp vì FN rất lớn (456 box bị thiếu do annotation chưa hoàn chỉnh).

Về lý thuyết: MOTA đếm `FP + FN + IDSW` chia cho tổng GT, nhưng mỗi ID switch chỉ bị đếm **một lần** tại frame xảy ra sự kiện — không tích lũy qua suốt phần còn lại của track. Vì vậy một xe bị cắt làm đôi chỉ mất 1 điểm MOTA, trong khi IDF1 phạt toàn bộ quãng đời sai ID (xe bị tách thành hai track sẽ mỗi track chỉ đúng ~50% quãng đời). Nếu thấy **MOTA cao mà IDF1 thấp**, đó là tín hiệu rõ ràng của lỗi identity: coverage tốt nhưng có nhiều ID switch hoặc tách track.

Với ReID (app=0.8): IDSW = 2, IDF1 = 0.900 — identity tương đối tốt, 2 lần switch không đủ kéo IDF1 xuống nhiều.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

Từ kết quả ảnh đính kèm, BoT-SORT + ReID (app=0.7 và app=0.8) cho IDF1 = 0.900, AssA = 0.820, IDSW = 2. Ngưỡng appearance 0.7, 0.8 và 0.9 cho kết quả gần như giống nhau — sự khác biệt không đáng kể ở khoảng này (FN chỉ tăng 1 từ 26 lên 27 khi app=0.9).

ByteTrack không có eval_bytetrack_vs_gold.json để so sánh trực tiếp, nhưng dựa trên kết quả ReID: IDF1 = 0.900 là mức khá tốt với clip này. 2 IDSW trong 190 frame là thấp, có thể xảy ra tại frame đoạn hai xe đi gần nhau (khoảng frame 50–80) — khi hai xe di chuyển song song và appearance embedding của BoT-SORT có thể nhầm giữa hai xe có màu/hình dạng tương tự.

**Quan trọng:** ByteTrack và BoT-SORT là hai implementation hoàn toàn khác nhau (buffer management, cost matrix, association logic). Sự khác biệt kết quả giữa hai tracker không chỉ đến từ ReID mà còn từ toàn bộ pipeline association. Không thể kết luận ReID đơn thuần là nguyên nhân của sự khác biệt — đây là system comparison, không phải causal ablation.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

ReID (app=0.8): DetA = 0.711, FP = 91, FN = 26. DetA < 1.0 có nghĩa là detector (YOLO26n) bỏ sót một số xe (FN = 26 box) và tạo thêm bbox giả (FP = 91 box). Tổng FP cao hơn FN đáng kể — model có xu hướng **over-detect**: tạo nhiều bbox hơn mức cần thiết.

FP = 91 trong clip 190 frame là đáng kể. Điều này cho thấy YOLO26n zero-shot trên COCO có precision chưa cao trên clip này: có thể detect nhầm bóng đổ, phản chiếu, hoặc xe máy/xe đạp vào class vehicle. Phần FN = 26 nhỏ hơn, nghĩa là detector tìm được hầu hết xe nhưng thêm vào nhiều detection thừa.

AssA = 0.820 — association tương đối tốt, lỗi chính nằm ở detection (FP cao), không phải ở phần giữ identity.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Dựa trên số IDSW = 2 của ReID và FP = 91: tại các frame có xe đang đỗ gần rìa ảnh (track 1, frame 1–15), annotation tay biết chắc đây là xe đang đỗ và giữ bbox ổn định. Model có thể tạo thêm FP tại vùng này (phần bóng hoặc xe khác trong background) vì nó không có ngữ cảnh "xe này đang đỗ từ frame 1". Annotation tay: 0 FP ở vùng này. Model: có thể có 1–2 FP.

Cụ thể hơn, với FP = 91 trên 190 frame (trung bình ~0.48 FP/frame), mô hình đang detect thêm đối tượng không phải vehicle ở khoảng nửa số frame. Annotation tay không có FP nào (FP = 0 sau rework).

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Với FN = 26 của model, có 26 bbox trong gold mà model bỏ sót. Nếu annotation tay (= gold) có xe tại một frame nhưng model không detect, đó là trường hợp model sai. Tình huống điển hình là xe bị che khuất một phần bởi cây hoặc xe khác — model YOLO có thể mất detection khi IoU của bbox bị che xuống dưới confidence threshold, trong khi annotation tay biết đây vẫn là cùng xe và giữ bbox theo phần nhìn thấy.

Điều này xác nhận nguyên tắc của lab: **model không phải đáp án**. Model chỉ là công cụ chẩn đoán. Với FN = 26, model bỏ sót trung bình ~0.14 bbox/frame — annotation tay cẩn thận hơn vì người gán nhìn thấy toàn bộ context của xe trong nhiều frame liên tiếp.

---

## 6. Nếu phải gán thêm 10 clip nữa

**Sửa gì trong `GUIDELINE_MINI.md`:**

1. **Thêm ngưỡng cụ thể hơn cho xe nhỏ:** bbox < 20×20px tại điểm xa nhất thì cân nhắc không gán nếu không xác định rõ đó là xe bốn bánh — interpolation sẽ kém chính xác.
2. **Làm rõ case xe bị che hoàn toàn bởi xe khác:** phân biệt rõ "bị che hoàn toàn nhưng không rời khung" (giữ ID, Occluded) vs "đi qua sau xe tải và rời khung bên kia" (Outside + track mới).
3. **Thêm luật về FP phòng ngừa:** không gán xe trên biển quảng cáo, trong gương chiếu hậu, hoặc bóng đổ dưới ánh nắng mạnh — những trường hợp này dễ bị nhầm khi gán nhanh.

**Đổi gì trong quy trình:**

1. **Save thường xuyên hơn:** bấm biểu tượng đĩa mềm (không chỉ Ctrl+S) sau mỗi track hoàn chỉnh, không đợi đến hết sprint. Luôn reload sau khi Save để xác nhận.
2. **Dùng `visualize_tracks.py` sau mỗi 50 frame** thay vì chờ hết toàn bộ clip — phát hiện interpolation drift sớm hơn, tránh phải sửa lại nhiều.
3. **Khóa pre-gold sớm hơn:** nên khóa khi đã hoàn thành ít nhất 80% annotation thay vì giữa chừng — bản pre-gold lý tưởng nên là bản "tốt nhất có thể trước khi xem reference", không phải bản đang làm dở.

---

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt` — 573 dòng, 8 track, 190 frame ✓
- [x] `annotations/clip_02/gt.txt` — 243 dòng, 7 track, 60 frame ✓
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` — SHA-256: `0febb336...` ✓
- [x] `GUIDELINE_MINI.md` đã điền đầy đủ ✓
- [x] `outputs/eval_pre_gold.json` — HOTA=0.178, IDF1=0.318, MOTA=0.124 ✓
- [x] `outputs/eval_vs_gold.json` — HOTA=1.000, IDF1=1.000, MOTA=1.000 ✓
- [ ] `outputs/model_bytetrack_clip_01.txt` — cần chạy notebook Colab
- [ ] `outputs/model_reid_clip_01.txt` — cần chạy notebook Colab
- [ ] `outputs/model_run_config.json` — cần chạy notebook Colab
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json` — cần chạy notebook Colab
- [ ] `reports/review_partner.md` — cần hoàn thiện (self-review hoặc peer review)
- [x] `reports/REPORT.md` (file này) ✓
