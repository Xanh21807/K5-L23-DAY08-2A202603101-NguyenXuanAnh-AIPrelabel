# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603101 - Nguyễn Xuân Anh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy / bẫy lọt / review thực tế. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện: `bicycle` hay `motorbike`? | Mọi phương tiện 2 bánh có động cơ (kể cả xe máy điện VinFast, xe đạp điện mini) đều xếp vào `motorbike`. Chỉ gán `bicycle` cho xe đạp thuần cơ học đạp bằng chân. | img_003.jpg, img_024.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Nếu xe đỗ ken dày bị che khuất > 80% hoặc không thể xác định ranh giới độc lập thì bao quanh cả khối bằng `ignore_region`. Chỉ vẽ từng xe nếu nhìn thấy rõ đầu xe, yên hoặc bánh xe riêng biệt. | img_005.jpg, img_018.jpg | CAL-03 |
| 3 | Hàng hoá cồng kềnh chở trên xe (thùng hàng, sọt tre, bó sắt): tính vào xe hay không? | Bounding box của phương tiện phải bao trọn cả phần hàng hoá được chở gắn liền trên giá/yên xe. Với mask phân đoạn, mask ôm cả thân xe và kiện hàng cố định. | img_009.jpg, img_022.jpg | CAL-01 |
| 4 | Người ngồi trong xe ba bánh / tuk-tuk nhìn xuyên qua mui che: gán thế nào? | Nếu nhìn thấy rõ đầu và vai hoặc ≥ 20% cơ thể hành khách/tài xế qua khung hở thì gán `person`. Nếu chỉ thấy mờ ảo dưới 20% thì không gán để tránh box nhiễu. | img_013.jpg, img_004.jpg | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_004.jpg, img_013.jpg | Xe tuk-tuk và xe ba gác có mui che | YOLO26 gọi là `car` hoặc `truck` | Sửa lại thành `three_wheeler` | Mục 2: Bổ sung định nghĩa `three_wheeler` bao gồm toàn bộ xe 3 bánh cơ giới hoặc thô sơ (tuk-tuk, ba gác, xích lô chở khách). |
| img_027.jpg, img_013.jpg | Bẫy lọt bỏ sót (missing) người/xe ở rìa hoặc khuất xa | Pre-label cố tình xoá box để bẫy người duyệt | Cần quét rìa ảnh trước khi tin nhãn máy | Quy trình review: Bắt buộc ẩn nhãn máy và quét ảnh độc lập theo 4 góc biên ảnh trước khi chấp nhận. |
| img_029.jpg, img_033.jpg | Bẫy lọt box lỏng (loose) bao rộng thừa nền | Pre-label cố tình nới rộng box gấp đôi vật | Co hẹp box sát mép ngoài cùng của vật thật | Mục 4.1: Bounding box phải ôm khít biên giới vật thể, sai số cạnh không vượt quá 5% kích thước đối tượng. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | Phiên bản cơ sở ban đầu |
| v1.1 | Quy định xe điện vào `motorbike`; chuẩn hóa cụm xe máy dùng `ignore_region`; định nghĩa khép kín `three_wheeler` cho tuk-tuk/ba gác; siết quy tắc bao khít và rà soát bẫy lọt | Giải quyết các ca CAL-01, CAL-02, CAL-03, CAL-04 sau đợt review 24 ảnh batch |
