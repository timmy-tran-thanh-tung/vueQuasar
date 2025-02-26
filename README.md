# Câu 1:
**- Cứ mở trực tiếp trên trình duyệt thôi ạ.**
**- Em tạo 3 khối: lấy Json, vẽ giao diện, highlight.**
    + Khối lấy Json: dùng fetch và lấy dialog từ jameslan.Json
    + Khối vẽ giao diện: em dùng chatGPT đưa code ghép các dialog (text) với nhau (có xử lí các dấu câu..)
    + Khối highlight em dựa trên nội dung của file json về timestamp, hỏi chatGPT code và style css vàng lên 

# Câu 2:
**- Cứ mở trực tiếp trên trình duyệt thôi ạ.**
**- Em tạo 5 khối chính: cấu hình API, gọi API, xử lý phản hồi, chạy chương trình, vẽ tin nhắn**
    + Khối cấu hình: vì groq yêu cầu 1 vài cấu hình, pass key... để lấy dữ liệu, em lưu vào đây
    + Khối gọi API: dùng fetch gọi API theo cấu hình đã làm
    + Khối xử lý phản hồi: kiểm tra thành công/ thất bại
    + Khối chạy chương trình: vì lí do fix lỗi bất đồng bộ nên em phải thêm khối này trung gian
    + Khối vẽ tin nhắn: xử lý và vẽ box tin nhắn lên giao diện

# Câu 3.
- App.vue là file App.vue mặc định mà khi em cài Vue CLI có sẵn, em code trực tiếp trên đó.
- Có riêng chức năng hỗ trợ toán học (em fix mãi không được nên tạm bỏ qua ạ @@).

**- App có chức năng cơ bản:**
    + New chat: lưu hội thoại hiện tại và mở hội thoại mới.
    + Show think: checkbox này sẽ làm hiển thị think của bot.
    + Ô nhập tin nhắn: có tính năng chống spam, khi chat box đang trả lời thì vô hiệu phím Enter gửi tin và nút Gửi.
    + Animation lúc chờ tin bot: hiệu ứng animation làm cho người dùng không cảm thấy trống rỗng.
    + Nút 3 gạch (cạnh new chat): Mở lịch sử các cuộc trò chuyện trước đó
    + Left panel (lịch sử cuộc trò chuyện): Hover vào sẽ hiện nút delete có thể xóa, hoặc bấm vào là sẽ mở lại cuộc hội thoại cũ.
    + Tự động lưu: thoát hoặc newchat thì hội thoại cũ sẽ tự lưu.

**- Em chia methods thành các khối rồi xử lí:**
    + Bước 1: Bắt đầu là giao tiếp API để lấy dữ liệu (bao gồm tinh chỉnh cấu hình API).
    + Bước 2: Xử lý tin nhắn (đưa nội dung, thẻ... hợp lí thân thiện với html).
    + Bước 3: Vẽ tin nhắn lên giao diện (hàm convertToHtml(messages.content)) rồi để template định dạng lại cho đẹp.
    + Ngoài ra sẽ có chức năng: lưu hội thoại, chống spam, xóa, mở mới, lưu khi thoát đột ngột...

**- Theo cấu trúc của vue, trong method em tạo các hàm:**
+ Xử lý dữ liệu và hiển thị HTML:
    > escapeHtml(text): Chống tấn công XSS, chuyển đổi nội dung text an toàn.
    > convertToHtml(content): Chuyển đổi text thuần thành các định dạng code block, heading...

+ Lưu trữ và quản lý lịch sử hội thoại:
    > saveToLocalStorage(): Lưu dữ liệu vào Local Storage, giúp giữ lại hội thoại ngay cả khi thoát ứng dụng.
    > loadFromLocalStorage(): Tải lại lịch sử trò chuyện khi mở trang.
    > toggleLeftDrawer(): Hiển thị thanh lịch sử trò chuyện.

+ Xử lý tin nhắn:
    > sendMessage(): Gửi tin nhắn, vẽ nội dung lên giao diện, và xử lý checkbox "Show Think".
    > saveChat(): Lưu toàn bộ hội thoại.
    > newChat(): Tạo mới một cuộc hội thoại.
    > loadChat(index): Tải dữ liệu khi truy cập vào hội thoại cũ.
    > removeChat(index): Xóa hội thoại, khi hover và nhấn vào biểu tượng thùng rác.
    > scrollToBottom(): Cuộn xuống tin nhắn mới nhất tự động.

+ Giao tiếp với API
    > fetchGroqResponse(messages): Gửi yêu cầu đến API Groq để lấy dữ liệu phản hồi.

+ Xử lý vòng đời của ứng dụng
    > beforeUnmount(): Lưu dữ liệu khi người dùng thoát tab, đóng trình duyệt hoặc tắt ứng dụng đột ngột.
    > mounted(): Chạy các hàm yêu cầu phải có dữ liệu trước (đợi sau khi ứng dụng tải xong).
