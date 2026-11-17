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
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
