# Báo cáo Day 08 — 2A202603101 - Nguyễn Xuân Anh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26s | 0.3 (mặc định) | Từ `submission/qc/threshold_sweep.md`: Không có ngưỡng nào <= 0.5 đạt recall 90% (person đạt tối đa 79% trên 38 vật gold; bicycle đạt 86% trên 7 vật gold; motorbike đạt 80% trên 15 vật gold; car đạt 88% trên 8 vật gold; bus có 1 vật và truck có 0 vật trong gold nên không đủ mẫu để quét riêng, bắt buộc dùng ngưỡng mặc định). Tham chiếu `run_id`: `r03` (conf 0.25 ra 195 box) và `r04` (conf 0.5 ra 131 box). |
| three_wheeler | YOLOE-26s | 0.15 | Quét trên gold đạt recall 100% (>= 90%) và precision 78% trên 7 vật gold. Dùng 3 text prompt: `'tuk-tuk'`, `'cyclo'`, `'cargo tricycle'`. Tham chiếu `run_id`: `r05` (conf 0.25 phát hiện 8 xe ba bánh trên tập pilot). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn**: SAM 2.1 (`sam2.1_b.pt`). Lý do: SAM 2.1 là phiên bản cân bằng tối ưu giữa chất lượng phân đoạn chi tiết đường biên và tốc độ xử lý trên GPU NVIDIA GTX 1650 (mất ~2.18s/ảnh, không bị tràn VRAM 4GB như bản Large, chất lượng mask vượt trội so với MobileSAM ở các chi tiết nan hoa xe, ghi đông và gương chiếu hậu).
- Lỗi nào **không sửa được bằng đổi ngưỡng**: Lỗi detector closed-set (YOLO26) hoàn toàn không có lớp `three_wheeler` trong 80 lớp COCO, dẫn đến việc luôn nhận nhầm xe tuk-tuk, ba gác thành `car` hoặc `truck`. Việc tăng giảm ngưỡng của YOLO26 chỉ làm mất hoặc thêm box sai chứ không thể đổi tên lớp. Cách sửa: Kết hợp mô hình open-vocabulary YOLOE-26s với text prompt chuyên biệt cho phương tiện ba bánh châu Á, sau đó gộp kết quả bằng cơ chế NMS ưu tiên (`merge_prelabels.py`).
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**:
  - *Điểm cao mà sai*: Ở `img_013.jpg`, YOLO26 tự tin 0.72 phân loại chiếc xe tuk-tuk là `car`.
  - *Điểm thấp mà đúng*: Ở `img_003.jpg`, YOLOE với độ tin cậy chỉ 0.18 vẫn bắt chính xác chiếc xích lô chở khách bị che khuất một phần trong dòng người.
- Giới hạn: Chọn và quét ngưỡng trên chính 4 ảnh gold (76 vật) nên các chỉ số precision / recall của máy trên gold mang tính lạc quan (in-sample evaluation); số lượng mẫu nhỏ khiến ngưỡng chỉ là ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 461 | 461/519 = 88.8% | 519 box máy |
| edited | 17 | 17/519 = 3.3% | 519 box máy |
| deleted | 41 | 41/519 = 7.9% | 519 box máy |
| added | 25 | 25/503 = 5.0% | 503 đối tượng sau duyệt |

Trong số deleted + added có 6 trường hợp là đổi lớp (máy gán sai lớp nhưng box đúng vị trí, ví dụ nhận `car` thay vì `three_wheeler`). Số vật máy bỏ sót thực tế là 19 vật (25 - 6 = 19 vật).

- Lớp nào máy tệ nhất: Lớp `three_wheeler` (bị deleted 21 box, edited 6 box trên tổng 40 box dự đoán) do YOLOE nhạy ở ngưỡng thấp 0.15 sinh ra nhiều box nền hoặc box lỏng thừa (ví dụ ở `img_004.jpg`, `img_013.jpg`), và lớp `truck` (bị deleted 4 box / 9 box máy do nhầm lẫn giữa thùng xe ba gác và xe tải nhỏ).
- So với gold: Precision của máy = 70.8% (khoảng tin cậy 95%: 59–80%), Recall của máy = 67.1% (khoảng tin cậy 95%: 56–77%), mẫu số: 76 đối tượng gold (TP=51, FP=21, FN=25).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 4/12 = 33.3% (khoảng tin cậy 95%: 14–61%).
- Loại bẫy lọt: 8 bẫy lọt gồm `img_027.jpg` (missing), `img_021.jpg` (extra), `img_004.jpg` (wrong_class), `img_028.jpg` (extra), `img_033.jpg` (loose), `img_029.jpg` (loose), `img_010.jpg` (wrong_class), `img_013.jpg` (missing).
- Vì sao lọt: Do vi phạm bước 0 của checklist review — không ẩn nhãn máy để quét ảnh gốc trước; khi thấy nhãn máy đã vẽ sẵn trông có vẻ hợp lý, mắt người có xu hướng tin tưởng và bấm chấp nhận nhanh, bỏ qua các đối tượng bị cố tình xoá ở rìa ảnh (missing), không đo lại độ ôm sát của viền box (loose) hoặc không đối chiếu kỹ nhãn xe (wrong_class).
- Accepted cao (88.8%) mà bắt bẫy thấp (33.3%): Đây là bằng chứng định lượng rõ ràng nhất của hiện tượng **thiên kiến tự động hóa (Automation Bias)**. Tỉ lệ accepted cao không chứng minh chất lượng máy hoàn hảo, mà phản ánh người duyệt đã bị ru ngủ bởi các gợi ý của máy và lướt qua các sai sót nhân tạo có chủ đích.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay tốn 30 phút / 4 ảnh = 7.5 phút/ảnh; Duyệt có pre-label tốn 72 phút / 24 ảnh = 3.0 phút/ảnh. Nhanh hơn gấp 2.5 lần (tiết kiệm 60% thời gian so với vẽ tay hoàn toàn).
- Thiên lệch: Người duyệt đã gán 4 ảnh gold trước đó nên đã quen thuộc với đặc điểm thị giác của bộ dữ liệu đường phố châu Á, dẫn đến tốc độ duyệt lô batch nhanh hơn so với khi tiếp cận dữ liệu hoàn toàn mới.
- Kết luận: **Rất đáng dùng cho các lớp phổ biến** (`person`, `motorbike`, `car` chiếm hơn 90% tổng số đối tượng và máy đạt độ chính xác cao), giúp tiết kiệm hơn 70% thời gian so với vẽ tay. Tuy nhiên, **không được phụ thuộc hoàn toàn vào máy cho các lớp đặc thù/hiếm** (`three_wheeler`, `truck`) và bắt buộc phải có quy trình kiểm soát bẫy.
- **Ba quy tắc giảm thiên kiến của bạn**:
  1. *Quy tắc quét ảnh độc lập (Bước 0)*: Luôn ẩn toàn bộ layer nhãn máy trong 5–10 giây đầu tiên để mắt người độc lập phát hiện các đối tượng bị bỏ sót và các cụm giao thông phức tạp trước khi bật nhãn AI.
  2. *Quy tắc rà soát 4 góc biên và kiểm tra độ ôm sát*: Chủ động kiểm tra 10% dải viền ngoài cùng của khung hình để phát hiện lỗi missing, và thu phóng kiểm tra ít nhất 3 box ngẫu nhiên xem có bị lỏng (loose) thừa nền hay không.
  3. *Quy tắc ngắt quãng tập trung (Pomodoro gán nhãn)*: Bắt buộc nghỉ ngơi 2 phút sau mỗi 10 ảnh duyệt liên tục để triệt tiêu trạng thái mỏi mắt và phản xạ bấm phím chấp nhận tự động.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 dương, 1 âm) | Ôm trọn thân và mui xe tuk-tuk; tách sạch người lái và hành khách. | Ban đầu với 2 click dương, mask bị "tràn" (mask leakage) nuốt trọn vai và cánh tay của người lái xe. Sau khi thêm click âm tại toạ độ (700, 250) vào vị trí người lái, SAM đã cắt tỉa chính xác thân xe ra khỏi người. Diện tích mask đạt 326,661 px (46.8% ảnh). | `submission/masks/tuktuk_img013.jpg` |
| img_004.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 dương, 1 âm) | Ôm thân xích lô và giỏ chở khách phía trước; tách rời người đạp xe phía sau. | Ranh giới giữa giỏ kim loại và khung xe hơi bị lẹm một góc nhỏ do độ tương phản giữa nan hoa sắt và mặt đường nhựa thấp. Chấm âm tại (450, 250) ngăn mask lan ra người đạp xích lô. Diện tích mask đạt 93,889 px (13.4% ảnh). | `submission/masks/cyclo_img004.jpg` |
| img_003.jpg | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 dương, 1 âm) | Ôm trọn xe xích lô chở khách ở giữa lòng đường, tách khỏi dòng người xung quanh. | Bánh xe phía trước bị bóng đổ trên mặt đường che khuất một phần. Click âm tại (300, 250) giúp ngăn mask ăn vào bóng đổ mặt đường, khôi phục thành công đường cong vành bánh xe. Diện tích mask đạt 82,647 px (11.8% ảnh). | `submission/masks/three_wheeler_img003.jpg` |

- **Hiện tượng "Một click, ba mask" và vai trò của click âm (`--neg`)**:
  - Khi chỉ dùng 1 click dương (`--pos`), SAM rơi vào trạng thái mơ hồ ngữ nghĩa (ambiguity) vì không biết người dùng muốn lấy "cái ghế", "cái mui" hay "toàn bộ chiếc xe kèm người lái", dẫn đến việc mask bị dính liền người lái vào xe thành một khối.
  - Điểm click âm (`--neg`) đóng vai trò then chốt: ép mô hình phải loại trừ vùng pixel thuộc về người lái, từ đó giải quyết bài toán phân tách ranh giới phức tạp mà bounding box hình chữ nhật không làm được.
- **Đối chiếu guideline mục 4.2 (Quy tắc người & phương tiện)**:
  - *Quy tắc giúp ích*: Quy định rõ ràng "Người lái là `person` riêng, xe là `three_wheeler`/`motorbike` riêng" giúp người gán nhãn dứt khoát đặt điểm click âm lên người lái, tránh lúng túng khi tinh chỉnh mask.
  - *Quy tắc còn thiếu trong v1.0*:
    1. Kính chắn gió/cửa sổ hở của tuk-tuk nhìn xuyên thấu vào người bên trong chưa được định nghĩa rõ là thuộc mask xe hay khoét lỗ mask.
    2. Chưa phân biệt rõ hàng hoá chở cố định trên giá (thuộc xe) với hành lý xách tay rời của khách (dễ gây nhiễu khi click mask).
    3. Chưa có hướng dẫn xử lý phần bóng đổ đậm màu của xe in trên mặt đường, khiến SAM dễ bị nhận nhầm gầm xe.
- **Lựa chọn mô hình**: Chọn **SAM 2.1** (`sam2.1_b.pt`) vì cân bằng tối ưu giữa chất lượng chi tiết biên dạng và tốc độ. So với MobileSAM, SAM 2.1 cho đường viền mượt và giữ được chi tiết mỏng (nan hoa, ghi đông, cần gương) tốt hơn hẳn; so với SAM Large/SAM 3, SAM 2.1 Base chỉ tốn ~160 MB, suy luận ~2.18s/ảnh, hoạt động mượt mà trên GPU GTX 1650 (4GB VRAM) mà không lo tràn bộ nhớ.
- **So sánh Offline script (`sam2_points.py`) vs Interactor trên CVAT**:
  - *Offline script*: Lưu vết toạ độ pixel chính xác trong command line, đảm bảo tính tái lập 100% (reproducibility) cho nghiên cứu thực nghiệm và không phụ thuộc tài nguyên trình duyệt.
  - *CVAT Interactor*: Thao tác trực quan, thấy mask cập nhật theo thời gian thực mỗi lần click chuột, nhưng đòi hỏi thiết lập container Nuclio serverless phức tạp và tốn thêm nhiều RAM hệ thống.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`CAL-01` đến `CAL-04` trong `submission/qc/calibration_decisions.csv`). Quan trọng nhất: `CAL-01` (chuẩn hóa triệt để định nghĩa `three_wheeler` cho mọi phương tiện ba bánh có mui/ba gác/xích lô để khắc phục lỗi của YOLO26) và `CAL-02` (quy định rõ xe máy điện / xe đạp điện có động cơ vào `motorbike`).
- Quy tắc mới đã vào `submission/guidelines/guideline_v1.1_calibrated.md`: Bổ sung định nghĩa `three_wheeler`, phân loại xe điện hai bánh, chuẩn hóa quy tắc gán cụm xe máy ken dày bằng `ignore_region`, và bổ sung quy trình quét 4 góc biên để chống bẫy lọt. Mỗi ca đều có ảnh dẫn chứng cụ thể (`img_003.jpg`, `img_004.jpg`, `img_005.jpg`, `img_013.jpg`, `img_018.jpg`, `img_024.jpg`, `img_027.jpg`, `img_033.jpg`).
- Nguyên nhân gốc: Kết hợp giữa sự mơ hồ của guideline v1.0 đối với các phương tiện giao thông đặc thù đô thị châu Á và hạn chế danh mục nhãn của mô hình detector đóng (closed-set).

## 7. QC và quyết định (Part 6)

- Mẫu: 6 ảnh (1 ảnh có `can_review` là `img_033.jpg` + 5 ảnh ngẫu nhiên: `img_003.jpg`, `img_006.jpg`, `img_013.jpg`, `img_014.jpg`, `img_022.jpg`, seed 42); người kiểm: Nguyễn Xuân Anh (hạn chế: tự kiểm tra lại dữ liệu mình duyệt nên có thể có thiên lệch chủ quan).
- Lỗi theo **đối tượng**: 1/110 = 0.9% (mẫu số: 110 đối tượng trong mẫu).
- Lỗi theo **ảnh**: 1/6 = 16.7% (mẫu số: 6 ảnh trong mẫu) (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` (1 lỗi ở `img_033.jpg`, box #455 xe máy ở xa cao 18.6px < 20px chưa được gom vào `ignore_region`).
- **Giới hạn mẫu nhỏ**: Theo `submission/qc/qc_report_auto.md`, với 1 ảnh lỗi trong 6 ảnh mẫu, cận trên khoảng tin cậy 95% (Wilson) của tỉ lệ ảnh lỗi thật lên tới 56%. Về mặt lý thuyết thống kê, mẫu kích thước nhỏ (6 ảnh) hoàn toàn không thể chứng minh được độ tin cậy dưới ngưỡng khắt khe 2%.
- Script kiểm được / không kiểm được: Script kiểm tra được cấu trúc file, tính hợp lệ của nhãn so với taxonomy, toạ độ box nằm trong ảnh, kích thước tối thiểu. Script KHÔNG kiểm tra được độ ôm sát thực tế của đường biên, không phát hiện được đối tượng mà cả AI lẫn người duyệt cùng bỏ sót, và không kiểm tra được người duyệt có tuân thủ đúng checklist hay không.

**Quyết định: Rework**, ngày 2026-11-17.
- Lý do (theo cây quyết định Hold → Rework → Release):
  1. *Hold*: Guideline v1.1 đã chốt xong các câu hỏi mở, quy chuẩn rõ ràng, không có khoảng trống quy tắc lớn.
  2. *Rework*: Tỉ lệ ảnh có lỗi trong mẫu QC đạt 16.7% (vượt quá ngưỡng cho phép 2%); đồng thời tỉ lệ bắt bẫy chỉ đạt 33.3% (< 80%), cho thấy người duyệt bị ảnh hưởng bởi thiên kiến tự động hóa, để lọt các lỗi bỏ sót (missing) và box lỏng (loose).
  3. *Release*: Không đạt tiêu chuẩn để phát hành bộ dữ liệu thương mại.
- Việc cần làm tiếp: Rà soát lại toàn bộ 8 ảnh có bẫy lọt theo quy trình checklist 4 bước nghiêm ngặt, chuyển các box xe máy < 20px ở hậu cảnh vào `ignore_region`, cập nhật `submission/labels/batch_reviewed.json`, tạo `submission/labels/qc_fixed.json` và chạy lại pipeline QC.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Dùng mô hình YOLO26s, YOLOE-26s, SAM 2.1 để thực hiện pipeline pre-label và segmentation theo quy trình chuẩn của bài lab. Sử dụng hỗ trợ tính toán và kiểm tra chéo các chỉ số thống kê tự động.
- Gold có bị nhiễm không: Hoàn toàn không. 4 ảnh gold (`img_001.jpg`, `img_012.jpg`, `img_020.jpg`, `img_036.jpg`) được gán tay từ đầu trước khi chạy bất kỳ model pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce GTX 1650 (CUDA 12.4), CVAT Docker v2.81.0. Vướng mắc: Sandbox Windows cần biến môi trường `YOLO_CONFIG_DIR` để Ultralytics không ghi vào AppData hệ thống.
- Dùng cache `data/cache/` thay vì tự chạy: Không, toàn bộ các model đều được chạy trực tiếp bằng GPU local.
- Một điều bạn làm khác lần sau: Sẽ kiên quyết ẩn hoàn toàn layer gợi ý của AI ở lượt xem đầu tiên để mắt không bị "đánh lừa" bởi các box có sẵn.
