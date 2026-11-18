# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603268 - Vũ Hoàng Nam

> Chỉ ghi các quy tắc được chốt hoặc bổ sung so với `guideline_v1.0_label_contract.md`.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện thuộc `bicycle` hay `motorbike`? | Nếu thấy rõ xe có động cơ, chọn `motorbike`; nếu không, chọn `bicycle`. Nếu ảnh chưa rõ, chọn lớp phù hợp nhất và bật `can_review`. | `img_015.jpg` | `CAL-01` |

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì | Quyết định | Quy tắc bổ sung |
|---|---|---|---|---|
| `img_004.jpg`, `img_013.jpg` | Tuk-tuk có mui dễ bị lẫn với ô tô; người ngồi gần xe | Model YOLO26 gán tuk-tuk là `car`; `review_stats.md` ghi 24 box `three_wheeler` bị xóa, 27 box `motorbike` bị xóa | Xe ba bánh là `three_wheeler`; người ngồi là `person` riêng. Sửa nhãn và tách box theo từng vật | Mục 2: mui và hàng gắn liền không làm tuk-tuk thành `car`; không dùng tăng/giảm confidence để chữa nhầm lớp (`CAL-02`) |
| `img_007.jpg`, `img_015.jpg`, `img_016.jpg`, `img_019.jpg`, `img_030.jpg` | Ảnh pilot không có dự đoán `three_wheeler`, có thể bỏ sót xe ba bánh | YOLOE r03 không trả box cho các ảnh trên theo `model_log.csv` | Rà ảnh gốc; bổ sung box nếu thấy vật thuộc lớp, không coi dự đoán rỗng là bằng chứng không có vật | Mục 2/checklist: rà cả ảnh không có box xe ba bánh; đánh dấu `can_review` khi chưa chắc (`CAL-03`) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản gốc | — |
| v1.1 | Chốt dấu hiệu phân biệt xe đạp điện; nhấn mạnh tuk-tuk có mui vẫn là `three_wheeler`; thêm lượt rà ảnh khi model không dự đoán xe ba bánh | Câu hỏi mở và lỗi model, `CAL-01` đến `CAL-03`; đối chiếu `submission/qc/calibration_decisions.csv`, `submission/model_log.csv`, `submission/qc/review_stats.md` |
