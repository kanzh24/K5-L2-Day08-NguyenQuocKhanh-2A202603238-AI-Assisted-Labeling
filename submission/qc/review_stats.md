# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 566; đối tượng sau duyệt: 513; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 425 | 425/566 = 75.1% | 566 box máy |
| edited (sửa) | 28 | 28/566 = 4.9% | 566 box máy |
| deleted (xoá) | 100 | 100/566 = 17.7% | 566 box máy |
| added (vẽ thêm) | 56 | 56/513 = 10.9% | 513 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 46.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 243 | 7 | 39 | 14 |
| bicycle | 4 | 0 | 4 | 6 |
| motorbike | 118 | 13 | 19 | 18 |
| car | 44 | 4 | 8 | 10 |
| bus | 2 | 0 | 1 | 1 |
| truck | 0 | 0 | 5 | 2 |
| three_wheeler | 14 | 4 | 24 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 86 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 58 | 20 | 28 | 74.4% (KTC 95%: 64–83%) | 67.4% (KTC 95%: 57–76%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 3/3
  - missing: 1/3
  - extra: 3/3
  - lọt: img_024.jpg (missing), img_003.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
