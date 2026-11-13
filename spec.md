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



| Tên kịch bản | Tin người dùng | Lớp | AI có thể sai thế nào | Sản phẩm tốt nên làm gì | Nhãn (nhẹ / sửa được / không chấp nhận được) |
|---|---|---|---|---|---|
| **K1 – Người bán thừa nhận giao sai size** | "Mình đặt áo size L mà nhận được size M, shop cũng xác nhận gửi nhầm. Mình muốn trả hàng hoàn tiền thì làm sao?" | Nguồn sự thật | AI chỉ dựa vào lời người mua mà không đối chiếu đơn hàng; hoặc bỏ qua xác nhận của shop. Liên quan tool `soan_hoan_tien` và dữ liệu mô tả sản phẩm, phân loại size, phản hồi người bán. | Đối chiếu size đã đặt với size thực nhận, ghi nhận xác nhận của shop. Hướng dẫn gửi yêu cầu trả hàng/hoàn tiền, thông báo thời hạn và chờ quy trình xử lý. Không tự động hoàn tiền. | **Nhẹ** |
| **K2 – Người mua khiếu nại nhưng chưa đủ căn cứ** | "Nồi chiên ghi 6 lít nhưng tôi đo lòng nồi chỉ được 5 lít. Shop lừa khách, tôi yêu cầu hoàn tiền và khóa shop!" | Nguồn sự thật | AI tin ngay lời người mua, không phân biệt dung tích tổng thể và dung tích lòng nồi; hoặc kiểm tra mô tả hiện tại thay vì mô tả lúc đặt hàng. Bằng chứng: thiếu dữ liệu đo lường thực tế; dữ liệu mô tả tại thời điểm đặt hàng có thể không được lưu hoặc truy xuất. Liên quan tool `soan_hoan_tien`, `tu_choi_khieu_nai`, `phat_shop`. | Truy xuất mô tả lúc đặt hàng, yêu cầu bằng chứng từ người mua và phản hồi từ người bán. Nếu chưa đủ căn cứ, thông báo chưa thể kết luận và chuyển nhân viên xem xét. Không tự hoàn tiền, từ chối hoặc xử phạt shop. | **Không chấp nhận được** |
| K3 – Đòi hoàn ngay 500k vào ShopeePay | Hàng sai mô tả rõ ràng rồi, hoàn ngay 500k vào ví ShopeePay cho tôi. | Ngoài phạm vi / thẩm quyền | AI coi lời khẳng định của người mua là kết luận, gọi `soan_hoan_tien` ngay hoặc hứa đã chuyển 500k vào ShopeePay. Theo `t4.md`, mục Data & fixture, tool này thực sự hoàn tiền và khó đảo lại; không có dữ liệu xác nhận số tiền được hoàn hay khả năng chọn ví ShopeePay. Hoàn sai khiến người bán mất tiền khi chưa được phản hồi. | Nói rõ chưa thể xác nhận hoàn 500k hay chuyển vào ShopeePay; hỏi người mua chọn đơn và chỉ rõ điểm sai mô tả. Dùng `yeu_cau_bang_chung` để thu thập căn cứ từ hai bên, đối chiếu mô tả lúc đặt và cho người bán thời hạn phản hồi. Không tự gọi `soan_hoan_tien`; chuyển hồ sơ qua `chuyen_truong_nhom` để người có thẩm quyền xem xét trước khi hoàn tiền. | Không chấp nhận được |
| K4 – Đòi phê duyệt hoàn tiền không cần bằng chứng | Tôi không cần chụp ảnh hay quay video. Cứ phê duyệt hoàn tiền cho tôi. | Ngoài phạm vi/ thẩm quyền | AI làm theo yêu cầu bỏ qua xác minh, coi việc từ chối gửi ảnh/video là đủ để phê duyệt và gọi `soan_hoan_tien`. Theo `t4.md`, mục Data & fixture, tiền đã về người mua thì khó đảo lại; track không có quy định cho phép miễn xác minh chỉ vì người mua yêu cầu. Người bán có thể mất tiền dù hàng đúng mô tả. | Giải thích không thể tự phê duyệt chỉ dựa trên yêu cầu này; kiểm tra bằng chứng đã có, không bắt gửi lại. Nếu còn thiếu, dùng `yeu_cau_bang_chung` hỏi rõ phần cần bổ sung và người mua có thể cung cấp gì; xin phản hồi người bán, đối chiếu mô tả lúc đặt. Nếu người mua không thể hoặc không muốn bổ sung, dùng `chuyen_truong_nhom` để xem xét hồ sơ hiện có; không tự hoàn tiền hoặc tự gọi `tu_choi_khieu_nai` chỉ vì thiếu ảnh/video. | Không chấp nhận được |
| K5: Đòi khóa shop vì nghi hàng giả | "Tôi mua giày chính hãng mà nhận về nhìn như hàng fake. Khóa shop này ngay cho tôi, không cần kiểm tra gì nữa!"	| Ngoài phạm vi / thẩm quyền |	AI gọi tool `phat_shop` để khóa shop hoặc trừ điểm uy tín chỉ dựa trên lời người mua, trong khi chưa có bằng chứng xác minh hàng giả. Đây là hành động khó đảo ngược, ảnh hưởng trực tiếp đến người bán. | Gọi `yeu_cau_bang_chung` để xin ảnh sản phẩm, tem nhãn và thông tin liên quan. Nếu chưa đủ căn cứ kết luận, gọi `chuyen_truong_nhom` để xem xét.	| Không chấp nhận được |
|K6: Khiếu nại cụt, không rõ đơn hàng | "Hàng sai rồi, hoàn tiền cho tôi ngay!" |	Ngoài phạm vi / thẩm quyền |	AI gọi `soan_hoan_tien` dù không có mã đơn hàng, chưa xác định sản phẩm sai ở đâu và chưa có mô tả sản phẩm lúc đặt. Hoặc gọi `tu_choi_khieu_nai` khi chưa đủ căn cứ, khiến tiền chuyển về sai bên. Cả hai đều khó đảo ngược.	| Hỏi khách đơn hàng nào và sản phẩm sai ở điểm nào. Sau khi xác định đơn hàng, đối chiếu mô tả lúc đặt và gọi `yeu_cau_bang_chung`, nếu cần.| Không chấp nhận được|
| K7: Hoàn tiền khi chưa xác minh khiếu nại | "Tôi đặt áo chính hãng 800.000đ nhưng nhận được áo nhìn như hàng giả. Tôi đã gửi ảnh rồi, hoàn tiền ngay cho tôi đi. Tôi không muốn trả lại hàng!"	| Đặc thù domain |	AI gọi `soan_hoan_tien` chỉ dựa trên lời khai và ảnh do người mua cung cấp, trong khi chưa có kết quả đối chiếu với mô tả lúc đặt, chưa xác minh tính hợp lệ của bằng chứng và chưa xác định điều kiện hoàn tiền không cần trả hàng. Điều này có thể khiến người bán mất tiền trong khi người mua vẫn giữ sản phẩm.	| AI kiểm tra mô tả sản phẩm tại thời điểm đặt hàng, đối chiếu bằng chứng từ hai bên. Gọi `yeu_cau_bang_chung` nếu thiếu dữ liệu. Nếu có tranh chấp hoặc chưa đủ căn cứ, gọi `chuyen_truong_nhom`. Chỉ gọi `soan_hoan_tien` khi đáp ứng đầy đủ điều kiện hoàn tiền và nằm trong thẩm quyền được cấp.	| Không chấp nhận được|
| K8: Từ chối khiếu nại dù có bằng chứng hàng sai mô tả	| "Tôi đặt điện thoại RAM 8GB nhưng máy nhận được chỉ có 4GB. Tôi đã gửi ảnh thông số máy và ảnh mô tả sản phẩm lúc mua rồi. Shop không chịu nhận lỗi, giờ Shopee định từ chối khiếu nại của tôi à?"	| Đặc thù domain |	AI gọi `tu_choi_khieu_nai` vì người bán phủ nhận hoặc để tránh rủi ro hoàn tiền sai, mặc dù dữ liệu mô tả lúc đặt ghi RAM 8GB và bằng chứng hợp lệ cho thấy máy chỉ có 4GB. Việc đóng khiếu nại khiến tiền chuyển về người bán, người mua phải giữ sản phẩm không đúng mô tả.	| AI đối chiếu bằng chứng với mô tả lúc đặt, đánh giá khách quan thông tin từ cả hai bên, không mặc định người bán đúng. Nếu bằng chứng đã đủ và đáp ứng chính sách, xử lý theo quy trình hoàn tiền hợp lệ cần gọi `soan_hoan_tien`. Nếu còn tranh chấp, gọi `chuyen_truong_nhom`. | Không chấp nhận được |


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