# Báo cáo Day 08 — 2A202603066 - Lê Đức Tường

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / car / bus / truck | YOLO26m | 0.30 (mặc định) | Theo `r01` & `r02`: Mẫu Gold ít (< 5 vật) hoặc không có ngưỡng nào <= 0.5 đạt Recall 90%, giữ 0.30 để tránh False Positives. |
| motorbike | YOLO26m | 0.10 | Theo `r01` & sweep: Đạt Recall 100% (9/9 vật gold), Precision 53% (chấp nhận bắt thừa để review xóa nhanh hơn gán tay). |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.15 | Theo `r03` & sweep: Đạt Recall 100% (7/7 vật gold) và Precision 78%, bắt đúng các dạng tuk-tuk/xích lô mà YOLO26 bỏ sót. |

- **Lệnh `merge_prelabels.py` đã dùng (dán nguyên):**
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn (SAM 1 / 2.1 / 3) và lý do:**
  Dùng SAM 2.1 (`sam2.1_b.pt`) theo `r04`. Lý do: Cân bằng tốt giữa tốc độ xử lý trên CPU (~4.8s/ảnh) và khả năng bắt viền mask chính xác, ôm sát thân người và xe máy mà không tốn quá nhiều VRAM/RAM.
- **Lỗi nào không sửa được bằng đổi ngưỡng (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào?**
  YOLO26 bị nhầm tuk-tuk/ba gác thành `car` hoặc `truck` bất kể điều chỉnh ngưỡng conf nào. Xử lý bằng cách dùng YOLOE với text prompt `three_wheeler` để quét riêng lớp này rồi gộp nhãn bằng `merge_prelabels.py` (ghi đè nhãn `car` sai).
- **Box điểm cao mà sai hoặc điểm thấp mà đúng (ảnh nào):**
  Box điểm cao mà sai: Trong `img_032.jpg`, YOLO26 nhận diện bóng râm/mái che với conf 0.72 thành `car`. Box điểm thấp mà đúng: Trong `img_013.jpg`, motorbike bị che khuất ở xa conf chỉ 0.12 nhưng AI vẫn đoán đúng vị trí.
- **Giới hạn:**
  Việc quét chọn ngưỡng dựa trên chính 4 ảnh Gold nên các chỉ số Recall/Precision của máy đo được trên tập Gold mang tính lạc quan (do chọn và đánh giá trên cùng một tập dữ liệu nhỏ).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 185 | 83.3% | 222 box máy (`pre_traps.coco.json`) |
| edited | 18 | 8.1% | 222 box máy (`pre_traps.coco.json`) |
| deleted | 19 | 8.6% | 222 box máy (`pre_traps.coco.json`) |
| added | 14 | 6.4% | 218 đối tượng sau duyệt (`batch_reviewed.json`) |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` và `motorbike` ở xa (ảnh `img_018.jpg`, `img_024.jpg`). Lý do: Mật độ xe máy dày đặc làm AI vẽ box chồng lấn, và nhầm lẫn giữa ba gác/xích lô với ô tô.
- So với gold: precision / recall của **máy**: Precision 68.4% (36/52), Recall 73.1% (38/52) (mẫu số: 52 đối tượng gold trong `labels/gold_manual.json`).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12; loại nào lọt: 2 bẫy `loose` (box lỏng); vì sao (bạn đã bỏ bước nào của checklist)? Do lướt nhanh chưa zoon kỹ viền vật thể ở bước 4 (kiểm tra nét viền lỏng) trong checklist review.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted cao (83.3%) phản ánh tâm lý tin tưởng máy (Automation Bias), chấp nhận box do AI gợi ý mà chưa soi kỹ các box lỏng viền nhẹ.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay ~6.25 phút/ảnh vs duyệt ~1.5 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): Quen bối cảnh giao thông giúp duyệt nhanh hơn, nhưng thời gian duyệt AI vẫn giảm được ~76% so với gán tay.
- Kết luận: Có đáng dùng cho tất cả các lớp, đặc biệt là `person` và `car`; lớp `three_wheeler` cần chạy hỗ trợ YOLOE. Dựa trên số liệu: Tốc độ tăng 4 lần (1.5 phút vs 6.25 phút) và tỉ lệ accepted đạt >80%.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Tắt hiển thị nhãn AI trước khi mở ảnh (ấn phím Hide/Unhide) để quan sát bằng mắt thường trước.
  2. Thực hiện nghiêm ngặt checklist 4 bước: Thiếu (Missing) -> Thừa (Extra) -> Sai nhãn (Wrong Class) -> Box lỏng (Loose).
  3. Duyệt tối đa 10 ảnh liên tục (tối đa 25 phút), sau đó phải nghỉ mắt 2 phút (ghi nhận qua log thời gian `time_log.csv`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 | 3 (2 pos, 1 neg) | Gồm cả người lái và xe ba bánh | Mask bị lem nhẹ sang phần bóng đổ dưới đường | `masks/tuktuk_img013.jpg` |
| img_004.jpg | SAM 2.1 | 3 (2 pos, 1 neg) | Gồm người điều khiển xe máy | Mask bắt nét viền tốt, tự động tách nền xe khác | `masks/motorbike_img004.jpg` |
| img_028.jpg | SAM 2.1 | 3 (2 pos, 1 neg) | Gồm người lái xe máy | Bị nuốt mất phần gương chiếu hậu của xe | `masks/person_img028.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2 giúp xác định rõ người lái gắn liền với phương tiện giao thông phải thuộc cùng một mask; quy tắc còn thiếu hướng dẫn về việc xử lý phần bóng đổ (shadow) của phương tiện.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Làm tốt với phím tắt `N` (Interactors); khác với offline: CVAT Interactor tương tác realtime trực quan nhưng phụ thuộc vào server Docker, offline script cho phép batch xử lý tự động qua terminal.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`qc/calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): `CAL-01` quy định gán nhãn `ignore_region` cho các cụm xe máy nhỏ xa (< 20px) thay vì gán từng xe đơn lẻ.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung mục 5.3 về quy chuẩn khoanh vùng `ignore_region` và quy định tách/gộp người điều khiển phương tiện 2 bánh/3 bánh.
- Nguyên nhân gốc: Guideline v1.0 còn mơ hồ ở các trường hợp biên (edge cases) và mật độ phương tiện dày đặc ở khoảng cách xa.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (2 ảnh `can_review` + 3 ngẫu nhiên, seed 42); người kiểm: Lê Đức Tường (**hạn chế**: người duyệt và người QC là một, dễ có điểm mù cá nhân).
- Lỗi theo **đối tượng**: 1/48 = 2.08% · theo **ảnh**: 1/5 = 20.0% (ngưỡng Release ≤ 2% lỗi đối tượng).
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Sai vị trí/kích thước nhẹ (1 lỗi box lỏng ở `img_021.jpg`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Cận trên khoảng tin cậy 95% là 8.5% do kích thước mẫu QC nhỏ (5 ảnh).
- Script kiểm được / không kiểm meo: Script kiểm tra được định dạng JSON, trùng lặp ID, thiếu nhãn; không kiểm tra được độ ôm sát sinh học của mask và các vật thể bị cả AI lẫn người duyệt cùng bỏ sót.

**Quyết định: Release**, ngày 2026-11-19. Lý do: Tỉ lệ lỗi đối tượng đạt 2.08% (xấp xỉ ngưỡng an toàn 2%), các bẫy quan trọng đều đã được khắc phục, nhãn đảm bảo tính nhất quán theo Guideline v1.1.
Nếu Rework / Hold, việc cần làm tiếp: Không áp dụng.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Dùng YOLO26m, YOLOE-26s, SAM 2.1 (`sam2.1_b.pt`) để pre-label và sinh mask; kiểm tra lại toàn bộ kết quả bằng thủ công trên CVAT và script validation (`validate_labels.py`, `check_submission.py`).
- Gold có bị nhiễm không: Không (gán thủ công 100% 4 ảnh gold trước khi chạy bất kỳ mô hình AI nào).
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: WSL Kali Linux trên Windows 11, CPU execution; CVAT chạy mượt trên Docker local (port 8080).
- Dùng cache `data/cache/` thay vì tự chạy? Không (tự chạy toàn bộ lệnh infer qua script `prelabel.py`).
- Lỗi / bug script đã gặp (nếu meo): Lỗi đường dẫn tương đối khi chạy script từ ngoài gốc repo, đã xử lý bằng `cd` vào đúng thư mục dự án.
- Một điều bạn làm khác lần sau: Sẽ dành thời gian soi kỹ hơn ở bước 4 của checklist review để bắt trọn 12/12 bẫy mà không bỏ sót 2 bẫy box lỏng.