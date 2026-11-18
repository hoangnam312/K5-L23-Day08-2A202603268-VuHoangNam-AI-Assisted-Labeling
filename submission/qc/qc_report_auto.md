# Báo cáo QC tự động (nháp)

> Số liệu được tính từ phiếu QC.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 127 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/127 = 0.0% | 127 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 0/5 = 0.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 638/727 = 87.8% / 14/727 = 1.9% / 75/727 = 10.3% | 727 box máy |
| Máy: added | 30/682 = 4.4% | 682 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Giới hạn thống kê
- Không thấy lỗi ở 5 ảnh mẫu, nhưng 5 ảnh quá ít để kết luận tỷ lệ lỗi của cả lô dưới 2%.

## Gợi ý quyết định
**RELEASE**

- Không thấy lỗi trong 5 ảnh mẫu; bắt bẫy đạt 11/12. Cần kiểm tra thêm vì mẫu còn nhỏ.

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
