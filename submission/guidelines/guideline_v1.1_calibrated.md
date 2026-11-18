# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603238 - Nguyễn Quốc Khánh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân loại dựa trên sự hiện diện của bàn đạp cơ học (pedals): nếu có bàn đạp trợ lực gán `bicycle`; nếu chạy thuần điện không có bàn đạp (hoặc kiểu dáng/tốc độ tương đương xe máy) gán `motorbike`. | `img_015.jpg`, `img_016.jpg` | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc gần camera / ban đêm: từng xe hay `ignore_region`? | Nếu xe đứng riêng rẽ, thấy rõ $\ge 20\%$ diện tích và nhận diện được ranh giới thì gán từng xe. Nếu xe đỗ ken đặc trong bóng tối/thiếu sáng không phân biệt được ranh giới từng chiếc thì vẽ một vùng `ignore_region` bao quanh cụm xe. | `img_026.jpg` | CAL-03 |
| 5 | Kính chắn gió / khoảng hở xuyên thấu xe ba bánh: thuộc mask xe hay không? | Kính chắn gió trong suốt thuộc mask của xe. Tuy nhiên, các khoảng hở nhìn xuyên thấu ở khung xe ba bánh (tuk-tuk/ba gác) có diện tích $> 5\%$ hoặc nhìn thấy người ngồi bên trong phải được khoét bỏ khỏi mask của xe. | `img_013.jpg`, `img_017.jpg` | CAL-04 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg`, `img_004.jpg` | Xe tuk-tuk có mui chở khách lưu thông trên phố | YOLO26-m (closed-set) gán nhầm thành `car 0.42` hoặc `truck 0.77` do COCO không có nhãn ba bánh | Luôn gán `three_wheeler` cho cả xe lẫn khung mui; người lái và hành khách ngồi trong xe vẽ box `person` độc lập nếu nhìn thấy $\ge 20\%$ | Bổ sung Mục 2 Taxonomy và Mục 4.1.4: Xe ba bánh có mui luôn là `three_wheeler`, người ngồi trên xe là `person` riêng không gộp chung vào xe |
| `img_026.jpg` | Xe đạp đỗ lẫn trong bãi xe máy ban đêm thiếu sáng (box #352) | YOLO26 phát hiện box nhưng khó phân biệt xe đạp thường hay xe máy điện | Tick thuộc tính `can_review` để hội chẩn; các cụm xe sâu trong bóng tối không tách được biên chuyển sang gán `ignore_region` | Bổ sung Mục 4.1.6: Khi điều kiện ánh sáng yếu không xác định chắc chắn được chủng loại giữa xe đạp/xe máy, tick `can_review` hoặc khoanh `ignore_region` |
| `img_013.jpg` | Người ngồi sau xe ba bánh bị khung kim loại che khuất một phần | Mô hình sinh 2 box đè nhau (1 box cụt ngực 63px và 1 box toàn thân 189px); bẫy `loose` cấy vào box cụt | Xóa box cụt lửng, chỉ giữ một box `person` duy nhất bao trọn phần cơ thể nhìn thấy | Bổ sung Mục 4.1.2: Đối với người ngồi sau xe hở, chỉ vẽ 1 box duy nhất bám phần nhìn thấy, cấm để box trùng lặp cắt cụt thân thể |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Chốt quy tắc phân loại xe điện hai bánh (dựa vào bàn đạp); quy định khoanh `ignore_region` cho bãi xe máy đêm thiếu sáng; chuẩn hóa nhãn `three_wheeler` và tách người lái độc lập; loại bỏ box cụt trùng lặp người ngồi trên xe ba bánh. | Khắc phục lỗi closed-set của YOLO26 (`img_013`, `img_004` - CAL-01), chốt câu hỏi mở xe điện (`img_015`, `img_016` - CAL-02), xử lý ca khó bãi xe đêm (`img_026` - CAL-03), khắc phục bẫy loose lệch do box trùng (`img_013` - CAL-04). |
