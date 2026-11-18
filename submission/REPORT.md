# Báo cáo Day 08 — 2A202603311 - Dương Quốc Huy

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.15 | Không có ngưỡng nào đạt recall 90%. Cao nhất là 25/31 = 81% ở 0.10 và 0.15. Em chọn 0.15 vì recall giữ nguyên mà precision tăng từ 0.50 lên 0.57. Để mặc định 0.3 thì recall còn 21/31 = 68%. (r01, r02: lên 0.5 mất 34% box person) |
| motorbike | YOLO26m | 0.2 | Recall cao nhất 11/14 = 79% ở 0.10, nhưng phải nhận thêm 10 box sai để được thêm 1 box đúng. Ở 0.2 recall 10/14 = 71%, precision 0.59. Để 0.3 thì recall chỉ còn 7/14 = 50%. (r02) |
| car | YOLO26m | 0.4 | Recall 7/8 = 88% không đổi từ 0.10 đến 0.5, nên em chọn 0.4 vì precision cao nhất (0.58). |
| bicycle / bus / truck | YOLO26m | 0.3 (mặc định) | Gold có 5 bicycle, 0 bus, 1 truck: quá ít nên không chọn ngưỡng riêng. |
| three_wheeler | YOLOE-26s-seg, prompt `three_wheeler` | 0.15 | Theo `threshold_sweep.md`: recall 7/7 = 100%, precision 7/9 = 78%. Ở 0.3 recall còn 5/7. YOLO26 không có lớp này (r01: tuk-tuk img_002 không có box), YOLOE bắt được (r03: 0.93). |

- Ứng viên lấy từ `work/prelabel/y26_all` (1669 box) và `work/prelabel/yoloe_all` (125 box), chạy conf 0.05 trên 36 ảnh.
- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,motorbike=0.2,car=0.4,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  - Kết quả 759 box cho 28 ảnh. Em để YOLOE đứng trước để box `three_wheeler` được giữ khi trùng với box YOLO26 (loại 6 `truck`, 6 `car`, 2 `bus`, 1 `bicycle`).
- **Phiên bản SAM:** SAM 2.1 base (`sam2.1_b.pt`). Mask bám viền khá tốt nhưng chạy 6.63 s/ảnh trên CPU (r04), nên em chỉ dùng cho vài vật cần polygon.
- **Lỗi không sửa được bằng đổi ngưỡng:**
  - Sai lớp: xe bán tải ở img_001 bị gọi `truck` 0.88 (box trùng 97% với box `car` trong gold). Tuk-tuk img_013 bị gọi `car` 0.42 (r04). Ở img_036, 3 xe đạp không có box `bicycle` nào khớp, máy chỉ bắt được người lái.
  - Thiếu lớp: YOLO26 không có `three_wheeler`.
  - Cách sửa: dùng YOLOE cho `three_wheeler`. Khi review phải kiểm tra lớp của các box `truck`/`car`, không chỉ xem box có đúng chỗ hay không.
- **Box điểm cao mà sai / điểm thấp mà đúng:**
  - Điểm cao mà sai: `truck` 0.88 trên xe bán tải img_001.
  - Điểm thấp mà đúng: `three_wheeler` của YOLOE ở khoảng 0.15–0.2 vẫn khớp gold (hạ ngưỡng từ 0.25 xuống 0.15 thì recall tăng từ 5/7 lên 7/7).
- **Giới hạn:** em chọn ngưỡng và chấm trên cùng 4 ảnh gold, nên các số trên là lạc quan. Mỗi lớp chỉ có 7–31 vật, nên lệch 1 vật là recall đổi vài điểm %. Ngoài ra có 6 người trong gold (img_020, img_036) mà máy không có box nào khớp kể cả ở conf 0.05, nên hạ ngưỡng cũng không lên được 90%.

## 2. AI error profile (từ `qc/review_stats.md`)

Số liệu tính trên `batch_reviewed.json` bản nộp.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 580 | 89.2% | 650 box máy |
| edited | 13 | 2.0% | 650 box máy |
| deleted | 53 | 8.2% | 650 box máy |
| added | 59 | 9.0% | 652 đối tượng sau duyệt |

Bảng đã bỏ các đối tượng quanh vùng bẫy (bẫy là lỗi nhân tạo, không phải lỗi của máy). Trong deleted + added có 12 ca là đổi lớp, nên số vật máy thực sự bỏ sót khoảng 59 − 12 = 47.

- **Lớp máy làm kém nhất:**
  - `three_wheeler`: chỉ giữ nguyên 13/39 box (22 xoá, 4 sửa). Do em đặt ngưỡng YOLOE thấp (0.15) để không sót xe, nên có nhiều box thừa. Ở r03 đã thấy YOLOE vẽ box lên xe bán tải và xe đẩy (img_002).
  - `truck`: chỉ giữ 1/6 box (5 xoá, 4 thêm), phần lớn là xe bán tải bị gọi nhầm như img_001.
  - `car`: phải thêm 18 box trên tổng 60 box sau duyệt.
- **So với gold:** precision của máy 50/85 = 58.8% (KTC 95%: 48–69%), recall 50/66 = 75.8% (KTC 95%: 64–84%), trên 66 đối tượng gold. Số này lạc quan vì ngưỡng chọn trên chính các ảnh này.
- accepted 89.2% không có nghĩa là máy đúng 89.2%: trên gold máy vẫn sót 16/66 vật.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy (bản nộp): 9/12 = 75.0%** (KTC 95%: 47–91%), chưa đạt ngưỡng 80%.
  - Theo loại: extra 3/3, wrong_class 2/3, loose 2/3, missing 2/3.
  - Còn lọt: img_023 (wrong_class), img_027 (loose), img_035 (missing).
- **Quá trình duyệt:** em duyệt nhiều vòng, tổng cộng 30 phút.
  - Lần export đầu tiên em chỉ bắt được 5/12 (loose 0/3, missing 1/3, wrong_class 2/3, extra 2/3).
  - Sau khi xem danh sách bẫy lọt trong `review_stats.md`, em duyệt lại và export lại: lần sau 7/12, bản cuối 9/12.
  - Các bẫy bắt thêm (img_008, img_022, img_013, img_028) đều nằm trong danh sách lọt em đã xem. Vì vậy con số phản ánh đúng lần duyệt đầu là 5/12. 9/12 là kết quả sau khi đã biết ảnh nào có lỗi.
- **Vì sao lọt:**
  - Loose lọt cả 3 ở lần đầu: em chưa kiểm kỹ 4 cạnh của từng box, chỉ xem box có nằm trên vật hay không. Đến bản cuối, dù biết img_027 có bẫy em vẫn chưa tìm ra.
  - Missing lọt 2/3 ở lần đầu: em nhìn các box có sẵn trước, chưa quét cả ảnh để tìm vật chưa có box. img_035 vẫn lọt ở bản cuối.
  - Wrong_class lọt ở img_023: em chưa soát kỹ lớp của từng box.
  - Bản nộp vẫn còn 13 cảnh báo box cao < 20 px (`validate_labels`): em giữ lại box máy mà chưa áp quy tắc 4.1.6.
- **Accepted cao mà bắt bẫy thấp:** lần đầu em giữ khoảng 92% box máy mà chỉ bắt được 5/12 bẫy. Nghĩa là em đã tin box máy nhiều hơn mức nên tin (automation bias). 30 phút cho khoảng 670 đối tượng là khoảng 2.7 giây một đối tượng, không đủ thời gian để kiểm kỹ từng box.

## 4. Máy có đáng dùng không?

- **Thời gian** (`time_log.csv`): gán tay 45/4 = 11.3 phút/ảnh, duyệt 30/24 = 1.25 phút/ảnh (tính cả các vòng duyệt lại).
  - Em gán gold trước nên lúc duyệt đã quen cảnh và quy tắc, nên duyệt nhanh hơn một phần là nhờ vậy.
  - Duyệt nhanh cũng một phần vì duyệt chưa kỹ (lần đầu chỉ bắt 5/12 bẫy). Nếu duyệt đủ kỹ để đạt QC thì thời gian sẽ lâu hơn.
- **Kết luận:** có nên dùng, nhưng phải duyệt kỹ hơn, và tùy lớp.
  - `person`, `motorbike` dùng được: giữ nguyên 355/373 và 163/174 box máy.
  - `three_wheeler` (13/39) và `truck` (1/6) thì chỉ nên coi là gợi ý vị trí, phải kiểm lớp từng box.
  - Bắt bẫy 9/12 vẫn dưới 80%, nên lô này cần **Rework**.
- **Ba quy tắc giảm thiên kiến của em:**
  1. Đếm số vật trong ảnh trước rồi mới xem box máy. Kiểm tra: số vật đếm được khớp số box sau duyệt.
  2. Phóng to kiểm 4 cạnh mọi box cao từ 100 px trở lên. Kiểm tra: lần duyệt đầu ở lô sau bắt được cả 3 bẫy loose.
  3. Lọc riêng từng lớp `three_wheeler`, `truck`, `car` để kiểm lớp, và chạy `validate_labels.py` trước khi export cho đến khi không còn cảnh báo box < 20 px.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base | Lần 1: 2 dương + 1 âm. Lần 2: 2 dương + 6 âm. Lần 3 (chọn): 4 dương + 4 âm | Lần 1 dính cả người lái và người phía sau khung xe. Lần 3 đã tách được phần lớn | Lần 1 mask tràn ra 45.4% ảnh. Lần 2 mất thân xe và bánh (12.4%). Lần 3 (38.9%) còn dính tay, chân người lái và lấn sang xe máy đỗ phía sau | `tuktuk_img013_sam2.jpg`, `tuktuk_img013_sam2_neg6.jpg`, `tuktuk_img013_sam2_v3.jpg` |
| img_004 | SAM 2.1 base | 2 dương + 1 âm | Không có người lái, người đứng cạnh không bị dính | Mép dưới sát mặt đường hơi răng cưa, có lỗ nhỏ ở chỗ bóng dưới gầm (13.3% ảnh) | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2.1 base | 2 dương + 1 âm | Không có người lái, ghế trong xe tính là một phần xe | Mask gọn, đủ mui, thân, bánh (12.1% ảnh). Điểm âm đặt trên mui không có tác dụng | `tuktuk_img017_sam2.jpg` |

- Em chạy bằng `scripts/sam2_points.py` trên CPU. Em không so sánh với MobileSAM vì không tải thêm `mobile_sam.pt`.
- **Đối chiếu guideline mục 4.2:**
  - Quy tắc 4.2.4 (hai vật chạm nhau thì mỗi vật một polygon) giúp em biết phải tách người lái ra khỏi mask xe bằng click âm. Mục 2 (mui gắn liền thuộc về xe) giúp em biết phải click dương lên mui.
  - Còn thiếu: quy tắc 4.2.2 về lỗ thủng không nói rõ người hoặc đồ vật nhìn thấy qua khung xe hở có phải bỏ ra khỏi mask không. Ở img_013 đây là chỗ mask bị lem nhiều nhất. Em đề xuất thêm vào v1.1: phần nhìn xuyên qua khung xe ba bánh mà có người hoặc vật khác thì bỏ ra khỏi mask xe.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: em không thử.

## 6. Calibration (Part 6)

- **Số quyết định:** 6 (`qc/calibration_decisions.csv`): 2 `loi_may`, 3 `cau_hoi_mo`, 1 `bay_lot`.
  - Quan trọng nhất là **CAL-01** (xe bán tải luôn là `car`). Lỗi này lặp ở nhiều ảnh, có điểm cao (`truck` 0.88 ở img_001), và đổi ngưỡng không sửa được. Trong review, `truck` chỉ giữ được 1/6 box máy.
  - Tiếp theo là **CAL-02** (xe đẩy tay không phải `three_wheeler`): `three_wheeler` là lớp bị xoá nhiều nhất (22/39 box máy).
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:**
  - Bán tải = `car`, `truck` chỉ khi buồng lái tách rời thùng hàng lớn.
  - Xe đẩy tay không có bánh thứ ba và không có người đạp / lái thì không phải `three_wheeler`. Xe đỗ sát nhau vẫn mỗi xe một box.
  - Chốt câu 2 (xe đỗ dày gần camera: tách được thì gán từng xe), câu 5 (vùng nhìn xuyên khung hở loại khỏi mask), câu 7 (không có ngưỡng `conf` tự chấp nhận).
  - Câu 1, 3, 6 chưa chốt vì em chưa gặp ảnh ví dụ rõ trong lô.
- **Nguyên nhân gốc:**
  - **Do máy:** CAL-01, CAL-02. YOLO26 không có lớp xe ba bánh và hay nhầm bán tải; YOLOE với ngưỡng thấp sinh box thừa.
  - **Do guideline mơ hồ:** CAL-03, CAL-04, CAL-05 (các câu hỏi còn mở ở mục 6 của v1.0).
  - **Do người duyệt (em):** CAL-06, bẫy loose lọt vì em không kiểm 4 cạnh box. Guideline đã đủ rõ nên không sửa guideline.

## 7. QC và quyết định (Part 6)

- **Mẫu:** 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035. Tổng 126 đối tượng.
  - Không có ảnh `can_review` vì bản nộp không có box nào được tick `can_review`. Đây cũng là một thiếu sót khi duyệt.
  - Người kiểm: em, có dùng AI hỗ trợ soi ảnh (mục 8). **Hạn chế:** em vừa là người duyệt vừa là người kiểm, nên dễ bỏ qua đúng những lỗi mình đã bỏ qua lúc duyệt.
- **Lỗi theo đối tượng:** 3/126 = 2.4%. **Theo ảnh:** 3/5 = 60.0% (ngưỡng Release ≤ 2%).
- **Top lỗi:**
  - Thừa (2): img_031 #576 (box `person` 0.15 nằm trên quần áo treo, không có người); img_014 #168 (`person` cao 19.3 px < 20 px, theo 4.1.6 không vẽ).
  - Bỏ sót (1): img_035, theo `review_stats.md` bẫy missing ở ảnh này vẫn lọt. Em chưa xác định được vị trí trên overlay.
- **Giới hạn mẫu nhỏ:** 3/5 ảnh lỗi thì cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **88%** (`qc_report_auto.md`). Kể cả mẫu sạch 0/5 cũng không chứng minh được tỉ lệ lỗi ≤ 2%. 5 ảnh chỉ đủ để phát hiện lô có vấn đề, không đủ để khẳng định lô đạt.
- **Script kiểm được / không kiểm được:**
  - Kiểm được: box cao < 20 px (`validate_labels`, bản nộp còn 13 cảnh báo), box cùng lớp trùng nhau, schema COCO, đủ 24 ảnh.
  - Không kiểm được: box có ôm sát vật không (loose), vật mà cả máy lẫn người cùng bỏ sót, sai lớp khi box đúng chỗ. Những lỗi này chỉ thấy khi nhìn ảnh. Ví dụ là bẫy img_027 (loose) và img_035 (missing), vẫn lọt cả sau khi duyệt lại.

**Quyết định: Rework**, ngày 2026-11-18.

Lý do, đi theo thứ tự Hold → Rework → Release:
- **Không Hold:** các câu hỏi guideline gặp trong lô đã chốt ở v1.1 (câu 2, 5, 7). Câu 1, 3, 6 không gặp ảnh ví dụ. Không có ảnh hỏng.
- **Rework:** ảnh có lỗi 3/5 = 60% > 2%. Bắt bẫy 9/12 = 75% < 80% (lần đầu chỉ 5/12). Lỗi lặp ở `three_wheeler` và `truck`.

Việc cần làm tiếp:
1. Sửa trên CVAT các lỗi đã thấy (img_031 #576, img_014 #168) và tìm vật bị sót ở img_035. Áp quy tắc v1.1 (bán tải → `car`, xoá box `three_wheeler` trên xe đẩy tay). Export `submission/labels/qc_fixed.json` rồi chạy lại `build_release.py`.
2. Duyệt lại cả lô theo checklist 4 bước, đặc biệt kiểm 4 cạnh box và quét vật thiếu box. Xoá 13 box < 20 px. Tick `can_review` cho ca khó.
3. Rút mẫu QC mới, nên lớn hơn 5 ảnh, và nhờ người khác kiểm.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude Code, hỗ trợ viết báo cáo, lý giải lý thuyết và chạy script; có sửa 8 box < 20 px trong gold theo quy tắc 4.1.6. Không dùng AI để gán gold hay review batch.
- **Gold có bị nhiễm không?** Không. Em gán tay 4 ảnh gold trên CVAT trước khi chạy model (Pha 2 chạy sau khi đã có `gold_manual.json`).
- **Máy / GPU / CVAT:** Windows 11, chỉ CPU (không GPU NVIDIA), Python 3.12, torch 2.14 CPU, ultralytics 8.4. CVAT chạy bằng Docker tại `http://localhost:8080`.
  - Vướng mắc: lúc đầu tạo Task `batch-review` không nằm trong Project nên import báo lỗi `Label 'person' is not registered for this task`. Em đã chuyển Task vào Project rồi import lại.
- **Dùng cache `data/cache/`?** Không. Em tự chạy YOLO26m và YOLOE trên 36 ảnh (khoảng 1 s/ảnh và 0.5 s/ảnh trên CPU).
- **Lỗi / bug script đã gặp:** trên PowerShell, script in tiếng Việt bị `UnicodeEncodeError` (bảng mã cp1252). Sửa bằng cách đặt `PYTHONUTF8=1` và `PYTHONIOENCODING=utf-8` trước khi chạy.
- **Một điều em làm khác lần sau:** duyệt chậm và theo đúng checklist ngay từ lần đầu: đếm vật trước, kiểm 4 cạnh từng box, lọc theo lớp. Lần này em duyệt nhanh (30 phút) nên lần đầu chỉ bắt 5/12 bẫy, sau đó phải duyệt lại.
