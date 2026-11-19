# Meridian Home & Paper

Meridian là bản demo phục vụ báo cáo kiến tập về quản lý và vận hành thương mại điện tử với Odoo. Ứng dụng minh họa việc tiếp nhận đơn liên sàn bằng dữ liệu mô phỏng. Cần Python 3.12 và không cần cài thêm thư viện. SQLite lưu đơn, SKU, ánh xạ và lịch sử. Mặc định máy chủ chỉ mở trên loopback.

## Chạy cục bộ

```powershell
python app.py
```

Mở [ứng dụng cục bộ](http://127.0.0.1:8765). Chọn sàn, chọn tệp JSON và bấm **Xem trước**. Kiểm tra SKU, số lượng và đơn giá rồi chọn **Xác nhận nhập** hoặc **Lưu vào ngoại lệ** theo kết quả. Thao tác xem trước chưa lưu đơn hay gửi sang Odoo. Các tệp mẫu có trong mục **Dữ liệu thử nghiệm**. Danh sách có bộ lọc thị trường, trạng thái và người phụ trách. Đơn hợp lệ sẽ ở **Sẵn sàng** khi chưa có kết nối Odoo.

- Năm mẫu đại diện là Amazon → Mỹ, eBay → Đức, AliExpress → Nam Phi, Etsy → Úc và Taobao → Nhật Bản. Đây là các cặp giả lập dùng để kiểm tra adapter và thị trường đích. Đơn Taobao giao Nhật sử dụng CNY theo giả định thanh toán của bộ dữ liệu.
- Mục **Sàn bán hàng** chọn adapter. Quốc gia giao được đọc từ tệp, ưu tiên trường `destination_country` khi tệp có cả mã thị trường sàn và quốc gia giao. Chẳng hạn, tệp Amazon có `Marketplace: "US"`, `destination_country: "DE"`, `CurrencyCode: "USD"` sẽ hiển thị Amazon · Đức · USD. Năm mẫu đại diện không khóa các sàn theo quốc gia.
- **Tiền tệ đơn nguồn** là luồng mặc định duy nhất của mẫu, dùng USD, EUR, ZAR, AUD hoặc CNY. Backend giữ nguyên số tiền và yêu cầu tiền tệ đã bật cùng một bảng giá Odoo phù hợp nếu tiền tệ khác báo giá tham chiếu. Các tệp JSON có thể nhập trực tiếp nằm trong thư mục [samples](samples).
- **Bộ dữ liệu vận hành** có 30 đơn nguồn, ba tệp sự kiện, danh mục 500 SKU và 52 ánh xạ. Trong Dữ liệu thử nghiệm, tải đơn đầu tiên của sàn đang chọn hoặc tải cả gói ZIP. Ngày đặt, thành phố, mã bưu chính và dịch vụ giao xuất hiện khi xem trước và mở chi tiết. Xem [hướng dẫn bộ dữ liệu](samples/operations/README.md).
- Mục **Dữ liệu thử nghiệm** hiện mẫu và ngoại lệ của sàn đang chọn. Trong bộ vận hành, Amazon và Taobao có ca thiếu SKU, eBay có ca lệch tổng, AliExpress có ca đơn giá âm và Etsy có ca số lượng bằng 0.
- Bấm bất cứ ô nào trong dòng đơn để mở chi tiết. Tại dòng thiếu mã, bấm **Ánh xạ SKU**, tìm theo tên hoặc mã SKU, chọn kết quả rồi lưu. App dùng sàn, tài khoản và mã listing từ đơn. Nếu đơn chưa giao người xử lý, bấm **Xử lý lại**. Với đơn đã giao, duyệt theo quy trình bản sửa. Màn hình **Ánh xạ SKU** trên điều hướng dùng để xem và quản lý các cặp mã đã lưu.
- Nếu chỉ thiếu ánh xạ và đơn đã giao, nhập lý do rồi bấm **Gửi ánh xạ để duyệt**. Bản duyệt chứa các dòng và SKU đề xuất, còn tệp gốc giữ nguyên. Đổi ánh xạ sau khi duyệt sẽ yêu cầu xác nhận lại trước bàn giao.
- Để thử hủy, nhập `amazon-cancellation-order-usd.json`, sau đó nhập `amazon-cancellation-notice.json` và bấm **Ghi nhận hủy**. Hai tệp thuộc thư mục [samples/acceptance](samples/acceptance). Đơn xuất hiện ở **Hủy trên sàn** và vẫn mở được báo giá cũ nếu đã có. Meridian không tự hủy báo giá trong Odoo.
- Ca đã nghiệm thu lệch tổng Amazon trong [samples/acceptance](samples/acceptance) có tổng nguồn 52,19 USD và tổng tính lại 50,69 USD. App giữ đơn và hiển thị khoản lệch cùng phí, giảm giá và thuế nguồn.

## Kết nối môi trường Odoo thử nghiệm

Chế độ cục bộ chạy được ngay sau khi tải mã nguồn. Để thử bàn giao báo giá nháp, cần một cơ sở dữ liệu Odoo thử nghiệm có quyền truy cập JSON-2 API, ứng dụng Sales, báo giá mẫu có khách hàng và bảng giá theo tiền tệ của đơn nguồn. Trên PowerShell, dùng thông tin của môi trường thử do bạn quản lý.

~~~powershell
$env:ODOO_URL = 'https://your-test-instance.odoo.com'
$env:ODOO_FIXTURE_QUOTE = 'S00003'
.\Start-Meridian-Odoo.ps1
~~~

Script hỏi khóa API bằng lời nhắc ẩn, tạo hai sản phẩm mẫu nếu cần, rồi chạy tại [127.0.0.1:8766](http://127.0.0.1:8766). Đặt ODOO_FIXTURE_QUOTE theo mã báo giá mẫu trong môi trường của bạn. Nếu cần chạy thử với một file SQLite mới, khởi động app bằng phương án dòng lệnh và thêm --db.

## Kiểm thử tự động

### Chạy kịch bản bàn giao có phê duyệt

1. Chọn eBay rồi nhập `ebay-exception-usd.json` từ [samples/acceptance](samples/acceptance) trên môi trường chưa có mã đơn này. Trong chi tiết đơn, nhập **Mai Linh** và bấm **Giao người xử lý**.
2. Nhập lý do sửa, chọn `ebay-corrected-usd.json` có cùng mã đơn, rồi **Gửi bản sửa**. Xem phần **Trước** và **Sau**.
3. Nhập **Minh An** và bấm **Xác nhận và kiểm tra lại**. Đơn đạt sẽ hiện **Bàn giao Odoo**. Có thể chọn **Trả lại bản sửa** cùng lý do để thử nhánh từ chối.
4. Bấm **Bàn giao Odoo**. Khi backend có kết nối, kiểm tra báo giá nháp và lịch sử xử lý. Ở chế độ cục bộ, đơn vẫn sẵn sàng.

Tên người xử lý và người duyệt là danh tính mô phỏng nhập tay. Bản cục bộ chưa có đăng nhập và phân quyền theo tài khoản.

### Chạy bộ test

```powershell
python -B check_flow.py
```

Các ca tự động chạy với SQLite thật và phản hồi Odoo mô phỏng. Bộ kiểm thử bao gồm xem trước chỉ đọc, năm cặp mẫu, luồng giao việc, duyệt bản sửa, duyệt ánh xạ, chặn thay đổi sau duyệt, ghi nhận hủy và kiểm tra bộ dữ liệu vận hành. Các tệp dùng trong nghiệm thu trước nằm ở [samples/acceptance](samples/acceptance).
