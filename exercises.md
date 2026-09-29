# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định như "changeme", trong tình huống chúng ta quên chưa thay đổi "changeme" thành api key đúng, app sẽ vẫn khởi động mà không "chết sớm", do đó chúng ta không biết và vẫn để cho app chạy, kết quả là người dùng gửi request đến và sẽ bị lỗi do api key không match.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log JSON của lần gọi `/ask`: `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T04:26:26.297921+00:00","user_id":"sv-test","tokens_in":458,"tokens_out":47,"cost_usd":0.0000969}`. Từ log này, mình có thể lọc request theo `user_id` và thời gian để tra cứu hoạt động của một người dùng; đồng thời tổng hợp token và `cost_usd` để tìm request tốn kém hoặc theo dõi chi phí. `print("đã trả lời xong")` không có các trường dữ liệu chuẩn để lọc và thống kê tự động.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 294 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Image multi-stage nhỏ hơn khoảng 1.44 GB, tương đương khoảng 83%. Mức giảm này không chỉ đến từ việc tách stage: bản đầu dùng `python:3.11` và `COPY . .`, còn bản hiện tại dùng `python:3.11-slim`, cài dependency với `--no-cache-dir` ở stage `builder`, rồi chỉ chép dependency cùng source cần chạy sang `runtime`. Vì vậy image cuối nhẹ hơn và không chứa các file chỉ cần trong build stage.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Sau khi sửa app/main.py, Docker dùng lại cache cho bước cài dependency vì requirements.txt không đổi. Các bước chép source chạy lại. Nếu đặt COPY . . trước RUN pip install, thay đổi source sẽ làm mất cache của bước copy và khiến bước cài dependency chạy lại

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu có lỗ hổng cho phép kẻ tấn công chạy mã hoặc lệnh tùy ý trong ứng dụng Python, các lệnh đó sẽ chạy với quyền của tiến trình ứng dụng. Nếu container chạy bằng root, kẻ tấn công có quyền root bên trong container, có thể đọc dữ liệu và bí mật được cấp cho container, sửa hệ thống hoặc khai thác cấu hình, quyền truy cập hay lỗi của runtime/kernel để thoát container và chiếm quyền cao trên host. Lệnh `USER app` chạy ứng dụng bằng tài khoản thường, nên khi bị khai thác kẻ tấn công chỉ có quyền hạn chế của tài khoản đó; nó cắt đứt mắt xích cấp quyền root bên trong container, nhưng không đảm bảo container không thể bị thoát.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Tối đa 20 request trong 2 giây: gửi 10 request ngay trước lúc phút kết thúc, rồi gửi thêm 10 ngay sau khi bộ đếm reset ở giây 00. Hai nhóm request nằm ở hai phút khác nhau nên đều được chấp nhận, dù tổng cộng có thể đến 20 request trong khoảng 2 giây. Sliding window 60 giây tránh được kiểu dồn request quanh ranh giới này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong một cửa sổ 60 giây; cost guard giới hạn tổng chi phí của mỗi người dùng trong tháng. Ví dụ, mình còn dưới 10 request trong phút này nhưng đã vượt ngân sách tháng thì rate limit vẫn cho request qua, còn cost guard chặn trước khi gọi LLM (402). Ngược lại, nếu mình gửi request thứ 11 trong 60 giây nhưng chi phí tháng vẫn còn trong ngân sách, cost guard chưa chặn vì ngân sách chưa hết, nhưng rate limit sẽ trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối nên endpoint gộp kiểm tra Redis sẽ báo lỗi trên cả 3 container. Nếu nền tảng dùng endpoint đó làm liveness check, nó đánh dấu các container không khỏe và khởi động lại chúng; load balancer cũng ngừng gửi traffic tới các instance lỗi. Redis vẫn mất kết nối trong 30 giây nên restart không khắc phục được nguyên nhân, các lần kiểm tra tiếp tục thất bại và dịch vụ có thể không còn instance nào nhận traffic. Khi Redis hoạt động lại, probe thành công, các instance khỏe trở lại và traffic được gửi vào. Tách riêng `/health` (chỉ kiểm tra process) với `/ready` (kiểm tra Redis) giúp ngừng gửi traffic mà không restart process còn sống.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Mình chạy `docker compose up --build --scale agent=3 -d`. Sau đó gọi `/ask` cùng `X-User-Id` lần lượt tới agent-1, agent-2, agent-3 rồi agent-1:
>
> ```text
> agent-1 (127.0.0.1:8001): HTTP 200, history_length=0
> agent-2 (127.0.0.1:8000): HTTP 200, history_length=2
> agent-3 (127.0.0.1:8002): HTTP 200, history_length=4
> agent-1 (127.0.0.1:8001): HTTP 200, history_length=6
> ```
>
> Kết quả tăng đều vì các replica đang dùng chung Redis, mỗi lượt thêm 2 message vào lịch sử. Nếu thay Redis bằng dict trong RAM, từng replica sẽ có bản riêng; với thứ tự A, B, C, A, kết quả sẽ là `0, 0, 0, 2` vì request đầu ở mỗi container không thấy lịch sử của hai container còn lại.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy, request tới `/ready` bị lỗi. Mình kiểm tra cấu hình và thấy `REDIS_URL` vẫn là `redis://localhost:6379/0`; trên cloud, `localhost` trỏ tới chính container ứng dụng chứ không phải Redis của Railway nên app không kết nối được Redis. Mình đổi `REDIS_URL` sang URL Redis Railway cung cấp rồi deploy lại. Sau đó `/ready` hoạt động.
