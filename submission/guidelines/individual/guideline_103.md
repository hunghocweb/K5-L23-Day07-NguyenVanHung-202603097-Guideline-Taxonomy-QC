# Guideline gán nhãn Bounding Box — v0.1-draft — Thành viên 103

> **Bản khảo sát cá nhân độc lập (Pha 1) dựa trên việc rà soát toàn bộ 40 ảnh trong `data/images/`.**  
> Bản này dùng để chốt các quy tắc mơ hồ trong `guideline_v0_starter.md`, chuẩn bị cho phiên họp thống nhất v1.0 của nhóm ở Pha 2.

- **Phiên bản:** v0.1-draft
- **Ngày:** 2026-11-16
- **Người viết:** Thành viên 103
- **Phạm vi dữ liệu:** 40 ảnh đường phố (`data/images/img_001.jpg` – `img_040.jpg`)
- **Định dạng nhãn:** Bounding Box 2D (hình chữ nhật ôm sát phần nhìn thấy), chuẩn xuất COCO 1.0

---

## 1. Mục tiêu

1. **Mục đích sử dụng:**
   - Tập dữ liệu sau gán nhãn được sử dụng để huấn luyện và kiểm thử mô hình phát hiện vật thể 2D (Object Detection) phục vụ hệ thống hỗ trợ lái xe an toàn (ADAS) và camera giám sát giao thông đô thị.
   - Nhãn cần phản ánh chính xác vị trí, kích thước thực tế nhìn thấy và mức độ bị che khuất của người và phương tiện di chuyển trên đường phố.

2. **Hậu quả sai sót cần phòng tránh:**
   - **Bỏ sót vật thể (False Negative):** Đặc biệt nguy hiểm với người đi bộ (`person`) và xe máy (`motorcycle`) ở cự ly gần hoặc khi bị che khuất một phần (như trong `img_005.jpg`, `img_014.jpg`). Bỏ sót sẽ khiến hệ thống tự hành không phát hiện chướng ngại vật sống, dẫn đến nguy cơ va chạm nghiêm trọng.
   - **Gán nhầm vật vô tri thành vật sống (False Positive):** Nhận diện tượng điêu khắc (`img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), hình chân dung trên biển quảng cáo (`img_018.jpg`) hoặc hình phản chiếu qua gương chiếu hậu (`img_020.jpg`) thành người/xe thật sẽ gây ra hiện tượng phanh gấp ngoài ý muốn (phantom braking), tiềm ẩn nguy cơ bị phương tiện phía sau tông dồn toa.
   - **Sai lớp (Misclassification):** Nhầm lẫn giữa phương tiện vận tải nặng (`truck`) và phương tiện nhẹ (`car`, `motorcycle`) làm sai lệch dự đoán quán tính và khoảng cách an toàn cần thiết của xe tự lái.

---

## 2. Lớp và thuộc tính

### 2.1. Bảng định nghĩa các lớp

| Lớp (`class`) | Định nghĩa (một câu) | Ví dụ cụ thể (trong 40 ảnh) | Phản ví dụ / Dễ nhầm lẫn |
|---|---|---|---|
| `car` | Ô tô con từ 4 đến 7 chỗ (sedan, SUV, hatchback, crossover) dùng chở người cá nhân. | Ô tô trong mưa `img_015.jpg`, xe wagon nội thất `img_024.jpg`, các xe con nằm trên giàn chở của `img_002.jpg`. | Tượng xe bán tải bằng đá `img_027.jpg`, xe mô hình đồ chơi `img_038.jpg`, xe hoạt hình diễu hành `img_019.jpg`. |
| `truck` | Phương tiện cơ giới chuyên chở hàng hóa, có khung gầm tải và thùng hàng riêng biệt phía sau. | Xe chuyên chở ô tô 2 tầng `img_002.jpg`, xe tải hàng trong kho `img_007.jpg`, xe tải nhỏ mini-truck `img_036.jpg`. | Xe bán tải nhẹ cá nhân dùng chở đồ nhẹ (đề xuất xét kỹ), xe van thùng kín một khối. |
| `van` | Xe thùng kín một khối thân liền, kích thước trung bình dùng chở hàng nhẹ hoặc chở khách 9–16 chỗ. | Xe van chở khách trên phố Bangkok `img_016.jpg`. | Xe tải có thùng rời phía sau, xe sedan thông thường. |
| `motorcycle` | Phương tiện hai bánh gắn động cơ đốt trong hoặc mô-tơ điện (xe số, xe tay ga, mô tô phân khối lớn). | Dòng xe máy trong `img_005.jpg`, `img_008.jpg`, `img_021.jpg`, `img_026.jpg`, `img_030.jpg`. | Xe đạp không động cơ `img_006.jpg`, xích lô đạp chân `img_009.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`. |
| `person` | Con người còn sống thực tế ở mọi tư thế (đang đi bộ, đứng, ngồi trên vỉa hè, lái xe hoặc ngồi sau xe). | Người qua đường `img_001.jpg`, `img_014.jpg`, `img_025.jpg`, người bán hàng rong `img_003.jpg`, `img_010.jpg`, người điều khiển xe máy `img_005.jpg`, `img_026.jpg`. | Tượng điêu khắc `img_017.jpg`, nhân vật quái vật cosplay Predator `img_011.jpg`, người in trên billboard `img_018.jpg`, ma-nơ-canh `img_033.jpg`. |
| `ignore_region` | Vùng đánh dấu khu vực tập trung đông đúc hoặc đối tượng quá nhỏ/mờ mà không gán riêng từng vật. | Cụm người tản bộ mờ xa ở bãi biển `img_025.jpg`, dòng xe cộ ken đặc mờ xa ở cuối phố `img_040.jpg`, tủ kính chứa đồ chơi dày đặc `img_038.jpg`. | Các đối tượng đứng đơn lẻ có chiều cao nhìn thấy $\ge 20\text{ px}$. |

### 2.2. Xử lý các đối tượng không thuộc lớp nào

Trong 40 ảnh khảo sát xuất hiện một số loại phương tiện giao thông đường phố thực tế không có trong 5 lớp ban đầu của `v0`:
- **Xe buýt (`bus`):** Xuất hiện trong `img_009.jpg`, `img_032.jpg`, `img_040.jpg`. Tạm thời **KHÔNG gán** vào `truck` hay `car` để tránh làm nhiễu dữ liệu huấn luyện, tick `can_review` hoặc ghi chú đề xuất nhóm bổ sung lớp `bus` vào `taxonomy.json` ở Pha 2.
- **Xe ba bánh / Xe lam / Tuk-tuk (`tuk_tuk`):** Xuất hiện trong `img_004.jpg`, `img_016.jpg`, `img_035.jpg`. Không phải `car`, cũng không phải `motorcycle`. Đề xuất nhóm bổ sung lớp `tuk_tuk` ở Pha 2; ở lượt này không gán ép vào `motorcycle`.
- **Xích lô / Xe đạp không động cơ:** Xuất hiện trong `img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`. Chỉ gán box `person` cho người đạp/người ngồi trên xe nếu người đó nhìn rõ; thân xích lô không gán vào `motorcycle`.
- **Động vật:** Ngựa gắn trên xe bán tải `img_028.jpg`, chim hải âu trên tượng `img_017.jpg`: Tuyệt đối không gán vào bất kỳ lớp nào.

### 2.3. Thuộc tính (`attributes`) với ngưỡng định lượng đo được

Để đảm bảo hai annotator gán nhãn độc lập vẫn cho ra cùng giá trị thuộc tính, các mức định tính của `v0` được chuẩn hóa thành các ngưỡng phần trăm cụ thể:

1. **`occluded_level` (Mức độ che khuất):**
   - `0` (Không bị che): Thấy rõ $100\%$ đường bao của vật thể ($0\%$ diện tích bị che khuất).
   - `1` (Che khuất nhẹ): Bị che từ $1\%$ đến $30\%$ diện tích nhìn thấy (ví dụ: bánh xe máy bị mép vỉa hè hoặc một chân người khác che lấp nhẹ).
   - `2` (Che khuất vừa): Bị che từ $31\%$ đến $70\%$ diện tích (ví dụ: xe máy chỉ thấy nửa thân trước do xe khác đỗ chắn ngang, hoặc người chỉ thấy từ ngực trở lên).
   - `3` (Che khuất nặng): Bị che trên $70\%$ diện tích, chỉ nhìn thấy một phần nhỏ (ví dụ: chỉ thấy mũ bảo hiểm hoặc cụm đèn pha), nhưng bối cảnh cho phép người gán khẳng định chắc chắn sự tồn tại của vật thể.

2. **`truncated` (Cắt mép ảnh):**
   - `no`: Toàn bộ vật thể nằm hoàn toàn bên trong khung ảnh (không chạm mép cắt của bức ảnh).
   - `yes`: Vật thể bị cắt bởi ít nhất một mép biên của bức ảnh (mép trên, dưới, trái, phải).

3. **`can_review` (Đánh dấu cần xem lại):**
   - `false` (Mặc định): Đã gán dứt khoát theo quy tắc rõ ràng.
   - `true`: Áp dụng cho các tình huống biên mơ hồ, ca tranh chấp giữa hai lớp (ví dụ xe bán tải gắn nắp thùng kín cao trong `img_037.jpg`), hoặc ranh giới bbox khó xác định do ánh sáng yếu/mưa nhòe (`img_013.jpg`, `img_015.jpg`) để đưa ra thảo luận nhóm.

---

## 3. Quy tắc vẽ Bounding Box

1. **Nguyên tắc ranh giới ôm sát phần nhìn thấy (Visible Boundary):**
   - Bounding box 2D là hình chữ nhật đứng (tiếp tuyến) bao khít toàn bộ các điểm ảnh nhìn thấy thực tế (visible pixels) của vật thể.
   - Không mở rộng box để suy đoán phần vật thể bị che khuất hoàn toàn hoặc nằm ngoài mép ảnh.

2. **Quy tắc phân tách Người và Xe máy (`person` vs `motorcycle`):**
   - Khi có người điều khiển hoặc ngồi sau xe máy (`img_005.jpg`, `img_026.jpg`, `img_030.jpg`), luôn gán **2 box riêng biệt**:
     - **Box 1 (`person`):** Ôm khít cơ thể người từ đỉnh đầu (hoặc chóp mũ bảo hiểm) đến hết mép bàn chân đang đặt trên xe.
     - **Box 2 (`motorcycle`):** Ôm trọn vẹn thân xe từ mép ngoài của bánh trước/sau, tay lái, gương chiếu hậu và pô xe.
   - Thân người che một phần xe máy sẽ làm tăng thuộc tính `occluded_level` của chiếc xe máy (thường là mức `1` hoặc `2`).
   - Nếu một xe máy chở 2 người (`img_026.jpg`): Gán **3 box** gồm 2 box `person` riêng cho 2 người và 1 box `motorcycle` cho xe.

3. **Quy tắc Xe chở xe (`img_002.jpg`):**
   - Chiếc xe tải chở hàng lớn được gán 1 box `truck` bao toàn bộ đầu kéo và hệ thống sàn/giá đỡ hàng phía sau.
   - Từng chiếc ô tô con 4 chỗ đang được chở trên các tầng sàn của xe tải, nếu nhìn rõ hình thể 3D, được gán các box `car` độc lập tương ứng và đánh dấu `occluded_level = 1` hoặc `2` tùy theo mức độ bị khung sàn che lấp.

4. **Quy tắc Hàng hóa chở trên xe (`img_012.jpg`, `img_029.jpg`, `img_003.jpg`):**
   - Bbox phương tiện (`motorcycle`, `truck`) chỉ ôm kết cấu cơ khí cố định của xe.
   - Hàng hóa rời rạc xếp trên giá/thùng xe (như bao tải, thùng xốp trên bán tải `img_012.jpg`, hoặc các sọt hàng cồng kềnh gấp nhiều lần xe máy `img_029.jpg`) **KHÔNG tính vào kích thước box của phương tiện**. Nếu hàng hóa che khuất thân xe, gán `occluded_level` tương ứng cho xe.

5. **Quy tắc Vật vô tri: Tượng, Biển quảng cáo, Ma-nơ-canh, Đồ chơi, Phản chiếu:**
   - **TUYỆT ĐỐI KHÔNG GÁN BOX** cho các trường hợp sau:
     - Tượng điêu khắc bằng đá/đồng/kim loại: Tượng công tước Wellington cưỡi ngựa `img_017.jpg`, tượng bia mộ khắc hình xe bán tải `img_027.jpg`, nhân vật Predator hóa trang `img_011.jpg`.
     - Hình vẽ, áp phích, biển quảng cáo pano có in hình chân dung người (`img_018.jpg`).
     - Ma-nơ-canh mặc quần áo đứng trước cửa hàng (`img_033.jpg`).
     - Đồ chơi, mô hình xe hơi nhỏ trong tủ kính (`img_038.jpg`, `img_019.jpg`).
     - Hình ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`) hoặc qua vũng nước mưa trên mặt đường (`img_013.jpg`).

6. **Quy tắc Xe chủ (Ego Vehicle):**
   - Các chi tiết nội thất khoang lái xe chụp (vô lăng, taplo, gương bên trong, ghế ngồi trong `img_020.jpg`, `img_024.jpg`) KHÔNG gán nhãn `car`.

7. **Ngưỡng kích thước tối thiểu và Vùng bỏ qua (`ignore_region`):**
   - Vật thể có chiều cao nhìn thấy $< 20\text{ px}$: **Không gán box đơn lẻ**.
   - Nếu xuất hiện cụm đông đúc nhiều người hoặc phương tiện nhỏ mờ xếp chồng lên nhau ở hậu cảnh xa xăm (như `img_025.jpg`, `img_040.jpg`): Vẽ 1 box lớn gán lớp `ignore_region` bao trọn cụm đó để tránh tính lỗi bỏ sót khi đo IAA và QC.

---

## 4. Bảng ca khó (Corner Cases trong 40 ảnh)

Dưới đây là 18 ca điển hình trong bộ dữ liệu 40 ảnh được khảo sát và đề xuất quy tắc giải quyết của thành viên 103:

| Ảnh | Tình huống thực tế | Rule ID | Quyết định của 103 | Lý do |
|---|---|---|---|---|
| `img_001.jpg` | Người phụ nữ đứng cạnh xe bán tải lúc chập tối | `R-01` | Gán 1 box `person` và 1 box `truck` cho xe bán tải | Xe bán tải có khoang chở hàng hở phía sau, phân biệt rõ với xe con 4 chỗ. |
| `img_002.jpg` | Xe tải chở 8 ô tô con trên hai tầng giá đỡ | `R-02` | Gán 1 box `truck` cho xe chở xe; gán thêm từng box `car` riêng cho các ô tô trên sàn | Ô tô trên sàn là vật thể 3D hoàn chỉnh, camera ADAS cần nhận biết sự hiện diện của chúng. |
| `img_003.jpg` | Người bán mít ngồi xổm bên mẹt hàng rong | `R-03` | Gán 1 box `person` (ngồi); không gán mẹt mít và quang gánh | Gánh hàng rong là nông sản rời, không phải kết cấu phương tiện giao thông. |
| `img_004.jpg` | Hai xe tuk-tuk 3 bánh đỗ trên vỉa hè ban đêm | `R-04` | Không gán vào `car` hay `motorcycle`; tick `can_review`, đề xuất bổ sung lớp `tuk_tuk` | Ép tuk-tuk vào xe máy hoặc ô tô sẽ làm sai lệch phân phối dữ liệu huấn luyện. |
| `img_005.jpg` | Dòng người đi xe máy dày đặc dừng đèn đỏ | `R-05` | Tách riêng từng box `person` và box `motorcycle`; xe nào $< 20\text{ px}$ ở xa thì bỏ qua | Tuân thủ triệt để nguyên tắc phân tách người và phương tiện để đo IoU chính xác. |
| `img_006.jpg` | Đoàn xích lô đạp chân chở khách tại Huế | `R-06` | Chỉ gán box `person` cho tài xế và hành khách; không gán xích lô vào `motorcycle` | Xích lô chạy bằng sức người, không thuộc định nghĩa xe gắn máy cơ giới. |
| `img_007.jpg` | Xe tải giao hàng bốc dỡ ban đêm trong kho mờ | `R-07` | Gán box `truck`, đánh dấu `occluded_level = 2` | Điều kiện ánh sáng yếu, xe bị che một phần bởi cổng kho nhưng dáng xe tải rõ ràng. |
| `img_011.jpg` | Nhân vật quái vật Predator cosplay đứng cạnh dàn mô tô | `R-08` | Gán các box `motorcycle`; KHÔNG gán box `person` cho Predator | Trang phục giả tưởng/tượng mô hình biểu diễn, không phải người thật tham gia giao thông. |
| `img_012.jpg` | Xe bán tải chở hàng hóa/thùng xốp cồng kềnh | `R-09` | Gán box `truck` chỉ ôm sát khung xe; không bao các thùng xốp nhô cao | Box phương tiện chỉ tính kích thước thân xe vật lý cố định. |
| `img_013.jpg` | Đường phố ướt sũng phản chiếu vệt đèn ban đêm | `R-10` | Bbox chỉ ôm sát xe thật; KHÔNG ôm vệt bóng phản chiếu dưới mặt đường | Bóng phản chiếu dưới mặt nước không phải là thực thể vật lý. |
| `img_014.jpg` | Du khách đi bộ băng qua đường giữa dòng xe | `R-11` | Gán box `person` (đứng thẳng), `truncated = no` | Người đi bộ băng cắt mặt đường là đối tượng ưu tiên cao nhất của hệ thống an toàn. |
| `img_015.jpg` | Ô tô nhìn qua kính chắn gió nhòe nước mưa | `R-12` | Gán box `car`, đánh dấu `occluded_level = 1`, tick `can_review` | Giọt nước mưa làm méo viền xe; cần reviewer kiểm chứng lại ranh giới box. |
| `img_017.jpg` | Tượng công tước Wellington đội nón giao thông | `R-13` | KHÔNG gán `person` cho tượng; KHÔNG gán con ngựa và chim hải âu | Tránh lỗi phanh khẩn cấp oan uổng do hệ thống nhận nhầm tượng điêu khắc là người. |
| `img_018.jpg` | Chân dung người in trên biển quảng cáo billboard | `R-14` | KHÔNG gán box `person` cho ảnh in trên pano quảng cáo | Hình ảnh 2D in trên biển báo tĩnh không phải vật cản sống di chuyển trên đường. |
| `img_019.jpg` | Xe mô hình Dangermouse trong lễ diễu hành đồ chơi | `R-15` | KHÔNG gán `car` cho xe đồ chơi diễu hành | Mô hình đạo cụ giải trí không tham gia lưu thông giao thông tiêu chuẩn. |
| `img_020.jpg` | Gương chiếu hậu ô tô phản chiếu dòng xe phía sau | `R-16` | KHÔNG gán box `car` cho ảnh phản chiếu trong gương; không gán nội thất xe chủ | Ảnh phản chiếu là ảo ảnh quang học trong gương, xe thật nằm ở hướng khác. |
| `img_022.jpg` | Xe mô tô 3 bánh có thùng chở người bên hông (sidecar) | `R-17` | Gán 1 box `motorcycle` trùm cả xe và thùng sidecar; gán box `person` cho người ngồi | Thùng sidecar là phụ kiện gắn liền thân xe máy tạo thành một khối di chuyển. |
| `img_026.jpg` | Hai người đèo nhau trên xe máy trong hầm tối | `R-18` | Gán 2 box `person` riêng cho 2 người và 1 box `motorcycle` ôm thân xe | Dù ánh sáng hầm nhân tạo, vẫn tách rõ 2 cá thể người ngồi trước và ngồi sau. |
| `img_027.jpg` | Tượng đài bia mộ bằng đá granite khắc hình xe bán tải | `R-19` | KHÔNG gán box `truck` hay `car` cho bia mộ bằng đá | Tượng đá đứng yên trong nghĩa trang, tuyệt đối không gán thành phương tiện. |
| `img_028.jpg` | Xe bán tải chở mô hình ngựa lớn phía sau thùng xe | `R-20` | Gán box `truck` cho xe bán tải; KHÔNG gán mô hình tượng con ngựa | Tượng ngựa là mô hình quảng cáo đặt trên thùng, không thuộc taxonomy. |
| `img_029.jpg` | Xe máy chở chồng sọt hàng hóa cồng kềnh phình to | `R-21` | Gán box `motorcycle` cho xe, box `person` cho người; không ôm khối sọt hàng | Tách bạch cấu trúc xe máy và tải trọng hàng chở rời. |
| `img_032.jpg` | Xe buýt lớn đỗ tại trạm chờ giờ cao điểm | `R-22` | Gán các box `person` ở trạm chờ; tạm thời không gán xe buýt, đề xuất thêm lớp `bus` | Xe buýt có kích thước vượt trội, gán vào `car` sẽ sai lệch hoàn toàn kích thước. |
| `img_033.jpg` | Ma-nơ-canh mặc quần áo bên ngoài cửa hàng thời trang | `R-23` | KHÔNG gán box `person` cho ma-nơ-canh | Ma-nơ-canh là hình nộm bằng nhựa/thạch cao vô tri giác. |
| `img_037.jpg` | Xe bán tải gắn nắp thùng kín cao (Master 75 Truck Cap) | `R-24` | Gán `truck`, tick `can_review` để nhóm chốt phân loại `van` hay `truck` ở Pha 2 | Nắp thùng cao biến bán tải thành dạng thùng kín, là ca tranh chấp ranh giới điển hình. |
| `img_038.jpg` | Tủ kính bày đầy búp bê và các mô hình ô tô đồ chơi | `R-25` | Vẽ 1 box lớn `ignore_region` bao quanh tủ kính; không gán từng món đồ chơi | Đồ chơi trong tủ kính không phải phương tiện thật tham gia giao thông. |

---

## 5. QC và kiểm soát phiên bản

### 5.1. Ngưỡng kiểm soát chất lượng (QC Targets)
Áp dụng các ngưỡng đo lường chuẩn hóa từ `submission/taxonomy.json` và `docs/QC_CHEATSHEET.md`:
- **Kích thước tối thiểu:** Chiều cao bbox $\ge 20\text{ px}$.
- **Ngưỡng IoU đánh giá:**
  - Vật thể lớn (chiều cao $\ge 50\text{ px}$): $\text{IoU} \ge 0.85$.
  - Vật thể nhỏ (chiều cao $< 50\text{ px}$): $\text{IoU} \ge 0.70$.
- **Ngưỡng đo độ đồng thuận (IAA):**
  - $\text{IoU trung bình} \ge 0.85$.
  - Tỉ lệ khớp lớp ($\text{Class Agreement}) \ge 95\%$.
- **Ngưỡng chấp thuận nghiệm thu lô:** Tỉ lệ ảnh bị lỗi $\le 2\%$ trên tổng số ảnh được duyệt.

### 5.2. Nguyên tắc đánh dấu ca mơ hồ
- Nếu một đối tượng thuộc trường hợp tranh chấp quy tắc hoặc ranh giới bbox mờ:
  1. Vẽ bbox bám sát khả năng nhận biết tốt nhất.
  2. Bật cờ `can_review = true`.
  3. Ghi chép mã ảnh và lý do vào `submission/individual_notes/103.md` để mang ra đối chiếu chéo trong phiên họp calibration của nhóm ở Pha 2 và Pha 5.

### 5.3. Lịch sử thay đổi tài liệu

| Phiên bản | Ngày | Người thực hiện | Nội dung thay đổi chính | Lý do / Căn cứ |
|---|---|---|---|---|
| `v0` | 2026-11-16 | Giảng viên | Bản khởi đầu sơ sài (5 lớp, thuộc tính định tính chung chung). | Khởi tạo dự án lab. |
| `v0.1-draft` | 2026-11-16 | Thành viên 103 | - Chuẩn hóa định lượng thuộc tính theo % cụ thể.<br>- Bổ sung quy tắc phân tách người và xe máy rõ ràng.<br>- Bổ sung quy tắc xe chở xe, hàng cồng kềnh, xe chủ.<br>- Loại bỏ dứt khoát tượng, hình in, ma-nơ-canh, bóng phản chiếu.<br>- Lập bảng phân tích 25 ca khó trên 40 ảnh. | Khảo sát thực tế toàn bộ 40 ảnh đường phố ở Pha 1 để loại bỏ triệt để các góc mơ hồ. |
