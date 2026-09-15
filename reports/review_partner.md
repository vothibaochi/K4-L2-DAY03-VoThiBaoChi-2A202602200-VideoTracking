# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | `Học viên cá nhân` |
| Reviewer | `Self review` |
| Pair ID | `single` |
| CVAT version | `2.74.1` |
| Thời điểm review | `2026-09-15` |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 84 | 85 | 5 | bbox drift | `track 5` mất IoU ở 0.54–0.57 quanh frame 84–88, là vùng biến hướng chênh và cần thêm keyframe | Thêm keyframe trước/sau 85–88; giữ bbox khít với phần nhìn thấy | fixed |
| 2 | 73–78 | 73–78 | 4 | bbox treo trước khi xe xuất hiện | `track 4` xuất hiện sớm hơn frame xe thật bắt đầu; theo rule `outside`, bbox không nên tồn tại trước khi xe ra rõ | Bấm `outside` ở frame đầu không hợp lệ và chỉnh lại khởi đầu track | fixed |
| 3 | 149–151 | 149–151 | 4 | bbox treo sau khi xe rời khung | `track 4` còn bbox khi xe đã rời khỏi khung; cần dừng ở frame đúng | Đặt `outside` ở frame cuối của xe, không kéo bbox tiếp | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 7 track, label `vehicle` |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | ID được giữ xuyên clip, không có reuse sai |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Không có đổi ID rõ ràng ở đoạn che và cắt nhau |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | frame 51–53, 149–151, ID 4 |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | các bbox được chỉnh sát với phần nhìn thấy |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | frame 84–88, ID 5; frame 110–113, ID 6 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `annotations/clip_01/gt.txt` export đúng MOT 1.1 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | 3 finding đã có closure |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Không có ID switch chính thức |
| 2 — endpoint/scope | ĐÃ SỬA | frame 51–53 và 149–151, ID 4 |
| 3 — geometry/interpolation | ĐÃ SỬA | frame 84–88, ID 5; frame 110–113, ID 6 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `BBox drift ở khung giữa hai keyframe và bbox treo ở đầu/cuôi track. Dùng rule `outside` và thêm keyframe quanh midpoint.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Không có`.
3. Một rule cần Lab Coach làm rõ (nếu có): `Không cần, quy tắc đã rõ trong mini guideline.`
