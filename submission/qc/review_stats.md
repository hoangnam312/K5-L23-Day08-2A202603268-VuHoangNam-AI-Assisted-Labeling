# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 727; đối tượng sau duyệt: 682; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 638 | 638/727 = 87.8% | 727 box máy |
| edited (sửa) | 14 | 14/727 = 1.9% | 727 box máy |
| deleted (xoá) | 75 | 75/727 = 10.3% | 727 box máy |
| added (vẽ thêm) | 30 | 30/682 = 4.4% | 682 đối tượng sau duyệt |

Trong số deleted + added, **15 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 15.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 360 | 3 | 13 | 6 |
| bicycle | 5 | 1 | 1 | 2 |
| motorbike | 209 | 5 | 27 | 8 |
| car | 50 | 0 | 5 | 5 |
| bus | 2 | 0 | 1 | 1 |
| truck | 2 | 0 | 4 | 5 |
| three_wheeler | 10 | 5 | 24 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 52 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 46 | 39 | 6 | 54.1% (KTC 95%: 44–64%) | 88.5% (KTC 95%: 77–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 2/3
  - wrong_class: 3/3
  - loose: 3/3
  - lọt: img_022.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
