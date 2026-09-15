# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Võ Thị Bảo Chi / SOLO`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `~20 phút` |
| Thời gian gán `clip_01` | `~90 phút` |
| Số track đã vẽ trong `clip_01` | `7` |
| Số keyframe trung bình mỗi track | `~8–10` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Track 5` bị drift quanh frame 84–88 do interpolation tạo bbox lệch khỏi phần xe nhìn thấy. Tôi thêm keyframe trước/sau vị trí lệch và kiểm lại ở midpoint bằng mắt.
2. `Track 4` xuất hiện sớm và tồn tại sau khi xe rời khung ở frame 51–53 và 149–151; tôi gắn đúng `outside` ở frame đầu/cuối và không để bbox treo.
3. `Track 8` không khớp hoàn toàn với gold vì rất ngắn và có đoạn bị mờ; tôi kiểm lại mục tiêu xác định xe bốn bánh và không đoán phần ngoài khung.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: phát hiện `ID switch` tiềm ẩn và câu hỏi về `track 5`/`track 6` khi xe cắt nhau hoặc bị che.
- Lượt 2: kiểm `entry/exit`, kết luận `track 4` và `track 5` cần `outside` rõ ràng ở đầu/cuối đoạn.
- Lượt 3: kiểm trường hợp `frame 84–88` và `110–113`, nơi bbox bị lệch do interpolation, phải thêm keyframe.

Kiểm chéo với: `self-review + metrics`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3`. Số lỗi bạn ấy tìm được trong bản của bạn: `0`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Sự khác nhau chủ yếu nằm ở việc khi nào cần đặt outside và khi nào cần thêm keyframe ở giữa track. Quy tắc trong mini guideline đã được bổ sung rõ hơn để tránh drift và bbox treo ở các frame đầu/cuối.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `31c2f4a32e444d82e3db2a173e7e873fe187ce73f2dbec8a6e21194a991c65a1` |
| Thời điểm khóa | `2026-09-15T15:46:55.897329+00:00` |
| Số row / frame / track trước khi mở reference | `555 row / 190 frame / 7 track` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.779 | 0.738 | 0.828 | 0.859 | 0.931 | 0.864 | 0.845 | 30 | 48 | 0 |
| Sau rework | 0.779 | 0.738 | 0.828 | 0.859 | 0.931 | 0.864 | 0.845 | 30 | 48 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| bbox treo trước khi xe xuất hiện | 51–53 | 4 | Bấm `outside` ở đúng frame đầu và không để bbox xuất hiện trước thời điểm xe rõ ràng |
| bbox treo sau khi xe rời khung | 149–151 | 4 | Đặt `outside` ở frame cuối và dừng bbox ngay khi xe rời hình |
| bbox drift / interpolation lệch | 84–88 | 5 | Thêm keyframe quanh midpoint và sửa bbox khít với phần nhìn thấy |
| bbox drift / interpolation lệch | 110–113 | 6 | Thêm keyframe quanh frame 110–113 để giữ khớp với xe đang đổi hướng |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.14.7 / 8.4.145 / 2.14.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt`, `bytetrack.yaml`, `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `conf=0.25, iou=0.7, imgsz=960, classes=[2,5,7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.779 | 0.738 | 0.828 | 0.859 | 0.931 | 0.864 | 0.845 | 30 | 48 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.711 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.716 | 0.6366 | 0.8127 | 0.8535 | 0.865 | 0.7135 | 0.8333 | 120 | 37 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của tôi cao hơn IDF1 không phải là vấn đề ở bản này; IDF1 = 0.931 và MOTA = 0.864 đều ở mức tốt. Nếu MOTA cao mà IDF1 thấp, thì khiếm khuyết chủ yếu sẽ nằm ở association: nhiều track được phát hiện đúng nhưng đổi nhầm ID hoặc chia tách. MOTA không phạt lỗi ID như IDF1/AssA vì nó tập trung vào FP/FN và cung cấp điểm tổng mô tả nó hơn là đúng duy trì identity qua video.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ByteTrack có IDF1 0.8746, MOTA 0.7487, IDSW 2. ReID có IDF1 0.9001, MOTA 0.7923, IDSW vẫn 2. ReID cải thiện mức phát hiện (DetA 0.711 vs 0.6487) và giảm FN (26 vs 54), nhưng không giải quyết hết lỗi ID ở các frame 87 và 113. Tại frame 87, track 5 đổi từ 17 sang 18; tại frame 113, track 6 đổi từ 24 sang 31. Điều này cho thấy ReID giúp trên các đoạn che ngắn nhưng không thể khẳng định rằng mọi cải tiến đều do appearance, vì hai tracker khác nhau về implementation.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`ReID giảm FN từ 54 xuống 26, nghĩa là detector bắt được nhiều xe hơn. FP lại gần như không đổi (ByteTrack 88, ReID 91), còn DetA tăng từ 0.6487 lên 0.711. Lỗi còn lại chủ yếu là association: nhiều track vẫn bị tách hoặc đổi ID ở các frame crossing/occlusion, mặc dù phát hiện cơ bản đã tốt hơn.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Đúng trong frame 84–88, track 5. ReID vẫn vấp ở chỗ này khi track 5 đổi từ 17 sang 18, do đoạn hai xe cắt nhau và bbox drift. Tôi đã sửa bằng keyframe quanh midpoint; ReID không giấu được lỗi này trong số liệu của nó.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Tại frame 104–115, track 6 của ReID đổi từ 24 sang 31; đây là dấu hiệu gợi ý rằng model có thể bị nhầm khi che ngắn hoặc ở vùng chuyển hướng. Tôi kiểm lại bằng mắt và thấy đây là vùng cần keyframe rõ hơn để giảm drift, nên xét việc thêm `outside` và keyframe là đúng.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ nêu rõ thêm hai quy tắc: (1) đặt `outside` ngay khi xe không còn nhìn thấy dù còn ở rìa khung; (2) nếu bbox trong 5 frame liên tiếp có IoU < 0.6, phải thêm keyframe ngay trước/sau điểm đó. Quy trình làm việc sẽ giảm nhịp đi qua từng track, tăng số lần check middle frame, và nén thời gian QC theo 3 lượt như đã làm.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
