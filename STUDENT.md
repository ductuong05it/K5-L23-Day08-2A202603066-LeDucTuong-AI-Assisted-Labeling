# Thông tin học viên

- Họ và tên: Lê Đức Tường
- MSHV: 2A202603066
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: https://github.com/ductuong05it
- Repo bài nộp (URL): https://github.com/ductuong05it/K5-L23-Day08-2A202603066-LeDucTuong-AI-Assisted-Labeling
- Commit chốt bài (hash): main
- Máy: Kali Linux trên WSL2 Windows 11, GPU: chạy CPU execution
- CVAT (Docker): v2.8.2 (Local Docker port 8080)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 185 / 18 / 19 (mẫu số: 222 box máy); added: 14 (mẫu số: 218 đối tượng)
- So với gold: precision / recall của **máy** 68.4% / 73.1%
- Bắt bẫy: 10/12 = 83.3%
- Thời gian: gán tay 6.25 phút/ảnh; duyệt có pre-label 1.5 phút/ảnh
- Lỗi QC: 20.0% theo ảnh (1/5), 2.08% theo đối tượng (1/48)
- **Quyết định:** Release — lý do một dòng: Tỉ lệ lỗi đối tượng 2.08% đạt ngưỡng an toàn, nhãn nhất quán theo Guideline v1.1.