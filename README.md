Google-ultra

Google One's AI Ultra có một offer dùng thử miễn phí 12 tháng, không hiển thị trên bất kỳ trang nào, chỉ tồn tại trong giá trị trả về của thư viện thanh toán Play (BillingClient). Script này nối chuỗi "tra offer → gọi Play checkout → xác nhận 0 đồng → kích hoạt" thành một lệnh duy nhất.

Nguyên lý là mượn chính tiến trình của Google One để gọi BillingClient, không cần Xposed, không cần sửa APK.

Lấy được gì

Nhận được là **Ultra 5x, 20TB, 12 tháng $0**, hủy trước khi hết hạn thì không phải trả tiền. Sau khi kích hoạt, trang đăng ký Play sẽ ghi "sắp trừ tiền 2027-11-17 US$99.99/tháng", đây là giá sau khi hết dùng thử, không phải trừ ngay bây giờ.

Gói 30TB (20x) cũng tra được offer, nhưng khi thanh toán Play sẽ kiểm tra lại lần hai, phần lớn tài khoản sẽ bị báo "bạn không đủ điều kiện dùng thử miễn phí". Script này sẽ không bấm vào đó.

Cần gì

Một chiếc điện thoại Android đã root, nối USB với máy tính. Magisk, KernelSU, APatch đều được, giả lập cũng được.

Máy tính cần Python 3.9 trở lên, cộng thêm hai gói:

```bash
pip3 install frida
```

adb dùng platform-tools trong Android SDK. Cài xong adb devices thấy được điện thoại là được. Nếu chưa cài, trên Mac brew install android-platform-tools, trên Windows tải gói nén platform-tools, giải nén rồi thêm thư mục vào PATH.

Trên điện thoại cần bật "Tùy chọn nhà phát triển → Gỡ lỗi USB", cắm vào rồi bấm cho phép trong hộp thoại bật lên.

Cách dùng

Trước tiên tự kiểm tra môi trường một lượt, nó sẽ kiểm tra thiết bị, root, frida-server, đẩy module thanh toán, và tiện tay tắt proxy với Clash có thể chặn mạng Play:

```bash
python3 ultra0.py --check
```

Tất cả xanh là có thể chạy tài khoản.

Tài khoản viết vào một file text, mỗi dòng một cái, các trường cách nhau bằng bốn dấu gạch ngang:

```
email----mật khẩu----email phụ----khóa 2FA----số thẻ----ngày hết hạn----CVC----chủ thẻ----mã bưu chính
```

Mấy mục sau để trống cũng được. Tài khoản mới mua thường không có 2FA, thì để trống:

```
abc123@gmail.com----mypassword----
```

Nếu có thẻ:

```
abc123@gmail.com----mypassword--------4242424242424242----10/29----326----ZHANG SAN----97201
```

Không điền địa chỉ sẽ mặc định dùng 123 Main St, Portland, OR 97201, Oregon miễn thuế tiêu thụ.

Chạy:

```bash
python3 ultra0.py accounts.txt
```

Một tài khoản cũng được:

```bash
python3 ultra0.py --account "abc@gmail.com:mật khẩu:khóa 2FA"
```

Muốn xem kết quả, mở trang đăng ký xác nhận một chút:

```
Cửa hàng Play → ảnh đại diện → Thanh toán và đăng ký → Đăng ký
```

Bên trong nó làm gì

1. Thêm tài khoản vào điện thoại (chưa có thì thêm, có rồi thì bỏ qua)
2. Chuyển tài khoản Play. Chuyển xong nhất định đọc lại từ bảng ảnh đại diện để đối chiếu, không khớp thì thoát ngay, không chạy nhầm sang tài khoản khác
3. Khởi động Google One, dùng Frida gắn vào, DexClassLoader nạp module thanh toán
4. queryProductDetailsAsync tra g1.play.20tb, lấy về 5 offer
5. Chọn cái có thời gian miễn phí dài nhất (eft-12month, P1Y), gọi launchBillingFlow trên main thread
6. Canh chừng quầy thanh toán. Thấy giá là 0 hoặc "dùng thử miễn phí 12 tháng" mới bấm đăng ký; thấy "không đủ điều kiện dùng thử miễn phí" thì đóng lại, tuyệt đối không bấm
7. Play đôi khi yêu cầu nhập lại mật khẩu, script tự điền
8. Mở trang đăng ký đối chiếu, kết quả ghi vào ultra0-report/

Khi thất bại thì thế nào

Tài khoản không đủ điều kiện. Play sẽ báo "bạn không đủ điều kiện dùng thử miễn phí, tuy vậy bạn vẫn có thể đăng ký". Một tài khoản cơ bản chỉ có một cơ hội: một khi đã khởi tạo dùng thử, dù chưa xác nhận, offer cũng sẽ biến mất. Nên đừng lấy tài khoản chính ra thử.

Không có phương thức thanh toán. Đăng ký bắt buộc phải có thẻ. Trang thêm thẻ của Play là WebView, adb không điền vào được, script sẽ dừng lại nhắc bạn điền xong trên điện thoại rồi nhấn Enter. Mỗi thiết bị thao tác thủ công một lần, sau đó mọi tài khoản dùng chung thẻ này.

Tài khoản Play chưa chuyển sang. Script sẽ in ra tài khoản thực tế đang đăng nhập là cái nào, rồi thoát. Lúc này chuyển thủ công trong Play một chút, thêm --no-switch chạy lại, để nó chỉ kiểm tra không thử chuyển nữa:

```bash
python3 ultra0.py accounts.txt --no-switch
```

Quầy thanh toán không bật lên. Chạy lại một lần. Hai lần liên tiếp không bật, kiểm tra mạng của thiết bị: settings get global http_proxy, không phải null thì xóa đi; VPN kiểu Clash sẽ chiếm DNS, cũng khiến Play không kết nối được.

Một số tham số

```bash
--plan 5x      # chỉ mở 20TB
--plan 20x     # chỉ tra 30TB (phần lớn tài khoản không lấy được dùng thử)
--plan auto    # mặc định, tra cả hai gói, chọn cái có thời gian miễn phí dài nhất
--no-add       # tài khoản đã có trên điện thoại, bỏ qua bước thêm
--no-switch    # không tự chuyển tài khoản, chỉ kiểm tra tài khoản hiện tại
```

Vài lưu ý

Thử xong nhớ vào Play hủy đăng ký, không thì hết hạn sẽ bị trừ $99.99.

Script chỉ chạy cục bộ, tài khoản mật khẩu và số thẻ không lên mạng, cũng không tải lên bất kỳ đâu. File báo cáo ở ultra0-report/, bên trong có chi tiết offer và kết quả từng bước.

Đoạn code này chỉ lợi dụng việc "client lấy được offer dùng thử" trong quy trình thanh toán Play, không sửa bất kỳ dữ liệu phía server nào.

Liên hệ

Có vấn đề vào nhóm hỏi. Ba nhóm vào cái nào cũng được, đều đầy thì thêm WeChat của tôi, ghi chú "Ultra", tôi kéo bạn vào.

<table>
<tr>
<td align="center"><b>Nhóm 2</b><br><img src="assets/wechat-group.jpg" width="300" alt="Mã QR nhóm 2"></td>
<td align="center"><b>Nhóm 3</b><br><img src="assets/wechat-group3.jpg" width="300" alt="Mã QR nhóm 3"></td>
<td align="center"><b>Đều đầy thì thêm tôi</b><br><img src="assets/wechat-admin.jpg" width="300" alt="Mã QR WeChat"></td>
</tr>
</table>

Liên kết bạn bè

Dự án này được công bố và thảo luận tại cộng đồng LINUX DO, cảm ơn sự ủng hộ của cộng đồng.

· LINUX DO —— cộng đồng những người lý tưởng mới
· LINUX DO Quảng bá mã nguồn mở —— chuyên mục quảng bá dự án mã nguồn mở

https://img.shields.io/badge/LINUX%20DO-%E7%A4%BE%E5%8C%BA-1f6feb

Giấy phép

MIT, xem LICENSE.
