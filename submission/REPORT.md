# Báo cáo Day 08 — 2A202603157 - Trần Thị Ngọc Ánh

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Căn cứ |
|---|---|---|---|
| person | YOLO26n | 0.10 | Quét ngưỡng trên gold; đối chiếu r01 và r02 |
| bicycle | YOLO26n | 0.30 | Gold chỉ có 4 vật; giữ mặc định vì mẫu quá ít |
| motorbike | YOLO26n | 0.10 | Quét ngưỡng trên gold; đối chiếu r01 và r02 |
| car | YOLO26n | 0.45 | Quét ngưỡng trên gold |
| bus | YOLO26n | 0.30 | Gold không có vật; giữ mặc định |
| truck | YOLO26n | 0.30 | Gold chỉ có 1 vật; giữ mặc định |
| three_wheeler | YOLOE-26s | 0.15 | Quét ngưỡng trên gold; r03 dùng prompt tuk-tuk, cyclo, cargo tricycle |

Lệnh gộp đã dùng:

~~~bash
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.1,motorbike=0.1,car=0.45,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
~~~

- Tour model: r01 có 148 box/8 ảnh; r02 có 81 box/8 ảnh; r03 có 8 box three_wheeler/8 ảnh; r04 tạo 8 mask/2 ảnh.
- SAM: thử MobileSAM ở r04; chọn SAM 2.1 base cho phần point-prompt để thử chất lượng mask trên CPU. Chưa có so sánh định lượng chất lượng giữa hai phiên bản.
- Đổi ngưỡng không giải quyết được việc model COCO thiếu lớp three_wheeler. Dùng YOLOE bổ sung rồi kiểm tra lớp, box và vật sót bằng mắt.
- img_015 cho thấy tăng ngưỡng loại được box bicycle trùng nhưng đồng thời làm mất box xe đạp thật. Điểm tin cậy không thay thế việc kiểm ảnh.
- Giới hạn: chọn và đánh giá ngưỡng trên cùng 4 ảnh gold nên số đo máy trên gold có xu hướng lạc quan. Các lớp ít vật chưa đủ căn cứ chọn ngưỡng riêng.

## 2. AI error profile

Nguồn: `submission/qc/review_stats.md`, kết quả sau các lần review bổ sung.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---|---|
| accepted | 351 | 351/624 = 56.2% | 624 box máy theo script |
| edited | 60 | 60/624 = 9.6% | 624 box máy theo script |
| deleted | 139 | 139/624 = 22.3% | 624 box máy theo script |
| added | 55 | 55/480 = 11.5% | 480 đối tượng sau duyệt theo script |

- Script loại 28 đối tượng quanh vùng bẫy khỏi thống kê. Tổng các trạng thái không cộng khớp hoàn toàn với mẫu số hiển thị; giữ nguyên đầu ra và chưa coi các tỷ lệ là một phân hoạch cộng thành 100%.
- Có 5 trường hợp đổi lớp; số vật máy sót thật được script ước tính là 55 − 5 = 50.
- Không khẳng định lớp tệ nhất chỉ từ số lượng tuyệt đối. Car có 20 đối tượng được thêm; three_wheeler có 17 box bị xóa. Tour cho thấy YOLO26 sai lớp tuk-tuk ở img_002; QC cho thấy box three_wheeler trùng và sai lớp ở img_003.
- So với gold: TP=43, FP=19, FN=7. Precision = 43/62 = 69.4%; recall = 43/50 = 86.0%.
- KTC 95%: precision 57–79%, recall 74–93%.
- Gold export có 57 đối tượng; phép đo dùng 50 đối tượng làm mẫu số recall. Đây là số đối tượng được script đưa vào đánh giá, không phải toàn bộ annotation trong file.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Lần review đầu: 7/12 = 58.3%.
- Sau review bổ sung: 9/12 = 75.0%, chưa đạt ngưỡng 80%.
- Kết quả cuối theo loại: wrong_class 3/3; extra 3/3; loose 1/3; missing 2/3.
- Script còn báo lọt img_034 (loose), img_005 (missing), img_031 (loose).
- Đối chiếu img_031 cho thấy box tham chiếu nằm trên quần áo treo, không phải người thật. Giữ việc xóa box sai và ghi nhận hạn chế đáp án bẫy; không sửa key hoặc số đo.
- Các lần sửa sau khi biết danh sách bẫy lọt là review có hướng dẫn, không phải phép đo mù độc lập.
- Accepted 56.2% không chứng minh review đáng tin khi bắt bẫy chỉ đạt 75%. Không thể xác định chính xác bước checklist đã bỏ chỉ từ số đo; cần siết việc quét vật sót và kiểm biên box.

## 4. Máy có đáng dùng không?

Nguồn: `submission/time_log.csv`.

- Gold: 20 phút/4 ảnh = 5 phút/ảnh.
- Review có pre-label: 60 phút/24 ảnh = 2.5 phút/ảnh.
- Thời gian trung bình ghi nhận giảm 50%, nhưng chưa đủ kết luận AI tăng tốc gấp đôi: hai tập khác độ khó, người làm đã quen cảnh sau gold, và log chưa tách riêng thời gian các vòng sửa bổ sung.
- Pre-label hữu ích như điểm bắt đầu, nhưng vẫn cần sửa đáng kể. Chưa thể chấp nhận tự động với kết quả QC và bắt bẫy hiện tại.

Ba quy tắc giảm thiên kiến:
1. Mỗi ảnh đều ẩn nhãn và quét đối tượng trước khi hiện dự đoán.
2. Kiểm đủ bốn nhóm: bỏ sót, thừa/trùng, sai lớp, biên box; rà riêng người ngồi trên xe và xe ba bánh.
3. Ghi thời gian thực, giữ kết quả review đầu và phân biệt các vòng sửa sau khi đã biết đáp án.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM | Số click | Phần lẫn ngoài vật | Lỗi quan sát | File |
|---|---|---|---|---|---|
| img_013, lần 1 | SAM 2.1 base | 2 dương + 1 âm | Còn lẫn người lái | Chưa tách sạch người khỏi xe | `masks/tuktuk_img013.jpg` |
| img_013, lần 2 | SAM 2.1 base | 2 dương + 3 âm | Còn lẫn người lái và xe phía sau | Bổ sung điểm âm chưa giải quyết hết; mask 308787 px, 44.2% ảnh | `masks/tuktuk_img013_v2.jpg` |
| img_004 | SAM 2.1 base | 3 dương + 2 âm | Tách được người đứng cạnh và xe xanh phía sau | Còn vùng khuyết ở ghế/thân xe; mask 87537 px, 12.5% ảnh | `masks/tuktuk_img004.jpg` |
| img_031 | SAM 2.1 base | 2 dương + 2 âm | Còn dính quần áo treo phía trái | Bao được phần người nhìn thấy; mask 26338 px, 3.8% ảnh | `masks/person_img031.jpg` |

- Có 3 vật khác nhau; img_013 có hai lần thử.
- Áp dụng mục 4.2: bám phần nhìn thấy, loại bóng và vật lân cận; khoảng hở lớn cần kiểm tra.
- CAL-02 làm rõ người nhìn qua cửa hở không thuộc mask xe. Mask img_013 chưa đáp ứng đầy đủ.
- Các ảnh trên là bằng chứng thử nghiệm, chưa khẳng định mask cuối đạt chất lượng.
- Không thực hiện Interactor SAM trong CVAT; dùng script offline.

## 6. Calibration (Part 6)

Nguồn: `submission/qc/calibration_decisions.csv`.

- Có 4 quyết định: CAL-01 đến CAL-04; gồm nguồn loi_may và cau_hoi_mo.
- CAL-01: kiểm lớp và vật sót trước khi chuyển box sang SAM.
- CAL-02: kính thuộc xe; phần người nhìn thấy qua cửa hở được tách riêng.
- CAL-03: không tạo person cho quần áo trưng bày; ghi nhận bất đồng với tham chiếu bẫy.
- CAL-04: kiểm mask dính vật/nền và phần bị thiếu; không đánh giá bằng số click hoặc diện tích.
- Các nội dung được ghi trong `submission/guidelines/guideline_v1.1_calibrated.md`.
- Nguyên nhân gồm giới hạn model, câu hỏi mask chưa rõ và lỗi tham chiếu. Các câu hỏi mở chưa đủ bằng chứng vẫn được ghi là chưa chốt.

## 7. QC và quyết định (Part 6)

Nguồn: `submission/qc/qc_report_auto.md`.

- Mẫu: 5/24 ảnh batch; 0 ca can_review + 5 ảnh ngẫu nhiên, seed 42; 94 đối tượng.
- Người kiểm: Trần Thị Ngọc Ánh, có ChatGPT hỗ trợ đối chiếu ảnh. Hạn chế: người kiểm cũng là người review; ảnh overlay có vùng nhãn chồng nhau.
- Lỗi theo đối tượng: 5/94 = 5.3%.
- Ảnh có lỗi: 2/5 = 40.0%; vượt ngưỡng 2%.
- Top lỗi: thừa 3 ca tại img_003; sai lớp 1 ca tại img_003; sai vị trí/kích thước 1 ca tại img_031.
- Bắt bẫy: 9/12 = 75%, chưa đạt 80%.
- Cận trên 95% Wilson của tỷ lệ ảnh lỗi là 77%. Mẫu nhỏ không chứng minh được tỷ lệ lỗi toàn lô dưới 2%.
- Script kiểm được cấu trúc file và tính số đo từ phiếu; không tự xác nhận box sát vật thật, lớp đúng hoặc vật bị cả người và máy cùng bỏ sót.

**Quyết định đề xuất: REWORK, ngày 2026-11-18.**

Căn cứ: tỷ lệ ảnh lỗi vượt ngưỡng, lỗi thừa lặp lại và bắt bẫy chưa đạt. Lỗi tham chiếu img_031 được ghi riêng, không phủ nhận các lỗi QC khác.

Việc cần làm tiếp: loại box trùng/thừa, sửa sai lớp ở img_003; sửa box bicycle ở img_031 theo guideline; rà các ảnh tương tự, tạo release mới và QC lại. Chưa thực hiện vòng sửa QC này.

## 8. Ghi chú cá nhân

- Dùng ChatGPT hỗ trợ cài môi trường, giải thích lệnh, xem kết quả model/mask, đối chiếu bẫy và ảnh QC; hỗ trợ soạn model_log, calibration, guideline và bản nháp báo cáo. Cần tự kiểm lại nội dung với ảnh, guideline và đầu ra script trước khi nộp.
- Theo trình tự thực hiện: gán tay và xuất gold trước khi chạy pre-label trên dữ liệu lab. Bước kiểm môi trường dùng ảnh bus.jpg của thư viện.
- Máy: Windows 10, RAM 16 GB, WSL2 Ubuntu, Python 3.12.16; chạy model bằng CPU.
- CVAT local bằng Docker, phiên bản 2.74.1.
- Không dùng cache dự đoán; đã tự chạy YOLO26, YOLOE và SAM.
- Vướng mắc: thiếu transformers/huggingface_hub đã bổ sung; sai tên batch_reviewed.json đã sửa; FutureWarning của torch.jit.load không làm dừng chạy.
- Hạn chế cần ghi nhận: đáp án bẫy img_031 không phù hợp ảnh thật; mẫu số bảng error profile chưa cộng khớp các trạng thái. Chưa sửa script.
- Lần sau sẽ ghi thời gian riêng từng vòng review và kiểm box trùng kỹ hơn trước export.