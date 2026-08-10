# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng chứa Câu trả lời của bạn bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Lương Đức Thắng  Mã học viên: 2A202601196

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định là "changeme", khi deploy lên cloud mà quên cấu hình biến môi trường, ứng dụng vẫn sẽ khởi động thành công và mở cổng HTTP. Bất cứ ai trên Internet biết URL đều có thể truy cập API của bạn bằng token "changeme" và sử dụng cạn kiệt tài nguyên hoặc ngân sách của bạn (gây tổn thất tài chính). Việc fail fast giúp phát hiện ngay lập tức lỗi cấu hình khi deploy, ngăn chặn nguy cơ bảo mật.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log: `{"event": "chat_request", "severity": "INFO", "ts": "2026-08-10T10:15:35", "client_id": "test", "usd_cost": 0.0003, "usage": {"prompt": 10, "completion": 20}}`
> Hai việc làm được: 1. Có thể dùng các hệ thống quản lý log (như ELK, Datadog) để parse JSON, lọc (filter) hoặc tổng hợp (aggregate) tổng chi phí theo từng `client_id` để thống kê. 2. Có thể set cảnh báo tự động (alert) khi log có `severity` là `ERROR` hoặc khi giá trị `usd_cost` vượt ngưỡng một cách dễ dàng.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~330 MB |
| Multi-stage | ~150 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch là các công cụ build (build tools như gcc), thư viện dev, cache của `pip`, và mã nguồn không cần thiết lúc chạy. Trong multi-stage build, stage cuối chỉ copy những gì cần thiết (môi trường đã cài đặt thư viện) từ stage builder sang, giúp thu nhỏ tối đa dung lượng container ở production.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa code trong `app/main.py`, các layer từ `COPY requirements.txt .` và `RUN pip install ...` vẫn được lấy từ cache, chỉ có layer `COPY app ./app` và các layer sau đó bị chạy lại (re-build). Nếu đặt `COPY . .` lên trước `RUN pip install`, Docker sẽ vô hiệu hóa cache cho lệnh `pip install` mỗi khi có MỘT file code bất kỳ thay đổi. Điều này khiến thời gian build lâu hơn rất nhiều vì phải tải và cài lại toàn bộ thư viện mỗi lần sửa code.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi sự kiện: Nếu có lỗ hổng trong code Python (vd: RCE), kẻ tấn công có thể chạy lệnh tùy ý bên trong container. Do container chạy bằng user `root`, kẻ tấn công có toàn quyền trong container. Nếu có lỗ hổng container escape hoặc mount thư mục nhạy cảm, root trong container có thể trở thành root trên host, phá hủy toàn bộ hệ thống. Lệnh `USER appuser` cắt đứt chuỗi này: kẻ tấn công khai thác được ứng dụng cũng chỉ có quyền của user thường, không thể chỉnh sửa file hệ thống.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> Header `WWW-Authenticate: Bearer` là quy chuẩn của HTTP (RFC 6750) để báo cho client biết server đang yêu cầu xác thực bằng Bearer Token. Việc trả cùng một thông báo lỗi giúp bảo mật: kẻ tấn công không thể dùng kỹ thuật dò tìm (enumeration) để biết được chúng vừa gửi sai định dạng hay token đó thực sự không tồn tại, tránh cung cấp manh mối.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Nó sẽ gửi được tối đa 10 request trước khi bị 429 vì số lượng token không vượt quá capacity=10. Nếu bỏ đoạn `min(capacity, ...)` trong hàm `available()`, thuật toán sẽ cứ cộng dồn vô hạn (10 phút thành 100 token). Lúc này, client có thể gửi 100 request liên tiếp, phá vỡ tính năng giới hạn bùng nổ (burst) của thuật toán.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Dùng $30/tháng: thiệt hại tối đa là toàn bộ ngân sách $30 trong vài giờ đầu, và hệ thống sẽ block client suốt cả phần còn lại của tháng (không tự hồi phục cho đến tháng sau). Dùng $1/ngày: thiệt hại tối đa cho sự cố hôm đó chỉ là $1. Hệ thống sẽ tự động hồi phục vào 0h00 ngày hôm sau, giảm thiểu tối đa rủi ro về chi phí và thời gian gián đoạn dài hạn.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Trình tự sự kiện: 1. Redis mất kết nối, cả 3 container đều báo `healthz` = fail. 2. Hệ thống (vd: Kubernetes) thấy health check thất bại sẽ lập tức kill (giết) cả 3 container. 3. Hệ thống tạo lại 3 container mới, nhưng chúng vẫn báo fail (vì Redis vẫn sập), lại tiếp tục bị kill (crash loop). 4. Khi Redis sống lại sau 30s, các app đang trong quá trình restart nên chưa thể phục vụ ngay, dẫn đến thời gian sập (downtime) kéo dài hơn việc dùng /readyz (chỉ tách khỏi load balancer chứ không kill app).

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi gặp phải: Ứng dụng báo lỗi 500 Internal Server Error (lỗi cụ thể trong log Render là `ValueError: Redis URL must specify one of the following schemes`) khi vừa deploy xong. 
> Nguyên nhân: Tôi chỉ copy phần host của Upstash mà quên mất scheme `rediss://`.
> Cách khắc phục: Sửa lại cấu hình Environment Variables trên Render thành `rediss://default:...`. Khi lưu lại, Render tự động deploy bản mới và ứng dụng đã hoạt động trở lại.
