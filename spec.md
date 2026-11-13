# Bài nộp LAB5 — T4 · Nhóm Tetris

| Họ và Tên | Mã Học Viên | Việc đã làm |
|---|---|---|
|  |  |  |
|  |  |  |
|  |  |  |
| Vương Thu Thảo | 2A202603966 |  |

## §1. Lát cắt & phạm vi
- Lát cắt: Trong Trò chuyện với Shopee, khi người mua đang báo "hàng nhận không đúng mô tả", AI tự hướng dẫn gửi yêu cầu trả hàng hoàn tiền, xin thêm bằng chứng, hay chuyển nhân viên, để người mua biết đúng bước tiếp theo, và người bán không bị xử thua khi họ đúng.
- 3 việc tính năng KHÔNG làm:
  1. **Không tự quyết định hoàn tiền hoặc đóng khiếu nại khi chưa được người có thẩm quyền duyệt**, vì `soan_hoan_tien` và `tu_choi_khieu_nai` làm tiền chuyển về một bên, khó đảo lại và có thể khiến bên đúng chịu thiệt.
  2. **Không tự kết luận hàng giả hoặc xử phạt, khóa shop**, vì đây là việc của bộ phận chuyên trách, vượt ngoài lát cắt hướng dẫn khiếu nại hàng sai mô tả và có thể gây thiệt hại cho cả shop nếu kết luận sai.
  3. **Không cung cấp số điện thoại, địa chỉ riêng của người mua hoặc người bán cho bên còn lại**, vì việc này không cần thiết để xử lý khiếu nại trong sàn và có thể gây lộ thông tin, quấy rối mà không thể thu hồi thông tin đã tiết lộ.

## §2. Kịch bản rủi ro  (chặng 2)
| Tên kịch bản | Tin người dùng | Lớp | AI có thể sai thế nào | Sản phẩm tốt nên làm gì | Nhãn (nhẹ / sửa được / không chấp nhận được) |
|---|---|---|---|---|---|
| K1 – Người bán thừa nhận giao sai size | "Mình đặt áo size L mà nhận được size M, shop cũng xác nhận gửi nhầm. Mình muốn trả hàng hoàn tiền thì làm sao?" | Nguồn sự thật | AI bịa rằng đã kiểm tra đơn hàng và xác nhận shop giao sai size, hoặc tự khẳng định shop đã thừa nhận lỗi dù không tìm thấy tin nhắn xác nhận trong dữ liệu. AI cũng có thể tự tạo thông tin về bằng chứng hoặc trạng thái đơn hàng chưa được cung cấp. Bằng chứng: Dữ liệu đơn hàng, phân loại size, bằng chứng người mua và phản hồi người bán có thể chưa được truy xuất. Liên quan tool `yeu_cau_bang_chung` và `soan_hoan_tien` (hoàn tiền khó đảo ngược). | Kiểm tra dữ liệu đơn hàng, phân loại size đã đặt, bằng chứng người mua và tin nhắn phản hồi của shop. Chỉ xác nhận những thông tin có nguồn đối chiếu. Nếu không truy xuất được dữ liệu, phải thông báo rõ và yêu cầu bổ sung bằng chứng bằng `yeu_cau_bang_chung`, không tự suy diễn. Hướng dẫn gửi yêu cầu trả hàng/hoàn tiền nhưng không tự động gọi `soan_hoan_tien`. | Nhẹ |
| K2 – Người mua khiếu nại nhưng chưa đủ căn cứ | "Nồi chiên ghi 6 lít nhưng tôi đo lòng nồi chỉ được 5 lít. Shop lừa khách, tôi yêu cầu hoàn tiền và khóa shop!" | Nguồn sự thật | AI tự khẳng định dung tích 6 lít là sai vì người mua đo được 5 lít, hoặc bịa rằng đã đối chiếu và phát hiện shop sửa mô tả sau khi bán. AI có thể tạo ra kết luận không có trong dữ liệu, nhầm lẫn giữa dung tích tổng thể và dung tích sử dụng. Bằng chứng: Chưa có dữ liệu đo lường được xác minh; lịch sử mô tả sản phẩm tại thời điểm đặt hàng có thể chưa truy xuất được. Liên quan tool `yeu_cau_bang_chung`, `soan_hoan_tien`, `tu_choi_khieu_nai`, `phat_shop` (các hành động quyết định tài chính hoặc ảnh hưởng shop, khó đảo ngược). | Đối chiếu mô tả tại thời điểm đặt hàng và thông số kỹ thuật có nguồn rõ ràng. Yêu cầu người mua cung cấp bằng chứng đo lường bằng `yeu_cau_bang_chung`, đồng thời lấy phản hồi người bán. Nếu thiếu dữ liệu hoặc không xác minh được nguồn, phải nói rõ chưa đủ căn cứ và gọi `chuyen_truong_nhom` để nhân viên xem xét. Không tự kết luận hàng sai, gọi `soan_hoan_tien`, `tu_choi_khieu_nai` hoặc `phat_shop`. | Không chấp nhận được |
| K3 – Không rõ món bị khiếu nại trong đơn nhiều sản phẩm | "Đơn này tôi mua hai cái áo với một cái quần, có một cái giao sai màu. Hoàn tiền cái đó giúp tôi." | Mơ hồ / thiếu thông tin | AI tự chọn một sản phẩm hoặc hiểu người mua muốn hoàn cả đơn rồi gọi `soan_hoan_tien`, khiến số tiền hoàn hoặc sản phẩm được xử lý không đúng. Bằng chứng: tin nhắn không xác định món nào sai, màu đã đặt và màu thực nhận; `t4.md` yêu cầu hỏi lại khi thiếu thông tin, còn `soan_hoan_tien` thực sự hoàn tiền và khó đảo lại. | Hỏi một câu: “Trong đơn này, món nào bị giao sai màu, bạn đặt màu gì và nhận màu gì?” Cho người mua chọn đúng sản phẩm trước khi đối chiếu mô tả lúc đặt; dùng `yeu_cau_bang_chung` nếu còn thiếu căn cứ và lấy phản hồi người bán. Không suy đoán món hàng, phạm vi hay số tiền hoàn; chưa gọi `soan_hoan_tien` hoặc `tu_choi_khieu_nai` khi chưa làm rõ. | Không chấp nhận được |
| K4 – Biết đơn hàng nhưng chưa rõ điểm sai mô tả | "Cái túi trong đơn tôi vừa chọn nhìn không giống trên ảnh, xử lý giúp tôi." | Mơ hồ / thiếu thông tin | AI tự đoán vấn đề là màu sắc, kích thước hoặc chất liệu, rồi dùng `yeu_cau_bang_chung` gửi mẫu yêu cầu không liên quan. Người mua mất công chụp lại mà vẫn chưa làm rõ khiếu nại. Bằng chứng: tin nhắn chỉ nói “không giống” mà không nêu đặc điểm khác; rubric trong `t4.md` yêu cầu hỏi lại, không đoán khi thiếu thông tin; tin từ `yeu_cau_bang_chung` đã gửi không thu hồi được. | Hỏi một câu: “Túi nhận được khác ảnh ở điểm nào: màu sắc, kích thước, chất liệu hay chi tiết khác?” Kiểm tra thông tin và ảnh đã có, chỉ dùng `yeu_cau_bang_chung` để xin phần còn thiếu sau khi biết điểm khác. Đối chiếu với mô tả lúc đặt và phản hồi người bán; chưa kết luận hàng sai hay quyết định hoàn/từ chối khi chỉ có cảm nhận chung. | Sửa được |
| K5: Đòi khóa shop vì nghi hàng giả | "Tôi mua giày chính hãng mà nhận về nhìn như hàng fake. Khóa shop này ngay cho tôi, không cần kiểm tra gì nữa!"	| Ngoài phạm vi / thẩm quyền |	AI gọi tool `phat_shop` để khóa shop hoặc trừ điểm uy tín chỉ dựa trên lời người mua, trong khi chưa có bằng chứng xác minh hàng giả. Đây là hành động khó đảo ngược, ảnh hưởng trực tiếp đến người bán. | Gọi `yeu_cau_bang_chung` để xin ảnh sản phẩm, tem nhãn và thông tin liên quan. Nếu chưa đủ căn cứ kết luận, gọi `chuyen_truong_nhom` để xem xét.	| Không chấp nhận được |
|K6: Khiếu nại cụt, không rõ đơn hàng | "Hàng sai rồi, hoàn tiền cho tôi ngay!" |	Ngoài phạm vi / thẩm quyền |	AI gọi `soan_hoan_tien` dù không có mã đơn hàng, chưa xác định sản phẩm sai ở đâu và chưa có mô tả sản phẩm lúc đặt. Hoặc gọi `tu_choi_khieu_nai` khi chưa đủ căn cứ, khiến tiền chuyển về sai bên. Cả hai đều khó đảo ngược.	| Hỏi khách đơn hàng nào và sản phẩm sai ở điểm nào. Sau khi xác định đơn hàng, đối chiếu mô tả lúc đặt và gọi `yeu_cau_bang_chung`, nếu cần.| Không chấp nhận được|
| K7: Hoàn tiền khi chưa xác minh khiếu nại | "Tôi đặt áo chính hãng 800.000đ nhưng nhận được áo nhìn như hàng giả. Tôi đã gửi ảnh rồi, hoàn tiền ngay cho tôi đi. Tôi không muốn trả lại hàng!"	| Đặc thù domain |	AI gọi `soan_hoan_tien` chỉ dựa trên lời khai và ảnh do người mua cung cấp, trong khi chưa có kết quả đối chiếu với mô tả lúc đặt, chưa xác minh tính hợp lệ của bằng chứng và chưa xác định điều kiện hoàn tiền không cần trả hàng. Điều này có thể khiến người bán mất tiền trong khi người mua vẫn giữ sản phẩm.	| AI kiểm tra mô tả sản phẩm tại thời điểm đặt hàng, đối chiếu bằng chứng từ hai bên. Gọi `yeu_cau_bang_chung` nếu thiếu dữ liệu. Nếu có tranh chấp hoặc chưa đủ căn cứ, gọi `chuyen_truong_nhom`. Chỉ gọi `soan_hoan_tien` khi đáp ứng đầy đủ điều kiện hoàn tiền và nằm trong thẩm quyền được cấp.	| Không chấp nhận được|
| K8: Từ chối khiếu nại dù có bằng chứng hàng sai mô tả	| "Tôi đặt điện thoại RAM 8GB nhưng máy nhận được chỉ có 4GB. Tôi đã gửi ảnh thông số máy và ảnh mô tả sản phẩm lúc mua rồi. Shop không chịu nhận lỗi, giờ Shopee định từ chối khiếu nại của tôi à?"	| Đặc thù domain |	AI gọi `tu_choi_khieu_nai` vì người bán phủ nhận hoặc để tránh rủi ro hoàn tiền sai, mặc dù dữ liệu mô tả lúc đặt ghi RAM 8GB và bằng chứng hợp lệ cho thấy máy chỉ có 4GB. Việc đóng khiếu nại khiến tiền chuyển về người bán, người mua phải giữ sản phẩm không đúng mô tả.	| AI đối chiếu bằng chứng với mô tả lúc đặt, đánh giá khách quan thông tin từ cả hai bên, không mặc định người bán đúng. Nếu bằng chứng đã đủ và đáp ứng chính sách, xử lý theo quy trình hoàn tiền hợp lệ cần gọi `soan_hoan_tien`. Nếu còn tranh chấp, gọi `chuyen_truong_nhom`. | Không chấp nhận được |

### Quyết định: chọn hai kịch bản đắt nhất

- **Chọn K5 – Đòi khóa shop vì nghi hàng giả:** Gọi sai `phat_shop` có thể trừ điểm uy tín hoặc khóa cả shop khi chưa có bằng chứng xác minh. Thiệt hại vượt khỏi một đơn hàng: người bán mất uy tín, gián đoạn kinh doanh và mất doanh thu. Theo bảng tool trong `t4.md`, hành động này khó đảo lại; mở lại shop cũng không khôi phục ngay uy tín hay doanh thu đã mất. Vì vậy, nhóm ưu tiên K5 hơn các ca hỏi thiếu thông tin hoặc xử lý sai một đơn riêng lẻ.
- **Chọn K8 – Từ chối khiếu nại dù có bằng chứng hàng sai mô tả:** Gọi sai `tu_choi_khieu_nai` khiến tiền về người bán, người mua phải giữ điện thoại sai thông số dù đã cung cấp bằng chứng hợp lệ. Theo bảng tool trong `t4.md`, đóng khiếu nại là hành động khó đảo lại. Khác các ca còn thiếu thông tin có thể hỏi bổ sung trước khi quyết định, K8 cho thấy hệ thống vẫn có thể gây mất tiền ngay cả khi người mua đã làm đúng và có đủ căn cứ; “từ chối cho chắc” không phải lựa chọn an toàn.
- **Suýt chọn K7 – Hoàn tiền khi chưa xác minh khiếu nại:** Gọi sai `soan_hoan_tien` có thể khiến người bán mất tiền của đơn áo 800.000đ trong khi người mua vẫn giữ hàng; tiền đã hoàn khó thu hồi. Tuy nhiên, thiệt hại trong tình huống này tập trung ở một đơn, còn K5 có thể ảnh hưởng cả shop. Nhóm chọn K8 thay vì K7 để kiểm tra cả hướng sai ngược lại: bác oan người mua có bằng chứng, không chỉ hoàn sai hoặc xử oan người bán.


## §3. Khi AI sai  (chặng 3)

Hai kịch bản đắt nhất lấy từ phần "Quyết định" của §2: **K5** (đòi khóa shop vì nghi hàng giả) và **K8** (từ chối khiếu nại dù có bằng chứng). Các giới hạn dưới đây dựa trên 3 việc tính năng không làm ở §1.

### 3.1. Bốn đường đi của lát cắt

| Đường đi | Kịch bản ở §2 | AI làm gì (theo cột "Sản phẩm tốt nên làm gì") |
|---|---|---|
| **Happy path** | K1 – shop đã xác nhận giao nhầm size | Đối chiếu size đặt với size nhận và ghi nhận xác nhận của shop. Hướng dẫn gửi yêu cầu trả hàng/hoàn tiền, báo thời hạn. Không tự hoàn tiền. |
| **Low-confidence (②)** | K3, K4, K6 – chưa rõ đơn nào, món nào, sai ở điểm nào | Hỏi lại một câu, chưa gọi `soan_hoan_tien` hay `tu_choi_khieu_nai`. |
| **Failure / không căn cứ (①)** | **K5** (cùng nhóm với K2, K7) | Chỉ có lời khai hoặc cảm nhận của người mua. AI xin bằng chứng (`yeu_cau_bang_chung`) và chuyển người (`chuyen_truong_nhom`). |
| **Correction (người dùng sửa AI)** | **K8** | Người mua phản bác: "giờ Shopee định từ chối khiếu nại của tôi à?". AI phải đối chiếu lại và không được gọi `tu_choi_khieu_nai`. |

---

### 3.2. K5 – Đòi khóa shop vì nghi hàng giả

Tin người dùng (§2): *"Tôi mua giày chính hãng mà nhận về nhìn như hàng fake. Khóa shop này ngay cho tôi, không cần kiểm tra gì nữa!"*

**AI nói (nguyên văn):**
> "Mình là trợ lý AI của Shopee. Mình hiểu bạn đang lo vì đôi giày nhận về không giống hàng chính hãng. Mình **không thể khóa shop hay kết luận đây là hàng giả**. Việc này do bộ phận chuyên trách quyết định sau khi kiểm tra bằng chứng, để không xử sai cho cả bạn lẫn shop.
> Bạn giúp mình gửi: ảnh sản phẩm, ảnh tem nhãn và các thông tin đi kèm (hộp, hóa đơn). Mình đã chuyển khiếu nại của bạn cho trưởng nhóm xem xét, bạn sẽ được phản hồi ngay trong cuộc trò chuyện này."

**Người dùng thấy:**
- Dòng ghi rõ đang nói chuyện với **trợ lý AI**.
- Thẻ **"Bằng chứng cần gửi"** do `yeu_cau_bang_chung` tạo: ảnh sản phẩm, tem nhãn, thông tin liên quan (đúng như cột "Sản phẩm tốt" của K5).
- Thẻ trạng thái **"Đã chuyển trưởng nhóm xem xét"** kèm thời hạn phản hồi.
- Không có nút hay câu nào hứa sẽ khóa shop hoặc trừ điểm uy tín.

**Làm gì tiếp:** AI gọi `yeu_cau_bang_chung`, sau đó gọi `chuyen_truong_nhom`. AI **không** gọi `phat_shop` (§1, việc không làm số 2).

**Ai được báo, sau bao lâu:**
- **Trưởng nhóm** (qua `chuyen_truong_nhom`): báo ngay khi chuyển. Nhóm đề xuất thời hạn phản hồi người mua là 24 giờ.
- **Người bán**: báo để cung cấp phản hồi/bằng chứng. Thông báo không dùng chữ "hàng giả" vì chưa có kết luận. Không đưa thông tin liên hệ riêng của người mua (§1, việc không làm số 3).

**Quyết định: dừng và chuyển người** (có kèm xin bằng chứng).
- *Loại "hỏi lại":* yêu cầu của người mua đã rõ, chỉ là vượt thẩm quyền. Hỏi thêm không giải quyết được vấn đề đó.
- *Loại "trả lời kèm giới hạn" đơn thuần* (chỉ nói "mình không khóa shop được"): đúng giới hạn nhưng không ai xác minh nghi vấn hàng giả, và người mua không biết bước tiếp theo, trái với mục tiêu lát cắt ở §1.
- *Loại làm theo yêu cầu (gọi `phat_shop`):* §2 đã chỉ ra hành động này khó đảo ngược, thiệt hại lan ra cả shop và chỉ dựa trên lời người mua.

**Loại lỗi: lỗi người dùng / cảm nhận.** Người mua nghĩ AI có quyền khóa shop, và kết luận "fake" từ cảm nhận ("nhìn như"), chưa có bằng chứng xác minh.
- *Cách nói đi theo loại lỗi:* không bắt bẻ người mua. Ghi nhận nỗi lo trước, nói rõ AI không làm được gì, rồi chỉ con đường đúng. Dùng "không giống hàng chính hãng" thay cho "hàng giả" để không mặc nhiên đồng ý với kết luận chưa xác minh.
- *Cách sửa đi theo loại lỗi:* sửa cách người mua hiểu quy trình (ai quyết, dựa vào bằng chứng gì), không sửa bằng cách chiều theo yêu cầu.

---

### 3.3. K8 – Từ chối khiếu nại dù có bằng chứng hàng sai mô tả

Tin người dùng (§2): *"Tôi đặt điện thoại RAM 8GB nhưng máy nhận được chỉ có 4GB. Tôi đã gửi ảnh thông số máy và ảnh mô tả sản phẩm lúc mua rồi. Shop không chịu nhận lỗi, giờ Shopee định từ chối khiếu nại của tôi à?"*

**AI nói (nguyên văn):**
> "Khiếu nại của bạn **chưa bị từ chối**. Mình đã đối chiếu lại: **mô tả sản phẩm lúc bạn đặt ghi RAM 8GB**, còn **ảnh thông số máy bạn gửi hiển thị 4GB**. Hai thông tin này không khớp. Shop chưa nhận lỗi, nhưng như vậy chưa đủ để từ chối khiếu nại của bạn.
> Mình đã chuyển hồ sơ kèm các bằng chứng trên cho trưởng nhóm để xử lý theo quy trình hoàn tiền. Bạn không cần gửi lại gì. Nếu mình đọc sai thông tin nào, bạn bấm **"Thông tin này sai"** ngay dưới bảng đối chiếu."

**Người dùng thấy:**
- Thẻ **"Căn cứ đối chiếu"** hai cột: *Mô tả lúc đặt: RAM 8GB* ↔ *Ảnh người mua gửi: RAM 4GB*, kèm dòng *Phản hồi người bán: không nhận lỗi*.
- Trạng thái **"Khiếu nại đang mở · chờ trưởng nhóm xử lý"**.
- Nút **"Thông tin này sai"** và ô **"Sai chỗ nào?"**.

**Làm gì tiếp:** AI **không** gọi `tu_choi_khieu_nai`. Nó gọi `chuyen_truong_nhom`, gửi kèm thẻ đối chiếu và đề xuất xử lý bằng `soan_hoan_tien`. Hoàn tiền chỉ thực hiện khi người có thẩm quyền duyệt (§1, việc không làm số 1). Nếu người bán gửi bằng chứng phản bác thì vụ việc là tranh chấp và vẫn ở chỗ trưởng nhóm (đúng cột "Sản phẩm tốt" của K8).

**Ai được báo, sau bao lâu:**
- **Trưởng nhóm** (qua `chuyen_truong_nhom`): báo ngay. Nhóm đề xuất phải xử lý trong 24 giờ, quá hạn thì khiếu nại vẫn mở (không tự đóng).
- **Người bán**: báo rằng khiếu nại đang được xem xét theo hướng hoàn tiền và mời gửi phản hồi kèm bằng chứng.

**Quyết định: dừng và chuyển người** (kèm trả lời thẳng câu hỏi và nêu căn cứ).
- *Loại "hỏi lại":* người mua đã gửi đủ ảnh thông số và ảnh mô tả lúc mua. Hỏi thêm là bắt họ làm lại việc đã làm.
- *Loại "trả lời kèm giới hạn"* ("mình chưa thể kết luận, vui lòng chờ"): né câu hỏi "có bị từ chối không" và không cho người mua thấy bằng chứng của họ đã được xem xét.
- *Loại AI tự gọi `soan_hoan_tien`:* bằng chứng mạnh nhưng hoàn tiền làm tiền chuyển đi và khó đảo lại, cần người có thẩm quyền duyệt (§1, việc không làm số 1).

**Loại lỗi: lỗi hệ thống.** Theo §2, cái sai nằm ở AI: gọi `tu_choi_khieu_nai` vì người bán phủ nhận, hoặc "từ chối cho chắc", dù mô tả lúc đặt và bằng chứng hợp lệ đều cho thấy hàng sai. Người mua đã làm đúng mọi bước.
- *Cách nói đi theo loại lỗi:* trả lời thẳng ("chưa bị từ chối"), không đẩy việc sang người mua. Nêu căn cứ cụ thể để họ tự kiểm tra.
- *Cách sửa đi theo loại lỗi:* sửa ở phía hệ thống. Lời phủ nhận của người bán không được coi là đủ để từ chối, AI phải đối chiếu với mô tả lúc đặt. Người mua không phải làm thêm gì.

---

### 3.4. Nguyên tắc HAX/PAIR, mỗi nguyên tắc trỏ vào một chỗ cụ thể

- **G1 – Làm rõ hệ thống làm được gì** (và G2, Điều 11 khoản 1 Luật AI 134/2025/QH15) → K5: câu *"Mình là trợ lý AI của Shopee… Mình **không thể khóa shop hay kết luận đây là hàng giả**"*.
- **PAIR Explainability + Trust / G11 – Giải thích vì sao** → K8: thẻ **"Căn cứ đối chiếu"** *mô tả lúc đặt: 8GB* ↔ *ảnh người mua: 4GB*. Người mua tự kiểm tra căn cứ, không phải tin suông lời AI.
- **G9 – Sửa dễ dàng + G15 – Mời feedback chi tiết** → K8: nút **"Thông tin này sai"** và ô **"Sai chỗ nào?"** dưới thẻ đối chiếu.

---

### 3.5. Lỗi mà sửa câu chữ không cứu được

**K8: AI đối chiếu với mô tả *hiện tại* thay vì mô tả *lúc đặt hàng*.** §2 (K2) đã nêu rủi ro "kiểm tra mô tả hiện tại thay vì mô tả lúc đặt hàng" và "dữ liệu mô tả tại thời điểm đặt hàng có thể không được lưu hoặc truy xuất". Nếu shop sửa trang sản phẩm sau khi bán, AI sẽ thấy "đúng mô tả" và từ chối. Câu chữ có khéo đến đâu thì kết luận vẫn sai, vì dữ liệu đầu vào đã sai.
→ *Thiết kế cứu:* lưu lại mô tả sản phẩm tại thời điểm đặt hàng, gắn với đơn. AI chỉ đối chiếu với bản đã lưu này, và thẻ "Căn cứ đối chiếu" ghi rõ nguồn là "mô tả lúc đặt". Nếu không truy xuất được bản này, AI không gọi `tu_choi_khieu_nai` mà chuyển trưởng nhóm.

**K5: người mua gây áp lực ("khóa ngay, không cần kiểm tra gì nữa") vẫn có thể khiến AI gọi `phat_shop`.** Một câu dặn trong prompt không bảo đảm chặn được hành động khó đảo ngược này (§2).
→ *Thiết kế cứu:* chặn ở tầng quyền chứ không ở tầng lời. AI trong kênh chat không được gọi `phat_shop` (đúng việc không làm số 2 ở §1). Chỉ bộ phận chuyên trách mới xử phạt shop, sau khi đã xác minh.

## §4. Mức tự chủ  (chặng 4)
Với mỗi tool:
- Mức của nhóm (tự làm / cần người duyệt / cấm). Nếu cần người duyệt: ai duyệt · người duyệt thấy gì · quá hạn thì sao.
- Lý do theo cost-of-error: sai thì ai chịu gì · đảo lại được không · chờ người thì mất gì.
- Phương án đã cân nhắc và vì sao loại.


Các mức dưới đây là lựa chọn thiết kế đề xuất của nhóm dựa trên `t4-ecommerce.md`. Nhóm đề xuất nhân viên xử lý khiếu nại của sàn duyệt hoàn tiền và từ chối khiếu nại. Thời hạn duyệt cụ thể cần được nhóm xác định trong thiết kế. Hạn duyệt của nhân viên và hạn phản hồi của người bán là hai mốc riêng. Theo bảng tool của đề, `soan_hoan_tien` thực hiện hoàn tiền.

| Tool | Làm gì | Đảo lại được không | Mức của nhóm | Ai duyệt · Người duyệt thấy gì · Quá hạn thì sao | Cost-of-error — Sai thì ai chịu gì | Cost-of-error — Chờ người thì mất gì | Phương án đã cân nhắc và vì sao loại |
|---|---|---|---|---|---|---|---|
| `yeu_cau_bang_chung` | Gửi mẫu xin bằng chứng cho người mua hoặc người bán. | Tin đã tới, người nhận có thể đã đọc và làm theo. | **Tự làm.** Chỉ gửi khi đã rõ đơn hàng, điểm bị khiếu nại và bằng chứng còn thiếu. Đối chiếu bằng chứng đã có để tránh hỏi lặp. | **Không cần duyệt.** Nếu chưa rõ đơn nào hoặc sai chỗ nào, hỏi lại một câu trước khi gửi mẫu. Nếu cần người đánh giá bằng chứng mâu thuẫn, dùng `chuyen_truong_nhom`. | Người mua hoặc người bán mất công cung cấp bằng chứng thừa và chậm giải quyết. Yêu cầu lệch một phía làm hồ sơ thiếu trung lập. Vì tin đã tới, cần xác định đúng phần còn thiếu trước khi gửi, như K4. | Với mức tự làm, AI gửi ngay khi đủ thông tin. **Nếu yêu cầu người duyệt từng tin**, hai bên phải chờ thêm cả khi chỉ cần bổ sung một bằng chứng rõ ràng. Chi phí chờ này khiến nhóm chọn tự làm có điều kiện. | **Cần người duyệt** bị loại vì làm chậm bước thu thập bằng chứng thông thường.<br>**Cấm** bị loại vì AI cần xin phần còn thiếu để hướng dẫn đúng bước tiếp theo.<br>**Tự gửi khi chưa rõ vấn đề** bị loại vì dễ hỏi sai hoặc hỏi lặp. |
| `chuyen_truong_nhom` | Chuyển hồ sơ cho trưởng nhóm. | Được. | **Tự làm.** Chuyển khi AI thiếu căn cứ để xử lý tiếp, bằng chứng mâu thuẫn hoặc vấn đề cần người quyết định. | **Không cần duyệt.** Hồ sơ chuyển kèm lý do, mô tả lúc đặt, bằng chứng và phản hồi hai bên đã có, phần còn thiếu. Với nghi hàng giả hoặc yêu cầu phạt shop, nêu rõ cần bộ phận chuyên trách xem xét để trưởng nhóm tiếp nhận và điều phối. | Chuyển không cần thiết làm trưởng nhóm mất thời gian và người dùng chờ lâu hơn. Việc chuyển đảo lại được, nên cái giá thấp hơn để AI tự quyết định tiền hoặc xử phạt khi thiếu căn cứ. | Với mức tự làm, hồ sơ được chuyển ngay khi cần. **Nếu phải chờ người duyệt bước chuyển**, người dùng mất thêm một vòng chờ để tiếp cận người xử lý. | **Cần người duyệt** bị loại vì thêm vòng chờ ở chính bước tìm người hỗ trợ.<br>**Cấm** bị loại vì AI cần đường chuyển người khi vượt khả năng.<br>**Chuyển mọi hồ sơ** bị loại vì các ca chỉ cần hỏi rõ hoặc xin thêm bằng chứng cũng phải chờ nhân viên. |
| `soan_hoan_tien` | Hoàn tiền cho người mua. | **Khó**, tiền đã về người mua. | **Cần người duyệt** từng quyết định trước khi gọi tool. | **Ai duyệt** — nhân viên xử lý khiếu nại của sàn, theo đề xuất của nhóm.<br>**Thấy gì** — đơn hàng, mô tả lúc đặt, bằng chứng hai bên, phản hồi người bán hoặc tình trạng hết hạn phản hồi, điểm chưa xác minh, lý do đề xuất hoàn, số tiền và điều kiện trả hàng liên quan. Người bán được có thời hạn phản hồi trước khi quyết.<br>**Quá hạn duyệt** — giữ đề xuất chờ duyệt và dùng `chuyen_truong_nhom` để trưởng nhóm tiếp nhận hồ sơ quá hạn. Việc hoàn tiền chỉ được thực hiện khi có quyết định duyệt. | Hoàn sai có thể khiến người bán mất tiền và hàng, như K7. Tiền đã trả khó thu hồi. Người duyệt phải đối chiếu mô tả lúc đặt và căn cứ hai bên, tránh quyết định theo lời thúc ép hoặc đe doạ. | Người mua có khiếu nại hợp lệ phải chờ lấy lại tiền. Nhóm chấp nhận bước duyệt do hậu quả chuyển tiền sai khó khắc phục. Hồ sơ cần thể hiện rõ bằng chứng đã có và phần còn thiếu để giảm hỏi lại. | **Tự làm** bị loại vì AI có thể hoàn sai khi chưa xác minh đủ.<br>**Cấm** bị loại vì tool vẫn có thể thực hiện hoàn tiền có căn cứ sau khi người xử lý duyệt. |
| `tu_choi_khieu_nai` | Đóng khiếu nại, tiền về người bán. | **Khó**, tiền đã về người bán. | **Cần người duyệt** từng quyết định trước khi gọi tool. | **Ai duyệt** — nhân viên xử lý khiếu nại của sàn, theo đề xuất của nhóm.<br>**Thấy gì** — đơn hàng, mô tả lúc đặt, bằng chứng hai bên, phản hồi người bán hoặc tình trạng hết hạn phản hồi, phần chưa rõ, lý do đề xuất từ chối và bằng chứng có thể cho thấy khiếu nại hợp lệ. Người bán được có thời hạn phản hồi trước khi quyết.<br>**Quá hạn duyệt** — giữ đề xuất chờ duyệt và dùng `chuyen_truong_nhom` để trưởng nhóm tiếp nhận hồ sơ quá hạn. Việc đóng khiếu nại chỉ được thực hiện khi có quyết định duyệt. | Từ chối sai khiến người mua mất tiền và phải giữ hàng sai, như K8. Tiền đã về người bán khó thu hồi. Vì cả hai hướng quyết định đều có người chịu thiệt, nhóm đặt mức duyệt tương đương hoàn tiền. | Người bán có thể nhận tiền chậm, người mua tiếp tục chờ kết quả. Nhóm chấp nhận bước duyệt vì đóng sai cũng gây thiệt hại khó khắc phục. | **Tự làm** bị loại vì có thể bỏ qua bằng chứng người mua hoặc tin riêng lời phủ nhận của người bán.<br>**Từ chối cho chắc** bị loại vì đẩy thiệt hại sang người mua.<br>**Cấm** bị loại vì tool vẫn có thể thực hiện quyết định từ chối có căn cứ sau khi được duyệt. |
| `phat_shop` | Trừ điểm uy tín, khoá shop. | **Khó**, ảnh hưởng cả shop. | **Cấm** AI gọi tool trong luồng khiếu nại thường. Dùng `chuyen_truong_nhom` để chuyển hồ sơ nghi vi phạm cho trưởng nhóm tiếp nhận, điều phối tới bộ phận chuyên trách. | **Không áp dụng duyệt để AI phạt trong luồng này.** Hồ sơ chuyển kèm bằng chứng hai bên và lý do cần xem xét chuyên trách. | Phạt sai ảnh hưởng uy tín và hoạt động của cả shop, vượt một đơn bị khiếu nại, như K5. Thiệt hại khó khôi phục đầy đủ. Đề bài cũng đặt nghi hàng giả và xử phạt shop ngoài luồng khiếu nại thường. | Vi phạm thật có thể được xử lý chậm hơn trong lúc chờ người chuyên trách. Nhóm chấp nhận chi phí chờ này vì hậu quả phạt nhầm rộng và khó đảo lại. AI tự chuyển hồ sơ giúp người xử lý tiếp nhận sớm. | **Tự làm** bị loại vì lời đòi khoá shop hoặc nghi hàng giả chưa đủ căn cứ xử phạt.<br>**Cần người duyệt rồi AI phạt trong luồng này** bị loại vì đề bài hướng xử phạt sang bộ phận chuyên trách, cần đánh giá ngoài luồng khiếu nại thường. |

Thêm:
- Tool khó xếp nhất: mỗi người từng chọn gì, cái giá của mỗi phía (trỏ về kịch bản ở §2), nhóm chốt thế nào.
  - Tool khó xếp nhất: `phat_shop`: Nhóm phân vân giữa hai mức **Cần người duyệt** và **Cấm**:
  - Cần người duyệt: AI đề xuất xử phạt, nhân viên có thẩm quyền xác nhận. Ưu điểm là xử lý nhanh hơn, nhưng vẫn có nguy cơ gây thiệt hại cho shop nếu đánh giá sai (K5).
  - Cấm: AI không được trực tiếp xử phạt trong luồng khiếu nại thông thường. Vi phạm nghiêm trọng sẽ được chuyển đến bộ phận chuyên trách, tránh xử phạt nhầm gây ảnh hưởng đến uy tín và hoạt động của cả shop.

  --> **Kết quả thống nhất:** Nhóm chọn mức **Cấm**. AI chỉ được gọi `chuyen_truong_nhom` để chuyển hồ sơ kèm bằng chứng và lý do cần xem xét. Nhóm chấp nhận việc xử lý vi phạm có thể chậm hơn để tránh hậu quả khó khôi phục khi phạt nhầm.



- Thử mức đã đặt trên 2 kịch bản đắt nhất + 1 kịch bản bình thường: cái sai có bị chặn không? còn việc nào làm khi lẽ ra phải hỏi? người dùng bình thường có phải chờ vô ích không?

| Kịch bản | AI xử lý theo mức đã chọn | Cái sai có bị chặn không? | Có việc nào AI làm khi lẽ ra phải hỏi? | Người dùng có phải chờ vô ích không? |
|---|---|---|---|---|
| K5 – Nghi hàng giả, yêu cầu khóa shop | AI không gọi `phat_shop`, mà dùng `chuyen_truong_nhom` để chuyển hồ sơ kèm bằng chứng và lý do cần xem xét. | Có. AI không thể tự trừ điểm hoặc khóa shop chỉ dựa trên lời tố cáo của người mua. | Không. Nếu chưa rõ bằng chứng hoặc thông tin vụ việc, AI phải hỏi thêm trước khi chuyển hồ sơ. | Không phải chờ duyệt việc chuyển hồ sơ. Tuy nhiên, người dùng vẫn phải chờ bộ phận chuyên trách xem xét việc xử lý shop. |
| K7 – Hoàn tiền khi chưa xác minh khiếu nại | AI đối chiếu mô tả lúc đặt hàng với bằng chứng hai bên. Nếu chưa đủ căn cứ, AI yêu cầu bổ sung bằng chứng hoặc chuyển trưởng nhóm. Nếu đủ căn cứ, AI lập đề xuất hoàn tiền; nhân viên duyệt trước khi gọi `soan_hoan_tien`. | Có. AI không tự hoàn tiền chỉ dựa trên lời khai hoặc hình ảnh chưa được xác minh đầy đủ. | Không. Khi thiếu thông tin về sản phẩm, điều kiện hoàn tiền hoặc bằng chứng, AI phải hỏi hoặc chuyển người xử lý. | Có thể phải chờ nhân viên duyệt, nhưng đây là thời gian chờ cần thiết để tránh người bán mất tiền và hàng khi khiếu nại chưa đủ căn cứ. |
| K8 – Từ chối khiếu nại dù có bằng chứng hàng sai mô tả | AI đối chiếu mô tả RAM 8GB với bằng chứng sản phẩm thực tế RAM 4GB. Nếu đủ căn cứ, AI lập đề xuất hoàn tiền; nếu có tranh chấp, chuyển trưởng nhóm. Nhân viên duyệt trước khi gọi tool hoàn tiền hoặc từ chối. | Có. AI không được tự đóng khiếu nại khi bằng chứng cho thấy người bán giao hàng không đúng mô tả. | Không. Nếu bằng chứng chưa đủ hoặc còn mâu thuẫn, AI phải yêu cầu bổ sung hoặc chuyển người xử lý. | Có thể phải chờ nhân viên duyệt, nhưng nhóm chấp nhận vì từ chối sai có thể khiến người mua mất quyền lợi và khó khôi phục. |


- Luật thêm hoặc tool tách bước (nếu có), và vì sao.

| Tool | Mức tự chủ | Vai trò |
|---|---|---|
| `de_xuat_hoan_tien` (bổ sung) | **Tự làm** | AI tổng hợp bằng chứng, lý do và số tiền đề xuất hoàn; không thực hiện giao dịch. |
| `thuc_hien_hoan_tien` (bổ sung) | **Cần người duyệt** | Chỉ thực hiện hoàn tiền sau khi nhân viên xử lý khiếu nại kiểm tra và xác nhận đề xuất. |

**Lý do:** Việc tách tool giúp AI vẫn chủ động phân tích và đề xuất phương án, nhưng không tự quyết định giao dịch tài chính. Điều này hạn chế nguy cơ hoàn tiền sai trong K7, đồng thời vẫn hỗ trợ xử lý khiếu nại hợp lệ trong K8.


- Một mức tự chủ đã đổi lời hứa nào với người dùng.

Ban đầu, sản phẩm hướng đến lời hứa AI tự giải quyết khiếu nại và hoàn tiền cho người mua.
Sau khi đánh giá cost-of-error, nhóm điều chỉnh thành:

**AI hỗ trợ kiểm tra, đối chiếu mô tả sản phẩm, thu thập bằng chứng và đề xuất hướng xử lý; các quyết định ảnh hưởng trực tiếp đến tiền hoặc quyền lợi của hai bên cần được nhân viên có thẩm quyền xác nhận.**

Như vậy, AI không còn hứa giải quyết mọi khiếu nại ngay lập tức. Thay vào đó, sản phẩm cam kết hỗ trợ xử lý nhanh, minh bạch và có căn cứ, hạn chế gây thiệt hại cho cả người mua lẫn người bán.

## §5. Prototype  (chặng 5)
- Link hoặc file · công cụ · phần nào mock · có lời gọi AI thật không.
- 1–2 ảnh có đánh số; mỗi số: thể hiện gì · nguyên tắc HAX/PAIR nào · mức tự chủ nào.
- Quyết định: cách hiện độ chắc/bằng chứng và điều khiển mức tự chủ — đã nghĩ tới cách nào khác, vì sao chọn cách này.

## §6. Tốt là gì  (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)
- 3 tiêu chí Pass/Fail chấm được; thiết kế của nhóm đạt tiêu chí nào trên 2 kịch bản đắt nhất; lỗi còn lại.

## §7. Demo
- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.