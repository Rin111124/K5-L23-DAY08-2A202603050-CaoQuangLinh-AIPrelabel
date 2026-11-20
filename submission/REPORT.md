# Báo cáo Day 08 — 2A202603050 - Cao Quang Linh

> Bản hoàn thiện có hỗ trợ ChatGPT. Các số liệu lấy từ log/script trong repo. Những nhận xét có ghi “bản nháp AI” cần học viên đọc, xác nhận và sửa theo trải nghiệm thực tế trước khi nộp.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person/car/bus/truck: mặc định 0.3; bicycle: 0.2; motorbike: 0.1 | r01–r02 cho thấy YOLO26 bắt nhiều vật nhưng ngưỡng 0.5 làm mất vật nhỏ/xa; threshold sweep chọn theo F1/recall trên 4 ảnh gold. |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | r03 phát hiện lớp mà COCO/YOLO26 không có; threshold sweep ghi 7/7 recall tại 0.15. Box vẫn có thể lỏng và cần review. |

- Lệnh merge ghi theo cấu hình trong `threshold_sweep.md` và lịch sử thao tác: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`. Lệnh lần đầu dán nhiều dòng bằng dấu backtick trong CMD nên lỗi; cần đối chiếu lịch sử terminal để xác nhận lần chạy thành công có cùng tham số.
- Chọn SAM 2.1 base (`sam2.1_b.pt`), vì `model_log.csv` ghi đây là segmenter được thử ở r04; SAM nhận box từ detector nên không sửa được lớp sai. Ba click-mask thử nghiệm cho thấy mask còn dính người hoặc bỏ sót phần xe.
- Lỗi không giải quyết bằng đổi ngưỡng: tuk-tuk bị gọi là `car` (img_013). Cần sửa lớp thành `three_wheeler`, tách nhãn `person` nhìn thấy; SAM chỉ tạo mask theo vùng đầu vào.
- Ví dụ điểm thấp nhưng đúng: YOLOE phát hiện tuk-tuk img_002 và img_032 với confidence cao trong r03; ví dụ điểm cao nhưng sai lớp: tuk-tuk img_013 bị YOLO26 gọi là `car` (r04). Cần đối chiếu trực tiếp ảnh và log run trước khi chốt mô tả này.
- Giới hạn: threshold được chọn trên chính 4 ảnh gold rồi đo lại trên chúng; kết quả in-sample lạc quan, không đại diện chắc chắn cho dữ liệu mới. Các lớp bus (0 vật gold) và truck (1 vật gold) quá ít để hiệu chỉnh đáng tin.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---:|
| accepted | 589 | 94.4% | 624 box máy |
| edited | 5 | 0.8% | 624 box máy |
| deleted | 30 | 4.8% | 624 box máy |
| added | 8 | 1.3% | 602 đối tượng sau duyệt |

- Theo threshold sweep, motorbike ở ngưỡng 0.1 có precision 52.4% (11 TP, 10 FP; 12 vật gold), đổi lại recall 91.7%. Truck chỉ có 1 vật gold và 0 TP ở sweep; mẫu quá nhỏ để kết luận chắc lớp nào tệ nhất. Review stats ghi ba-wheeler có 14 box deleted, cần xem xét ánh xạ lớp/box chứ không chỉ hạ ngưỡng.
- So với gold: precision 46/78 = 59.0%; recall 46/54 = 85.2% (46 TP, 32 FP, 8 FN; nguồn `review_stats.md`). Đo trên 4 ảnh đã dùng chọn ngưỡng nên lạc quan.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy 11/12 = 91.7%; lọt một bẫy loose ở `img_026.jpg` (`review_stats.md`). Nguyên nhân hành vi cần học viên xác nhận; giả thuyết bản nháp AI là đã chưa siết bước phóng to và kiểm tra độ ôm sát theo checklist.
- Accepted 94.4% và bắt bẫy 91.7% đều cao trong mẫu này, nhưng vẫn có một bẫy lọt và 12 bẫy là mẫu nhỏ. Vì vậy pre-label có ích để bắt đầu, không thay thế bước rà toàn ảnh và từng box.

## 4. Máy có đáng dùng không?

- `time_log.csv`: gán tay 22 phút / 4 ảnh = 5.5 phút/ảnh; review có pre-label 75 phút / 24 ảnh = 3.1 phút/ảnh. Chênh lệch trung bình ghi nhận khoảng 44%, nhưng hai tập ảnh khác nhau về nội dung và độ khó; không xem đây là so sánh kiểm soát. Gán gold trước cũng có thể khiến người làm quen cảnh.
- Kết luận dự thảo AI: dùng pre-label như bản nháp cho person/motorbike/car và dùng YOLOE để gợi ý `three_wheeler`; vẫn duyệt thủ công từng ảnh. Căn cứ: review giữ nguyên 589/624 box (94.4%) và tiết kiệm thời gian ghi nhận, nhưng precision gold chỉ 59.0%, có 32 FP và một bẫy loose bị lọt. Học viên cần xác nhận cách diễn đạt kết luận.
- Ba quy tắc giảm thiên kiến (đề xuất để học viên xác nhận):
  1. Trước khi mở nhãn máy, quét toàn ảnh và ghi vật thể nhìn thấy; lưu checklist theo từng ảnh.
  2. Với mỗi box, kiểm tra đủ bốn bước trong review checklist; phóng to các box xe mờ và box điểm cao có vẻ sai.
  3. Giữ bẫy kín đến khi export xong; ghi số bẫy bắt được và đọc cả danh sách bẫy lọt, không chỉ phần trăm.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| `img_013.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Có; mask vẫn dính người lái/khách | Mask phủ cả tuk-tuk và người dù đã thêm click âm | `img013_tuktuk_sam21.jpg` |
| `img_010.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Dính một phần người lái | Chỉ lấy một phần mui/khung; bỏ sót phần lớn bánh và thân xe | `img010_cyclo_sam21.jpg` |
| `img_026.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Không thấy người trong mask | Chỉ lấy một dải yên/thân xe máy mờ, chưa phủ toàn xe | `img026_blurry_motorbike_sam21.jpg` |

Các kết quả là mask thử nghiệm, chưa đạt ranh giới vật thể tốt. Cần xem lại ảnh gốc và ghi đúng mức độ chất lượng; click âm trên người và click dương ở nhiều phần xe có thể giúp, nhưng không đảm bảo tách đúng.

- Interactor SAM trong CVAT: chưa có bằng chứng trong log để kết luận đã chạy thành công; các file đang có là kết quả thử offline bằng script click-point.

## 6. Calibration (Part 6)

- `calibration_decisions.csv` có ba đề xuất CAL-01 đến CAL-03. CAL-01 đề xuất tách từng xe khi thấy được thân/bánh; chỉ dùng `ignore_region` cho cụm ít nhất 5 xe không thể tách đáng tin. CAL-02 sửa tuk-tuk bị gọi `car` thành `three_wheeler` và gán `person` riêng khi nhìn thấy. CAL-03 phóng to xe máy mờ và siết box theo phần xe nhận diện được.
- `guideline_v1.1_calibrated.md` là bản nháp có hỗ trợ AI. Q1 và Q3–Q7 vẫn được ghi là chưa chốt; học viên cần xác nhận các quy tắc có khớp quyết định gán nhãn thực tế hay không trước khi xem đây là guideline cuối.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 `can_review` + 5 ngẫu nhiên, seed 42), tổng 117 đối tượng. Phiếu hiện do ChatGPT hỗ trợ rà overlay; chưa phải lượt kiểm độc lập của học viên. `img_006` và `img_014` là cảnh dày/xa, cần học viên xác nhận.
- Lỗi theo đối tượng: 3/117 = 2.6%; ảnh có lỗi: 2/5 = 40.0% (`qc_report_auto.md`).
- Lỗi ghi nhận: sai vị trí/kích thước — `img_031.jpg`, box #512 gộp/bao quá rộng quanh xe đạp chở hàng; `img_035.jpg`, boxes #608 và #609 là hai phần của cùng xe ba bánh. Đây là các phát hiện AI cần người nộp xác nhận trên ảnh gốc.
- Cận trên 95% Wilson cho tỷ lệ ảnh lỗi là 77%; với 5 ảnh, mẫu không đủ để chứng minh ngưỡng Release 2%.
- Script đếm lỗi được ghi trong phiếu, tỷ lệ theo ảnh/đối tượng và cận Wilson. Script không phát hiện lỗi chưa được người rà ghi vào phiếu, không đánh giá box có ôm sát vật thể thật hay không, và không biết người rà có làm theo checklist hay không.

**Quyết định dự thảo: Rework**, ngày 2026-11-19. Theo số liệu QC hiện tại, 2/5 ảnh có lỗi, cao hơn ngưỡng 2%; cần kiểm lại phát hiện AI, sửa các box được xác nhận (#512, #608, #609), dựng lại release và lấy mẫu QC vòng tiếp theo. Đây là gợi ý từ số liệu; học viên phải xác nhận quyết định cuối.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** ChatGPT hỗ trợ rà overlay QC, đọc số liệu, đề xuất calibration và tạo/kiểm tra prompt click SAM. Các phát hiện AI được ghi rõ; người nộp cần xác nhận trên ảnh và chịu trách nhiệm với quyết định cuối.
- Gold có bị nhiễm không: chưa xác nhận được từ log. Học viên cần ghi trung thực liệu có xem pre-label hoặc hỏi AI trước khi hoàn tất gán gold hay không.
- Máy: Windows 11, CPU không GPU; CVAT Docker v2.74.1 theo `STUDENT.md`.
- Cache `data/cache/`: thư mục chỉ có `.gitkeep`, không thấy kết quả cache; đối chiếu cách bạn chạy model để xác nhận.
- Lỗi đã gặp: lệnh merge nhiều dòng dùng dấu backtick trong CMD làm các tham số bị tách thành lệnh riêng và báo thiếu `--out`; cần ghi rõ nếu đây đúng là lần lỗi bạn muốn nêu.
- Bài học dự thảo: SAM có thể tạo mask trông có vẻ hợp lý nhưng không tự sửa nhãn lớp, và review nhanh có thể bỏ lọt box lỏng; cần xác nhận đây có đúng trải nghiệm của bạn.
