# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 650; đối tượng sau duyệt: 652; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 580 | 580/650 = 89.2% | 650 box máy |
| edited (sửa) | 13 | 13/650 = 2.0% | 650 box máy |
| deleted (xoá) | 53 | 53/650 = 8.2% | 650 box máy |
| added (vẽ thêm) | 59 | 59/652 = 9.0% | 652 đối tượng sau duyệt |

Trong số deleted + added, **12 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 47.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 355 | 4 | 14 | 16 |
| bicycle | 5 | 0 | 2 | 5 |
| motorbike | 163 | 4 | 7 | 9 |
| car | 41 | 1 | 2 | 18 |
| bus | 2 | 0 | 1 | 4 |
| truck | 1 | 0 | 5 | 4 |
| three_wheeler | 13 | 4 | 22 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 66 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 50 | 35 | 16 | 58.8% (KTC 95%: 48–69%) | 75.8% (KTC 95%: 64–84%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - loose: 2/3
  - missing: 2/3
  - extra: 3/3
  - lọt: img_035.jpg (missing), img_027.jpg (loose), img_023.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
