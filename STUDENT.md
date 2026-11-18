# Thông tin học viên

- Họ và tên: Dương Quốc Huy
- MSHV: 2A202603311
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: dqhuy11
- Repo bài nộp (URL): https://github.com/dqhuy11/K5-L2B-Day08-DuongQuocHuy-2A202603311-AI-Assisted-Labeling
- Commit chốt bài (hash): cc7f542
- Máy: Windows 11, không GPU (chạy CPU)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 580 / 13 / 53 (mẫu số: 650 box máy); added: 59 (mẫu số: 652 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 50/85 = 58.8% / 50/66 = 75.8%
- Bắt bẫy: 9/12 = 75.0% (lần duyệt đầu 5/12)
- Thời gian: gán tay 11.3 phút/ảnh (45/4); duyệt có pre-label 1.25 phút/ảnh (30/24)
- Lỗi QC: 60.0% theo ảnh (3/5), 2.4% theo đối tượng (3/126)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 3/5 > 2% và bắt bẫy 9/12 < 80%, lỗi lặp ở `three_wheeler` và `truck`.
