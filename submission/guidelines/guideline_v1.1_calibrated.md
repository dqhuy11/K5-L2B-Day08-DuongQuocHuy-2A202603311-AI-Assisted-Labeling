# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603311 - Dương Quốc Huy

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Còn tách được từng xe (thấy riêng bánh hoặc yên, cao ≥ 20 px) thì gán từng xe. Chỉ dùng `ignore_region` khi ≥ 5 xe dính nhau không tách được. | img_006 | CAL-04 |
| 5 | Kính chắn gió / cửa sổ / khung hở có thuộc mask xe không? | Kính và khung mui thuộc mask xe. Vùng nhìn xuyên qua khung hở mà thấy người hoặc vật khác là lỗ, phải loại khỏi mask xe. | img_013 | CAL-03 |
| 7 | Có đặt mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | Không đặt. `conf` chỉ để sắp thứ tự duyệt; mọi box vẫn phải nhìn ảnh. Ví dụ img_001 có box sai `truck` 0.88. | img_001, img_002 | CAL-05 |

Câu chưa chốt:
- **Câu 1** (xe đạp điện), **câu 3** (hàng nhô ra), **câu 6** (box chồng nhau ảnh đêm): em chưa gặp ảnh ví dụ rõ trong lô 24 ảnh batch để quyết định, nên giữ nguyên v1.0 và tick `can_review` nếu gặp.
- **Câu 4** (người ngồi trong xe ba bánh có mui): giữ ngưỡng ≥ 20% của v1.0. Ở img_013 em thấy hành khách ngồi trong tuk-tuk vẫn nhìn rõ, nên chưa có ca nào nằm sát ngưỡng để đổi quy tắc.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_001, img_002 | Xe bán tải có thùng hở | YOLO26m gọi `truck` (0.88 ở img_001, 0.73 ở img_002) | Gán `car` | Mục 2, lớp `car` / `truck`: bán tải luôn là `car`; `truck` chỉ khi buồng lái tách rời thùng hàng lớn (CAL-01) |
| img_002, img_032 | Xe đẩy hàng tay cạnh xe máy; nhiều xích lô đỗ sát nhau | YOLOE gán `three_wheeler` cho xe đẩy và xe bán tải (0.37–0.40); gộp 2 xích lô làm 1 box (0.27) | Xe đẩy tay không gán; mỗi xích lô một box | Mục 2, lớp `three_wheeler`: thêm "không gồm xe đẩy tay không có bánh thứ ba và không có người đạp / lái"; mục 4.1: xe đỗ sát nhau vẫn mỗi xe một box (CAL-02) |
| img_013 | Tuk-tuk khung hở, người lái và hành khách ngồi bên trong | SAM 2.1 với 1 click âm: mask nuốt người lái và người phía sau khung | Tách người ra khỏi mask xe | Mục 4.2.2: vùng nhìn xuyên khung hở là lỗ (CAL-03) |
| img_008, img_022, img_027 | Bẫy box lỏng (loose) | (bẫy nhân tạo) | Lỗi của người duyệt, không phải guideline mơ hồ | Không đổi guideline; thêm bước kiểm 4 cạnh box vào quy trình duyệt (CAL-06) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 5, 7 của mục 6. Thêm: bán tải = `car`; xe đẩy tay không phải `three_wheeler`; mỗi xe đỗ sát một box; vùng nhìn xuyên khung hở loại khỏi mask; `conf` không dùng để tự chấp nhận box. Quy trình duyệt thêm bước kiểm 4 cạnh. | CAL-01 đến CAL-06 |
