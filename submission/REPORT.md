# Báo cáo Day 08 — 2A202603268 - Vũ Hoàng Nam

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.15; motorbike=0.10; bicycle/car/bus/truck=0.30 (mặc định) | `r01`: YOLO26m làm pre-label cho các lớp COCO; sweep trên 4 ảnh gold đề xuất person (18 vật: recall 94%, precision 50%) và motorbike (11 vật: recall 91%, precision 40%). Bicycle có 7 vật, recall tối đa 71%; car có 9 vật, tối đa 89%; bus và truck không có vật gold nên giữ mặc định. |
| three_wheeler | YOLOE-26s, prompt theo taxonomy | 0.15 | `r03`: YOLOE tìm lớp xe ba bánh; sweep trên 7 vật gold đạt recall 100%, precision 78% tại 0.15. |

- Lệnh `merge_prelabels.py` đã dùng: không có lệnh merge được lưu trong `model_log.csv`; log ghi recipe YOLO26-M r01 và YOLOE-S r03 riêng, cùng ngưỡng/đầu ra ở cột `params`. Vì vậy không khôi phục chính xác nguyên văn lệnh gộp.
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`), chạy CPU; theo r05 đã tạo 3 mask. Chọn do đã có quy trình click và đầu ra cho đủ ba vật; chất lượng ở `img_013` còn dính chân người lái và cần dọn biên.
- Lỗi **không sửa được bằng đổi ngưỡng**: YOLO26 gọi tuk-tuk là `car` ở `img_004`/`img_013`; sửa lớp theo taxonomy thành `three_wheeler`, tách người thành box `person` riêng.
- Box điểm cao mà sai: r01 ghi xe xanh sát mép ở `img_011` bị gán `truck` với 0.77 và box quá rộng; tăng ngưỡng không sửa được nhầm lớp.
- Giới hạn: sweep chỉ có 4 ảnh gold. Vì dùng cùng ảnh để chọn ngưỡng và đo kết quả, cần kiểm tra thêm trên ảnh mới. Bicycle có 7 vật; bus và truck không có vật gold nên chưa đủ dữ liệu chọn ngưỡng riêng. Xem `submission/qc/threshold_sweep.md` và `.csv`.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 638 | 87.8% | 727 box máy |
| edited | 14 | 1.9% | 727 box máy |
| deleted | 75 | 10.3% | 727 box máy |
| added | 30 | 4.4% | 682 đối tượng sau duyệt |

- Lớp có nhiều box máy bị xóa nhất là `three_wheeler`: 24 deleted và 5 edited; `motorbike` đứng tiếp theo với 27 deleted. Đây là số đếm theo lớp trong `qc/review_stats.md`. Model log r03 cũng ghi YOLOE có dự đoán xe ba bánh yếu hoặc thiếu ở một số ảnh pilot; đây là dấu hiệu cần rà lớp này kỹ hơn.
- Trong 30 added có 15 trường hợp đổi lớp, nên máy bỏ sót khoảng 15 vật. Trên 4 ảnh gold: precision 54.1% (46/85), recall 88.5% (46/52). Cần kiểm tra thêm ảnh mới vì ngưỡng được chọn trên chính 4 ảnh này.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 11/12 bẫy (91.7%), đạt ngưỡng yêu cầu 80%. Bắt được extra 3/3, missing 2/3, wrong_class 3/3 và loose 3/3. Bẫy còn lọt là `img_022.jpg` (missing), theo `qc/review_stats.md`.
- Kết quả sau khi cập nhật export cho thấy các bẫy wrong_class và loose trước đó đã được sửa đủ; bẫy missing ở `img_022.jpg` vẫn còn. Chỉ 12 bẫy nên khoảng tin cậy rộng; tỷ lệ này là phép kiểm tra quy trình duyệt, không chứng minh không còn lỗi khác.
- Accepted (87.8%) cho biết phần lớn box máy được giữ với IoU đạt ngưỡng, nhưng không đo được vật bị bỏ sót. Cần đọc cùng kết quả bẫy 91.7% và giới hạn mẫu nhỏ; hai số đo phản ánh hai việc khác nhau.

## 4. Máy có đáng dùng không?

- Thời gian review là 120 phút / 24 ảnh = 5 phút/ảnh (`time_log.csv`). Chưa có dòng `gold_manual` trong time log, nên chưa đủ dữ liệu tính thời gian gán tay mỗi ảnh hoặc phần trăm tiết kiệm. So sánh thời gian nếu bổ sung sau cũng có thể thiên lệch vì 4 ảnh gold được gán trước giúp làm quen cảnh.
- Kết luận: pre-label có ích như bản nháp có người kiểm tra, không đủ cơ sở dùng tự động. 87.8% box máy được giữ và bắt bẫy đạt 11/12 (91.7%), nhưng precision máy trên gold là 54.1%; cần review trực quan và sửa nhãn trước khi sử dụng. Gold chỉ có 4 ảnh và 12 bẫy là mẫu nhỏ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy và tự quét từng ảnh trước khi hiện box; checklist phải được hoàn tất cho đủ 24/24 ảnh.
  2. Kiểm tra mọi box, kể cả box confidence cao; với từng ảnh rà theo thứ tự bỏ sót → thừa → sai lớp → box lỏng.
  3. Sau export, đọc kết quả bẫy; chỉ coi bước review đạt khi bắt ít nhất 10/12 bẫy (>= 80%), nếu thấp hơn thì ghi nhận chưa đạt và review lại lô trước khi chốt.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_002` tuk-tuk xanh | SAM 2.1 base (`sam2.1_b.pt`, Colab CPU) | 4 (2 dương, 2 âm) | Không cố ý lấy người lái; không thấy hàng hoá. Có lẫn một ít người/nền nhìn xuyên qua khung hở. | Mask bám sát mái và thân xe, nhưng vùng khung hở còn lẫn cảnh phía sau. | `tuktuk_img002.jpg` |
| `img_013` tuk-tuk có người lái | SAM 2.1 base (`sam2.1_b.pt`, Colab CPU) | 10 ở lượt cuối (5 dương, 5 âm; tinh chỉnh qua 3 lượt) | Tách được phần lớn người lái/hành khách; còn dính ít vùng chân người lái. Không có hàng hoá đáng kể. | Cần nhiều click âm; biên quanh khung hở và người ngồi gần xe chưa sạch hoàn toàn. | `tuktuk_driver_img013.jpg` |
| `img_032` xích lô | SAM 2.1 base (`sam2.1_b.pt`, Colab CPU) | 4 (3 dương, 1 âm) | Không; không thấy hàng hoá. | Bao được ghế, khung và bánh xe; các lỗ giữa khung là vùng dễ sai nếu cần polygon thật sát. | `cyclo_img032.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc đường biên phần nhìn thấy và tách hai vật chạm nhau giúp xác định mask xe riêng với người. Quy tắc về lỗ nhỏ (<5%) cho phép không vẽ các khe nhỏ của khung xích lô. Cần chỉnh tay mask `img_013` quanh người lái và các vùng nhìn xuyên qua khung; click âm giảm lẫn người nhưng chưa làm biên hoàn toàn sạch.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`submission/qc/calibration_decisions.csv`), gồm 1 câu hỏi mở và 2 lỗi máy. `CAL-02` quan trọng nhất: tuk-tuk bị YOLO26 gọi là `car`; xe ba bánh phải gán `three_wheeler`, người ngồi là `person` riêng.
- Quy tắc mới trong `submission/guidelines/guideline_v1.1_calibrated.md`: phân biệt xe đạp điện theo dấu hiệu động cơ nhìn thấy; tuk-tuk có mui vẫn là `three_wheeler`; rà ảnh không có dự đoán three_wheeler để phát hiện bỏ sót.
- Nguyên nhân gốc: guideline v1.0 để mở câu hỏi xe đạp điện; lỗi tuk-tuk là giới hạn phân loại của máy; dự đoán rỗng của YOLOE không đảm bảo ảnh không có xe ba bánh.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh batch ngẫu nhiên (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42), 127 đối tượng; không có ảnh ca khó. Ảnh: `img_003`, `img_006`, `img_014`, `img_031`, `img_035`.
- Lỗi theo **đối tượng**: 0/127 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%), theo `submission/qc/qc_report_auto.md`.
- Top lỗi: không ghi nhận lỗi cụ thể trong 5 ảnh mẫu; không suy ra các loại lỗi khác không tồn tại trong toàn bộ tập.
- **Giới hạn mẫu nhỏ**: chưa thể kết luận tỷ lệ lỗi của cả lô dưới 2% chỉ từ 5 ảnh mẫu.
- Script đếm lỗi trong phiếu và số box; không xác nhận box có sát vật không, hoặc ảnh có vật bị bỏ sót không.

**Quyết định: Release**, ngày 2026-11-18. Lý do: đã chốt câu hỏi calibration; QC không thấy lỗi trong 5 ảnh mẫu; bắt bẫy đạt 11/12 = 91.7% (cao hơn ngưỡng 80%). Mẫu chỉ gồm 5/24 ảnh nên cần kiểm tra thêm trước khi dùng nhãn.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Có; đã đối chiếu nội dung với log và ảnh mẫu.
- Gold có bị nhiễm không: log hiện có không ghi lại thứ tự gán gold so với việc xem pre-label/hỏi AI, nên không thể xác nhận hồi cứu.
- Máy / GPU / CVAT dùng; vướng mắc: Windows 11, không có GPU; CVAT Docker 2.74.1. Lệnh Python trực tiếp gặp runtime bật isolated/safe-path nên không tìm thấy module `common` trong script subprocess.
- Dùng cache `data/cache/`: có; thư mục chứa cache YOLO26-M và YOLOE-26s, theo danh sách file hiện có.
- Lỗi / bug script đã gặp: runtime Python của môi trường hiện tại chặn đường dẫn import script; `check_submission.py` vì vậy không chạy được lệnh con `validate_labels.py` trực tiếp.
- Một điều làm khác lần sau: ghi lại lệnh merge nguyên văn, thời gian gán gold và quy trình tránh nhiễm trước khi bắt đầu từng Part.
