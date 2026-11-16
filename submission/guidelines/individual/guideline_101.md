# Guideline gán nhãn — v1.0-draft (bản cá nhân, viết lại từ v0)

- Phiên bản: v1.0-draft
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0
- Ngày: 2026-11-16
- Người viết / duyệt: Thành viên 101 (chưa duyệt, chờ họp Pha 2)

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố châu Á (camera giao thông, hỗ trợ lái xe). Đường phố ở đây đông xe máy, nhiều loại xe ba bánh, xe chở hàng cồng kềnh, đám đông và nhiều vật trông giống xe hoặc người nhưng không phải vật thật.

Hai kiểu sai nguy hiểm nhất:

- **Bỏ sót** người hoặc xe máy ở sát đường (mô hình không thấy → không phanh).
- **Dạy mô hình coi vật không thật là vật thật** (tượng, ma-nơ-canh, hình in, phản chiếu, xe đồ chơi → mô hình phanh gấp hoặc đếm sai).

Ngưỡng chất lượng của nhóm: IoU trung bình ≥ 0.85 và đồng thuận lớp ≥ 95% giữa hai người gán độc lập (mục 5).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

Vật không thuộc lớp nào thì **không gán** (xem R01).

| Lớp               | Định nghĩa (một câu)                                                                                                                  | Ví dụ                                 | Phản ví dụ / dễ nhầm với                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `car`             | Ô tô con bốn bánh thân kín, mui thấp, cabin và khoang hành lý liền một khối (sedan, hatchback, SUV, taxi).                            | img_006, 015, 023, 031, 040           | Có thùng hở phía sau → `pickup`; thân hộp mui cao, cửa trượt → `van`; xe mô hình, xe trang trí → `non_real_object`.  |
| `pickup`          | Xe bán tải: cabin phía trước, thùng hở có thành phía sau (kể cả có bạt hoặc mui trùm thùng).                                          | img_001, 012, 037                     | Thùng cao liền khối hoặc khung thùng riêng, nhiều bánh → `truck`; SUV không có thùng hở → `car`.                     |
| `truck`           | Xe tải: cabin tách khỏi thùng hàng cao, dài, gầm cao, từ xe tải nhỏ đến xe chở ô tô nhiều tầng.                                       | img_002, 007, 036                     | Nhỏ hơn, thùng hở thấp → `pickup`; không có cabin tách rời → `van`; chở khách → `bus`.                               |
| `van`             | Xe thùng kín cỡ nhỏ một khối, mui cao hơn ô tô, kính hông phẳng, chở người hoặc hàng nhẹ.                                             | img_005 (xe xám phía sau), 016        | Mui thấp, có khoang hành lý riêng → `car`; dài, nhiều cửa sổ, nhiều cửa → `bus`.                                     |
| `bus`             | Xe buýt hoặc xe khách mini: thân dài, cao, một khối, nhiều cửa sổ dọc thân.                                                           | img_009, 031, 037, 040                | Ngắn, ít cửa sổ → `van`; thùng chở hàng → `truck`.                                                                   |
| `motorcycle`      | Xe hai bánh có động cơ, tay lái ngang, một yên (xe tay ga, xe số, mô tô), kể cả có thùng bên (sidecar).                               | img_006, 010, 021, 022, 026, 030      | Không động cơ, có bàn đạp → `bicycle`; ba bánh → `three_wheeler`. Người ngồi trên xe là `person` riêng.              |
| `bicycle`         | Xe đạp hai bánh mảnh, khung hở, bàn đạp, không động cơ.                                                                               | img_003, 006, 029                     | Khung dày, có động cơ → `motorcycle`; ba bánh → `cyclo`.                                                             |
| `three_wheeler`   | Xe ba bánh chạy động cơ chở người hoặc hàng (tuk-tuk, auto-rickshaw), khung lộ, có mái che hoặc buồng nhỏ.                            | img_004, 016, 035, 040                | Đạp chân, không động cơ → `cyclo`; hai bánh → `motorcycle`.                                                          |
| `cyclo`           | Xích lô đạp ba bánh, chạy bằng sức người, ghế khách ở phía trước hoặc sau.                                                            | img_023                               | Có động cơ → `three_wheeler`; hai bánh → `bicycle`.                                                                  |
| `person`          | Người thật (đi bộ, đứng, ngồi trên xe, trẻ em) nhìn thấy ít nhất đầu hoặc thân trên.                                                  | img_003, 014, 034, 021                | Ma-nơ-canh, tượng, người trong ảnh in hoặc áp phích → `non_real_object`; người dưới ngưỡng nhỏ → R07.                |
| `non_real_object` | Vật **trông giống** xe hoặc người nhưng không phải vật thật: tượng, ma-nơ-canh, xe mô hình/đồ chơi, xe bằng đá, hình in, tranh tường. | img_017, 033, 038, 027, 028, 018, 030 | Phản chiếu trong gương/kính/mặt đường ướt → không gán (R09). Vật không giống lớp nào (vịt cao su) → không gán (R01). |
| `ignore_region`   | Vùng bỏ qua: không gán từng vật; bỏ sót trong vùng này không tính là lỗi khi QC.                                                      | img_021, 030, 032, 019, 038           | Vùng có vật nhìn rõ và tách được thì phải gán từng vật, không dùng ignore để lười.                                   |

**Vật không thuộc lớp nào** (vịt cao su, nón giao thông, biển báo, cây, hàng hóa không phải xe) thì **không gán** và coi là nền (R01). Lớp `non_real_object` chỉ dành cho vật **giống** một lớp thật.

### Thuộc tính

Mọi lớp vật (trừ `ignore_region`) đều có ba thuộc tính dưới đây.

| Thuộc tính       | Giá trị      | Ý nghĩa                                                                                                                |
| ---------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------- |
| `occluded_level` | `0`          | Thấy trọn vẹn: phần thân bị vật khác che dưới 5%.                                                                      |
|                  | `1`          | Che ít: che từ 5% đến dưới 30%.                                                                                        |
|                  | `2`          | Che nhiều: che từ 30% đến 70%.                                                                                         |
|                  | `3`          | Che gần hết: che trên 70% nhưng vẫn nhận ra lớp. Không nhận ra lớp thì không gán (R14).                                |
| `truncated`      | `no`, `yes`  | `yes` khi bị cắt ở mép ảnh từ khoảng 10% thân vật trở lên; dưới mức đó là `no`. Bị vật khác che không tính là cắt mép. |
| `can_review`     | tick / không | Tick khi không chắc lớp, mức che hoặc cách vẽ, để đem ra calibration (R14).                                            |

## 3. Quy tắc vẽ box

Mỗi quy tắc có mã (R01–R14) để tham chiếu ở mục 4.

1. **R01 — Chỉ gán vật thuộc taxonomy.** Vật không thuộc lớp nào (vịt cao su, nón giao thông, biển báo, cây, hàng hóa lẻ) thì không gán, coi là nền.
2. **R02 — Một vật một box một lớp.** Không gán hai lớp cho một vật; nếu phân vân thì chọn lớp theo cột "phản ví dụ" ở mục 2 và tick `can_review`.
3. **R03 — Box ôm sát phần nhìn thấy**, hình chữ nhật nhỏ nhất bao hết phần nhìn thấy của vật. Không đoán phần bị che, không để viền thừa. Gương hậu, tay lái, bạt thùng xe nằm trong box; hàng chất trên xe nằm trong khung xe cũng tính; vật dài thò ra khỏi xe (cây, ống) không tính.
4. **R04 — Người ngồi trên xe:** một box `person` cho người và một box riêng cho xe; hai box được phép chồng lấn. Người trong thùng xe, trong xích lô, trong tuk-tuk cũng gán `person` nếu thấy rõ.
5. **R05 — Vật bị che:** vẽ box theo phần nhìn thấy, gán `occluded_level` theo ngưỡng mục 2.
6. **R06 — Vật bị cắt mép ảnh:** vẽ box tới đúng mép ảnh, gán `truncated=yes` nếu bị cắt từ khoảng 10% trở lên.
7. **R07 — Vật quá nhỏ:** cao dưới 20 px (đúng `min_box_height_px`) thì không gán từng vật. Nếu cả cụm nhỏ thì vẽ một `ignore_region`; một vật lẻ nhỏ thì bỏ qua. Vật cao 20–50 px vẫn gán, ngưỡng IoU nới còn 0.7.
8. **R08 — Đám đông và hàng xe đậu dày đặc:** gán từng vật ở hàng gần nhìn rõ, tách được; phần phía sau chồng chéo không tách được thì phủ một `ignore_region`. Không dùng ignore cho vật rõ ràng.
9. **R09 — Vật không phải thật:** tượng, ma-nơ-canh, xe mô hình/đồ chơi, xe bằng đá, hình in, áp phích, tranh tường → `non_real_object` (hoặc phủ `ignore_region` nếu cả cụm quá nhiều như tủ đồ chơi). **Phản chiếu** trong gương, kính, mặt đường ướt thì **không gán**.
10. **R10 — Xe chở xe:** xe chở ô tô nhiều tầng gán một `truck` cho xe tải; mỗi ô tô thật trên giá chở nhìn rõ thì gán `car` riêng (thường `occluded_level` cao).
11. **R11 — Xe cũ, xe trong bảo tàng, xe đỗ trong nhà** vẫn là xe thật, gán lớp theo hình dạng.
12. **R12 — Xe giả tưởng, xe trang trí diễu hành** có biển số, có người lái, chạy được trên đường: gán lớp gần hình dạng nhất và tick `can_review`; chưa chốt, cần họp.
13. **R13 — Ảnh nhoè, tối, ngược sáng, đen trắng:** vẫn gán nếu nhận ra lớp; không chắc thì tick `can_review`; không nhận ra thì không gán.
14. **R14 — Không chắc thì tick `can_review`, không đoán.** Ca nào tick nhiều thì mang ra calibration.

## 4. Ca khó

Các ca dưới đây là bản nháp. Cột "Quyết định" là đề xuất của mình, ca nào ghi _chưa chốt_ thì cần họp ở Pha 2.

| Ảnh               | Tình huống                                                                                                  | Rule ID  | Quyết định                                                                                      | Lý do                                                                           |
| ----------------- | ----------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| img_038           | Tủ kính đồ chơi: vịt cao su, nhân vật hoạt hình, hàng xe mô hình xếp hàng                                   | R01, R09 | Vịt không gán; cả tủ phủ một `ignore_region` (chưa chốt: hoặc `non_real_object` cho xe mô hình) | Vịt không giống lớp nào; vẽ từng xe mô hình tốn công và dễ lệch giữa người gán. |
| img_017           | Tượng cưỡi ngựa đội nón giao thông                                                                          | R01, R09 | Tượng người và ngựa → `non_real_object`; nón giao thông không gán                               | Giống `person` nhưng là tượng; nón nằm ngoài taxonomy.                          |
| img_033           | Ma-nơ-canh, quần áo trong tủ kính, phản chiếu                                                               | R09      | Ma-nơ-canh → `non_real_object`; hình phản chiếu không gán                                       | Giống `person` nhưng không phải người thật; phản chiếu không phải vật.          |
| img_028           | Tượng ngựa đặt trong thùng xe bán tải cổ                                                                    | R01, R09 | Xe → `pickup`; tượng ngựa → không gán (ngựa không giống lớp nào)                                | Xe là xe thật; ngựa nằm ngoài taxonomy.                                         |
| img_027           | Mộ bằng đá hình xe bán tải                                                                                  | R09      | `non_real_object`                                                                               | Giống `pickup` nhưng là bia mộ bằng đá, không phải xe.                          |
| img_019           | Xe hình hộp màu vàng kiểu diễu hành, có người lái và nhân vật chuột, có biển số; đám đông đứng sau rào chắn | R12, R08 | Xe → `car` + `can_review` (chưa chốt); đám đông phía sau → `ignore_region`                      | Chạy được trên đường nên khó coi là đồ chơi; hình dạng khác xe thường.          |
| img_024           | Xe nhỏ màu vàng đặt trên nóc xe thật; nhiều xe cổ trong bảo tàng                                            | R09, R11 | Xe nhỏ trên nóc → `non_real_object`; xe cổ → `car`                                              | Xe nhỏ trên nóc trông như mô hình trưng bày; xe cổ trong nhà vẫn là xe thật.    |
| img_002           | Xe tải chở nhiều ô tô nhiều tầng                                                                            | R10      | `truck` cho xe tải; từng ô tô nhìn rõ trên giá → `car`                                          | Mỗi ô tô là xe thật, có thể bị che một phần.                                    |
| img_020           | Gương chiếu hậu, thấy xe và cầu vồng phản chiếu                                                             | R09      | Không gán hình phản chiếu                                                                       | Phản chiếu không phải vật thật.                                                 |
| img_015, 031      | Xe và đèn phản chiếu trên mặt đường ướt                                                                     | R09      | Không gán bóng phản chiếu; xe thật vẫn gán                                                      | Bóng phản chiếu dễ bị gán thừa.                                                 |
| img_029           | Người chở cây dài trên xe máy, thò ra xa                                                                    | R03, R04 | Box `motorcycle` và `person` theo thân xe và người; không tính phần cây thò ra                  | Vật dài thò ra làm box quá rộng và không ôm sát.                                |
| img_035           | Tuk-tuk (auto-rickshaw) chở bao hàng cao ngất                                                               | R03      | `three_wheeler`; box bao cả hàng chất trên xe trong khung xe                                    | Hàng gắn liền xe; tách ra sẽ làm hai người vẽ lệch nhau.                        |
| img_004, 016, 040 | Tuk-tuk kiểu Thái nhiều màu, đôi khi bị xe khác che                                                         | R05      | `three_wheeler`, `occluded_level` theo ngưỡng                                                   | v0 không có lớp phù hợp, dễ bị gán `car` hoặc `motorcycle`.                     |
| img_023           | Xích lô đạp phía trước ô tô                                                                                 | R02      | `cyclo` (chưa chốt: có thể gộp vào `bicycle`); người đạp → `person`                             | Ba bánh, đạp chân; chỉ xuất hiện ít.                                            |
| img_022           | Xe máy có thùng bên, ảnh đen trắng, ở xa                                                                    | R07, R13 | `motorcycle`; nếu cao dưới 20 px thì bỏ qua                                                     | Vật nhỏ và ảnh đen trắng.                                                       |
| img_021           | Đám đông xe máy dày đặc lúc kẹt xe                                                                          | R08      | Gán từng xe và người ở hàng gần; phần phía sau dày đặc → `ignore_region`                        | Không thể gán hết, cần ranh giới thống nhất.                                    |
| img_030           | Bãi xe máy đậu san sát, bị che nhiều                                                                        | R08, R05 | Xe nhìn rõ → `motorcycle`; cụm chồng chéo → `ignore_region`                                     | Che nhiều, dễ bất đồng giữa hai người gán.                                      |
| img_032           | Hàng người đứng chờ ở trạm, xa                                                                              | R08, R07 | Người ở gần nhìn rõ → `person`; cả hàng nhỏ, xa → `ignore_region`                               | Vật nhỏ và đông.                                                                |
| img_013, 025, 034 | Cảnh rộng, người và xe rất nhỏ ở xa                                                                         | R07      | Dưới 20 px không gán; từ 20 px trở lên vẫn gán                                                  | Theo `min_box_height_px`.                                                       |
| img_014           | Người đi bộ rõ ở giữa, xe máy phía sau nhoè chuyển động                                                     | R13      | Người rõ → `person`; xe nhoè nhận ra lớp thì gán, không chắc thì `can_review`                   | Nhoè làm khó nhận ra lớp.                                                       |
| img_005           | Cận cảnh nhiều người đội mũ bảo hiểm, bị che, bị cắt mép; xe van phía sau                                   | R05, R06 | `person`/`motorcycle` theo phần nhìn thấy, `occluded_level` và `truncated` theo ngưỡng          | Cận cảnh, che chồng và cắt mép cùng lúc.                                        |
| img_018           | Biển quảng cáo có chân dung người; xe đỗ sau hàng rào                                                       | R09      | Chân dung trên biển → `non_real_object` (hoặc bỏ qua); xe đỗ → `car`                            | Người trên biển không phải người thật.                                          |
| img_001           | Xe bán tải đầy hàng, người đứng sát bên                                                                     | R04, R02 | `pickup` và `person` mỗi vật một box                                                            | Người đứng sát xe, box chồng lấn.                                               |

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.

Ngưỡng QC áp dụng (lấy từ `submission/taxonomy.json`, không đổi):

- IoU mỗi box: cao từ 50 px trở lên ≥ 0.85; cao 20–50 px ≥ 0.7; ghép box khi IoU ≥ 0.5.
- IAA (hai người gán độc lập): IoU trung bình ≥ 0.85 và đồng thuận lớp ≥ 95%.
- Tỉ lệ lỗi theo ảnh trong mẫu QC ≤ 2% mới được Release; luôn ghi mẫu số.
- Quyết định theo thứ tự Hold → Rework → Release.

Ai quyết ca mơ hồ: ca nào tick `can_review` hoặc bị cờ trong `calibration_candidates.csv` thì cả nhóm họp ở Pha 5, mỗi quyết định một dòng trong `calibration_decisions.csv`, quy tắc mới đưa vào guideline v1.1. Mã Rule ID ở mục 3 dùng để tham chiếu trong nhật ký QC; nếu nhóm đổi cách đặt mã thì cập nhật ở đây.

Lịch sử thay đổi:

| Phiên bản            | Thay đổi                                                                                                                               | Lý do (ca nào)                                                                                                                                           |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v0                   | Bản khởi đầu của giảng viên: 6 lớp, 3 quy tắc, chưa có ca khó                                                                          | —                                                                                                                                                        |
| v1.0-draft (cá nhân) | Thêm 6 lớp (`pickup`, `bus`, `bicycle`, `three_wheeler`, `cyclo`, `non_real_object`); mức che có ngưỡng %; 14 quy tắc có mã; 22 ca khó | img_004, 016, 035 (xe ba bánh), img_009, 031 (buýt), img_003, 029 (xe đạp), img_017, 033, 038, 027, 028 (vật không thật), img_021, 030 (đông, che nhiều) |
