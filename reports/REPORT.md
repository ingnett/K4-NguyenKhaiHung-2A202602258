# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: 'Nguyễn Khải Hưng'
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `Chưa ghi nhận` |
| Thời gian gán `clip_01` | `Chưa ghi nhận` |
| Số track đã vẽ trong `clip_01` | `10` |
| Số keyframe trung bình mỗi track | `Chưa ghi nhận` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che hoặc đi sát nhau: giữ ID khi còn đủ bằng chứng là cùng một xe.
2. Xe xuất hiện hoặc rời khỏi rìa khung hình: đặt điểm bắt đầu/kết thúc đúng frame nhìn thấy.
3. Bbox ở giữa hai keyframe bị lệch: kiểm tra lại frame giữa và thêm keyframe khi cần.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: kiểm tra tính liên tục của ID trong toàn clip.
- Lượt 2: kiểm tra frame bắt đầu và kết thúc của từng track.
- Lượt 3: kiểm tra bbox tại các đoạn chuyển động hoặc che khuất.

Kiểm chéo với: `Chưa ghi nhận`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `Chưa ghi nhận`. Số lỗi bạn ấy tìm được trong bản của bạn: `Chưa ghi nhận`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Chưa có biên bản kiểm chéo để kết luận. Các luật cần ghi rõ hơn gồm: thời điểm tạo track mới sau khi xe rời khung, cách xử lý xe bị che lâu, và mật độ keyframe ở đoạn xe đổi hướng.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `771009c934de4096b8b675b799f5bf746995b3836a6a0b2137a595c53d45af43` |
| Thời điểm khóa | `2026-09-15T10:23:00.113069+00:00 UTC` |
| Số row / frame / track trước khi mở reference | `714 / 190 / 10` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.743 | 0.667 | 0.831 | 0.854 | 0.890 | 0.754 | 0.837 | 141 | 0 | 0 |
| Sau rework | 0.743 | 0.667 | 0.831 | 0.854 | 0.890 | 0.754 | 0.837 | 141 | 0 | 0 |

> SHA-256 của annotation hiện tại trùng với snapshot pre-gold, nên chưa có thay đổi giữa hai mốc.

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox lệch | 79 | 5 | Cần kiểm tra và chỉnh bbox quanh frame này |
| Bbox lệch | 108 | 6 | Cần kiểm tra và chỉnh bbox quanh frame này |
| Bbox lệch | 168 | 8 | Cần kiểm tra và chỉnh bbox quanh frame này |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.12.6 / 8.4.145 / 2.14.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.743 | 0.667 | 0.831 | 0.854 | 0.890 | 0.754 | 0.837 | 141 | 0 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| ReID vs bạn | 0.688 | 0.584 | 0.814 | 0.861 | 0.811 | 0.646 | 0.845 | 87 | 163 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của bản annotation là `0.754`, thấp hơn IDF1 `0.890`. Bản nhãn có ID khá ổn nhưng có 141 FP và không có FN. IDF1 tập trung vào độ đúng của identity precision/recall, còn MOTA cộng FP, FN và ID switch nên phản ánh cả lỗi detection. Khi MOTA cao nhưng IDF1 thấp, điều đó thường cho thấy bbox và số lượng detection khá đúng nhưng identity bị gán sai; MOTA không phạt nặng lỗi ID vì ID switch chỉ là một thành phần trong công thức và có thể nhỏ so với tổng số bbox.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

ReID tăng IDF1 từ `0.875` lên `0.900` và AssA từ `0.776` lên `0.820`. Cả hai đều có `2` IDSW, nhưng xảy ra ở các frame khác nhau: ByteTrack có switch tại frame `59` và `94`, còn ReID có switch tại frame `87` và `113`. Vì vậy ReID cải thiện chất lượng identity tổng thể nhưng không loại bỏ hoàn toàn ID switch. Đây là system comparison, không phải causal test riêng của ReID, vì hai run dùng hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Từ ByteTrack sang ReID, DetA tăng `0.649 -> 0.711`, FP tăng nhẹ `88 -> 91`, FN giảm mạnh `54 -> 26`, còn IDSW giữ ở `2`. Điều này cho thấy ReID run bắt được nhiều bbox hơn và cải thiện recall; lỗi còn lại là kết hợp của detector và association. FN giảm cho thấy vấn đề detection/khả năng giữ track vẫn đáng kể, trong khi IDSW và fragmented tracks cho thấy association vẫn còn lỗi ở các đoạn che khuất.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Evidence hiện có cho thấy ReID không khớp được các track annotation `4` và `5`; đây là ứng viên để kiểm tra các đoạn frame tương ứng trong ảnh. Chưa đủ bằng chứng từ JSON để khẳng định frame cụ thể hoặc kết luận model sai mà chưa xem overlay. Cần bổ sung frame sau khi kiểm tra trực quan.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Ứng viên cần xem lại là các track model bị ghost trong khoảng frame `16-116` (model track `7`), `106-121` (track `27`) và `158-178` (track `38`). Tuy nhiên đây mới là cảnh báo từ metric; cần mở ảnh để xác định đó là false positive của model hay annotation bị thiếu. Không sửa annotation chỉ dựa trên việc model khác nhãn.

## 6. Nếu phải gán thêm 10 clip nữa

Bổ sung vào `GUIDELINE_MINI.md` các luật có frame cụ thể: giữ ID bao nhiêu frame khi bị che, khi nào xe quay lại phải tạo ID mới, cách xử lý xe ở rìa ảnh, và vị trí cần thêm keyframe. Quy trình nên có ba lượt kiểm tra bắt buộc: kiểm ID, kiểm frame đầu/cuối, và kiểm frame giữa; sau đó ghi lại các frame bất đồng để kiểm chéo trước khi mở gold/reference.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` (chưa có trong workspace)
- [x] `reports/REPORT.md`
