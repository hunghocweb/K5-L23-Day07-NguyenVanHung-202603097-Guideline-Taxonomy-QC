# Guideline gán nhãn phát hiện vật 2D — v1.1

- **Phiên bản:** `v1.1` — bản calibration v1.1; nhãn độc lập và IAA giữ theo v1.0. Người phụ trách 101 theo phân công nhóm, QC cuối 103.
- **Ngày:** 2026-11-17.
- **Nguồn tổng hợp:** guideline cá nhân của [101](individual/guideline_101.md), [102](individual/guideline_102.md), [103](individual/guideline_103.md), [104](individual/guideline_104.md). Giữ nguyên cả bốn bản nguồn.
- **Phạm vi:** 40 ảnh `data/images/img_001.jpg`–`img_040.jpg`; CVAT; bounding box 2D thẳng trục; xuất COCO 1.0.
- **Đặc tả đi kèm:** [taxonomy.json](../taxonomy.json), phiên bản `v1.0`; cấu hình CVAT sinh từ taxonomy.
- **Dẫn chiếu:** mã `R-01`–`R-26` trong tài liệu này là mã chung; mã cùng số trong bản cá nhân không mặc nhiên có cùng nghĩa.
- **Nguyên tắc tổng hợp:** lấy hợp các nội dung và giữ đề xuất bổ sung mới của cả bốn bản; không loại một đề xuất chỉ vì các bản còn lại chưa có. Chỗ trùng được gộp diễn đạt; chỗ mâu thuẫn được ghi rõ cách thống nhất để có một quyết định gán duy nhất.

## 1. Mục tiêu

Nhãn dùng để huấn luyện và đánh giá mô hình phát hiện vị trí 2D của người và phương tiện trên ảnh đường phố, phục vụ nghiên cứu camera giao thông và hỗ trợ lái xe. Mỗi nhãn mô tả **phần nhìn thấy trên ảnh**, không suy diễn kích thước 3D, hướng chuyển động, khả năng vận hành hay tính hợp pháp khi lưu thông.

Ưu tiên tránh bỏ sót người, xe máy và vật bị che nhưng còn nhận ra lớp; đồng thời tránh gán nhầm tượng, đồ chơi, hình in thành người/xe thật; vật giả có lớp riêng, còn phản chiếu không gán. Sai lớp giữa xe con, xe tải, van, buýt và tuk-tuk làm sai phân bố dữ liệu; box bao cả hàng hóa hoặc đoán phần khuất làm giảm độ nhất quán hình học.

Ngoài phạm vi: biển số, biển báo, hành vi, danh tính, động vật và hàng hóa riêng lẻ. Giữ đề xuất gán xe đạp, xích lô và một lớp riêng cho vật giả giống người/xe; lớp vật giả giúp phân biệt với đối tượng thật, không được đổi thành nhãn người/xe thật. Các loại bị loại trừ còn lại được xử lý theo R-01, R-10–R-12; không tự thêm lớp trong lúc gán. Các ngưỡng QC ở phần 5 là tiêu chí của bài lab, không chứng minh mô hình đủ an toàn để triển khai ngoài thực tế.

## 2. Lớp và thuộc tính

### 2.1. Lớp

Có **12 lớp đối tượng và 1 lớp vùng bỏ qua**. Một đối tượng thuộc đúng một lớp; `ignore_region` không phải một đối tượng để huấn luyện phát hiện. Giữ toàn bộ tên lớp được đề xuất trong bốn bản, đồng thời tách rõ phạm vi `pickup`/`truck` và `tuk_tuk`/`three_wheeler` để không gán trùng.

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / cách phân biệt |
|---|---|---|---|
| `car` | Ô tô con dạng sedan, hatchback, wagon, SUV hoặc MPV, kể cả taxi, xe trưng bày và xe diễu hành cỡ thật có hình dạng ô tô. | Sedan img_023; SUV img_012; xe bảo tàng img_024; xe vàng img_019. | Bán tải → `pickup`; van thân hộp cao → `van`; đồ chơi/tượng đá giống xe → `non_real_object`. |
| `pickup` | Xe bán tải có cabin kiểu ô tô con và thùng hàng thấp có thành phía sau, kể cả thùng gắn nắp cao hoặc phủ bạt. | Img_012, 028, 037. | Tải nhỏ cabin dựng cao, sàn/thùng chuyên dụng → `truck`; SUV không có thùng hàng → `car`; nắp thùng không biến bán tải thành van. |
| `truck` | Xe tải từ tải nhỏ đến đầu kéo hoặc xe chuyên dụng có thùng/sàn/cơ cấu chở hàng riêng, không thuộc hình thái bán tải `pickup`. | Xe chở xe img_002; xe tải img_007; tải nhỏ img_001, 036. | Không phân theo kích thước hay hàng đang chở; bán tải kiểu ô tô con → `pickup`; van khoang kín liền thân → `van`. |
| `van` | Xe thân hộp kín cỡ nhỏ/vừa, khoang hàng hoặc khách liền buồng lái, không có thùng riêng và không mang hình thái xe buýt. | Van phía sau img_005; van img_016. | SUV/MPV thân thấp → `car`; thùng rời → `truck`; thân xe buýt nhiều hàng cửa sổ → `bus`. |
| `motorcycle` | Xe hai bánh có động cơ, gồm xe số, tay ga, mô tô và xe máy điện; tính cả sidecar hoặc thùng kéo nối trực tiếp. | Img_005, 010, 022, 025, 026, 029. | Ba bánh tích hợp → `tuk_tuk` hoặc `three_wheeler`; xe đạp không động cơ → `bicycle`; xích lô → `cyclo`. |
| `bicycle` | Xe đạp hai bánh chạy bằng sức người, có khung và bộ truyền động bàn đạp, không có dấu hiệu động cơ. | Img_003, 006, 029, 034, 039. | Xe máy điện có động cơ → `motorcycle`; xích lô ba bánh đạp chân → `cyclo`; xe đẩy không có cấu trúc xe đạp → ngoài phạm vi. |
| `cyclo` | Xích lô ba bánh chạy bằng sức người, có ghế hoặc khoang chở khách phía trước/sau người đạp. | Img_023, 039. | Tuk-tuk có động cơ → `tuk_tuk`; xe ba bánh máy chở hàng khác → `three_wheeler`; xe đạp hai bánh → `bicycle`. |
| `person` | Người thật ở mọi tư thế, kể cả người nhìn thấy trong/trên phương tiện, người đi xe đạp và người bị che một phần. | Img_003, 005, 014, 019, 026, 032. | Tượng người, ma-nơ-canh, búp bê, ảnh người → `non_real_object`; phản chiếu → không gán; xem R-10, R-11. |
| `bus` | Xe buýt hoặc xe khách có thân dài/cao chuyên chở nhiều hành khách, thường nhiều hàng cửa sổ và cửa lên xuống đặc trưng. | Xe khách img_009; buýt img_031, 037, 040. | Van khách thân ngắn dạng hộp → `van`; không đoán số ghế không nhìn thấy để quyết lớp. |
| `tuk_tuk` | Xe ba bánh có động cơ mang hình thái tuk-tuk/auto-rickshaw, có khoang hoặc ghế khách đặc trưng, kể cả khi đang chở hàng. | Img_004, 016, 035, 040. | Ba gác máy chỉ có sàn/thùng hàng và không có hình thái tuk-tuk → `three_wheeler`; xích lô đạp chân → `cyclo`; sidecar → `motorcycle`. |
| `three_wheeler` | Xe ba bánh có động cơ và khung tích hợp dạng ba gác/thùng/sàn hàng, không mang hình thái tuk-tuk/auto-rickshaw. | Ba gác máy chở hàng dạng thùng/sàn; chưa có ảnh xác nhận riêng trong bộ 40 ảnh. | Tuk-tuk kể cả chở bao tải ở img_035 → `tuk_tuk`; xe đạp ba bánh chở khách → `cyclo`; mô tô có sidecar → `motorcycle`. |
| `non_real_object` | Vật giả hoặc hình thể hiện giống người hay phương tiện thuộc taxonomy, gồm tượng, ma-nơ-canh, mô hình/đồ chơi, ảnh in và tranh tường. | Tượng người img_017; chân dung img_018; tượng xe img_027; ma-nơ-canh img_033; xe đồ chơi img_038. | Người/xe thật → lớp thật tương ứng; phản chiếu → không gán; tượng ngựa/vịt cao su không giống lớp đích → ngoài phạm vi. |
| `ignore_region` | Vùng chứa cụm đối tượng thuộc taxonomy quá nhỏ hoặc chồng dày không thể tách từng vật đáng tin cậy, kể cả cụm vật giả. | Phần xa img_021; bãi xe img_030; cụm người img_032; cụm mô hình dày img_038. | Không phủ vật rõ, đủ kích thước; không dùng cho phản chiếu hoặc chỉ vì vật ngoài taxonomy. |

Giữ các đề xuất mới `pickup`, `bicycle`, `cyclo`, `three_wheeler`, `non_real_object` từ 101; giữ `bus` và đề xuất nhận diện xe ba bánh ở 101/103/104; giữ tên lớp `tuk_tuk` từ 103/104. Do `three_wheeler` trong bản 101 bao cả tuk-tuk, bản chung tách **nhóm đặc thù `tuk_tuk`** và **ba bánh máy còn lại `three_wheeler`**. Đây là phân chia một đề xuất rộng thành hai lớp loại trừ nhau, không gán cả hai nhãn cho một xe. Không tự tạo ảnh ví dụ cho lớp chưa có ca xác nhận.

Khi quyết lớp: xác định vật giả trước; vật giả giống lớp đích → `non_real_object`, không đồng thời gán lớp thật. Với vật thật, phân biệt theo cấu trúc trong bảng và R-15–R-18, R-24. Phương tiện không nằm trong định nghĩa nào, như xe đẩy tay, vẫn là nền; người trên/cạnh phương tiện đó xét `person` riêng.

### 2.2. Thuộc tính

Mọi box đối tượng đều có ba thuộc tính; `ignore_region` không có thuộc tính. Giá trị mặc định chỉ là trạng thái khởi tạo, người gán phải kiểm lại từng box.

Gọi **p** là phần trăm diện tích hình chiếu của vật **nằm trong ảnh** bị vật khác che. Ước lượng p theo hình dạng vật, không dùng diện tích hình chữ nhật box làm diện tích cơ thể/xe. Phần nằm ngoài ảnh không thuộc mẫu số. Chỉ ước lượng hình dạng để chọn thuộc tính, không vẽ box đầy đủ phần khuất.

| Thuộc tính | Giá trị cố định | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` (mặc định) | p = 0%; không có phần bị vật khác che. |
| | `1` | 0% < p ≤ 30%. |
| | `2` | 30% < p ≤ 70%. |
| | `3` | 70% < p < 100%; vẫn còn dấu hiệu nhìn thấy đủ để nhận ra lớp. Vật bị che hoàn toàn không có box. |
| `truncated` | `no` (mặc định) | Không có phần vật bị cắt ra ngoài ảnh; chạm mép nhưng vẫn thấy đầy đủ không phải cắt mép. |
| | `yes` | Bất kỳ phần nào của vật bị mép ảnh cắt, không cần đạt 10%. |
| `can_review` | `false` (mặc định), `true` | `true` khi còn phân vân về lớp, box, thuộc tính hoặc việc có nên gán; ghi rõ lý do và mã quy tắc. Không dùng cờ này thay cho quyết định bắt buộc của guideline. |

Ví dụ: p = 30% → mức `1`; p = 30,5% → mức `2`; p = 70% → mức `2`; p = 71% → mức `3`. Nếu thực sự không phân biệt được hai mức lân cận, tạm chọn mức cao hơn và bật `can_review` (R-22). Không tự đặt mức che theo tư thế ngồi, ảnh tối hoặc độ nhòe. Người và xe có mức che riêng; che khuất và cắt mép có thể xuất hiện đồng thời.

### 2.3. Các điểm khác nhau đã thống nhất

| Vấn đề | Khác nhau giữa các bản cá nhân | Quyết định v1.0 và lý do | Dẫn chiếu |
|---|---|---|---|
| Bán tải | 101 tách `pickup`; 102 thường gán `car`; 103 còn phân vân nhưng nhiều ca dùng `truck`; 104 gộp `truck`. | Giữ đề xuất lớp mới `pickup`; các bản chưa tách lớp được mở rộng theo lớp này. `truck` giữ tải nhỏ/tải chuyên dụng, không còn bao bán tải hình thái ô tô con. | R-15, R-26 |
| Buýt và xe ba bánh máy | 101 thêm `bus`/`three_wheeler`; 102 để ngoài phạm vi; 103 đề xuất thêm; 104 dùng `bus`/`tuk_tuk`. | Giữ cả `bus`, `tuk_tuk`, `three_wheeler`; chia tuk-tuk đặc thù và ba bánh máy khác theo cấu trúc, không theo hàng đang chở. | R-16, R-17, R-26 |
| Xe đạp, xích lô | 101 thêm lớp; 102–104 loại xe vì chưa có lớp tương ứng. | Giữ đề xuất `bicycle`, `cyclo`; gán xe và từng người riêng. Không giữ cách bỏ xe của taxonomy cũ khi đã bổ sung lớp. | R-04, R-24 |
| Vật giả và tủ đồ chơi | 101 thêm `non_real_object`; 102/103 dùng ignore cho tủ đồ chơi; 104 bỏ toàn bộ vật giả. | Giữ lớp mới `non_real_object`: không gán lớp người/xe thật cho vật giả; vật giả tách được có nhãn riêng, cụm quá nhỏ/dày dùng ignore. Giữ đề xuất xử lý tủ đồ chơi thay vì bỏ cả tủ. | R-09, R-10, R-25 |
| Người trong xe | 101/102/103 có phạm vi người gồm người lái/ngồi xe; 104 loại người trong cabin. | Gán người trong cabin nếu thấy dấu hiệu cơ thể và box nhìn thấy cao ≥ 20 px; không suy ra người từ một chiếc xe. | R-05 |
| Che khuất | Các bản dùng ngưỡng 5%, 10%, 30%, 35%, 70%, 75% khác nhau. | Dùng các khoảng liên tục ở mục 2.2, gần cách chia 103/104; mức `3` là che nặng nhưng nhận ra lớp. | R-06 |
| Cắt mép | 101 yêu cầu khoảng 10%; 102–104 dùng bất kỳ phần bị cắt. | Bị cắt dù ít → `yes`, tránh phải đo phần ngoài ảnh không nhìn thấy. | R-07 |
| Hàng chở trên xe | 101 bao cả hàng xếp cao; 102–104 loại phần nhô ra, một số bản vẫn tính hàng nằm gọn trong thùng. | Hàng rời không quyết định cực trị box; phần nằm gọn trong khung xe không làm đổi box. Nắp/bạt gắn với thùng là kết cấu xe. | R-13 |
| Predator, mô tô trưng bày, xe diễu hành, xe trên nóc | 102 đề xuất giữ người hóa trang; 103 coi Predator là vật giả; 101 coi xe trên nóc img_024 là mô hình nhỏ; 104 xử lý xe cỡ thật theo hình thái. | Giữ cả quy tắc người hóa trang thật → `person` và vật giả → `non_real_object`. Đối chiếu ảnh: Predator dạng tượng → `non_real_object`; xe diễu hành và thân xe cỡ thật trên nóc → `car`; mô tô trưng bày có cấu trúc xe → `motorcycle`. Không coi mọi mô hình trưng bày là đồ chơi thu nhỏ. | R-10, R-14, R-19, R-25 |
| Mờ, kính và hàng rào | Có bản gán mức che vì mưa/mờ; 104 bỏ qua mọi song rào thưa khi tính che. | Mờ không tự tăng mức che; kính trong không che; phần song rào thực sự che vật vẫn tính vào p. | R-06, R-20, R-21 |
| Release | 101 có câu cho phép kết luận từ mẫu QC; 104 và tài liệu lab yêu cầu đủ bằng chứng cả lô. | Duyệt toàn bộ 40 ảnh; QC lấy mẫu chỉ cho kết quả Hold khi bàn giao. | R-23 |

## 3. Quy tắc vẽ box

Áp dụng theo thứ tự: xác định vật có thuộc phạm vi → xác định từng cá thể và lớp → vẽ phần nhìn thấy → kiểm chiều cao → chọn thuộc tính. Các mã sau là điểm dẫn chiếu chuẩn cho ghi chú, calibration và phiếu QC.

| Mã | Quy tắc đã chốt |
|---|---|
| **R-01** | **Chỉ gán đúng phạm vi đã mở rộng.** Gán người, phương tiện và vật giả thuộc 12 lớp đối tượng ở mục 2.1, bao gồm xe đạp và xích lô. Xe đẩy tay không thuộc định nghĩa xe đạp/xích lô, động vật, nón giao thông, biển báo, vịt cao su và hàng hóa rời là nền; không ép sang lớp gần nhất, không tạo ignore chỉ vì ngoài taxonomy. Người trên/cạnh phương tiện vẫn xét riêng. |
| **R-02** | **Một cá thể, một box, một lớp.** Các phần rời nhìn thấy nhưng thuộc cùng một vật chỉ dùng một box. Không gán hai lớp cho một vật; không gộp hai người thành một `person`. Hai vật khác nhau có thể có box chồng nhau. |
| **R-03** | **Ôm sát phần nhìn thấy.** Dùng hình chữ nhật song song trục ảnh, bao các cực trị nhìn thấy của chính vật. Nếu phần nhìn thấy rời nhau, box bao tất cả các phần ấy; khoảng trống giữa chúng nằm trong box là hợp lệ. Không kéo box đến vị trí dự đoán của phần bị che. Xe gồm bánh, gương, tay lái, ống xả, thân và kết cấu gắn liền; người gồm cơ thể, quần áo, tóc, mũ bảo hiểm. Loại bóng đổ, phản chiếu và hàng mang theo. |
| **R-04** | **Người trên phương tiện hở.** Mỗi người đi xe máy, xe đạp hoặc xích lô có một box `person`; xe có một box riêng theo lớp `motorcycle`, `bicycle` hoặc `cyclo`. Hai người trên một xe máy cho 2 `person` + 1 `motorcycle` nếu cả ba box đều đạt ngưỡng R-08. Box xe không mở rộng để bao đầu/thân người; box người không mở rộng để bao xe. Áp dụng cả khi xe đang đỗ; người/xe có mức che riêng, không mặc định một mức cho mọi người ngồi xe. |
| **R-05** | **Người trong cabin/khoang xe.** Người nhìn trực tiếp qua kính trong, cửa mở, trên thùng bán tải hoặc trong tuk-tuk/buýt vẫn gán `person` nếu nhận ra cơ thể và box phần nhìn thấy cao ≥ 20 px. Chỉ thấy đầu vẫn được gán nếu đạt điều kiện. Phần bị thân xe che tính vào mức che của người; ảnh chân dung dán trên kính và phản chiếu không phải người trong cabin. |
| **R-06** | **Vật bị che.** Box theo R-03; `occluded_level` theo mục 2.2. Che > 70% vẫn gán nếu phần còn lại đủ phân biệt lớp và đạt R-08. Không suy ra một xe chỉ vì có người ngồi tư thế lái; phải thấy dấu hiệu xe. Vật che hoàn toàn hoặc không thể nhận ra lớp không có box đoán, ghi vấn đề theo R-22. |
| **R-07** | **Cắt mép ảnh.** Box dừng đúng tại mép ảnh của phần vật còn thấy, không có tọa độ vượt ảnh. Đặt `truncated=yes` khi có phần thực sự bị cắt dù ít; không gọi vật bị xe khác che là truncated. Vẫn kiểm chiều cao nhìn thấy theo R-08, không dùng chiều cao giả định ngoài ảnh. |
| **R-08** | **Ngưỡng vật nhỏ.** Đo chiều cao box phần nhìn thấy trên ảnh gốc, không đo trên màn hình sau zoom. Cao < 20 px: không gán lớp đối tượng; một vật nhỏ đơn lẻ bỏ qua, cụm nhiều vật nhỏ dùng R-09. Cao đúng 20 px trở lên: gán nếu nhận ra lớp và tách được. Chiều rộng không thay cho chiều cao; vật bị che nặng không được miễn ngưỡng này. |
| **R-09** | **Đám đông và vùng ignore.** Gán từng vật đủ kích thước, còn tách được; chỉ bao phần còn lại quá nhỏ hoặc chồng dày không thể tách đáng tin cậy bằng `ignore_region`. Một cụm liền dùng một vùng; các cụm tách rời dùng vùng riêng, không phủ cả ảnh. Vẽ sát cụm, tránh che các vật đã gán rõ; nếu hình chữ nhật bao cụm nuốt vật rõ, chia vùng ignore nhỏ hơn. Chỉ khó vì phải vẽ nhiều không phải lý do dùng ignore. |
| **R-10** | **Tượng, hình in, đồ chơi.** Giữ đề xuất lớp `non_real_object`: tượng người, ma-nơ-canh, búp bê hình người, mô hình xe thu nhỏ, xe bằng đá nguyên khối và người/xe trên ảnh in, áp phích, tranh tường gán lớp này nếu tách được và đạt R-08. Không gán lớp người/xe thật cho chúng. Tượng ngựa, vịt đồ chơi hoặc hình không giống bất kỳ lớp người/xe đích nào vẫn ngoài phạm vi. Người hóa trang có bằng chứng là người thật → `person`; không tự giả định bên trong tượng/linh vật có người. Phương tiện trưng bày theo R-19; cách bao vật giả theo R-25. |
| **R-11** | **Phản chiếu và bóng.** Không gán ảnh phản chiếu trong gương, kính hoặc mặt đường ướt; không bao bóng đổ/vệt đèn vào box vật. Vật thật nhìn xuyên qua kính vẫn gán theo R-21. Không dùng ignore để che phản chiếu. |
| **R-12** | **Xe chủ (ego).** Không gán bộ phận của xe đang mang camera: gương, cửa, capo, taplo, vô lăng. Xe khác chụp cạnh xe chủ vẫn gán. Chỉ loại một xe là ego khi xác định được quan hệ với vị trí chụp, không vì nó ở gần mép ảnh. |
| **R-13** | **Hàng hóa và đồ mang.** Không dùng cây, ống sắt, bao tải, sọt hàng, tượng chở trên thùng, quang gánh, túi/giỏ rời để mở rộng box xe/người. Nếu hàng nằm gọn trong đường bao kết cấu xe thì box xe giữ nguyên, không cần khoét phần hàng. Nắp thùng, bạt che cố định theo khung thùng và giá chở gắn với xe thuộc kết cấu xe. Hàng che thân xe thì tăng p tương ứng. |
| **R-14** | **Xe chở xe.** Một box `truck` cho đầu xe cùng khung/sàn/giá chở, không mở rộng theo cực trị ô tô được chở. Mỗi xe được chở còn nhận ra lớp và đạt R-08 có box riêng, kể cả bị khung giá che. Xe trên sàn vẫn phân lớp theo hình thái: bán tải là pickup theo R-15, sedan/SUV/MPV là car; không mặc định mọi xe được chở là car. Thân xe cỡ thật nhận ra được trên nóc img_024 cũng có box riêng; không áp dụng cho xe đồ chơi thu nhỏ. Không sao chép số lượng xe từ mô tả khi chưa kiểm trên ảnh. |
| **R-15** | **Bán tải và tải nhỏ.** Giữ lớp `pickup` cho bán tải có cabin kiểu ô tô con và thùng hàng thấp có thành; lắp nắp cao hoặc bạt vẫn là `pickup` nếu còn dấu hiệu cấu trúc bán tải. Tải nhỏ cabin dựng cao/cab-over với sàn/thùng chuyên dụng, tải thùng, đầu kéo và xe chuyên dụng → `truck`. Không chỉ dựa vào kích thước, số bánh hay hàng đang chở; tải nhỏ img_001/036 không tự thành pickup vì nhỏ. Thiếu dấu hiệu phân biệt thì R-22. |
| **R-16** | **Car, van, bus.** Dùng hình thái thân xe theo bảng lớp. Xe chở khách dạng buýt/coach → `bus`; xe hộp kín liền thân ngắn → `van`; sedan/SUV/MPV → `car`. Không suy ra số ghế, nhãn hiệu hoặc tải trọng không thấy. Không đủ dấu hiệu để phân biệt thì R-22. |
| **R-17** | **Ba bánh có động cơ.** Giữ cả hai đề xuất: có hình thái tuk-tuk/auto-rickshaw với khoang/ghế khách đặc trưng → `tuk_tuk`; ba gác máy dạng sàn/thùng hàng khác → `three_wheeler`. Một tuk-tuk đang chở bao tải như img_035 vẫn là `tuk_tuk`. Xích lô đạp chân → `cyclo`. Không đổi lớp vì bị che mất một bánh; dựa vào cấu trúc còn thấy, không suy đoán động cơ chỉ từ số bánh. Không đồng thời gán `tuk_tuk` và `three_wheeler` cho cùng xe. |
| **R-18** | **Sidecar và thùng kéo.** Xe máy cùng sidecar hoặc thùng kéo nối trực tiếp thành một box `motorcycle`. Người trên đó xét riêng theo R-04; hàng rời theo R-13. Xe ba bánh tích hợp không xuất phát từ mô tô gắn phụ kiện xử lý theo R-17. |
| **R-19** | **Xe đỗ, xe trưng bày và xe khác thường.** Xe cỡ thật đang đỗ, trong kho, bảo tàng hoặc diễu hành vẫn gán theo hình dạng; không cần chứng minh xe chạy được. Quy ước của bộ ảnh 2D: mô tô trưng bày cỡ thật có bánh, khung và bộ phận xe nhận ra được ở img_011 được gán `motorcycle`, kể cả trang trí bằng kim loại. Quy ước này không mở rộng sang tượng xe nguyên khối trên bệ hay mô hình thu nhỏ (R-10). |
| **R-20** | **Đêm, mưa, đen trắng, nhòe.** Gán nếu vẫn nhận ra lớp và đạt R-08. Box ôm phần vật còn xác định được, gồm vệt nhòe gắn trực tiếp với đường bao vật; loại ánh sáng/bóng không thuộc vật. Nhòe gây mơ hồ ranh giới phải bật `can_review`. Tối/mờ không tự tăng `occluded_level`; không nhìn ra lớp thì R-22. |
| **R-21** | **Kính và hàng rào.** Xe/người thật sau kính trong hoặc rào vẫn gán nếu còn nhận ra lớp và đạt R-08. Kính trong không tính là vật che; cột và song rào thực sự che diện tích vật thì tính phần ấy vào p, không coi toàn bộ vùng giữa các song là bị che. Kính mờ/đục che mất hình vật có thể tạo che khuất; không nhầm phản chiếu với vật nhìn xuyên kính. |
| **R-22** | **Ca chưa chắc.** Nếu đủ bằng chứng để có box và lớp tạm hợp lý: bật `can_review=true`, ghi ảnh, vùng/ID, lý do và mã quy tắc. Nếu chưa thể xác định có vật/lớp/box: không tạo box giả để đặt cờ; ghi `image, annotator, region, note` trong `submission/qc/image_issues.csv`. Không tự thêm lớp hoặc sửa quy tắc lúc gán độc lập; đưa vào calibration. |
| **R-23** | **QC và thay đổi phiên bản.** Dùng nguyên ngưỡng phần 5; kiểm nhãn độc lập theo v1.0 trước calibration. Ca chưa giải quyết phải có quyết định truy được đến người và mã quy tắc. Mọi thay đổi ảnh hưởng chọn vật, lớp, box hoặc thuộc tính phải tăng phiên bản và rà lại phạm vi chịu ảnh hưởng; không sửa ngược bản cá nhân. |
| **R-24** | **Xe đạp và xích lô.** Giữ đề xuất gán `bicycle` cho xe đạp hai bánh không động cơ và `cyclo` cho xích lô ba bánh đạp chân; gán người đạp/ngồi riêng theo R-04. Xe đang dắt, đỗ hoặc chở hàng vẫn giữ lớp; hàng rời không làm rộng box. Không gộp xích lô vào xe đạp chỉ vì ít ảnh. Nếu không phân biệt được động cơ thì bật `can_review`/ghi vấn đề theo R-22, không tự bỏ xe. |
| **R-25** | **Box của vật giả.** Một ma-nơ-canh, tượng người, búp bê hình người hoặc xe đồ chơi tách được là một cá thể `non_real_object`; box ôm phần nhìn thấy của hình người/xe giả, không ôm cả biển quảng cáo, tủ kính hay bệ trưng bày. Hình người cưỡi ngựa liền một tượng ở img_017 gán phần hình người; ngựa/nón là phần ngoài phạm vi. Tủ img_038 gán từng mô hình người/xe đủ kích thước và tách được; chỉ phần cụm còn quá nhỏ/chồng dày mới dùng ignore theo R-09. Phản chiếu của vật giả vẫn không gán. |
| **R-26** | **Bổ sung đề xuất, không thu hẹp theo đa số.** Hợp nhất tất cả lớp và tình huống mới từ bốn bản cá nhân. Nội dung trùng gộp diễn đạt; tên/định nghĩa chồng lấn phải có ranh giới loại trừ nhau, như `pickup`/`truck` và `tuk_tuk`/`three_wheeler`. Cách bỏ qua do taxonomy cũ thiếu lớp được thay bằng lớp bổ sung tương ứng. Với hai cách gán mâu thuẫn không thể cùng áp dụng, lưu hai phương án và lý do chốt tại mục 2.3; không bỏ mất tình huống gốc và không tạo hai nhãn cho một vật. |

## 4. Ca khó

Chọn **8 ca đại diện**, không liệt kê cả 40 ảnh. Các ảnh tham chiếu có đuôi `.jpg` trong `data/images/`; bốn bản cá nhân giữ chi tiết khảo sát ban đầu. Đây là quyết định theo quy tắc, không phải kết quả đếm nhãn hoặc QC đã thực hiện. Mọi box đối tượng, kể cả vật giả, đều phải đạt R-08.

| Ca / ảnh tham chiếu | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| C-01 — img_005, img_026 | Người trên xe máy, nhiều người chung một xe, xe bị người che. | R-02, R-04, R-06, R-08 | Mỗi người một `person`, xe một `motorcycle`; hai người trên một xe cho 3 box nếu đều đạt ngưỡng. Phần xe còn thấy phải đủ nhận ra lớp; không kéo box để bù phần khuất. | Tách cá thể và mức che riêng; không suy ra một xe chỉ từ tư thế người. |
| C-02 — img_002, img_024 | Xe tải chở ô tô; thân xe vàng cỡ thật đặt trên nóc xe bảo tàng. | R-13, R-14, R-19 | Xe chở chuyên dụng → `truck`; từng xe được chở phân theo hình thái; các xe bán tải trên img_002 → pickup theo R-15. Ở img_024, xe đen và thân xe vàng cỡ thật có box `car` riêng; không mở rộng box xe dưới theo xe trên nóc. | Giữ quy tắc xe chở xe và xe trưng bày; không nhầm thân xe cỡ thật với đồ chơi thu nhỏ. |
| C-03 — img_001, img_012, img_037 | Tải nhỏ và bán tải, bán tải gắn nắp cao. | R-07, R-13, R-15 | Tải nhỏ cabin dựng cao/thùng chuyên dụng img_001 → `truck`; bán tải img_012/037 → `pickup`, kể cả có nắp cao. Không bao hàng nhô ra; cắt mép thật → `truncated=yes`. | Giữ lớp mới `pickup`, phân biệt bằng cấu trúc thay vì chỉ kích thước hoặc loại hàng. |
| C-04 — img_004, img_023, img_029, img_035 | Tuk-tuk chở người/hàng, xích lô và xe đạp cạnh xe máy điện. | R-04, R-05, R-17, R-24 | Tuk-tuk img_004/035 → `tuk_tuk`; xích lô đạp img_023 → `cyclo`; xe đạp img_029 → `bicycle`, xe máy điện → `motorcycle`; từng người có box riêng. Ba gác máy kiểu sàn/thùng không có hình thái tuk-tuk dùng `three_wheeler` khi gặp. | Giữ đủ đề xuất mới, chia lớp theo cấu trúc; việc chở hàng không đổi tuk-tuk sang lớp khác. |
| C-05 — img_011, img_019 | Predator trưng bày, mô tô kim loại và xe diễu hành hình dạng khác thường. | R-10, R-19, R-22, R-25 | Predator dạng tượng người → `non_real_object`; mô tô có cấu trúc xe cỡ thật → `motorcycle`; xe vàng diễu hành → `car`; người lái đủ ngưỡng → `person`. Người hóa trang thật vẫn là `person` nếu có bằng chứng, không áp cách bỏ tượng cho mọi người mặc đồ hóa trang. | Giữ cả đề xuất xử lý cosplay và phương tiện khác thường; phân biệt bằng dấu hiệu nhìn thấy. |
| C-06 — img_017, img_027, img_033, img_038 | Tượng người, tượng xe đá, ma-nơ-canh và tủ đồ chơi. | R-01, R-09, R-10, R-25 | Hình người trên tượng, tượng xe, ma-nơ-canh và từng đồ chơi giống người/xe đủ điều kiện → `non_real_object`. Phần tủ có mô hình quá nhỏ/dày không tách được → `ignore_region`; không phủ vật giả rõ đã gán. Ngựa/nón/vịt ngoài phạm vi. | Giữ lớp mới cho vật giả và đề xuất ignore cụm đồ chơi; không gán lớp người/xe thật cho đồ giả. |
| C-07 — img_018, img_020, img_033 | Hình in, vật sau kính/rào, phản chiếu và xe chủ. | R-10, R-11, R-12, R-21, R-25 | Chân dung in → `non_real_object`; xe thật nhìn xuyên kính/rào → lớp xe tương ứng; phần song/cột thực sự che tính vào p. Phản chiếu và bộ phận xe chụp không gán. Img_020 không có mục tiêu khác thì giữ với 0 box. | Tách rõ hình in có nhãn vật giả, vật thật sau kính, ảnh phản chiếu và ego. |
| C-08 — img_014, img_015, img_021, img_030, img_032 | Đêm/mưa/nhòe và cụm người, xe quá nhỏ hoặc chồng dày. | R-06, R-08, R-09, R-20, R-22 | Gán từng vật cao ≥ 20 px, nhận ra lớp và tách được; phần cụm còn quá nhỏ/chồng dày → ignore. Nhòe còn nhận ra thì gán, bật `can_review` khi mép mơ hồ; không tự tăng mức che vì ảnh tối/mờ. | Áp dụng cùng ngưỡng trên ảnh gốc, tránh dùng ignore vì nhiều vật hoặc dùng độ mờ thay mức che. |

Ca có quy tắc rõ xử lý trực tiếp; chỉ bật `can_review` khi bằng chứng vẫn chưa đủ để áp dụng dứt khoát. Không thể tạo box thì ghi vấn đề cấp ảnh theo R-22. Ảnh có 0 box vẫn phải có trong danh sách `images` của COCO và được kiểm bằng mắt.

## 5. QC và phiên bản

### 5.1. Ngưỡng và cách đo

Giữ nguyên toàn bộ mục `rules` của [taxonomy.json](../taxonomy.json); không hạ ngưỡng để đạt nghiệm thu.

| Tham số | Giá trị | Cách áp dụng |
|---|---|---|
| `min_box_height_px` | 20 | Chiều cao box phần nhìn thấy trên ảnh gốc; không áp dụng ngưỡng này cho vùng ignore. |
| `large_px` | 50 | Cao ≥ 50 px là lớn; 20 ≤ cao < 50 px là nhỏ. Khi so hai box, script dùng trung bình chiều cao hai box để chọn ngưỡng. |
| `iou_thr_large` | 0.85 | IoU tối thiểu của cặp box lớn. |
| `iou_thr_small` | 0.70 | IoU tối thiểu của cặp box nhỏ; đúng 50 px dùng ngưỡng lớn. |
| `match_iou` | 0.50 | Ghép box 1–1 theo hình học, không phân biệt lớp; đây không phải ngưỡng chấp nhận box. |
| `iaa_iou` | 0.85 | IoU trung bình trên các cặp đã ghép phải đạt ≥ 0.85. |
| `iaa_class` | 0.95 | Số cặp cùng lớp / số cặp ghép phải đạt ≥ 95%. |
| `error_rate_img_max` | 0.02 | Ảnh có ít nhất một lỗi / ảnh đã duyệt phải ≤ 2%. |

Đủ 100% lượt gán theo `assignment.csv` mới đo IAA; với nhóm bốn người, mỗi ảnh cần ít nhất hai người gán độc lập. Báo riêng số box chưa ghép, bất đồng ignore và thuộc tính; IAA tốt trên cặp đã ghép không chứng minh không bỏ sót. Không có cặp ghép ghi N/A. Giữ số đo trước calibration; mọi tỉ lệ phải ghi tử số/mẫu số.

### 5.2. Kiểm tra và quyết định

1. **Trước khi gán:** đối chiếu guideline với taxonomy `v1.0`; chạy `python scripts/make_cvat_labels.py` để kiểm schema và sinh `cvat/labels_generated.json`, sau đó dùng cấu hình này trong CVAT. Cờ `occluded` sẵn của CVAT không thay cho thuộc tính `occluded_level`.
2. **Khi gán độc lập:** kiểm đủ từng vật và thuộc tính, giữ ảnh âm tính, ghi ca chưa chắc theo R-22. Không xem nhãn người khác để điều chỉnh nhãn của mình.
3. **Calibration:** người gán nêu ca và mã quy tắc; cả nhóm thống nhất cách xử lý, ghi mã người chốt thực tế trong `calibration_decisions.csv`. Nếu chưa thống nhất thì tiếp tục giữ vấn đề mở, không coi cờ `can_review` là đã giải quyết. Vai trò người chốt/reviewer phải được xác nhận trong `TEAM.md` trước bàn giao.
4. **QC cuối:** kiểm bằng mắt toàn bộ 40 ảnh và dùng script kiểm cấu trúc, phạm vi, lớp/thuộc tính. Phiếu QC dùng `bo_sot`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh`; ảnh sạch ghi `khong`. Mỗi lỗi ghi Rule ID, vị trí hoặc ID và lý do; kiểm cả trường hợp mọi người cùng sai.
5. **Quyết định lô:** Hold nếu thiếu bằng chứng, còn vấn đề guideline/dữ liệu, review lệch phiên bản hoặc chỉ mới lấy mẫu; Rework nếu tỉ lệ ảnh lỗi > 2% hoặc lỗi lặp theo quy định lab; Release chỉ sau khi reviewer xác nhận toàn lô và bằng chứng đạt yêu cầu. Với 40 ảnh, 1 ảnh lỗi = 2,5% nên vượt ngưỡng. Ghi riêng tỉ lệ lỗi theo đối tượng và mẫu số theo công cụ của lab.

Tài liệu này chốt quy tắc, không xác nhận đã gán nhãn, đo IAA, calibration hay Release. Dùng [QC_CHEATSHEET.md](../../docs/QC_CHEATSHEET.md) và [hướng dẫn phiếu QC](../../scripts/README.md) để ghi bằng chứng thực tế.

### 5.3. Khóa phiên bản và lịch sử thay đổi

Bản v1.0 này đã được chỉnh theo yêu cầu tổng hợp giữ đề xuất mới, trước khi xác nhận khóa để gán độc lập. Khi nhóm bắt đầu lượt gán, khóa cặp guideline/taxonomy `v1.0`. Nếu phát hiện khoảng trống, ghi ca theo R-22 để calibration; không sửa âm thầm v1.0. Thay đổi sau calibration được ghi vào `guideline_v1.1_calibrated.md` và `submission/guideline_change.json`, có lý do, Rule ID, người quyết định và phạm vi rà lại. Thay đổi ngữ nghĩa cần quyết định phạm vi lô và kiểm lại cả lô theo hướng dẫn lab; nếu đổi taxonomy, lưu `taxonomy_v1.1.json` và sinh lại cấu hình CVAT. Giữ nguyên mã cho quy tắc còn cùng nghĩa; mã mới cấp tiếp, không tái dùng mã cũ cho nội dung khác.

| Phiên bản | Thay đổi | Lý do / ca tham chiếu |
|---|---|---|
| `v0` | Bản khởi đầu: 5 lớp đối tượng + ignore; quy tắc và thuộc tính còn sơ lược. | Cơ sở của bài lab. |
| Bốn bản cá nhân Pha 1 | Bốn cách diễn giải độc lập của 101–104; giữ nguyên phiên bản và nội dung nguồn. | Bằng chứng để đối chiếu bất đồng ở mục 2.3. |
| `v1.0` — 2026-11-16 | Hợp nhất đủ 5 phần theo nguyên tắc giữ đề xuất bổ sung; 26 mã quy tắc; 8 ca đại diện; giữ đủ 13 tên lớp gồm `pickup`, `bicycle`, `cyclo`, `three_wheeler`, `non_real_object`, `bus`, `tuk_tuk` và các lớp gốc; phân rõ lớp chồng lấn; đồng bộ taxonomy, giữ nguyên `rules`. | Img_004/009/035/040 (lớp mới); img_012/037 (bán tải); img_011/019/024 (trưng bày/giả tưởng); img_017/027/033/038 (vật giả); img_021/030/032 (cụm dày); img_014/015 (nhòe). |

### 5.4. Kiểm tra độ đầy đủ của bản tổng hợp

| Đề xuất bổ sung | Nguồn đề xuất | Vị trí áp dụng trong bản chung |
|---|---|---|
| Tách bán tải `pickup` | 101 | Mục 2.1, R-15, ca C-03; giữ `truck` cho tải nhỏ/chuyên dụng. |
| Thêm `bicycle`, `cyclo` | 101 | Mục 2.1, R-04, R-24, ca C-04; không bỏ phương tiện chỉ vì bản v0 thiếu lớp. |
| Thêm `three_wheeler` và `tuk_tuk` | 101 (`three_wheeler`), 103/104 (`tuk_tuk`) | Mục 2.1, R-17, ca C-04; chia hai phạm vi loại trừ nhau và giữ cả hai tên lớp. |
| Thêm `bus` | 101, 103, 104 | Mục 2.1, R-16; không ép buýt vào car/truck/van. |
| Thêm `non_real_object` | 101 | Mục 2.1, R-10, R-25, ca C-05–C-07; giữ phân biệt vật giả với người/xe thật của 102–104. |
| Ignore phần cụm đồ chơi khó tách | 101, 102, 103 | R-09, R-25, ca C-06; gán riêng vật đủ điều kiện, chỉ ignore phần cụm còn lại. |
| Người trong/trên xe và nhiều người chung xe | 101, 102, 103; quy tắc tách người/xe cũng có ở 104 | R-04, R-05, ca C-01/C-04; người trong cabin không bị loại chỉ vì vị trí ngồi. |
| Người hóa trang, xe giả tưởng, mô tô trưng bày cỡ thật | 102 (người hóa trang), 101/104 (xe khác thường), 102–104 (mô tô) | R-10, R-19, ca C-05; giữ quy tắc tổng quát, đối chiếu ảnh để không nhầm tượng thành người. |
| Sidecar/thùng kéo gắn xe máy | 101/103 (sidecar), 104 (thùng kéo) | R-18; giữ một box cho cụm xe, người và hàng xử lý riêng. |
| Box nhìn thấy; hàng nhô ra; xe chở xe; xe đỗ/bảo tàng | Cả bốn bản với mức chi tiết khác nhau | R-03, R-13, R-14, R-19; thống nhất ranh giới để không có hai cách vẽ. |
| Xe chủ; kính/rào; đêm/mưa/nhòe; vấn đề chưa có box | 103, 104; các bản khác có một phần | R-11, R-12, R-20–R-22; bổ sung đầy đủ thay vì chỉ giữ quy tắc chung của v0. |
| Mức che có ngưỡng, cắt mép, kích thước, IAA và QC/phiên bản | Cả bốn bản | Mục 2.2, R-06–R-09, R-23, phần 5; các phương án mâu thuẫn lưu ở mục 2.3, ngưỡng QC gốc không đổi. |

Giữ đề xuất mới không có nghĩa áp dụng đồng thời hai chỉ dẫn trái nhau: mỗi ca vẫn chỉ có một quyết định theo quy tắc chung. Bốn bản nguồn được giữ nguyên để truy lại chi tiết, các ví dụ và phương án ban đầu.

## 6. Calibration v1.1

- Phụ trách: 101 theo phân công nhóm và ủy quyền xử lý của người điều phối; QC độc lập tiếp theo: 103. Không ghi nhận 103 đã review.
- Thay đổi ngữ nghĩa: làm rõ R-14/C-02 theo R-15, xe được chở không mặc định car; pickup trên img_002 vẫn là pickup. Phạm vi rà là toàn bộ 40 ảnh, gồm img_020 có 0 đối tượng.
- R-03/R-13: box ôm cực trị nhìn thấy của chính vật; không bao hàng rời, bóng hoặc nền. Không lấy trung bình hai box để thay việc đọc ảnh. Một xe cùng sidecar hợp nhất theo R-18.
- R-04/R-05/R-17: tách người và xe; img_004 là tuk_tuk, hai xe ba bánh có động cơ ở img_016 không phải cyclo.
- R-06/R-07: xét riêng che và cắt mép; tối/mờ không tự làm tăng mức che.
- R-10/R-19/R-25: tượng xe img_027 là non_real_object; thân xe cỡ thật img_024 là car. Tượng người img_017 chỉ ôm người/trang phục, không bao ngựa/nón giao thông. Vịt và động vật đồ chơi không tự thành lớp đích.
- R-09/R-22: ignore chỉ dành cho cụm quá nhỏ/dày; không bao hàng trên nóc xe hoặc phủ vật rõ đã tách được. Cờ can_review chỉ tắt sau xử lý trong bộ calibration; chưa thay QC độc lập.
- Mỗi ca IAA có dòng trong calibration_decisions.csv; ca M001 ghi thêm khả năng hai người cùng sai ở img_002.
- Taxonomy v1.0 và toàn bộ ngưỡng giữ nguyên, không cần taxonomy_v1.1.json. Guideline_change ghi semantic vì việc làm rõ R-14/C-02 tác động cách phân lớp, cần rà toàn lô.
- Nhãn cuối và manifest do build_final.py sinh sau khi có COCO đúng tên phiên bản. Bộ này chưa có quyết định Release; cần CP6.

| Phiên bản | Thay đổi | Phạm vi |
|---|---|---|
| v1.1 — 2026-11-17 | Làm rõ R-14/C-02 theo R-15; áp dụng lại quy tắc box, lớp, thuộc tính, vật giả và ignore. | 40 ảnh, giữ nguyên nhãn độc lập và IAA trước sửa. |
