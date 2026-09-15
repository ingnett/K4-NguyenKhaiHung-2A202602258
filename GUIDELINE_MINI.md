# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Khải Hưng`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Chỉ gán xe bốn bánh nhìn thấy rõ trong clip; không suy đoán phần bị che hoặc phần nằm ngoài ảnh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Chuyển động, vị trí và appearance vẫn đủ để xác định cùng xe. |
| Xe bị che lâu hơn ngưỡng trên | tạo ID mới khi xe xuất hiện lại, trừ khi có bằng chứng chắc chắn đó là cùng xe | Tránh nối nhầm ID sau một đoạn mất dấu dài. |
| Xe rời khung hình rồi quay lại | tạo **track mới** | Không coi một lần xuất hiện sau khi rời khung là cùng track nếu không có liên tục quan sát. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo vị trí trước/sau khi cắt; kiểm tra frame ngay trước và sau vùng chồng lấp | Không đổi ID chỉ vì bbox tạm thời bị che hoặc hai xe gần nhau. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; nếu chưa chắc thì chờ frame kế tiếp, không đoán bbox |
| Xe đang đỗ, không di chuyển | vẫn gán nếu là xe bốn bánh nhìn thấy rõ; giữ cùng ID trong toàn thời gian còn quan sát được |
| Keyframe đặt dày ở đâu | đặt dày hơn ở lúc xe đổi hướng, bị che, đi sát xe khác, xuất hiện/rời khung và tại frame giữa nếu bbox nội suy bị lệch |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 79 / ID 5`
- Tình huống: diagnostics báo bbox có IoU thấp hơn tại frame giữa.
- Quyết định: kiểm tra lại bbox quanh frame 79 và thêm keyframe nếu bbox không ôm đúng xe.
- Lý do: bbox lệch làm giảm LocA/DetA dù ID vẫn có thể đúng.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 108 / ID 6`
- Tình huống: xe có đoạn bị che hoặc chuyển động khiến bbox nội suy lệch; IoU diagnostics là 0.585.
- Quyết định: xem lại frame 108 và các frame lân cận, giữ ID nếu vẫn nhận ra cùng xe.
- Lý do: cần phân biệt lỗi bbox với lỗi association trước khi đổi ID.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 168 / ID 8`
- Tình huống: bbox lệch khỏi xe ở đoạn cuối track; IoU diagnostics là 0.581.
- Quyết định: kiểm tra frame 168 và đặt lại bbox theo phần xe thực sự nhìn thấy.
- Lý do: không kéo bbox sang phần ngoài ảnh hoặc phần bị che.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi xe rời khung rồi xuất hiện lại, luôn tạo ID mới; không nối lại chỉ dựa trên màu hoặc vị trí gần nhau.
- Ở các frame 79, 108 và 168, cần kiểm tra bbox ở đoạn giữa thay vì chỉ kiểm tra keyframe đầu/cuối.
- Khi hai xe chồng lên nhau, phải xem frame trước và sau vùng chồng lấp để giữ ID nhất quán; không đổi ID chỉ vì một frame khó nhìn.
