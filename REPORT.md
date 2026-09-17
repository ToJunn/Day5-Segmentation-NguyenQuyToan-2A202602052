# Báo cáo Day 5 — Segmentation — Bản nộp

- Mã học viên: **2A202602052**.
- Ngày / CVAT local: 17/09/2026 / http://localhost:8080.
- Công cụ: Brush và Polygon trên CVAT; Save và Export để lưu bài nộp.

Bài thực hành của tôi gồm semantic, instance và panoptic segmentation, cùng sáu checkpoint về các tình huống gán nhãn. Tôi đã có bản export của cả chín task trong `submissions/`. Kết quả tự đánh giá cho thấy Easy đạt yêu cầu về điểm, còn Medium và Hard cần cải thiện việc nhận diện đủ vật, chọn đúng class và tách instance.

## 1. Bài đã nộp

Đã có đủ ZIP của chín task. Số ảnh dưới đây là số ảnh có annotation/mask trong export, không khẳng định mọi vật/vùng đã được gán nhãn đầy đủ và chính xác.

| Task | File trong submissions/ | Ảnh có annotation/mask | Điểm tối đa |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng** | **9 ZIP** | **14 / 14** | **100** |

### Kết quả tự đánh giá ba tier

Dùng bộ chấm của repo, môi trường `VinAI/venv`, các ZIP bài nộp và reference tại `tiers/<task>/groundtruth`. Kết quả chi tiết lưu tại [results.json](reports/local_tiers/results.json).

| Task | Metric | Điểm tự đánh giá |
| --- | --- | ---: |
| easy_semantic | mIoU = 0,8584; coverage = 97,26% | 20 / 20 |
| medium_instance | mean matched IoU × recall = 0,4342 | 2,4 / 32 |
| hard_panoptic | PQ = 0,3252 | 8,3 / 30 |
| **Tổng ba tier** | | **30,7 / 82** |

Sáu checkpoint chưa được chấm với reference. Tổng trên chưa phải điểm cuối /100, chưa cộng bonus và chưa có xác nhận PASS/top 3.

## 2. Một quyết định trước khi dùng gợi ý

Trong phần ghi chép ban đầu, tôi chọn một xe nhỏ ở phía trái dưới ảnh thuộc class `car` để giải thích quyết định gán nhãn. Tôi chọn biên theo phần thân xe nhìn thấy, dừng tại vật che và mép ảnh, không đoán phần khuất hoặc kéo mask sang nền. Nếu có xe khác sát bên, mỗi xe vẫn phải là một instance riêng.

Tôi chưa lưu chính xác tên ảnh và ID của object đầu tiên trong ghi chép. Bộ ảnh Medium đã nộp gồm `000000181542.jpg`, `000000373353.jpg` và `000000458325.jpg`. Do đó, mô tả trên giải thích quyết định chọn biên nhưng chưa đủ để xác định lại object đầu tiên.

Bản ghi ban đầu của tôi nêu không dùng gợi ý cho object đầu tiên; việc dùng SAM/gợi ý ở các object tiếp theo chưa được ghi lại. Đây là hạn chế của phần ghi chép quá trình thực hiện.

## 3. Một lỗi tôi tìm thấy và sửa

### Lỗi đã ghi nhận trong quá trình làm

- Task/ảnh: `cp2_slice`, `000000017627.jpg`.
- Vùng đã ghi nhận: hai xe sát nhau ở tâm ảnh, khe giữa hai thân xe.
- Loại lỗi: gộp hai vật thành một instance.
- Dấu hiệu tôi ghi nhận: mask nối qua khe giữa hai xe, hai xe được biểu diễn bằng một object.
- Cách sửa theo ghi chép: tách thành hai object riêng, mỗi mask bám phần xe nhìn thấy; Save và export lại. Chỉ xóa khe giữa hai vùng trong cùng một object chưa đủ để tách instance.
- Trạng thái: đã có `cp2_slice.zip` chứa 13 annotation. Chưa có kết quả chấm với reference checkpoint hoặc số liệu trước/sau để định lượng hiệu quả sửa.

### Các vấn đề còn lại qua tự đánh giá

**Easy:** mIoU 0,8584 đạt ngưỡng điểm tối đa. IoU từng lớp: road 0,9835; sidewalk 0,7033; building 0,8243; vegetation 0,8209; sky 0,9599. Sidewalk là lớp cần ưu tiên kiểm tra biên, dù task đã đạt 20/20.

**Medium:** bài nộp có 52 instance, reference có 71. Bộ chấm ghép đúng 39 cặp, có 13 dự đoán không khớp và 32 instance reference chưa được ghép ở ngưỡng IoU 0,5. Mean matched IoU đạt 0,7905 nhưng recall chỉ 54,93%, khiến metric tổng còn 0,4342. Cần rà lại vật bị bỏ sót, class, tách/gộp và biên trên cả ba ảnh. FN không đồng nghĩa toàn bộ là vật chưa vẽ: mask sai class hoặc chưa đạt ngưỡng cũng có thể không được ghép.

**Hard:** PQ đạt 0,3252. Các lớp person, motorcycle và bus có lần lượt 9, 4 và 3 FN, không có TP/FP trong kết quả chấm. Sidewalk và bicycle có dự đoán nhưng chưa ghép được; car có 23 FP. Cần rà các vật chưa được biểu diễn đúng lớp, mask xe không khớp và vùng road–sidewalk. FP không tự chứng minh một vật bị vẽ thừa; cần đối chiếu ảnh và mask để xác định nguyên nhân.

Các vấn đề phát hiện qua scorer ở trên là **việc cần kiểm tra/sửa tiếp**, chưa ghi nhận một vòng Save/export và chấm lại sau sửa. Không có số liệu trước/sau để khẳng định mức cải thiện.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Ba ca dưới đây được đối chiếu trực tiếp với ảnh đầu vào khi hoàn thiện báo cáo. Các quyết định nêu cách xử lý theo quy tắc; không đồng nghĩa mask trong ZIP hiện tại đã được sửa theo các nhận xét này.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic`, `7ee6d192-89e2408b.jpg`, xe van trắng ở nửa phải ảnh và mặt đường xung quanh | Tô road liên tục qua vùng xe hay dừng tại biên xe? | Xe che mặt đường; Easy chỉ có năm class vùng, không có class car. Chỉ gán nhãn phần nhìn thấy thuộc class yêu cầu. | Mask road phải dừng tại biên xe; không tô xe thành road để tăng coverage và không tự thêm class ngoài taxonomy. |
| `hard_panoptic`, `000000350023.jpg`, xe buýt bị cắt ở góc dưới bên phải | Bỏ vật bị cắt mép hay giữ một instance bus? | Một phần thân xe vẫn nhìn thấy; bị cắt bởi khung ảnh không làm vật biến mất. Hard có class bus. | Giữ một instance bus cho phần nhìn thấy trong ảnh, dừng mask tại mép ảnh, không đoán phần nằm ngoài khung. |
| `hard_panoptic`, `000000350023.jpg`, vùng cây trước công trình ở bên phải ảnh | Gán cả vùng cho building hay tách vegetation khỏi building? | Tán cây che một phần công trình; panoptic phải biểu diễn bề mặt nhìn thấy và tránh chồng lấn class. | Phần tán cây nhìn thấy thuộc vegetation, phần công trình lộ ra thuộc building; không kéo building xuyên qua vùng cây che. |

Qua kết quả tự đánh giá, tôi nhận thấy mask bám biên tốt ở một số vật chưa đủ để đạt điểm cao nếu còn nhiều vật không được ghép đúng. Hướng cải thiện tiếp theo là rà Medium theo từng vùng ảnh và từng class, sau đó kiểm các class còn thiếu ở Hard, rồi mới tinh chỉnh biên. Sau mỗi lần sửa cần Save, export lại và chấm lại cùng bộ reference để so sánh kết quả.
