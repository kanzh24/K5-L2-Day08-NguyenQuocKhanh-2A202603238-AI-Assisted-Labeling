# Báo cáo Day 08 — <2603238 - Nguyễn Quốc Khánh>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-m | 0.3 (mặc định) | Dẫn `r02`, `r03`: Trên 4 ảnh gold không có ngưỡng nào <= 0.5 đạt mục tiêu recall 90% (person đạt 84%, motorbike 76%, car 82%, bicycle 44%; bus/truck < 5 vật). Do đó giữ mặc định 0.3 để tránh overfit tập gold quá nhỏ. |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk, cyclo, cargo tricycle` | 0.15 | Dẫn `r04`: Đạt recall 100% (>= 90%) và precision 67% trên gold; phát hiện được tuk-tuk/xích lô/ba gác mà YOLO26 bị mù. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`) vì cân bằng tối ưu giữa chất lượng bám biên sắc nét và tốc độ/RAM trên GPU; không bị lem mờ như SAM 1 (`mobile_sam.pt`) và không đòi hỏi quyền truy cập gated nặng như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Xe ba bánh bị YOLO26 gán nhầm thành `truck` (`img_011` conf 0.77, `img_002` conf 0.73) hoặc `car` (`img_013` conf 0.42). Lỗi này do taxonomy đóng của COCO không có xe ba bánh, nên dù tăng ngưỡng lên 0.5 (`r03`) thì box sai vẫn tồn tại với điểm cao. Sửa bằng cách kết hợp model open-vocabulary (YOLOE-26s) và dùng `merge_prelabels.py` ưu tiên `three_wheeler` ghi đè `car`/`truck` khi IoU >= 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: `truck` conf 0.77 ở `img_011.jpg` (thực chất là xe ba bánh/songthaew). Điểm thấp mà đúng: `three_wheeler` conf 0.28 ở `img_011.jpg` và `person` conf 0.32 ở `img_011.jpg`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Ngưỡng được chọn và đánh giá trên cùng một tập dữ liệu (in-sample evaluation) nên precision/recall đo được cao hơn thực tế ngoài thực địa; 4 ảnh gold là kích thước mẫu quá bé, các lớp hiếm (bus: 1 vật, truck: 0 vật) không đủ ý nghĩa thống kê để chọn ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 425 | 75.1% | 566 box máy |
| edited | 28 | 4.9% | 566 box máy |
| deleted | 100 | 17.7% | 566 box máy |
| added | 56 | 10.9% | 513 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` tệ nhất với 24/42 box bị xóa (57.1%) do YOLOE bắt nhầm xe bán tải/sạp hàng (`img_002`) và box còn lỏng trùm cả người lái; kế tiếp là `person` bị xóa 39 box và thêm 14 box do người ở xa bị dính cụm hoặc bỏ sót (`img_011`, `img_018`).
- So với gold: precision / recall của **máy**: Precision = 74.4% (58/78, KTC 95%: 64–83%), Recall = 67.4% (58/86, KTC 95%: 57–76%) (mẫu số: 86 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 (83.3%); loại nào lọt: 2 bẫy `missing` ở `img_003.jpg` và `img_024.jpg`; vì sao (bạn đã bỏ bước nào của checklist)? Đã bỏ qua bước 1 của checklist (chưa ẩn toàn bộ nhãn máy để đếm nhẩm tổng thể trước khi soi nhãn), dẫn tới bị neo tâm lý vào các box máy đã vẽ sẵn và lướt qua những vật thể máy bỏ sót hoàn toàn.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Tỷ lệ accepted 75.1% và bắt bẫy 83.3% (> 80%) cho thấy người duyệt duy trì được độ tỉnh táo tốt trước các bẫy đổi lớp (`wrong_class` 3/3), bẫy box ma (`extra` 3/3) và lệch viền (`loose` 3/3). Tuy nhiên, việc lọt 2 bẫy missing cho thấy vẫn có tâm lý ỷ lại vào máy ở các vùng không có box gợi ý.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 5.0 phút/ảnh (20 phút / 4 ảnh) vs duyệt 3.75 phút/ảnh (90 phút / 24 ảnh). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Do đã gán 4 ảnh gold trước nên đã nắm vững taxonomy và nhận diện cảnh quen thuộc hơn; việc có box gợi ý sẵn giúp giảm thời gian gán khoảng 25% mỗi ảnh.
- Kết luận: Có đáng dùng, đặc biệt hiệu quả cao cho các lớp phổ biến tiền cảnh (`car`, `motorbike`, `person`) với tỷ lệ accepted đạt 75.1% (425/566 box) giúp giảm bớt việc vẽ box thủ công. Tuy nhiên, không đáng tin cậy hoàn toàn ở lớp `three_wheeler` (tỷ lệ xóa lên tới 57.1% do box lỏng và nhận nhầm) và các vật nhỏ ở xa; người duyệt bắt buộc phải chủ động bổ sung vật sót thay vì ỷ lại vào máy.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quy tắc ẩn nhãn (Blind Scan First):** Luôn bấm ẩn toàn bộ nhãn máy (phím tắt ẩn/hiện hoặc icon mắt trên CVAT) trong 30 giây đầu mở ảnh; quét mắt từ trái sang phải đếm nhẩm số lượng người/xe thực tế trước khi bật lại box máy để tránh bị "neo nhãn".
  2. **Quy tắc rà soát 4 cạnh và tách người lái (4-Edge & Rider Separation Audit):** Với mọi box máy đưa ra, bắt buộc kiểm tra 4 cạnh viền xem có hở nền > 5% (bẫy loose) không; luôn tách riêng box `person` độc lập khỏi xe theo đúng quy tắc 4.1.4, không để box xe bao trùm luôn cả người lái.
  3. **Quy tắc lọc lớp đơn lẻ trước khi chốt (Class-by-Class Sweep):** Sử dụng bộ lọc Filters của CVAT bật/tắt từng lớp nhãn một (đặc biệt là `three_wheeler`, `bicycle`, `bus`) để rà soát lỗi nhầm lẫn nhãn chéo và xóa box trùng lặp trước khi bấm lưu và chuyển sang ảnh kế tiếp.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không (xe đỗ, người đứng cạnh được tách sạch) | Mask bám viền thân xe và bánh xe rất chuẩn; hơi lẹm nhẹ răng cưa ở bóng đổ gầm xe. | `masks/tuktuk_img004_sam2.jpg` |
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Có (dính một phần chân, mặt và cánh tay người lái) | Khung xe hở khiến mask lem vào người lái và người ngồi phía sau; cần thêm 2–3 click âm để tách triệt để. So với SAM 1 (`mobile_sam.pt`), SAM 2.1 bám biên kim loại sắc nét hơn, ít lem nền hơn. | `masks/tuktuk_img013_sam2.jpg` (kèm `masks/tuktuk_img013_sam1.jpg`) |
| img_017.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không (xe đỗ, không có người lái) | Bám sát viền bánh xe và khung thân; tuy nhiên mui vải có chữ vẽ graffiti bị khuyết một mảng do hoa văn bị hiểu nhầm là nền. | `masks/tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2:
  - **Quy tắc giúp ích:** Mục 4.2.1 (độ lệch biên $\le 3$ px) và 4.2.3 (bóng đổ không thuộc vật) giúp định hướng rõ vị trí đặt click âm để cắt bỏ bóng gầm xe.
  - **Quy tắc còn thiếu:** Chưa có quy định rõ về việc khoét rỗng các khoảng hở nhìn xuyên thấu ở xe ba bánh khung mở (mục 4.2.2 chỉ quy định chung cho lỗ thủng $< 5\%$ diện tích); chưa hướng dẫn xử lý khi mask xe tự động trùm lên người lái (trong khi mục 4.2.4 yêu cầu không chồng lấn polygon giữa 2 vật).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: So với script offline (`sam2_points.py`), Interactor trên giao diện CVAT cho phản hồi trực quan tức thì (real-time feedback) giúp ta click thêm điểm âm/dương để sửa vùng lem ngay lập tức, không phải ước lượng tọa độ pixel $(x, y)$ thủ công.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`calibration_decisions.csv`); quan trọng nhất: `CAL-01` (khắc phục lỗi closed-set của YOLO26 khi gán nhầm tuk-tuk thành `car`/`truck`, chuẩn hóa gán `three_wheeler` và tách người lái `person`) và `CAL-02` (chốt câu hỏi mở về phân loại xe điện hai bánh dựa trên sự hiện diện của bàn đạp cơ học).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  1. Phân biệt `bicycle` và `motorbike` điện dựa trên bàn đạp trợ lực cơ học (có bàn đạp $\rightarrow$ `bicycle`, thuần điện không bàn đạp $\rightarrow$ `motorbike`).
  2. Khối xe máy đỗ dày đặc trong đêm thiếu sáng không tách được biên chuyển sang gán `ignore_region` (áp dụng ở `img_026.jpg`).
  3. Xe ba bánh có mui (tuk-tuk/ba gác) luôn là `three_wheeler`, người ngồi trên xe vẽ `person` độc lập nếu thấy $\ge 20\%$, loại bỏ hoàn toàn các box trùng cắt cụt thân thể (`img_013.jpg`, `img_004.jpg`).
- Nguyên nhân gốc: Kết hợp cả ba yếu tố: `loi_may` do detector COCO đóng không nhận diện được xe ba bánh đặc thù Châu Á; `cau_hoi_mo` do guideline v1.0 chưa bao quát các phương tiện mới (xe điện); và `guideline mơ hồ` trong việc xử lý ranh giới khối xe đỗ đêm thiếu sáng.

## 7. QC và quyết định (Part 6)

- Mẫu: 7 ảnh (2 ảnh `can_review` + 5 ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt nhãn vừa là người kiểm QC nên có thể tồn tại điểm mù chủ quan).
- Lỗi theo **đối tượng**: 0/140 = 0.0% · theo **ảnh**: 0/7 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Không ghi nhận lỗi nào trong 7 ảnh mẫu kiểm định.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trong 7 ảnh mẫu, khoảng tin cậy Wilson 95% có cận trên lên tới **35%**. Điều này đồng nghĩa rằng trên mẫu nhỏ 7 ảnh, việc đạt 0.0% chưa đủ cơ sở thống kê để khẳng định tuyệt đối tỷ lệ lỗi thực tế toàn bộ lô dưới 2%.
- Script kiểm được / không kiểm được:
  - Script kiểm được: tính toàn vẹn file COCO, taxonomy, bounding box nằm trong ảnh, số lượng đối tượng, tỷ lệ bẫy bị chỉnh sửa, các công thức thống kê toán học.
  - Script không kiểm được: box có ôm khít từng pixel của vật thể thật hay không, nhãn có đúng bản chất thực tế hay không, các vật thể mà cả người duyệt lẫn mô hình cùng bỏ sót ngoài gold.

**Quyết định: Release**, ngày 2026-11-19.
Lý do (đi theo Hold → Rework → Release):
1. Không bị Hold vì các câu hỏi mở guideline và ca khó đã được chốt hoàn toàn trong v1.1, dữ liệu ảnh không có ảnh lỗi/hỏng;
2. Không bị Rework vì tỷ lệ ảnh lỗi đạt 0.0% <= 2%, không có lỗi hệ thống lặp lại, tỷ lệ bắt bẫy đạt 10/12 = 83.3% >= 80%;
3. Do đó đủ điều kiện thông qua quyết định **Release** bản nhãn.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng trợ lý AI Antigravity để hỗ trợ chạy kịch bản kiểm tra tự động, phân tích nguyên nhân kỹ thuật lỗi bẫy loose trên `img_013.jpg`, định dạng báo cáo markdown và soát lỗi submission; toàn bộ nhãn gán trên CVAT và kết quả quan sát trên ảnh đều do học viên tự kiểm chứng và thực hiện bằng mắt.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không bị nhiễm (4 ảnh gold được gán tay hoàn toàn từ đầu trên CVAT trước khi chạy bất kỳ mô hình pre-label nào).
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce GTX 1650 (CUDA), CVAT chạy qua Docker v2.81.0 (port 8080).
- Dùng cache `data/cache/` thay vì tự chạy? Không, tự chạy toàn bộ lệnh mô hình cục bộ trên máy cá nhân với GPU GTX 1650.
- Lỗi / bug script đã gặp (nếu có): Lệnh PowerShell ban đầu thiếu tiền tố `scripts/`; bẫy loose ở `img_013.jpg` cấy vào box cụt 63px nên khi người duyệt giữ lại box toàn thân 189px theo guideline thì IoU chỉ đạt 0.28 khiến script chấm nhầm là lọt bẫy.
- Một điều bạn làm khác lần sau: Luôn kiểm tra kỹ kích thước box gốc trước khi sửa bẫy nhân tạo và luôn tuân thủ nghiêm ngặt quy tắc ẩn nhãn (Blind Scan) ngay từ giây đầu tiên mở ảnh để tránh bỏ sót các bẫy missing.
