# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 7 ảnh (2 ca khó + 5 ngẫu nhiên), 140 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/140 = 0.0% | 140 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 0/7 = 0.0% | 7 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 425/566 = 75.1% / 28/566 = 4.9% / 100/566 = 17.7% | 566 box máy |
| Máy: added | 56/513 = 10.9% | 513 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Giới hạn thống kê
- 0 ảnh lỗi trong 7 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **35%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**RELEASE**

- ảnh lỗi 0.0% <= 2%, không lỗi hệ thống, bắt bẫy / nhất quán đạt (nếu đã đo)

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
