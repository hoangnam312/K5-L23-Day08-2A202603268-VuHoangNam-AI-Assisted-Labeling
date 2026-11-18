# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.15; motorbike=0.10; bicycle/car/bus/truck=0.30 (mặc định) | `r01`: YOLO26m làm pre-label cho các lớp COCO; sweep trên 4 ảnh gold đề xuất person (18 vật: recall 94%, precision 50%) và motorbike (11 vật: recall 91%, precision 40%). Bicycle có 7 vật, recall tối đa 71%; car có 9 vật, tối đa 89%; bus và truck không có vật gold nên giữ mặc định. |
| three_wheeler | YOLOE-26s, prompt theo taxonomy | 0.15 | `r03`: YOLOE tìm lớp xe ba bánh; sweep trên 7 vật gold đạt recall 100%, precision 78% tại 0.15. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: sweep chỉ có 4 ảnh gold; ngưỡng và precision/recall được chọn, đo trên cùng tập nên số đo trên gold lạc quan. Các lớp ít mẫu (bicycle 7, bus/truck 0) chưa đủ dữ liệu để chọn ngưỡng riêng; xem `submission/qc/threshold_sweep.md` và `.csv`.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 638 | 87.8% | 727 box máy |
| edited | 14 | 1.9% | 727 box máy |
| deleted | 75 | 10.3% | 727 box máy |
| added | 30 | 4.4% | 682 đối tượng sau duyệt |

- Lớp có nhiều box máy bị xóa nhất là `three_wheeler`: 24 deleted và 5 edited; `motorbike` đứng tiếp theo với 27 deleted. Đây là số đếm theo lớp trong `qc/review_stats.md`. Model log r03 cũng ghi YOLOE có dự đoán xe ba bánh yếu hoặc thiếu ở một số ảnh pilot; đây là dấu hiệu cần rà lớp này kỹ hơn.
- Trong 30 added có 15 trường hợp tương ứng với đổi lớp, nên số vật máy bỏ sót thật ước tính là 15. So với gold: precision máy 54.1% (46 TP / 85 dự đoán; 39 FP), recall 88.5% (46 TP / 52 đối tượng gold; 6 FN). Kết quả gold có tính lạc quan vì ngưỡng được chọn và chấm trên cùng 4 ảnh.

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
