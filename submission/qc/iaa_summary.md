# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 267 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 267 | 0.791 | 94.8% | 46.1% | 87.6% | 152/266 | 0 | CHƯA ĐẠT |
| 101|102 | 6 | 7 | 0.649 | 100.0% | 85.7% | 85.7% | 28/16 | 0 | CHƯA ĐẠT |
| 101|103 | 9 | 89 | 0.786 | 94.4% | 23.6% | 87.6% | 53/17 | 0 | CHƯA ĐẠT |
| 101|104 | 5 | 74 | 0.835 | 97.3% | 28.4% | 89.2% | 11/26 | 0 | CHƯA ĐẠT |
| 102|103 | 5 | 10 | 0.737 | 100.0% | 60.0% | 60.0% | 14/36 | 0 | CHƯA ĐẠT |
| 102|104 | 9 | 21 | 0.664 | 95.2% | 66.7% | 95.2% | 16/129 | 0 | CHƯA ĐẠT |
| 103|104 | 6 | 66 | 0.811 | 90.9% | 83.3% | 87.9% | 30/42 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 976 ({'bo_sot_thua': 335, 'sai_thuoc_tinh': 159, 'co_can_review': 227, 'iou_thap': 140, 'sai_lop': 14, 'bat_dong_ignore': 101}), trên 39 ảnh cần calibration.
