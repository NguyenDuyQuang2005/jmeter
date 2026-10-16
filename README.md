## 1. Mục tiêu thực hành

- Làm quen với cách xây dựng kịch bản kiểm thử trên Apache JMeter.
- Cấu hình người dùng mô phỏng và gửi yêu cầu HTTP GET.
- Thu thập số lượng yêu cầu, tỷ lệ lỗi, thời gian phản hồi và thông lượng.
- Đọc kết quả, đối chiếu các chỉ số và nhận xét trong phạm vi kịch bản.
- Phân biệt kiểm thử một trang web với kiểm thử endpoint API trả dữ liệu.

## 2. Tổng quan công cụ và chỉ số

JMeter là công cụ dùng để xây dựng và thực hiện các kịch bản kiểm thử, trong đó có kiểm thử tải HTTP. Một kịch bản cần xác định rõ đối tượng, mức tải, thời gian và tiêu chí đánh giá.

| Thành phần | Vai trò |
| --- | --- |
| Test Plan | Chứa cấu hình bài kiểm thử |
| Thread Group | Mô phỏng nhóm người dùng |
| HTTP Request | Gửi yêu cầu HTTP/HTTPS |
| View Results Tree | Xem từng phản hồi để kiểm tra kịch bản |
| Aggregate Report | Tổng hợp kết quả |

| Chỉ số | Cách đọc |
| --- | --- |
| Samples | Số mẫu được ghi nhận |
| Average | Thời gian phản hồi trung bình |
| Median | Trung vị; khoảng 50% mẫu không vượt mức này |
| 90% Line / P90 | Khoảng 90% mẫu không vượt mức này |
| Error % | Tỷ lệ mẫu thất bại |
| Throughput | Thông lượng; phải đọc kèm đơn vị |

Công thức tính tỷ lệ:

```text
Tỷ lệ thành công (%) = Số yêu cầu thành công / Tổng số yêu cầu × 100
Tỷ lệ thất bại (%)   = Số yêu cầu thất bại / Tổng số yêu cầu × 100
```

## 3. Môi trường và quy trình thực hành

### 3.1. Thông tin có trong repository

Repository chứa `README.md` và một tệp `.drawio`. Chưa có tệp kế hoạch `.jmx` hoặc dữ liệu kết quả đầy đủ `.jtl`/`.csv` để tái chạy và tính lại các thống kê.

Những thông tin chưa được nguồn cung cấp gồm phiên bản JMeter/Java, cấu hình máy tạo tải, đường truyền, thời điểm chạy, cấu hình lặp, bộ định thời và tiêu chí chấp nhận. Vì vậy, báo cáo không tự bổ sung các thông tin này như dữ kiện đã xác nhận.

### 3.2. Các bước dựng lại kịch bản

1. Chuẩn bị Java tương thích với phiên bản JMeter sử dụng, rồi mở JMeter.
2. Tạo **Test Plan** và thêm **Thread Group**.
3. Khai báo số thread, ramp-up và điều kiện kết thúc.
4. Thêm **HTTP Request**, tách riêng protocol, server và path.
5. Thêm **View Results Tree** để kiểm tra phản hồi và **Aggregate Report** để xem thống kê.
6. Kiểm tra trước bằng một thread, một vòng lặp; xác nhận URL, response code và nội dung đúng mục tiêu.
7. Lưu kế hoạch, thực hiện kịch bản đã thống nhất và lưu dữ liệu kết quả.
8. Chụp cấu hình, bảng tổng hợp và chi tiết mẫu lỗi; phân tích kết quả theo tiêu chí đặt trước.

**Lưu ý về tải:** Các mức tải trong báo cáo mô tả tài liệu nguồn. Chỉ áp dụng thử tải lên hệ thống được phép kiểm thử và trong giới hạn dịch vụ cho phép.

### 3.3. Làm rõ số thread, thời gian và số mẫu

Cả hai kịch bản trong README đều nêu **100 thread**, **ramp-up 10 giây**, **thời gian chạy 60 giây**. Ramp-up là thời gian khởi động dần các thread, không phải tổng thời gian chạy. Với cấu hình này, khoảng cách khởi động danh nghĩa là `10 / 100 = 0,1 giây`.

Nguồn không nêu **Loop Count**. Vì vậy, không thể suy ra 1.000 mẫu chỉ từ 100 thread và thời gian 60 giây.

- Nếu muốn chạy theo thời gian: có thể chọn lặp vô hạn, bật giới hạn thời gian và đặt Duration 60 giây. Đây là cấu hình đề xuất; số mẫu phụ thuộc tốc độ xử lý và các yếu tố trong kịch bản.
- Nếu muốn khoảng 1.000 lần gọi một sampler: 100 thread × 10 vòng lặp tạo ra 1.000 lượt thực hiện khi mọi thread hoàn tất, không bị dừng sớm và không có nhánh điều khiển làm đổi số lượt. Thời gian thực tế không tự động bằng 60 giây.

Nếu có nhiều sampler hoặc ghi thêm mẫu con/chuyển hướng, phải kiểm tra phạm vi đếm trước khi đối chiếu với tổng mẫu.

## 4. Kịch bản 1 — Kiểm thử trang web Phenikaa

### 4.1. Mục tiêu và cấu hình theo README nguồn

Mô phỏng việc truy cập trang `https://phenikaa-uni.edu.vn/vi`, ghi nhận phản hồi và phân tích các thống kê được công bố.

| Thông số | Giá trị |
| --- | --- |
| Number of Threads | 100 |
| Ramp-up Period | 10 giây |
| Thời gian chạy mô tả | 60 giây |
| Method | GET |
| Protocol | `https` |
| Server Name or IP | `phenikaa-uni.edu.vn` |
| Path | `/vi` |
| Content encoding | `UTF-8` |
| Listener | View Results Tree, Aggregate Report |

### 4.2. Kết quả trong ảnh tổng hợp

![Bảng tổng hợp kết quả kiểm thử trang Phenikaa](https://github.com/quocbinh93/JMeter/assets/96821644/a22ed4bb-a39d-43e2-8093-5681495311c9)

*Hình 1. Bảng số liệu được repository công bố; đây là ảnh bảng tính tổng hợp, không phải ảnh trực tiếp giao diện Aggregate Report.*

| Chỉ số | Giá trị đọc từ ảnh |
| --- | ---: |
| Tổng số yêu cầu | 1.000 |
| Yêu cầu thành công | 998 |
| Yêu cầu thất bại | 2 |
| Tỷ lệ thành công | 99,8% |
| Tỷ lệ thất bại | 0,2% |
| Thời gian phản hồi trung bình | 40 ms |
| Thời gian phản hồi trung vị | 38 ms |
| P90 | 70 ms |
| Throughput | 16 yêu cầu/giây |

Kiểm tra phép tính:

```text
998 / 1.000 × 100 = 99,8%
  2 / 1.000 × 100 =  0,2%
```

### 4.3. Đối chiếu với ảnh dữ liệu chi tiết

![Ảnh dữ liệu chi tiết có elapsed, responseCode, success và Latency](https://github.com/quocbinh93/JMeter/assets/96821644/dab81193-b61b-49b9-9386-d775ec357874)

*Hình 2. Một phần dữ liệu chi tiết trong repository, gồm nhiều nhãn và phản hồi chuyển hướng.*

Các điểm quan sát được:

- Cột `elapsed` có những giá trị như `1080`, `45`, `426`, `606`; cột `Latency` ở dòng đầu là `40`. Không được lấy giá trị Latency của một dòng làm thời gian phản hồi trung bình của toàn bộ lần chạy.
- Có mã `200`, `302` và `308`, trong khi các dòng đang hiển thị đều có `success = TRUE`. Vì vậy, không thể đồng nhất “thành công” trong dữ liệu này với “tất cả phản hồi đều là HTTP 200”.
- Có các nhãn liên quan đến `Phenikaa`, `STEM`, `News` và các nhãn có hậu tố. Phạm vi hiển thị rộng hơn mô tả một request `/vi`; cần tệp `.jmx` và dữ liệu đầy đủ để biết cấu trúc thực tế.

Ảnh chỉ chứa một phần kết quả, nên chưa đủ để xác minh hay bác bỏ số trung bình 40 ms và P90 70 ms. Một số mẫu lớn hơn các mức này vẫn có thể xuất hiện trong một tập dữ liệu có trung bình/P90 thấp.

### 4.4. Nhận xét

Theo bảng tổng hợp, tỷ lệ thành công cao và các giá trị thời gian được báo cáo thấp. Tuy nhiên, cần xem chi tiết 2 mẫu lỗi để xác định mã phản hồi và nguyên nhân; chưa có căn cứ quy lỗi cho máy chủ, mạng hay timeout.

Ngoài ra, `1.000 / 60 ≈ 16,67 yêu cầu/giây`, khác con số 16 trong bảng. Đây là điểm cần đối chiếu với khoảng thời gian tính throughput và phạm vi mẫu thực tế, không phải bằng chứng tự nó xác nhận số liệu sai.

**Kết luận cho kịch bản 1:** Nguồn báo cáo 998/1.000 yêu cầu thành công. Các chỉ số gợi ý phản hồi nhanh trong phép đo được mô tả, nhưng chưa đủ chứng cứ để kết luận khả năng chịu tải tối đa hoặc hiệu năng ở mọi thời điểm.

## 5. Kịch bản 2 — Nội dung được nguồn gọi là “kiểm thử API thời tiết”

### 5.1. Cấu hình theo README nguồn

| Thông số | Giá trị |
| --- | --- |
| Number of Threads | 100 |
| Ramp-up Period | 10 giây |
| Thời gian chạy mô tả | 60 giây |
| Method | GET |
| Protocol | `https` |
| Server Name or IP | `openweathermap.org` |
| Path | `/current` |
| Content encoding | `UTF-8` |
| Listener | View Results Tree, Aggregate Report |

**Điểm cần làm rõ:** `https://openweathermap.org/current` là trang tài liệu về dữ liệu thời tiết hiện tại; tại thời điểm đối chiếu, địa chỉ này chuyển tới `/api/current`. Nó không phải endpoint JSON lấy dữ liệu thời tiết. Nếu request thực tế đúng như README mô tả, kết quả đang đo trang tài liệu, chưa chứng minh hiệu năng API thời tiết.

### 5.2. Kết quả trong ảnh

![Bảng kết quả OpenWeather với thời gian trung bình 100 ms](https://github.com/quocbinh93/JMeter/assets/96821644/d82c597c-dad1-425e-b717-47d019b41224)

*Hình 3. Bảng tổng hợp của kịch bản OpenWeather trong repository.*

| Chỉ số | Giá trị đọc từ ảnh |
| --- | ---: |
| Tổng số yêu cầu | 1.000 |
| Yêu cầu thành công | 996 |
| Yêu cầu thất bại | 4 |
| Tỷ lệ thành công | 99,6% |
| Tỷ lệ thất bại | 0,4% |
| Thời gian phản hồi trung bình | 100 ms |
| Thời gian phản hồi trung vị | 98 ms |
| P90 | 150 ms |
| Throughput | 16 yêu cầu/giây |

Kiểm tra phép tính:

```text
996 / 1.000 × 100 = 99,6%
  4 / 1.000 × 100 =  0,4%
```

**Đối chiếu văn bản:** README nguồn kết thúc ở dòng “Thời gian phản hồi trung bình: 10”, không có đơn vị. Ảnh kết quả ghi rõ **100 ms**, nên báo cáo sử dụng 100 ms và ghi nhận sự không thống nhất này.

### 5.3. Cấu hình đề xuất nếu mục tiêu là kiểm thử API thật

Theo tài liệu OpenWeather, có thể gọi dữ liệu thời tiết hiện tại theo tọa độ [4]:

```text
https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric
```

| Trường trong HTTP Request | Cấu hình đề xuất |
| --- | --- |
| Protocol | `https` |
| Server Name or IP | `api.openweathermap.org` |
| Method | `GET` |
| Path | `/data/2.5/weather` |
| Parameters | `lat`, `lon`, `appid`, `units` |
| Biến JMeter | `${lat}`, `${lon}`, `${apiKey}` |

Ví dụ tọa độ để thử chức năng: `lat = 21.0285`, `lon = 105.8542`. Giá trị `${apiKey}` phải được khai báo bằng khóa của tài khoản thực hành. Không đưa khóa thật vào ảnh chụp hoặc repository công khai.

Khi kiểm tra request, cần xác nhận HTTP 200, body là JSON thời tiết và có các trường phù hợp như `weather`, `main.temp`, `name`. Chỉ ghi nhận hiệu năng API sau khi đã xác minh đúng endpoint và nội dung phản hồi. Mức tải phải phù hợp với quyền kiểm thử và hạn mức tài khoản.

**Trạng thái:** Đây là cấu hình sửa đề xuất, chưa được chạy trong báo cáo. Không chuyển các số liệu 100 ms, 98 ms, 150 ms ở ảnh nguồn sang endpoint mới.

### 5.4. Kết luận cho kịch bản 2

Ảnh nguồn báo cáo 996/1.000 yêu cầu thành công và 4 thất bại. Chưa có dữ liệu chi tiết để xác định nguyên nhân lỗi. Do địa chỉ trong README là trang tài liệu, chưa đủ căn cứ kết luận về hiệu năng API trả dữ liệu thời tiết.

## 6. So sánh hai bảng kết quả nguồn

| Chỉ số | Phenikaa | OpenWeather theo nguồn |
| --- | ---: | ---: |
| Tổng yêu cầu | 1.000 | 1.000 |
| Thành công | 998 | 996 |
| Thất bại | 2 | 4 |
| Tỷ lệ lỗi | 0,2% | 0,4% |
| Average | 40 ms | 100 ms |
| Median | 38 ms | 98 ms |
| P90 | 70 ms | 150 ms |
| Throughput | 16 yêu cầu/giây | 16 yêu cầu/giây |

Theo các giá trị công bố, nhóm OpenWeather có Average cao hơn 60 ms và tỷ lệ lỗi cao hơn 0,2 điểm phần trăm. Tuy nhiên, đây chỉ là đối chiếu hai bảng số liệu. Hai mục tiêu, nội dung phản hồi và điều kiện chạy chưa được chuẩn hóa, nên không dùng bảng này để xếp hạng chất lượng hai hệ thống.

## 7. Hoàn thiện phép đo và báo cáo

### 7.1. Cách chạy và lưu minh chứng

Theo hướng dẫn Apache JMeter [3], dùng GUI để xây dựng/gỡ lỗi, còn phép đo tải nên dùng chế độ dòng lệnh và hạn chế listener nặng. Tắt View Results Tree trong lúc đo tải; lưu kết quả để phân tích sau.

Ví dụ lệnh sau khi đã tự tạo và lưu `test-plan.jmx`:

```bash
jmeter -n -t test-plan.jmx -l results.jtl
```

Trên Windows có thể dùng `jmeter.bat` từ thư mục `bin`. Tên file trong lệnh là ví dụ, không phải tệp đã được repository nguồn cung cấp. Dùng tệp kết quả mới cho từng lần chạy để tránh trộn dữ liệu cũ.

Minh chứng nên bao gồm:

- Ảnh Thread Group thể hiện số thread, ramp-up, số vòng lặp và thời gian.
- Ảnh HTTP Request thể hiện đúng server, path và tham số.
- Ảnh Aggregate Report đầy đủ tiêu đề cột và dòng tổng.
- Chi tiết từng lỗi, gồm mã phản hồi, thông báo và nội dung liên quan.
- File `.jmx` và dữ liệu `.jtl`/`.csv` sau khi loại bỏ thông tin bí mật.

### 7.2. Điều kiện để đánh giá đạt/không đạt

Trước khi chạy cần xác định ngưỡng chấp nhận cho tỷ lệ lỗi, P90/P95 hoặc throughput theo yêu cầu hệ thống. Repository chưa đưa ra những ngưỡng này, nên báo cáo không tự gán kết quả “đạt SLA”.

Cần giữ điều kiện chạy nhất quán, thực hiện nhiều lần và ghi lại môi trường. Để đánh giá khả năng chịu tải, phải khảo sát các mức tải phù hợp thay vì suy ra giới hạn hệ thống từ một mức 100 thread.

## 8. Kết luận và bài học

Bài thực hành giúp hệ thống hóa cách tạo kịch bản JMeter, cấu hình HTTP Request và đọc các chỉ số hiệu năng. Hai bảng nguồn đều ghi nhận tỷ lệ thành công trên 99%, nhưng vẫn có yêu cầu thất bại cần phân tích.

Những điểm quan trọng rút ra:

1. Phân biệt số thread, số vòng lặp, thời gian chạy và số mẫu thu được.
2. Phân biệt thời gian phản hồi toàn bộ với latency; không suy thống kê tổng từ một dòng dữ liệu.
3. Kiểm tra đúng đối tượng đo, đặc biệt giữa trang tài liệu và endpoint API.
4. Đối chiếu văn bản, ảnh và dữ liệu gốc trước khi kết luận.
5. Đánh giá trong phạm vi tải, thời điểm và tiêu chí đã xác định.
