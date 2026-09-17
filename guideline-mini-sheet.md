# Phiếu quy tắc gán nhãn Day 5

Học viên: **2A202602052**. Dùng phiếu này khi rà bài trong CVAT trước khi Save và export lại.

## Mục tiêu nhanh trước khi vẽ

- Chọn đúng task và đúng loại segmentation: semantic, instance hay panoptic.
- Dùng đúng class theo `classes.json` của task đó, không copy danh sách từ task khác.
- Vẽ đúng phần nhìn thấy; không đoán phần ẩn hoặc phần bị che.
- Kiểm lại danh sách object, ranh giới, khoảng trống và tách/gộp trước khi Save và export.

## Chọn đúng loại trước khi vẽ

| Loại | Câu hỏi | Trong bài |
| --- | --- | --- |
| Semantic | Pixel này thuộc **loại vùng** nào? | Easy, `cp3_thin`, `cp4_curb`, `cp6_coverage` |
| Instance | Pixel này thuộc **vật nào**? | Medium, `cp1_holes`, `cp2_slice`, `cp5_occlusion` |
| Panoptic | Vùng thuộc loại nào **và** vật đếm được nào? | Hard |

Tên lớp phải giống từng chữ trong `classes.json` của task. `traffic sign` khác `traffic_sign`. Không dùng chung một danh sách lớp cho mọi task.

## Quy tắc hình học chung

- Vẽ sát **phần nhìn thấy**, không tự đoán phần bị che.
- Hai vật cùng lớp sát nhau vẫn là **hai instance**.
- Một vật bị cột hay vật khác che có thể có các vùng nhìn thấy rời nhau nhưng vẫn là **một instance**.
- Kính/chi tiết trên xe không tự động là lỗ phải khoét khỏi mask; theo quy tắc task.
- Ranh `road`–`sidewalk` xác định theo chức năng và bó vỉa, không chỉ theo màu.
- Phóng to kiểm nét mảnh và khe hở; Save rồi xem lại danh sách Objects.

## Class và format cho ba tier

| Task | Class cần dùng | Export |
| --- | --- | --- |
| `easy_semantic` | road, sidewalk, building, vegetation, sky | Segmentation mask 1.1 |
| `medium_instance` | person, bicycle, car, motorcycle, bus, truck; mỗi vật một instance | COCO 1.0 |
| `hard_panoptic` | Stuff: road, sidewalk, building, vegetation, sky. Things: person, car, bus, truck, motorcycle, bicycle, traffic light | COCO 1.0 |

Với Hard, phủ đúng vùng stuff và tách từng thing. Kiểm tra mask chồng lấn hoặc khoảng trống giữa các vùng; không dùng một mask car cho tất cả xe. Các checkpoint dùng class riêng trong `data/checkpoints/<task>/classes.json`.

## Ưu tiên rà lại theo kết quả bài nộp

| Bài | Bằng chứng từ scorer | Việc cần làm trong CVAT |
| --- | --- | --- |
| Easy — 20/20 | Sidewalk IoU 0,7033, thấp nhất trong năm lớp | Kiểm ranh bó vỉa và phần sidewalk còn thiếu/tràn; đạt điểm tối đa không có nghĩa mask hoàn hảo. |
| Medium — 2,4/32 | 39 TP, 13 FP, 32 FN; recall 54,93% | Quét cả ba ảnh theo từng vùng, rà đủ sáu class, vật nhỏ/bị che, biên và instance bị gộp/tách sai. |
| Hard — 8,3/30 | Person, motorcycle, bus chưa có dự đoán được ghi nhận ở các lớp này; sidewalk/bicycle chưa có cặp ghép đúng; car có 23 FP | Rà vật thuộc các lớp thiếu, kiểm class và mask xe, sửa ranh road–sidewalk, kiểm chồng lấn stuff–thing. |

TP là cặp ghép đúng theo bộ chấm; FP là dự đoán không được ghép; FN là đối tượng reference không được ghép. FP/FN có thể do sai class hoặc biên chưa đạt ngưỡng, không chỉ do thừa/thiếu vật. Cần xem ảnh và mask trước khi quyết định sửa.

## Nhắc nhanh sáu checkpoint

| Checkpoint | Quy tắc cần kiểm |
| --- | --- |
| `cp1_holes` | Kính/khe thuộc mask vật theo quy tắc task; không khoét lỗ tùy tiện. |
| `cp2_slice` | Hai vật sát nhau phải có hai instance, không chỉ xóa khe trong một object chung. |
| `cp5_occlusion` | Một vật bị che vẫn giữ một instance; chỉ tô các phần nhìn thấy. |
| `cp3_thin` | Phóng to cột/biển, dùng nét nhỏ phù hợp, tránh làm dày hoặc bỏ sót. |
| `cp4_curb` | Ranh road–sidewalk theo chức năng và bó vỉa. |
| `cp6_coverage` | Phủ vùng nhìn thấy thuộc các class yêu cầu; ghi lại vùng chưa chắc để hỏi. |

## Tự kiểm trước export

Rà từng ảnh từ trái sang phải, từ trên xuống dưới. Với Medium và Hard, kiểm lần lượt từng class để tránh chỉ tập trung vào các xe lớn, rõ nét. Tại vật bị che, kiểm riêng hai việc: mask chỉ phủ phần nhìn thấy và các phần của cùng một vật vẫn giữ đúng instance.

1. Đúng ảnh và đúng loại semantic/instance/panoptic chưa?
2. Mọi tên lớp có khớp `classes.json` không?
3. Có vật thiếu, vật thừa, gộp hai vật hoặc tách sai một vật không?
4. Mask có tràn sang nền/bóng hoặc bỏ sót vùng rõ ràng không?
5. Đã Save và export đúng format của task chưa?

Nếu dùng SAM hoặc gợi ý tự động, kiểm lại class, biên, vật bị gộp và vùng nền bị tô nhầm trước khi giữ mask. Khi sửa, ghi tên ảnh, vị trí, lỗi và thao tác thực tế để có thể trình bày lại trong report.

Khi không chắc, ghi ảnh/vị trí, dấu hiệu nhìn thấy, quy tắc đã dùng và điều cần hỏi trong `REPORT.md`. Không ép đoán cho đủ coverage.

Sau sửa, Save và export lại vào `submissions/<task>.zip`, rồi chấm lại để có metric trước/sau. Ghi đúng ảnh, vùng và thao tác thực tế trong [REPORT.md](REPORT.md). Tổng tự đánh giá hiện tại là **30,7/82**, chưa gồm checkpoint hoặc bonus; chưa có kết quả chấm lại sau sửa.

