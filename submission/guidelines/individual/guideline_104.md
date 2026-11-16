# Guideline gán nhãn — v0.1 (bản cá nhân Pha 1) — 104

- Phiên bản: v0.1-104 (sửa từ `guideline_v0_starter.md`)
- Ngày: 2026-11-16
- Người viết: 104 · Duyệt: chờ họp nhóm Pha 2

## 1. Mục tiêu

Nhãn dùng để huấn luyện **mô hình phát hiện vật 2D** (bounding box) cho camera giao thông và xe tự lái:
tìm **người thật** và **phương tiện thật** trên đường phố.

- Sai nguy hiểm nhất: **bỏ sót người thật** (người đi xe máy, người bị che một phần, người ở xa). Hệ thống không phanh.
- Sai thứ hai: **báo nhầm vật không thật** (tượng, ma-nơ-canh, biển quảng cáo, phản chiếu). Hệ thống phanh vô cớ.
- Ngoài phạm vi: hành vi, hướng di chuyển, 3D, hàng hóa, biển số.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người tối đa 9 chỗ (sedan, hatchback, SUV, MPV, taxi), kể cả xe trưng bày và xe giả tưởng cỡ thật chạy được. | sedan (023, 034), SUV (012, 040), taxi (040), xe trưng bày (024), xe Dangermouse (019) | bán tải (→ `truck`); van thùng kín (→ `van`); xe đồ chơi (038), tượng xe (027): không gán |
| `truck` | Xe chở hàng có **thùng hoặc sàn hàng tách khỏi buồng lái**, kể cả mọi bán tải (thùng mở, phủ bạt hay gắn nắp) và xe chở ô tô. | xe chở xe (002), xe tải nhỏ (001, 036), bán tải (012, 028), bán tải gắn nắp cao (007, 037), xe tải thùng (007) | van (khoang hàng liền buồng lái) |
| `van` | Xe cỡ nhỏ, **khoang hàng/khách kín liền với buồng lái**, không có thùng tách rời. | xe van (005, 016) | bán tải gắn nắp (→ `truck`) |
| `bus` (**lớp mới**) | Xe chở khách từ 10 chỗ trở lên, kể cả xe khách cỡ trung. | xe buýt (031, 037, 040), xe khách cỡ trung (009) | van chở khách (→ `van`) |
| `tuk_tuk` (**lớp mới**) | Xe ba bánh **có động cơ**, có mái hoặc khoang chở người/hàng. | tuk-tuk (004, 016, 035, 040) | xích lô (không động cơ, không gán); xe máy có sidecar (→ `motorcycle`) |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, xe tay ga, mô tô, xe máy điện), tính cả sidecar hoặc thùng kéo gắn liền. | 005, 010, 029 (xe máy điện), 022 (sidecar), 025 (thùng kéo), 011 (mô tô điêu khắc cỡ thật) | xe đạp (không gán) |
| `person` | **Người thật** đi bộ, đứng, ngồi, hoặc ngồi trên xe hai bánh / xe đạp / xích lô. | người đi bộ (014, 032), người đi xe máy (005, 026) | tượng (011, 017), ma-nơ-canh (033), người trên biển quảng cáo / áp phích / ảnh dán (018, 030, 040), người ngồi trong xe (R-08) |
| `ignore_region` | Vùng không gán từng vật: cụm người/xe cao dưới 20 px hoặc chồng dày tới mức không tách được box. | khán giả sau rào (019), xe máy kẹt ở xa (021), bãi xe (030) | **không** dùng cho tượng hay phản chiếu |

Phép thử lớp hay thuộc tính: *"mô hình có phải tự đoán ra điều này khi chạy không?"* Có thì là lớp
(`bus`, `tuk_tuk` khác `car` về kích thước và cách di chuyển). Không thì là thuộc tính.

**Vật không thuộc lớp nào:**

- Phương tiện không động cơ (xe đạp, xích lô, xe đẩy): **không gán xe**; người trên đó vẫn gán `person` (R-11).
- Vật không thật và phản chiếu: **không gán**, cũng không vẽ `ignore_region` (R-06).
- Động vật (chó ở 009, chim ở 017): không gán.
- Phương tiện thật nhưng không chắc lớp: chọn lớp gần nhất theo bảng trên và tick `can_review`.

### Thuộc tính

| Thuộc tính | Mức | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Không bị vật khác che (0%). |
| | `1` | Bị che 1–30% diện tích vật. |
| | `2` | Bị che 31–70%. |
| | `3` | Bị che trên 70% nhưng **vẫn nhận ra lớp** (ví dụ xe máy chỉ thấy gương và tay lái, 005, 040). |
| `truncated` | `no` | Toàn bộ vật nằm trong khung ảnh. |
| | `yes` | Một phần vật nằm ngoài khung ảnh, dù ít. |
| `can_review` | tick | Chưa chắc lớp, phạm vi box, hoặc có nên gán không. Ghi lý do vào vấn đề cấp ảnh. |

Cách đo `occluded_level`:

- Ước lượng hình dạng đầy đủ của **phần vật nằm trong khung ảnh**, rồi tính tỉ lệ diện tích bị vật khác che.
- Phần bị mép ảnh cắt **không** tính vào che khuất; phần đó đã ghi bằng `truncated`.
- Song rào thưa và kính trong suốt **không** tính là che (R-14).
- Phân vân giữa hai mức thì chọn **mức cao hơn**.
- Người đi xe máy: phần chân bị thân xe che **có** tính vào che khuất của `person`.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| R-01 | Box ôm sát **phần nhìn thấy** của vật, lệch không quá khoảng 2 px so với mép vật. Không đoán phần bị che. |
| R-02 | Người đi xe máy: **box riêng** cho từng người (`person`) và cho xe (`motorcycle`). Box `person` ôm cả người, kể cả phần chồng lên xe. Box xe ôm cả xe (bánh, gương, tay lái), kể cả phần người ngồi lên. Hai box được chồng nhau. Người ngồi trên xe đang đỗ cũng áp dụng quy tắc này (010). |
| R-03 | Vật bị che trên 70% nhưng **vẫn nhận ra lớp** thì vẫn vẽ box phần nhìn thấy, `occluded_level=3`. Ví dụ xe máy chỉ còn thấy gương, tay lái (005, 040). |
| R-04 | Hàng hóa nhô ra ngoài thân xe (bao tải, ống sắt, cuộn hàng, tượng chở trên thùng) **không** nằm trong box xe. Hàng nằm gọn trong thùng xe tải thì thuộc box `truck`. Đồ người mang (quang gánh, giỏ, túi) **không** nằm trong box `person` (003). |
| R-05 | Xe chở xe: xe chở là `truck`. Mỗi xe được chở (trên sàn xe tải hoặc trên nóc xe khác) có **box riêng** theo lớp của nó, kể cả khi bị khung che nhiều (002, 024). |
| R-06 | Vật không thật: tượng (kể cả tượng xe nguyên khối trên bệ), ma-nơ-canh, đồ chơi, mô hình thu nhỏ, linh vật, người hoặc xe trên biển quảng cáo / áp phích / ảnh dán / tranh tường; và **mọi hình phản chiếu** (gương, kính, đường ướt): **không gán**, không vẽ `ignore_region`. Ảnh chỉ có các vật này vẫn giữ trong Task, xuất COCO với 0 box (017, 020, 027, 033, 038). |
| R-07 | Phương tiện **cỡ thật, có bánh, đứng trên mặt đất** (xe độ, xe giả tưởng chạy được, mô tô điêu khắc cỡ thật) thì gán theo lớp của nó (011, 019). Khác R-06: tượng nguyên khối trên bệ (027) và mô hình thu nhỏ (038) không gán. |
| R-08 | Người **ngồi trong** ô tô, tuk-tuk, xe buýt (thấy qua kính hoặc qua cửa mở, kể cả xe mui trần): **không gán**; người đó là một phần của xe (004, 009, 019). |
| R-09 | Xe chủ (ego) và bộ phận xe của người chụp (gương, cửa, capo): **không gán** (020). |
| R-10 | Vật cao dưới 20 px (`min_box_height_px`): không gán riêng. Vật từ 20 px trở lên và tách được: gán từng vật. Phần còn lại của cụm dày (người chồng nhau, xe đỗ sát nhau, xe ở xa): **một** `ignore_region` bao cụm. |
| R-11 | Phương tiện không động cơ (xe đạp, xích lô, xe đẩy): không gán xe; người trên đó gán `person` (003, 006, 023, 029, 039). |
| R-12 | Phương tiện thật đang đỗ, không người lái, trong nhà kho, bãi xe hay bảo tàng: **vẫn gán** (007, 024, 030). |
| R-13 | Ảnh đêm, mưa, mờ do chuyển động: gán nếu nhận ra lớp, box ôm vùng nhìn thấy (kể cả vệt mờ); vật bị mờ thì **tick `can_review`** (014, 015). |
| R-14 | Vật sau rào song thưa hoặc sau kính trong suốt: **vẫn gán**; song rào và kính không tính vào che khuất (018, 032). |
| R-15 | Sidecar hoặc thùng kéo gắn liền xe máy là một phần của xe: box `motorcycle` bao cả phần đó (022, 025). Hàng chất trên thùng thì theo R-04. |
| R-16 | Chưa vẽ được box (không rõ là gì, không biết thuộc lớp nào): không đoán; ghi `image, annotator, region, note` để nhóm thu vào `image_issues.csv`. |

## 4. Ca khó

Loại ca: **QT** = quyết theo quy tắc ở trên; **RV** = cần hỏi reviewer / đưa ra họp Pha 2.

| Ảnh | Tình huống | Rule ID | Loại ca | Quyết định | Lý do |
|---|---|---|---|---|---|
| img_001 | Quán ven đường; nhiều người ngồi sau bàn; một phụ nữ đứng sau xe tải nhỏ Suzuki cũ chở ống thép (xe bị cắt mép phải); banner hot dog | R-04, R-10 | QT | `truck` cho Suzuki (`truncated=yes`), không bao ống thép nhô ra; `person` cho từng người từ 20 px; banner không gán | Suzuki có thùng hàng tách buồng lái; ống thép là hàng, không phải xe |
| img_002 | Xe tải chở 8 ô tô trên 2 tầng, sàn dưới bị khung che nhiều; 1 người đứng ở đuôi xe | R-05, R-03 | QT | 1 `truck` + 8 `car` (xe sàn dưới `occluded_level` 2–3) + 1 `person` | Xe được chở là xe thật, mô hình vẫn thấy; bỏ không gán là dạy mô hình bỏ sót |
| img_003 | Vỉa hè Hà Nội: người dắt xe đạp, người gánh hàng bằng quang gánh, người đi xe máy, hàng xe máy đỗ dày, xe và người nhỏ ở xa | R-02, R-04, R-10, R-11 | QT | `person` cho từng người, box người gánh hàng không bao quang gánh; xe đạp không gán; `motorcycle` cho xe tách được; cụm xe đỗ chồng nhau → `ignore_region` | Quang gánh là đồ mang theo, không phải một phần cơ thể; bao vào sẽ làm box `person` rộng gấp nhiều lần người |
| img_004 | Hai tuk-tuk đỗ trên vỉa hè ban đêm, tài xế ngồi trong; cọc giao thông | lớp `tuk_tuk`, R-08 | QT | 2 `tuk_tuk`; tài xế không gán | Tuk-tuk khác xe máy và ô tô về hình dạng, mô hình phải tự phân biệt → lớp riêng |
| img_005 | Nhóm người đi xe máy sát nhau, xe gần như bị người che hết; xe van phía sau; ô tô bạc cắt mép trái | R-02, R-03 | QT | `person` cho từng người; `motorcycle` cho từng xe còn thấy gương/tay lái (`occluded_level=3`); `van`; `car` (`truncated=yes`) | Mô hình phải đếm đủ người và xe trong dòng xe máy đông |
| img_006 | Huế nhìn từ phía sau: người đi xe máy, người đi xe đạp, taxi trắng cắt mép trái, xe ở xa | R-02, R-10, R-11 | QT | `person` + `motorcycle` từng cặp; người đi xe đạp chỉ `person`; `car` (`truncated=yes`); vật xa dưới 20 px bỏ qua | Xe đạp ngoài taxonomy, người trên đó vẫn là người thật |
| img_007 | Nhà kho ban đêm: bán tải gắn thùng hàng cao, xe tải Isuzu cắt mép phải, một người khuất giữa hai xe; nền ướt phản chiếu | lớp `truck`, R-03, R-06, R-12 | QT | 2 `truck`; người khuất → `person` `occluded_level=3`; phản chiếu không gán | Người bị che trên 70% nhưng vẫn nhận ra là người → vẽ theo R-03 |
| img_008 | Chợ đêm: hàng xe máy đỗ dày, tài xế xe ôm áo cam (một người ngồi trên xe, một người đứng cạnh xe), xe bán tải chở khách ở nền, đám đông nền, hình hoạt hình trên biển | R-02, R-10, R-12, R-06 | QT | `person` + `motorcycle`; xe đỗ tách được → `motorcycle`, phần chồng dày → `ignore_region`; bán tải chở khách → `truck`; hình hoạt hình không gán | Bán tải có thùng tách buồng lái → `truck` dù chở người |
| img_009 | Huế: xe khách cỡ trung, hành khách sau kính; người đi xe máy nhỏ; người trên ban công; con chó; biển quảng cáo | lớp `bus`, R-08, R-10 | QT | `bus`; hành khách không gán; `person` + `motorcycle` cho người đi xe máy từ 20 px; người ban công `person` nếu từ 20 px; chó không gán | Xe trên 10 chỗ → `bus`; người trong xe là một phần của xe |
| img_010 | Hà Nội: 2 người trên một xe tay ga; người bán hàng đứng cạnh xe đạp chở chuối; cô gái ngồi trên xe tay ga đỗ dùng điện thoại; xe tay ga đỗ | R-02, R-11 | QT | 4 `person`; `motorcycle` cho từng xe máy; xe đạp chở hàng không gán | Người ngồi trên xe đỗ vẫn tách 2 box như người đang chạy |
| img_011 | Tượng Predator cỡ lớn cắt mép phải; hàng mô tô điêu khắc kim loại cỡ thật; đám đông nhỏ ở nền; xe trắng ở nền bị tượng che | R-06, R-07, R-10 | QT | Tượng không gán; mô tô kim loại → `motorcycle`; người nền từ 20 px → `person`, cụm nhỏ → `ignore_region`; xe trắng → `van` (che theo %) | Mô tô cỡ thật có bánh, mô hình sẽ thấy như xe thật; tượng không phải người |
| img_012 | Bán tải chở hàng tạp hóa trong thùng (chiếm tiền cảnh, cắt mép dưới và phải); SUV Mazda; xe tải thùng phía trước | lớp `truck`, R-04 | QT | `truck` (`truncated=yes`), hàng trong thùng thuộc box; `car` cho Mazda; `truck` cho xe tải thùng | Hàng nằm gọn trong thùng nên là một phần hình dạng xe |
| img_013 | Đêm mưa: 2 người đi bộ, xe ở xa nhỏ, xe tay ga đỗ cạnh cọc, tượng rồng vàng, tượng đá, bóng phản chiếu trên đường ướt | R-06, R-10, R-13 | QT | 2 `person`; xe xa từ 20 px gán theo lớp, nhỏ hơn bỏ qua; `motorcycle` cho xe tay ga; tượng và phản chiếu không gán | Tượng và bóng phản chiếu không phải vật thật trên đường |
| img_014 | 2 người đi bộ rõ nét, 3 người đi xe máy bị nhòe do chuyển động | R-13, R-02 | RV | 2 `person`; người và xe bị nhòe: `person` + `motorcycle`, box ôm vệt mờ, tick `can_review` | Vệt mờ khó xác định mép box, hai người gán dễ lệch IoU |
| img_015 | Xe đỏ cắt mép trái, SUV bạc cắt mép phải, xe đỗ ở nền, một xe nhòe như bóng ma giữa ảnh; phản chiếu trên đường ướt | R-13, R-06 | RV | `car` cho từng xe; xe nhòe → `car` + `can_review`; phản chiếu không gán | Xe nhòe vẫn là xe thật đang chạy, nhưng mép box không rõ |
| img_016 | Bangkok: tuk-tuk xanh dương có khách, bán tải phía sau, người đi Vespa chở cuộn hàng, sedan tối, xe van trắng, tuk-tuk xanh lá có tài xế, phụ nữ áo đỏ tiền cảnh (cắt), đầu người mờ góc trái dưới | lớp `tuk_tuk`, R-08, R-04 | QT | 2 `tuk_tuk` (người bên trong không gán); `truck`; `person` + `motorcycle` (không bao cuộn hàng); `car`; `van`; `person` cho phụ nữ (`truncated=yes`); đầu người góc: `person` nếu nhận ra (`truncated=yes`) | Ảnh gom nhiều lớp dễ nhầm; mỗi vật theo đúng định nghĩa lớp |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông; chim | R-06 | QT | 0 box | Tượng không phải người thật |
| img_018 | Biển quảng cáo có chân dung; bãi xe sau rào song sắt (xe tối bên trái cột, sedan, xe đỏ cắt mép trái); chụp qua kính xe | R-06, R-14, R-09 | QT | Người trên biển không gán; xe sau rào gán theo lớp, song rào không tính che; xe tối bên trái cột → `car` | Không thấy thùng hàng tách khỏi buồng lái → không đủ căn cứ gán `truck` |
| img_019 | Xe Dangermouse có người lái và biển số; linh vật chuột ngồi trên xe; khán giả dày đặc sau rào; đầu người mờ ở tiền cảnh | R-07, R-08, R-06, R-10 | QT | `car`; người lái không gán; linh vật không gán; khán giả tách được từ 20 px → `person`, phần chồng dày → `ignore_region`; đầu người tiền cảnh → `person` (`truncated=yes`) | Xe chạy được trên đường thật nên là `car` dù hình dạng giả tưởng |
| img_020 | Gương chiếu hậu: một xe trong gương; thân xe của người chụp | R-06, R-09 | QT | 0 box | Xe trong gương là phản chiếu; gương và cửa là xe chủ |
| img_021 | Sài Gòn chập tối: hàng trăm xe máy kẹt; ô tô trắng cắt mép trái; ô tô bạc; xe buýt nhỏ ở xa | R-02, R-10, lớp `bus` | QT | `person` + `motorcycle` cho cặp gần tách được; vùng xa dày → `ignore_region`; `car`; `bus` nếu từ 20 px, không thì nằm trong vùng ignore | Không thể tách từng xe ở xa; gán vùng ignore để không tính là bỏ sót |
| img_022 | Ảnh đen trắng: mô tô có sidecar trên đường rừng, chỉ thấy đầu người lái sau kính chắn | R-15, R-03 | QT | 1 `motorcycle` bao cả sidecar; `person` `occluded_level=3` | Sidecar gắn liền là một phần của xe |
| img_023 | Huế: sedan Honda Civic, xích lô có người đạp (nhìn từ sau), người đi xe máy, hàng xe máy đỗ, người đứng và ngồi trên vỉa hè | R-11, R-02, R-10 | QT | `car`; xích lô không gán, người đạp → `person`; `person` + `motorcycle`; xe đỗ tách được → `motorcycle`, cụm → `ignore_region`; người vỉa hè → `person` (người cắt mép phải `truncated=yes`) | Xích lô ngoài taxonomy, người đạp vẫn là người thật |
| img_024 | Bảo tàng: xe Pacer đen chở thân vỏ xe vàng trên nóc; nhiều xe trưng bày xung quanh | R-05, R-12 | QT | `car` cho Pacer; `car` riêng cho thân vỏ vàng; `car` cho mỗi xe trưng bày từ 20 px (cắt mép → `truncated=yes`); cụm xe xa → `ignore_region` | Xe trưng bày là xe thật; thân vỏ trên nóc xử lý như xe được chở |
| img_025 | Phố ven biển: người đi bộ, người ngồi ghế bãi biển ở xa, taxi, xe máy kéo thùng bán hàng có người lái, ô tô, bán tải cắt mép phải | R-15, R-10 | QT | `person`; cụm người ngồi xa → `ignore_region`; `car` (taxi); `motorcycle` bao cả thùng kéo + `person`; `truck` (`truncated=yes`) | Thùng kéo gắn liền là một phần của xe máy |
| img_026 | Cặp đôi trên xe tay ga đỏ trong hầm, chụp từ xe bên cạnh; tranh tàu thủy trên tường | R-02, R-06 | QT | 2 `person` + 1 `motorcycle` (`truncated=yes`); tranh không gán | Hai người trên một xe: mỗi người một box, xe một box |
| img_027 | Tượng bán tải bằng đá trong nghĩa trang | R-06, R-07 | QT | 0 box | Tượng nguyên khối trên bệ, không phải xe |
| img_028 | Bán tải cổ chở tượng ngựa trên thùng; SUV đỏ phía sau bị che | lớp `truck`, R-04, R-06 | QT | `truck` (không bao phần tượng ngựa nhô lên); tượng ngựa không gán; `car` cho SUV đỏ (che theo %) | Tượng ngựa là vật chở, không thuộc lớp nào |
| img_029 | Người đi xe máy điện và người đi xe đạp cùng khiêng ống sắt dài; người đi bộ, người đi xe đạp ở nền | R-04, R-02, R-11 | QT | `person` + `motorcycle` (không bao ống sắt); người đi xe đạp → `person`, xe đạp không gán; người nền từ 20 px → `person` | Ống sắt là hàng; nếu bao vào, box xe sẽ dài gấp mấy lần xe |
| img_030 | Bãi xe máy dày đặc; nhân viên giữ xe; áp phích có người | R-10, R-12, R-06 | QT | `motorcycle` cho xe tiền cảnh tách được; phần dày phía sau → `ignore_region`; `person` cho nhân viên; áp phích không gán | Không thể tách từng xe trong bãi dày |
| img_031 | Đài Bắc đêm mưa: SUV trắng, sedan, hatchback tối, xe buýt, xe tay ga có người lái, người đi bộ, tranh hoạt hình trên tường, phản chiếu trên đường | lớp `bus`, R-10, R-13, R-06 | QT | `car` cho từng ô tô; `bus`; `person` + `motorcycle`; `person` cho người đi bộ; tranh và phản chiếu không gán; vật xa nhỏ bỏ qua hoặc `ignore_region` | Ảnh đêm vẫn nhận ra lớp nên gán bình thường |
| img_032 | Khoảng 30 người đứng chờ ở trạm; ô tô xanh cắt mép trái; xe đỗ phía sau đám người; SUV sau kính nhà chờ; cọc giao thông | R-10, R-14 | QT | `person` cho từng người tách được; phần chồng dày → `ignore_region`; `car` (`truncated=yes`); xe sau kính → `car` (kính không tính che); xe đỗ bị người che → `car` nếu nhận ra | Đám đông tách được thì gán từng người để mô hình học đếm người |
| img_033 | Tủ kính: ma-nơ-canh, ảnh người mẫu, xe van phản chiếu trên kính | R-06 | QT | 0 box | Ma-nơ-canh, ảnh và phản chiếu đều không phải vật thật |
| img_034 | Huế: người đi bộ qua đường, sedan, người đi xe máy gần và xa, người đi xe đạp, người đi xe máy cắt mép phải | R-02, R-10, R-11 | QT | `person`; `car`; `person` + `motorcycle`; người đi xe đạp → `person`; cặp cắt mép phải → `truncated=yes` | Ảnh giao thông điển hình, áp dụng đúng R-02 |
| img_035 | Hyderabad: tuk-tuk vàng chất bao tải cao; tuk-tuk vàng thứ hai sau người đi xe máy; người đi xe máy phía trước và ở nền; xe con trắng; người đi bộ nhỏ | lớp `tuk_tuk`, R-04, R-08 | QT | `tuk_tuk` (không bao bao tải); tài xế không gán; `tuk_tuk` thứ hai (che theo %); `person` + `motorcycle`; `car` | Bao tải là hàng; nếu bao vào, box sẽ cao gấp đôi xe |
| img_036 | Xe tải nhỏ CMC Veryca thùng phủ bạt; hàng xe tay ga đỗ hai bên; đầu người sau xe; người trên áp phích | lớp `truck`, R-10, R-06 | QT | `truck`; xe tay ga tách được → `motorcycle`, cụm → `ignore_region`; `person` `occluded_level=3` nếu nhận ra; áp phích không gán | Thùng phủ bạt vẫn là thùng tách buồng lái → `truck` |
| img_037 | Bán tải Silverado gắn nắp cao Master 75; 2 xe buýt GO (một chiếc bị che); người đi bộ | lớp `truck`, lớp `bus` | QT | `truck`; 2 `bus` (che theo %); `person` | Nắp chỉ che thùng, thùng vẫn tách khỏi buồng lái → `truck` |
| img_038 | Tủ kính: xe đồ chơi, búp bê, vịt đồ chơi | R-06 | QT | 0 box | Mô hình thu nhỏ không phải xe thật |
| img_039 | Huế: xe tay ga đỗ tiền cảnh, người đi xe tay ga, nhóm 3 phụ nữ đi bộ, người đi xe đạp, xích lô bên phải, người ngồi trên vỉa hè, xe ở xa | R-02, R-11, R-10 | QT | `motorcycle` cho xe đỗ; `person` + `motorcycle`; `person`; người đi xe đạp → `person`; xích lô không gán, người trên/cạnh → `person`; xe xa dưới 20 px bỏ qua | Mỗi vật theo đúng quy tắc người–xe và xe không động cơ |
| img_040 | Bangkok: taxi xanh cắt mép; người đội mũ bảo hiểm đen trên xe máy (chỉ thấy 2 gương); tuk-tuk sau taxi có tài xế; SUV tối; xe buýt cam ở nền; tuk-tuk và xe buýt đỏ ở xa; ảnh chân dung dán trên kính taxi | R-03, lớp `tuk_tuk`, R-08, lớp `bus`, R-06 | QT | `car` (taxi, `truncated=yes`); `person` + `motorcycle` (`occluded_level=3`, `truncated=yes`); `tuk_tuk` (tài xế không gán); `car` cho SUV; 2 `bus`; `tuk_tuk` ở xa; ảnh dán không gán | Xe máy chỉ thấy gương vẫn là xe thật cạnh người lái |

Ca chưa chốt khi gán: tick `can_review` trên box hoặc ghi vấn đề cấp ảnh; dẫn mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản

Ngưỡng QC (lấy từ mục `rules` trong `taxonomy.json`, không sửa):

| Ngưỡng | Giá trị |
|---|---|
| Chiều cao box tối thiểu | 20 px |
| IoU đạt với vật lớn (từ 50 px) / vật nhỏ | 0.85 / 0.7 |
| IoU để ghép hai box khi đo IAA | 0.5 |
| IAA: IoU trung bình / đồng thuận lớp | 0.85 / 0.95 |
| Tỉ lệ ảnh lỗi tối đa để Release | 2% |

- Ca mơ hồ: người gán tick `can_review`; nhóm chốt trong họp calibration (Pha 5); người chốt ghi mã của mình.
- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-104 | Thêm lớp `bus`, `tuk_tuk`; `truck` gồm mọi bán tải (v0 `.md` ghi "xe tải", `taxonomy.json` ghi "có thùng hàng riêng"); `occluded_level` mức 3 đổi từ "không rõ" sang "che trên 70%, vẫn nhận ra lớp"; thêm ngưỡng % cho từng mức; thêm R-01 đến R-16 | img_002, img_004, img_005, img_009, img_011, img_017, img_019, img_035, img_037 |
