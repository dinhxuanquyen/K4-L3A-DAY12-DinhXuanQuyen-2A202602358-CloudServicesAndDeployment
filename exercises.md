# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng quote mặc định bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đinh Xuân Quyền  Mã học viên: 2A202602358

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Bắt lỗi ngay lúc deploy giúp phát hiện sớm việc quên cấu hình biến môi trường. Nếu để changeme, app vẫn chạy ngầm và người lạ dùng chùa api thả ga thì đến cuối tháng xem bill tiền mới biết là bị hớ.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ: `{"level":"INFO","method":"POST","path":"/ask","status":200,"duration_ms":45.2}`
>
> 1. Dùng tool đọc field `duration_ms` để đếm số request chạy chậm.
> 2. Đọc field `status` để thống kê lỗi 400, 500 vẽ biểu đồ tự động. Log bằng chữ bình thường không làm được 2 cái này.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản                 | Dung lượng |
| -------------------- | ------------ |
| 1 stage (bản đầu) | ~900 MB      |
| Multi-stage          | ~150 MB      |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản 1 to vì dính đủ thứ tool cài cắm ban đầu . Bản Multi-stage siêu nhẹ vì nó chỉ bốc mỗi cái app đã build xong chạy qua stage 2, vứt lại đống rác ở stage 1.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Bản chuẩn: Chỉ mất cache từ lệnh `COPY . .` trở xuống. Lệnh `pip install` ở trên vẫn giữ nguyên không phải chạy lại.
> Nếu đảo ngược: Cứ mỗi lần sửa 1 dòng code python là mất sạch cache, lệnh `pip install` lại phải hì hục tải mớ thư viện lại từ đầu rất tốn tgian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu chạy root: Hacker tìm được bug -> chui vào tiêm mã độc -> nghiễm nhiên có quyền root để phá nát hệ thống máy chủ thật. Lệnh `USER` ép app chạy bằng acc con, nên hacker có hack được code python thì cũng chỉ có quyền acc con, không phá phách hệ thống được.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> 20 request. Lý do: Họ spam 10 request ở 12:00:59, sang giây tiếp theo 12:01:00 đồng hồ đếm phút reset lại 0, họ spam tiếp thêm 10 request nữa là thành 20.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate Limit chặn request spam quá nhanh, Cost Guard chặn dùng quá nhiều token (sợ tốn tiền).
>
> - Rate Limit tha, Cost Guard chặn: Ít gọi api nhưng toàn gửi văn bản PDF dài 100 trang -> cực tốn tiền -> Cost Guard chặn.
> - Ngược lại: F5 bấm gửi "hi" 100 lần 1 giây -> ko tốn tiền nhưng làm máy chủ lag -> Rate Limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Gộp chung: Redis sập -> health check đỏ -> Kubernetes tưởng app sập nên kill và khởi động lại nguyên cụm liên tục.
> Tách riêng: Redis sập -> /ready đỏ (tạm ngừng nhận request mới) -> chờ xíu Redis lên lại thì tự thông bình thường, app ko bị reset oan.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> `history_length` nhảy loạn xạ số (1, 2, 0...) vì mỗi request chui ngẫu nhiên vào 1 trong 3 agent, RAM thằng nào thằng nấy xài. Đẩy ra Redis thì 3 thằng xài chung 1 database nên đồng bộ 100%.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Bị lỗi Application failed to respond trên giao diện Railway dù Deploy báo success. Nguyên nhân do Railway tự gán port mặc định 8080 cho app, nhưng lúc add custom domain lại set nhầm port 8000 nên kết nối lệch pha. Sửa bằng cách vào tab Variables gán biến PORT=8000 để ép app chạy trùng với cấu hình web là thông mạng liền.
