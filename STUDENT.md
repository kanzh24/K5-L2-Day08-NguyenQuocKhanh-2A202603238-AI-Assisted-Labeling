# Thông tin học viên

- Họ và tên: Nguyễn Quốc Khánh
- MSHV: 2A202603238
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: kanzh24
- Repo bài nộp (URL): https://github.com/kanzh24/K5-L2-Day08-NguyenQuocKhanh-2A202603238-AI-Assisted-Labeling
- Commit chốt bài (hash): 
- Máy: Windows 11, GPU NVIDIA GeForce GTX 1650 (CUDA)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 425/566 = 75.1% / 28/566 = 4.9% / 100/566 = 17.7% (mẫu số: 566 box máy); added: 56/513 = 10.9% (mẫu số: 513 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 74.4% (58/78) / 67.4% (58/86) (mẫu số: 86 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 5.0 phút/ảnh (20 phút / 4 ảnh); duyệt có pre-label 3.75 phút/ảnh (90 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/7), 0.0% theo đối tượng (0/140)
- **Quyết định:** Release — lý do một dòng: Tỉ lệ ảnh lỗi 0.0% <= 2%, không có lỗi hệ thống, bắt bẫy đạt 83.3% >= 80% và đã giải quyết toàn bộ ca khó trong guideline v1.1.
