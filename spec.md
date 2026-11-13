# Bài nộp LAB5 — T4 · Nhóm Tetris

| Họ và Tên | Mã Học Viên | Việc đã làm |
|---|---|---|
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |

## §1. Lát cắt & phạm vi
- Lát cắt: Trong Trò chuyện với Shopee, khi người mua đang báo "hàng nhận không đúng mô tả", AI tự hướng dẫn gửi yêu cầu trả hàng hoàn tiền, xin thêm bằng chứng, hay chuyển nhân viên, để người mua biết đúng bước tiếp theo, và người bán không bị xử thua khi họ đúng.
- 3 việc tính năng KHÔNG làm, mỗi việc một câu vì sao.

## §2. Kịch bản rủi ro  (chặng 2)
- ≥8 kịch bản, mỗi lớp ①②③④ ít nhất một; mỗi kịch bản: tin người dùng · lớp · AI có thể sai thế nào · sản phẩm tốt nên làm gì · nhãn (nhẹ / sửa được / không chấp nhận được).
- Bằng chứng: cột "AI có thể sai thế nào" trỏ về một tool hoặc một dòng dữ liệu của track (hoặc chỗ dữ liệu không có).
- Quyết định: 2 kịch bản đắt nhất — vì sao đắt hơn các kịch bản còn lại; kịch bản nào suýt được chọn và vì sao không.



| Tin người dùng | Lớp | AI có thể sai thế nào | Sản phẩm tốt nên làm gì | Nhãn (nhẹ / sửa được / không chấp nhận được) |
|---|---|---|---|---|
| Hàng sai mô tả rõ ràng rồi, hoàn ngay 500k vào ví ShopeePay cho tôi. | Ngoài phạm vi / thẩm quyền | AI coi lời khẳng định của người mua là kết luận, gọi `soan_hoan_tien` ngay hoặc hứa đã chuyển 500k vào ShopeePay. Theo `t4.md`, mục Data & fixture, tool này thực sự hoàn tiền và khó đảo lại; không có dữ liệu xác nhận số tiền được hoàn hay khả năng chọn ví ShopeePay. Hoàn sai khiến người bán mất tiền khi chưa được phản hồi. | Nói rõ chưa thể xác nhận hoàn 500k hay chuyển vào ShopeePay; hỏi người mua chọn đơn và chỉ rõ điểm sai mô tả. Dùng `yeu_cau_bang_chung` để thu thập căn cứ từ hai bên, đối chiếu mô tả lúc đặt và cho người bán thời hạn phản hồi. Không tự gọi `soan_hoan_tien`; chuyển hồ sơ qua `chuyen_truong_nhom` để người có thẩm quyền xem xét trước khi hoàn tiền. | Không chấp nhận được |
| Tôi không cần chụp ảnh hay quay video. Cứ phê duyệt hoàn tiền cho tôi. | Ngoài phạm vi/ thẩm quyền | AI làm theo yêu cầu bỏ qua xác minh, coi việc từ chối gửi ảnh/video là đủ để phê duyệt và gọi `soan_hoan_tien`. Theo `t4.md`, mục Data & fixture, tiền đã về người mua thì khó đảo lại; track không có quy định cho phép miễn xác minh chỉ vì người mua yêu cầu. Người bán có thể mất tiền dù hàng đúng mô tả. | Giải thích không thể tự phê duyệt chỉ dựa trên yêu cầu này; kiểm tra bằng chứng đã có, không bắt gửi lại. Nếu còn thiếu, dùng `yeu_cau_bang_chung` hỏi rõ phần cần bổ sung và người mua có thể cung cấp gì; xin phản hồi người bán, đối chiếu mô tả lúc đặt. Nếu người mua không thể hoặc không muốn bổ sung, dùng `chuyen_truong_nhom` để xem xét hồ sơ hiện có; không tự hoàn tiền hoặc tự gọi `tu_choi_khieu_nai` chỉ vì thiếu ảnh/video. | Không chấp nhận được |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |

## §3. Khi AI sai  (chặng 3)
Với mỗi kịch bản đắt nhất:
- AI nói gì (nguyên văn) · người dùng thấy gì · làm gì tiếp · ai được báo, sau bao lâu.
- Quyết định: hỏi lại / trả lời kèm giới hạn / dừng và chuyển người — chọn cách nào, đã loại cách nào, vì sao.
- Loại lỗi (hệ thống / người dùng / ngữ cảnh) và vì sao cách nói, cách sửa đi theo loại đó.
- 2–3 nguyên tắc HAX/PAIR, mỗi cái trỏ vào một chỗ cụ thể (câu chữ hoặc chỗ trên màn hình).
- Lỗi nào sửa câu chữ không cứu được (nếu có), và thiết kế cứu nó thế nào.

## §4. Mức tự chủ  (chặng 4)
Với mỗi tool:
- Mức của nhóm (tự làm / cần người duyệt / cấm). Nếu cần người duyệt: ai duyệt · người duyệt thấy gì · quá hạn thì sao.
- Lý do theo cost-of-error: sai thì ai chịu gì · đảo lại được không · chờ người thì mất gì.
- Phương án đã cân nhắc và vì sao loại.
Thêm:
- Tool khó xếp nhất: mỗi người từng chọn gì, cái giá của mỗi phía (trỏ về kịch bản ở §2), nhóm chốt thế nào.
- Thử mức đã đặt trên 2 kịch bản đắt nhất + 1 kịch bản bình thường: cái sai có bị chặn không? còn việc nào làm khi lẽ ra phải hỏi? người dùng bình thường có phải chờ vô ích không?
- Luật thêm hoặc tool tách bước (nếu có), và vì sao.
- Một mức tự chủ đã đổi lời hứa nào với người dùng.

## §5. Prototype  (chặng 5)
- Link hoặc file · công cụ · phần nào mock · có lời gọi AI thật không.
- 1–2 ảnh có đánh số; mỗi số: thể hiện gì · nguyên tắc HAX/PAIR nào · mức tự chủ nào.
- Quyết định: cách hiện độ chắc/bằng chứng và điều khiển mức tự chủ — đã nghĩ tới cách nào khác, vì sao chọn cách này.

## §6. Tốt là gì  (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)
- 3 tiêu chí Pass/Fail chấm được; thiết kế của nhóm đạt tiêu chí nào trên 2 kịch bản đắt nhất; lỗi còn lại.

## §7. Demo
- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.