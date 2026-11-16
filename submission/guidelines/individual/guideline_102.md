# Guideline gán nhãn — v0.1_draft — Mã thành viên: 102

- **Phiên bản:** v0.1_draft (Bản cá nhân Pha 1)
- **Ngày:** 2026-11-16
- **Người thực hiện:** Thành viên 102
- **Mục đích:** Huấn luyện mô hình phát hiện 2D Bounding Box cho người và phương tiện giao thông đường phố.

---

## 1. Mục tiêu và Giới hạn sử dụng

Nhãn hộp bao (2D Bounding Box) được dùng để huấn luyện và đánh giá mô hình thị giác máy tính phát hiện người đi bộ, người điều khiển phương tiện và các loại xe cộ trên đường phố đô thị.
- **Mục tiêu ưu tiên:** Đảm bảo an toàn giao thông cao nhất (không bỏ sót người đi bộ, người đi xe máy gần đầu xe/làn đường).
- **Nguy cơ lỗi:** 
  - *False Negative (bỏ sót người/xe):* Cực kỳ nguy hiểm cho hệ thống tự hành hoặc cảnh báo va chạm.
  - *Nhầm lẫn lớp (ví dụ nhầm người thật với tượng/manocanh/hình in):* Gây phanh gấp hoặc xử lý sai tình huống giao thông.

---

## 2. Lớp và Thuộc tính

### 2.1. Định nghĩa các Lớp đối tượng (Classes)

| Lớp | Định nghĩa chuẩn | Ví dụ điển hình | Phản ví dụ / Dễ nhầm lẫn |
|---|---|---|---|
| `person` | Người thật bằng xương bằng thịt ở mọi tư thế (đứng, ngồi, đi bộ, lái xe, chở hàng). | Người đi bộ qua đường (`img_014`), người lái xe máy (`img_005`, `img_026`), người bán hàng rong (`img_003`). | Tượng đá Wellington (`img_017`), ma-nơ-canh shop (`img_033`), hình in trên pano quảng cáo (`img_018`), búp bê đồ chơi (`img_038`). |
| `car` | Ô tô con từ 4–9 chỗ, xe SUV, xe sedan, hatchback, xe bán tải (pickup) không có thùng hàng rời cỡ lớn. | Xe sedan, taxi, xe Pacer wagon (`img_024`), xe bán tải pickup chở đồ gia dụng (`img_001`, `img_012`, `img_037`). | Xe tải có thùng hàng chở hàng chuyên dụng (`truck`), xe thùng kín cỡ nhỏ (`van`), tượng xe bán tải (`img_027`). |
| `truck` | Xe tải chở hàng, xe tải thùng kín/hở, xe đầu kéo, xe chuyên dụng (chở xe, bồn, cẩu). | Xe tải giao hàng (`img_007`), xe tải chuyên dụng chở ô tô con (`img_002`), xe tải thùng CMC Veryca (`img_036`). | Xe ô tô con (`car`), xe bán tải gia đình thông thường. |
| `van` | Xe ô tô thùng kín cỡ nhỏ/vừa, dùng chở người hoặc chở bưu phẩm (thường dạng 1 khoang liền thân). | Xe van chuyển phát bưu phẩm, xe chở hàng nhẹ dạng hộp kín liền cabin. | Xe tải thùng rời (`truck`), xe con 7 chỗ (`car`). |
| `motorcycle` | Xe mô tô, xe gắn máy 2 bánh, bao gồm cả xe phân khối lớn và xe tay ga. | Xe máy lưu thông trên đường (`img_005`), xe máy dừng đỗ ở chợ (`img_008`, `img_030`), mô tô độ (`img_011`). | Xe đạp, xe xích lô 3 bánh đạp chân (`img_006`), xe ba gác máy. |
| `ignore_region` | Vùng bỏ qua: vùng chứa nhiều vật nhỏ li ti ở hậu cảnh xa, hoặc phương tiện hỗn hợp không thuộc taxonomy mà khó phân tách. Bỏ sót trong vùng này không tính là lỗi. | Đám đông người hoặc phương tiện ở rất xa hậu cảnh (`img_025`), khu vực quầy hàng đồ chơi phức tạp (`img_038`), đoàn xích lô/xe ba gác chen chúc khó tách box. | Các vật thể nhìn rõ, chiều cao $\ge 20\text{ px}$. |

*Vật không thuộc lớp nào (xe đạp, xích lô, xe kéo tay, xe tuk-tuk 3 bánh):* Không gán box phương tiện nếu không khớp định nghĩa, chỉ gán box `person` cho người điều khiển. Nếu che khuất phức tạp thì bao quanh bằng `ignore_region`.

---

### 2.2. Thuộc tính (Attributes)

| Thuộc tính | Kiểu | Các giá trị cho phép | Định nghĩa và Ngưỡng đo lường |
|---|---|---|---|
| `occluded_level` | Select | `0`, `1`, `2`, `3` | **Mức độ bị che khuất của phần thân thể / xe:**<br>- `0`: Thấy rõ ràng, không bị che hoặc bị che $< 10\%$ diện tích.<br>- `1`: Bị che khuất nhẹ ($10\% - 35\%$).<br>- `2`: Bị che khuất vừa/nặng ($35\% - 75\%$).<br>- `3`: Bị che khuất rất nặng ($> 75\%$), chỉ nhận biết được nhờ ngữ cảnh xung quanh. |
| `truncated` | Select | `no`, `yes` | **Bị cắt mép ảnh:**<br>- `no`: Toàn bộ vật thể nằm trọn vẹn trong khung hình.<br>- `yes`: Một phần vật thể bị đường biên/cạnh ảnh cắt cụt ra ngoài khung hình. |
| `can_review` | Checkbox | `true`, `false` | Đánh dấu `true` khi người gán phân vân, không chắc chắn về lớp hoặc ranh giới box, cần đưa ra nhóm/reviewer kiểm tra lại. |

---

## 3. Quy tắc vẽ Box chi tiết

- **R-01 (Người đi xe máy / người ngồi trên xe):**
  - Luôn gán **2 box riêng biệt**: 1 box lớp `motorcycle` bao trọn chiếc xe, và 1 box lớp `person` bao trọn người ngồi trên xe (từ đỉnh mũ bảo hiểm đến bàn chân/gác chân).
  - Box `person` đặt `occluded_level: 1` hoặc `2` (do nửa dưới cơ thể bị xe che khuất).
  - Nếu xe chở 2 người (`img_026`), gán 1 box `motorcycle` và 2 box `person` riêng rẽ cho từng người.
- **R-02 (Xe chở xe & Hàng hóa trên xe):**
  - Với xe chuyên dụng chở ô tô (`img_002`): Bản thân xe chở gán là `truck`. Các ô tô con đang được chở trên các tầng giá đỡ gán là `car` với `occluded_level: 1` hoặc `2` tương ứng nếu chúng còn nguyên hình dạng ô tô nhìn thấy rõ.
  - Với xe máy chở hàng cồng kềnh (`img_029`): Box `motorcycle` bao khung xe và bánh xe; hàng hóa chở kèm nếu quá khổ không làm phình to box xe máy mà gán sát thân xe thực tế.
- **R-03 (Loại trừ Tượng, Tranh ảnh, Ma-nơ-canh, Đồ chơi):**
  - **Không** gán `person` cho: Tượng đá/tượng đồng nhân vật (`img_017`), ma-nơ-canh trưng bày (`img_033`), hình người in trên biển quảng cáo/billboard (`img_018`), búp bê/mô hình đồ chơi (`img_038`).
  - **Không** gán `car`/`truck` cho tượng đài xe điêu khắc bằng đá (`img_027`).
- **R-04 (Loại trừ Hình ảnh ảo / Phản chiếu):**
  - Không gán box cho các hình ảnh phản chiếu trong gương chiếu hậu ô tô (`img_020`) hoặc phản chiếu trên mặt kính cửa hàng.
- **R-05 (Ngưỡng kích thước tối thiểu):**
  - Không vẽ box cho các vật thể có chiều cao dưới 20 pixels (`height < 20px`). Nếu có nhiều vật nhỏ li ti ở hậu cảnh, sử dụng `ignore_region` bao phủ khu vực đó.
- **R-06 (Nguyên tắc ôm sát phần nhìn thấy):**
  - Box dạng chữ nhật thẳng đứng (axis-aligned rectangle), viền ngoài bao khít các điểm cực trị (trên, dưới, trái, phải) của phần vật thể nhìn thấy được trên ảnh. Không đoán mò vẽ thêm vùng bị che khuất hoàn toàn.
- **R-07 (Trang phục hóa trang / Mascot):**
  - Người mặc trang phục hóa trang (như nhân vật Predator trong `img_011`): Vẫn gán là `person` vì có người thật vận động bên trong, ghi chú `can_review: true` nếu trang phục làm biến dạng cơ thể.

---

## 4. Bảng ca khó khảo sát qua 40 ảnh

| Mã ảnh | Tình huống đặc thù | Mã Rule áp dụng | Quyết định đề xuất | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chuyên dụng đang chở nhiều ô tô con trên sàn | `R-02` | Gán xe mẹ là `truck`; gán các xe con trên sàn là `car` (occluded 1 hoặc 2). | Các ô tô con trên xe vẫn có đặc trưng thị giác ô tô rõ ràng, mô hình cần phát hiện được vật cản lớn. |
| `img_005.jpg` | Đám đông người đi xe máy chen chúc tại TP.HCM | `R-01`, `R-08` | Mỗi xe vẽ 1 box `motorcycle`, mỗi người vẽ 1 box `person`. Đặt `occluded_level: 2` cho các đối tượng bị che nhiều. | Đảm bảo tính nhất quán giữa việc phát hiện người và phát hiện xe máy. |
| `img_006.jpg` | Đoàn xe xích lô chở khách du lịch | `R-09` | Gán người đạp xe và khách ngồi là `person`. Phần xe xích lô không gán (hoặc gom `ignore_region` nếu bị che rối). | Taxonomy v0 không có lớp `bicycle` hay `cyclo`. |
| `img_011.jpg` | Nhân vật Predator đứng cạnh dàn mô tô | `R-07`, `R-01` | Dàn xe máy gán `motorcycle`. Nhân vật hóa trang gán `person` kèm `can_review: true`. | Là người thật mang trang phục biểu diễn, cần reviewer xác nhận. |
| `img_017.jpg` | Tượng đá công tước Wellington trên lưng ngựa | `R-03` | **Không gán box `person`.** | Tượng đá tĩnh, không phải người thật tham gia giao thông. |
| `img_018.jpg` | Biển quảng cáo pano bầu cử có ảnh chân dung người | `R-03` | **Không gán box `person`.** | Ảnh in 2D trên biển bảng, không có vật lý chuyển động. |
| `img_020.jpg` | Cảnh phản chiếu qua gương chiếu hậu ô tô | `R-04` | **Không gán box** cho hình ảnh trong gương. | Ảnh ảo phản chiếu, vẽ box sẽ làm sai lệch định vị 3D của hệ thống xe tự lái. |
| `img_026.jpg` | Xe máy chở 2 người trong hầm | `R-01` | Vẽ 1 box `motorcycle` và 2 box `person` riêng biệt cho 2 người. | Tách rõ số lượng người để đếm tải chính xác. |
| `img_027.jpg` | Bia tưởng niệm hình xe bán tải pickup bằng đá trong nghĩa trang | `R-03` | **Không gán box `car` hay `truck`.** | Tượng đài điêu khắc tĩnh, không phải xe cơ giới. |
| `img_033.jpg` | Ma-nơ-canh mặc đồ trưng bày trên phố đi bộ | `R-03` | **Không gán box `person`.** | Vật thể giả người vô tri vô giác. |
| `img_038.jpg` | Tủ kính cửa hàng bày hàng loạt mô hình xe và búp bê đồ chơi | `R-03`, `R-05` | Khoanh toàn bộ tủ kính bằng một box `ignore_region`. | Đồ chơi tĩnh nhỏ sau gương kính, không gán chi tiết từng con/xe. |

---

## 5. QC và Quy trình Cập nhật Phiên bản

1. **Ngưỡng kiểm soát chất lượng (QC Thresholds):**
   - Box kích thước lớn ($\ge 50\text{ px}$): Độ trùng khớp $IoU \ge 0.85$.
   - Box kích thước nhỏ ($< 50\text{ px}$): Độ trùng khớp $IoU \ge 0.70$.
   - Đồng thuận nhãn lớp đối tượng (Class Agreement) $\ge 0.95$.
   - Tỉ lệ lỗi tối đa theo ảnh: $\le 2\%$.
2. **Quy tắc giải quyết bất đồng:**
   - Khi có sự khác biệt giữa hai người gán độc lập, căn cứ vào mã quy tắc (`R-01` đến `R-10`).
   - Mọi sửa đổi ảnh hưởng đến ranh giới và phân loại nhãn phải được thảo luận tại Pha 2 để nâng cấp lên bản `guideline_v1.0_agreed.md`.
