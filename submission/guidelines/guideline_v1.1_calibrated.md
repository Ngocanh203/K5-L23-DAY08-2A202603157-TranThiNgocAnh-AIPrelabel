# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603157 - Trần Thị Ngọc Ánh

Bản này chỉ bổ sung, làm rõ guideline v1.0. Những quy tắc không đề cập vẫn áp dụng theo v1.0. Các ca tương ứng được ghi trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 5 | Kính chắn gió/cửa sổ và người nhìn thấy qua xe được xử lý thế nào trong mask? | Kính thuộc mask xe. Phần người nhìn thấy qua cửa hở không thuộc mask xe, được gán `person` riêng. Chỉ gán phần nhìn thấy, không suy đoán phần người bị che. | `img_013.jpg` | CAL-02 |

Các câu hỏi 1, 2, 3, 4, 6 và 7 của v1.0 chưa chốt trong phiên bản này vì chưa có đủ đối chiếu để xây dựng quy tắc cụ thể. Các ca liên quan tiếp tục đánh dấu `can_review` nếu chưa chắc quy tắc; không tự coi chúng đã được giải quyết.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_004.jpg`, `img_013.jpg` | Xe ba bánh có mui không được nhận diện đúng; box đầu vào thiếu dẫn đến thiếu mask xe. | YOLO26 không nhận đúng hoặc bỏ sót xe ba bánh trong lần chạy thử; SAM chỉ tạo mask cho box được cung cấp. | Xác định lớp theo ảnh và guideline; bổ sung vật sót, kiểm tra lớp và box trước khi tạo mask. | CAL-01 — Không đổi định nghĩa lớp ở mục 2. Bổ sung checklist: kiểm lớp và vật sót trước bước box → SAM. |
| `img_013.jpg` | Người lái ngồi trong phần cửa hở của tuk-tuk. | SAM 2.1 base còn tô lẫn người lái và xe máy phía sau dù đã bổ sung điểm âm. | Người và xe là các vật riêng; mask thử còn lẫn vật phải được ghi nhận là chưa đạt, không chấp nhận chỉ vì script chạy thành công. | CAL-02 — Làm rõ mục 4.2: kính thuộc xe; phần người nhìn thấy qua cửa hở không thuộc mask xe. |
| `img_031.jpg` | Vùng quần áo treo phía trên ảnh bị coi là người trong đáp án bẫy. | Box tham chiếu của bẫy `loose` nằm trên quần áo treo; xóa box này vẫn bị script tính là lọt. | Không tạo `person` cho quần áo trưng bày. Giữ nguyên đáp án và kết quả script; giải thích bất đồng giữa tham chiếu và ảnh thật trong báo cáo. | CAL-03 — Không đổi phạm vi lớp `person` ở mục 2. Bổ sung kiểm tra: phân biệt người thật với quần áo trưng bày; ghi nhận lỗi tham chiếu khi có bằng chứng. |
| `img_013.jpg`, `img_031.jpg` | Mask lẫn người, xe lân cận hoặc quần áo treo phía sau. | SAM 2.1 base chưa tách sạch các vật; tăng điểm âm chưa giải quyết hết lỗi. | Kiểm tra phần thừa và phần thiếu trên ảnh; dùng điểm âm để thử loại vùng thừa; nếu chưa chỉnh sạch thì ghi rõ hạn chế. | CAL-04 — Bổ sung checklist mục 4.2: rà vật lân cận, nền và phần vật bị mất. Số click, diện tích mask và số polygon không chứng minh mask đúng. |

### Kiểm tra mask sau khi chạy SAM

1. Xác định đúng vật cần phân vùng trước khi chọn điểm.
2. Kiểm tra mui, thân, bánh xe hoặc các phần nhìn thấy của vật có bị mất không.
3. Kiểm tra mask có lẫn người, phương tiện khác hoặc nền không.
4. Đối chiếu khoảng hở lớn và phần bị che với mục 4.2 của v1.0.
5. Ghi phiên bản SAM, số điểm dương/âm và lỗi còn lại.
6. Phân biệt ảnh thử nghiệm với nhãn cuối đã được kiểm tra chất lượng.

Các ảnh mask trong `submission/masks/` là bằng chứng thử nghiệm. Mask `img_013` vẫn còn lẫn người lái và xe phía sau; mask `img_031` còn dính quần áo treo. Không coi các kết quả này là mask cuối đã đạt chất lượng.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên. | — |
| v1.1 | Thêm bước kiểm lớp và vật sót trước khi tạo mask từ box. | CAL-01: YOLO26 thiếu hoặc sai lớp xe ba bánh. |
| v1.1 | Làm rõ kính xe và phần người nhìn thấy qua cửa hở trong mask. | CAL-02: người lái tuk-tuk ở `img_013.jpg`. |
| v1.1 | Thêm kiểm tra vật trưng bày và ghi nhận bất đồng với đáp án bẫy. | CAL-03: quần áo treo ở `img_031.jpg`. |
| v1.1 | Bổ sung checklist kiểm biên mask và phân biệt kết quả thử với nhãn đạt chất lượng. | CAL-04: lỗi mask ở `img_013.jpg`, `img_031.jpg`. |