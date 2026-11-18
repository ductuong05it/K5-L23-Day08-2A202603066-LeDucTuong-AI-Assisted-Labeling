# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603066 - Lê Đức Tường

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Cum xe may / nguoi nho o xa (< 20px) xu ly the nao? | Quy dinh ve 1 rectangle trum ca cum va chon nhan ignore_region | img_021.jpg | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk / xe ba banh co thiet ke mo bi nham lan | YOLO26 nham thanh car / truck | Su dung YOLOE prompt three_wheeler de bat chinh xac va doi label thanh three_wheeler | Bo sung vao Muc 2.3: Quy dinh nhan three_wheeler cho phuong tien 3 banh |
| img_032.jpg | Vung bong ram / mai che phan chieu tren duong | YOLO26 nhan dien nham voi conf 0.72 thanh car | Xoa box va quy dinh khong ve nhan car vao vung bong ram hinh nen | Bo sung vao Muc 3.2: Quy dinh loai bo nhieu do bong ram va anh sang |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Quy dinh ignore_region cho cum xe xa, khoanh vung three_wheeler cho xe ba banh va loai bo box rac do bong ram | Khac phuc loi may va chot cau hoi mo (CAL-01, CAL-02, CAL-03) |