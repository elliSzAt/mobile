# Hacker Mindset

Là một khái niệm chỉ tư duy và quan điểm của mỗi cá nhân hoặc tổ chức về bảo mật thông tin. Nó bao gồm kiến thức về bảo mật, nhận thức về mối đe dọa và rủi ro, tư duy phòng ngừa và đối phó với tấn công. Việc phát triển mindset trong security là một quá trình liên tục.

### Tính tò mò, ham học hỏi

Là nền tảng của ngành.

 - Luôn khám phá điều mới.
 - Tự học và cập nhật kiến thức.
 - Phân tích các cuộc tấn công đã xảy ra.
 - Thực hành và thử nghiệm.

### Tính chủ động

 - Đặt mình vào vị trí của kẻ tấn công để có thể hiểu cách thức hoạt động của chúng.
 - Sử dụng các phương pháp và công cụ phân tích để tìm kiếm các lỗ hổng tồn tại trong hệ thống, dịch vụ cũng như cơ sở dữ liệu.
 - Tích cực tìm hiểu và nghiên cứu các lỗ hổng mới (Zero-day).

Mục đích: là để phát hiện và xử lý sớm các rủi ro và tìm ra các lỗ hổng ngay từ trong trứng, ví dụ:

 - Bruteforce ``username`` với ``password`` cố định.
 - Các lỗi như ``SQL injection`` hay ``XSS`` ví dụ được viết rule để loại bỏ hết các cụm từ ``script`` hay các biến thể của nó (SCrIpt, scRiPt,...) ra khỏi chuỗi. Vậy thì bypass như thế nào? Có thể chúng ta sẽ sử dụng các thẻ khác hoặc là ``urlencode`` để thử, tuy nhiên ngoài những thứ đó ra ta có thể thử cụm ``scrscriptipt``.

### Tư duy giải quyết vấn đề

 - Xác định vấn đề.
 - Phân tích nguyên nhân gốc rễ.
 - Tìm kiếm giải pháp.
 - Thử nghiệm và điều chỉnh.

### Kiên trì

Trong lĩnh vực bảo mật, kiên trì là một yếu tố cực kỳ quan trọng. Công việc bảo mật thường gặp phải nhiều thách thức và đôi khi việc tìm ra lỗ hổng hoặc giải pháp không phải lúc nào cũng nhanh chóng.