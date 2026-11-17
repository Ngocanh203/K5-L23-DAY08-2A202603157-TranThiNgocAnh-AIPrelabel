# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 624; đối tượng sau duyệt: 474; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 378 | 378/624 = 60.6% | 624 box máy |
| edited (sửa) | 48 | 48/624 = 7.7% | 624 box máy |
| deleted (xoá) | 132 | 132/624 = 21.2% | 624 box máy |
| added (vẽ thêm) | 44 | 44/474 = 9.3% | 474 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 39.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 237 | 18 | 73 | 5 |
| bicycle | 3 | 0 | 1 | 1 |
| motorbike | 93 | 27 | 35 | 10 |
| car | 25 | 1 | 3 | 20 |
| bus | 1 | 0 | 1 | 2 |
| truck | 0 | 0 | 2 | 1 |
| three_wheeler | 19 | 2 | 17 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 50 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 43 | 19 | 7 | 69.4% (KTC 95%: 57–79%) | 86.0% (KTC 95%: 74–93%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - extra: 2/3
  - loose: 1/3
  - missing: 2/3
  - lọt: img_029.jpg (wrong_class), img_022.jpg (extra), img_034.jpg (loose), img_005.jpg (missing), img_031.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
