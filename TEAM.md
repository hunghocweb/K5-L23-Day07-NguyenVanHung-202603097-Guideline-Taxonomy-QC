# Thông tin nhóm

- Tên nhóm: NguyenVanHung-202603097
- Mã thành viên (ba chữ số): 101, 102, 103, 104
- Người điều phối: 101
- Người review bản cuối: 103
- Người phụ trách commit chốt bài: 101
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80 trên 40 ảnh; 101, 102, 103, 104 mỗi mã 20 ảnh; cả bốn file COCO qua kiểm tra cấu trúc.
- IAA trước calibration (kèm mẫu số): 267 cặp box ghép 1–1 (IoU ghép >= 0,5); IoU trung bình 0,791 < 0,85; đồng thuận lớp 253/267 = 94,8% < 95%: CHƯA ĐẠT. Có 976 ca cần xem trên 39 ảnh, chưa phải các lỗi đã xác nhận; xem submission/qc/iaa_summary.md.
- Số ảnh phải calibration: 40/40 do guideline v1.1 làm rõ quy tắc có tác động toàn lô; CP4 gắn cờ 39 ảnh. Đã xuất/gộp 589 annotation (586 đối tượng, 3 ignore), 0 can_review; xem submission/qc/batch_manifest.json. Kết quả QC vòng 2 do 103 thực hiện được ghi trong báo cáo.
- Vòng QC và số ảnh đã duyệt / cả lô: Vòng 1: 40/40, Rework; đã sửa Q01–Q03. Vòng 2: reviewer 103 đã sửa và kiểm tra bản hiện tại, duyệt đủ 40/40 ảnh.
- Tỉ lệ lỗi: Vòng 2 có 0/40 ảnh lỗi phát hiện = 0,0%; 0/586 đối tượng lỗi = 0,0%; bỏ sót ghi nhận 0. Vòng 1: 3/40 = 7,5%, 3/587 = 0,511%.
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Release theo QC vòng 2 do 103 thực hiện, đủ 40/40 ảnh, 0/40 ảnh lỗi và 0/586 đối tượng lỗi; không còn lỗi lặp hoặc can_review.
