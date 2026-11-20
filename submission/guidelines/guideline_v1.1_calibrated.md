# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603050 - Cao Quang Linh

> Bản bổ sung có AI hỗ trợ soạn; người nộp cần xác nhận các quyết định dưới đây khớp với cách gán nhãn của mình.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định đề xuất | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Cụm xe máy đỗ dày gần camera: gán từng xe hay `ignore_region`? | Gán từng xe khi còn nhận diện được ranh giới từng thân xe hoặc bánh xe. Chỉ dùng một `ignore_region` khi cụm có ít nhất 5 xe và không thể tách cá thể đáng tin cậy. | `img_018.jpg` | CAL-01 |

Các câu hỏi v1.0 chưa chốt trong lượt này: xe đạp điện (Q1), hàng nhô ra khỏi xe (Q3), người trong xe có mui (Q4), kính xe (Q5), box chồng lấn ban đêm (Q6), và ngưỡng confidence (Q7). Chưa có ví dụ đủ rõ để kết luận nhất quán; khi gặp, đánh dấu `can_review` và không suy ra quy tắc từ confidence.

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì | Quyết định đề xuất | Quy tắc bổ sung |
|---|---|---|---|---|
| `img_013.jpg` | Tuk-tuk bị nhận thành ô tô; mask theo box giữ nguyên lỗi lớp | YOLO26/SAM có thể cho mask tốt nhưng lớp vẫn là `car` | Kiểm tra lớp theo vật thể trước khi chấp nhận mask; tuk-tuk là `three_wheeler`, người nhìn thấy gán `person` riêng | Mục 2 và 4.2: mask không thay thế kiểm tra lớp |
| `img_026.jpg` | Xe máy mờ, box lỏng dễ được giữ vì nhìn có vẻ hợp lý | Box bao nhiều nền đường | Phóng to ảnh và siết box theo phần xe nhìn thấy; không dùng confidence làm lý do giữ box lỏng | Mục 4.1.1 và checklist review bước 4 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Bổ sung tiêu chí tách cụm xe máy; yêu cầu kiểm tra lớp trước mask; nhắc chỉnh box xe mờ theo vật thể | CAL-01 đến CAL-03 |
