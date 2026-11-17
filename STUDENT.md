# Thông tin học viên

- Họ và tên: Nguyễn Xuân Anh
- MSHV: 2A202603101
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Xanh21807
- Repo bài nộp (URL): https://github.com/Xanh21807/K5-L23-DAY08-2A202603101-NguyenXuanAnh-AIPrelabel
- Commit chốt bài (hash): d3f4723
- Máy: Windows 11, GPU CUDA (NVIDIA GeForce GTX 1650)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 452 / 25 / 42 (mẫu số: 519 box máy; 87.1% / 4.8% / 8.1%); added: 29 (mẫu số: 506 đối tượng; 5.7%)
- So với gold: precision / recall của **máy**: 70.8% (51/72) / 67.1% (51/76) (mẫu số: 76 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 7.5 phút/ảnh (30 phút / 4 ảnh); duyệt có pre-label 3.0 phút/ảnh (72 phút / 24 ảnh)
- Lỗi QC: 16.7% theo ảnh (1/6), 0.9% theo đối tượng (1/111)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh lỗi trong mẫu QC đạt 16.7% (> 2%), cần rà soát các box vật nhỏ < 20px đưa vào ignore_region.
