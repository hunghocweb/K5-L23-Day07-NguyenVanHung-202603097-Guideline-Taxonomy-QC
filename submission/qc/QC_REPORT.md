# Báo cáo QC — nhóm 101–104

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC bounding box 2D trên 40 ảnh tuyển chọn; chưa đánh giá độ đại diện cho vận hành. Kết quả QC áp dụng cho phiên bản nhãn vòng 2, không chứng minh dữ liệu đủ đại diện hoặc an toàn cho triển khai thực tế.
- Nguồn và phép sử dụng dữ liệu: Giữ nguyên tác giả, nguồn và giấy phép trong data/ATTRIBUTION.csv; manifest ghi thông tin nguồn và checksum từng ảnh.
- Manifest và checksum nhãn cuối: submission/qc/batch_manifest.json; labels_calibrated.json SHA-256 cf7739fa219f0dd8b65262bd3d454589e5d36ca47b4cbb839890942441e412f5. Đã đối chiếu checksum nhãn, CSV, ảnh và đầu vào manifest.
- Guideline và taxonomy khi gán / khi bàn giao: Nhãn độc lập theo guideline v1.0 và taxonomy v1.0; bàn giao calibration theo guideline_v1.1_calibrated.md. Taxonomy và ngưỡng không đổi. guideline_change.json ghi semantic, rà toàn lô theo R-14/R-15.

## 2. Phân công và IAA trước calibration

- Lượt hoàn thành / được giao: 80/80, mỗi mã 101–104 có 20 ảnh; mỗi ảnh hai người gán độc lập.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 267 cặp ghép 1–1 với IoU ghép >= 0,5; IoU TB 0,790773 < 0,85; cùng lớp 253/267 = 94,8% < 95%, chưa đạt. Giữ số đo gốc trong iaa_summary.json/.md.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 152/266 phía A/B, tổng 418 lượt box chưa ghép; 0 bỏ qua do ignore; 101 ca bat_dong_ignore. Không đồng nhất với số vật bỏ sót thực tế.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: M001, img_002.jpg, hai bản cùng chọn car cho xe bán tải trên sàn; đối chiếu ảnh với R-14/R-15, đổi 8 box thành pickup. Chưa ghi nhận đối chiếu bộ tham chiếu riêng đã được coach duyệt.

## 3. Calibration và nhật ký Rework

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: calibration_decisions.csv có 980 dòng: 976 ca IAA, M001, Q01–Q03. Mã 101 phụ trách theo phân công và ủy quyền xử lý. Lớp/tọa độ trong phieu_sua và các snapshot cho phép lần lại bản đã sửa; kết quả duyệt vòng 2 do 103 thực hiện được ghi trong phiếu vòng 2.
- Tác động v1.1 lên nhãn và ảnh: 40/40 ảnh; img_020 giữ 0 đối tượng. COCO hiện tại labels_calibrated_d8f68a100ea97a90.json xuất qua CVAT task 32/job 32; bản cuối 589 annotation, gồm 586 đối tượng và 3 ignore_region, 0 can_review.
- Tham chiếu bằng chứng khi Rework: Vòng 1 giữ nguyên tại reviews/round-01/ với bản nhãn 590 annotation và kết quả REWORK. Q01 sửa truncated của truck #6 ở img_001; Q02 mở box person #66 ở img_005 theo phần áo/thân nhìn thấy; Q03 bỏ non_real_object #202 trên hươu trang trí ở img_013. ID trên đây thuộc labels_snapshot.json của vòng 1. Sau sửa đã mở vòng 2, không ghi đè hồ sơ vòng 1.

| Nhật ký | Ảnh / đối tượng vòng 1 | Rule | Việc sửa |
|---|---|---|---|
| Q01 | img_001 / #6 | R-07 | truncated=no thành yes |
| Q02 | img_005 / #66 | R-03/R-07 | bbox [318,116,257,524], truncated=yes |
| Q03 | img_013 / #202 | R-01/R-10 | Bỏ box trên hươu, động vật ngoài taxonomy |

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 103; ngày 2026-11-17. 103 đã sửa và kiểm tra bản nhãn hiện tại sau Q01–Q03, hoàn thành duyệt vòng 2.
- Đã duyệt / cả lô: Vòng 2: 40/40 ảnh, reviewer 103 kiểm tra bản nhãn sau sửa. Giữ kết quả và snapshot vòng 1 để truy vết.
- Ảnh lỗi / ảnh đã duyệt: Vòng 2: 0/40 = 0,0%. Vòng 1: 3/40 = 7,5%, vượt ngưỡng 2%.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: Vòng 2: 0/586 = 0,0%; 3 ignore không nằm trong mẫu số đối tượng. Vòng 1: 3/587 = 0,511%.
- Vật bỏ sót, ảnh và vùng: Không ghi nhận vật bỏ sót trong lượt duyệt vòng 2. Kết quả kiểm tra giới hạn trong bộ ảnh và phiên bản nhãn đang bàn giao.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Vòng 2: ca_kho 0/39 ảnh lỗi, con_lai 0/1; không có lỗi lặp được ghi nhận. Vòng 1 có ba loại lỗi, mỗi loại một; Q01 cắt mép truck, Q02 box person, Q03 box trên hươu ngoài taxonomy, đã xử lý trước vòng 2.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script chặn bbox vượt biên (bản nháp img_040 đáy 640,01 trên ảnh cao 640), kiểm phạm vi, lớp/thuộc tính và checksum. Mắt đọc ảnh phát hiện box gán cho hươu ngoài phạm vi và box thiếu phần thân người. Bốn kiểm thử gộp/calibration đã qua; kiểm cấu trúc không thay kiểm nội dung.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Reviewer 103 hoàn thành vòng 2 full 40/40 ảnh: 0/40 ảnh lỗi, 0/586 đối tượng lỗi; không còn can_review hoặc lỗi lặp. Script kết luận RELEASE. Bàn giao cho người điều phối 101 và coach trong phạm vi bộ nhãn thực hành.
- Nếu còn việc: Thành viên xác nhận phần ghi chú cá nhân; nhờ người ngoài nhóm thử truy vết trước nộp. Leader nộp một repo riêng tư có quyền coach hoặc gói LMS theo SUBMISSION.md, không gồm work hay thông tin đăng nhập. Khi phát hiện lỗi mới, giữ cả hai vòng hiện tại, sửa theo nhật ký và mở vòng QC mới.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Q03 → calibration_decisions.csv → reviews/round-01/labels_snapshot.json #202 → img_013 → R-01/R-10 → box bị bỏ trong bản vòng 2. C649 → img_027 → R-10/R-25 → non_real_object trong bản cuối. Chưa ghi nhận người ngoài nhóm đã thử truy vết.
