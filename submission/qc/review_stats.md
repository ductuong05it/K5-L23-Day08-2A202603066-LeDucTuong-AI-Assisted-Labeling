# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 621; đối tượng sau duyệt: 539; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 468 | 468/621 = 75.4% | 621 box máy |
| edited (sửa) | 28 | 28/621 = 4.5% | 621 box máy |
| deleted (xoá) | 125 | 125/621 = 20.1% | 621 box máy |
| added (vẽ thêm) | 43 | 43/539 = 8.0% | 539 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 37.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 240 | 9 | 22 | 14 |
| bicycle | 7 | 0 | 1 | 2 |
| motorbike | 157 | 13 | 67 | 9 |
| car | 45 | 0 | 10 | 11 |
| bus | 2 | 0 | 1 | 1 |
| truck | 4 | 0 | 2 | 3 |
| three_wheeler | 13 | 6 | 22 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 45 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 39 | 25 | 6 | 60.9% (KTC 95%: 49–72%) | 86.7% (KTC 95%: 74–94%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 3/12 = 25.0% (KTC 95%: 9–53%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 0/3
  - wrong_class: 2/3
  - missing: 1/3
  - extra: 0/3
  - lọt: img_026.jpg (loose), img_033.jpg (missing), img_006.jpg (loose), img_022.jpg (extra), img_034.jpg (loose), img_029.jpg (wrong_class), img_027.jpg (extra), img_024.jpg (extra), img_025.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
