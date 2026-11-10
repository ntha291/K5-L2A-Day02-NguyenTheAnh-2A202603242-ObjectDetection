# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` |Phương tiện chở người có kích thước nhỏ, thường có 4 bánh và tối đa khoảng 5–7 chỗ ngồi. |Sedan, hatchback, SUV, crossover. Có khoang hành khách và khoang động cơ riêng hoặc liền thân. |
| `truck` |Phương tiện vận tải hàng hóa, thường có kích thước lớn, kết cấu chuyên chở hàng hóa. |Xe tải thùng, xe tải ben, xe đầu kéo, xe container. Có thể có thùng hàng hoặc rơ-moóc. |
| `bus` |Phương tiện vận tải hành khách với sức chứa lớn, có nhiều hàng ghế và thường có kích thước lớn hơn car và van. |Xe buýt nội thành, xe khách, xe buýt trường học, xe khách đường dài. |
| `van` |Phương tiện có thân xe dạng hộp, kích thước trung bình, được thiết kế để chở người hoặc hàng hóa. |Minivan, xe tải van, xe giao hàng, xe chở khách dạng van. Thường có khoang sau kín. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding Box phải bao quanh phương tiện sát nhất có thể, bao gồm toàn bộ phần phương tiện nhìn thấy được. Không lấy dư quá nhiều background.
- Vật thể bị che khuất: Chỉ vẽ Bounding Box theo phần phương tiện nhìn thấy được. Không tự suy đoán hoặc vẽ thêm phần bị che khuất. Chỉ gán nhãn nếu vẫn xác định được loại phương tiện.
- Vật thể bị cắt bởi rìa ảnh: Vẽ Bounding Box đến sát mép ảnh. Không loại bỏ vật thể chỉ vì nó không xuất hiện đầy đủ trong ảnh.
- Vật thể rất nhỏ/mơ hồ: Chỉ gán nhãn khi vẫn có thể xác định tương đối chắc chắn loại phương tiện. Nếu không thể phân biệt hoặc nhận diện đáng tin cậy, bỏ qua theo quy định của dataset.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_008.jpg` | Xuất hiện xe buýt khớp nối 2 toa (vàng-trắng-xanh), xe tải thùng đỏ chở đầy hàng ở giữa ngã tư, hai xe minivan trắng và người đi xe máy/xe đạp bên trái. | Gán 1 box `bus` (ID 2) ôm trọn xe buýt khớp nối, 1 box `truck` (ID 1) cho xe tải đỏ, `van` (ID 3) cho xe microvan hộp trắng; bỏ qua xe máy/xe đạp. | Mỗi phương tiện thuộc schema là một box riêng theo phần nhìn thấy; mô tô/xe đạp nằm ngoài 4 lớp của schema nên không gán. |
| `drive_014.jpg` | Xe cứu hộ giao thông (tow truck màu trắng có cần trục kéo phía sau) đỗ sát dải phân cách, xe van/cứu thương màu đỏ ở bên phải và 3 xe con bị cắt một phần ở sát mép dưới ảnh. | Gán xe cứu hộ là `truck` (ID 1), xe hộp đỏ bên phải là `van` (ID 3), và vẽ box chạm sát biên dưới (`y_max = 1.0`) cho 3 xe `car` (ID 0) bị cắt khung. | Xe cứu hộ có kết cấu cabin + sàn/cần cẩu tải chuyên dụng nên thuộc `truck`; các xe bị cắt khung ở mép dưới vẫn nhìn rõ trên 50% thân xe con nên phải gán đủ và dừng đúng rìa ảnh. |
| `drive_024.jpg` | Làn gần nhất có một phương tiện 3 bánh có mái che (xe ba gác/xe lam nhỏ) đi phía trước hai xe minivan/microvan thân hộp màu bạc-xám và hàng xe con/taxi. | Không gán phương tiện 3 bánh có mái che; gán hai xe thân hộp màu bạc-xám là `van` (ID 3) và các xe sedan/taxi xung quanh là `car` (ID 0). | Phương tiện 3 bánh có mái che thuộc nhóm mô tô/xe ba bánh không nằm trong 4 lớp ô tô của schema; hai xe màu bạc có dáng hộp đứng đầu ngắn đặc trưng của `van`. |
| `drive_033.jpg` | Làn giữa có xe tải thùng kín màu trắng dễ nhầm với `van`/`bus` khi nhìn từ trên cao; phía xa gần cầu vượt có nhiều đốm xe rất nhỏ và mờ. | Gán xe thùng kín màu trắng là `truck` (ID 1); chỉ gán các xe ở khoảng cách trung bình–gần còn nhận diện rõ hình khối và bỏ qua các đốm xe quá nhỏ phía cầu vượt. | Xe màu trắng có cabin đầu kéo tách biệt với thùng hàng vuông phía sau nên là `truck`; các xe quá xa ở chân trời không đủ chi tiết xác định lớp theo quy tắc không tự đoán. |
| `drive_042.jpg` | Xe tải chở các bao hàng xanh/trắng chất cao vượt quá nóc cabin đi cạnh xe buýt 2 tầng (bên phải) và xe buýt đỏ-trắng bị cắt khung ở góc dưới. | Gán `truck` (ID 1) bao gồm cả phần thân xe và khối hàng hóa chất trên thùng xe; gán cả hai xe buýt là `bus` (ID 2) bám sát phần nhìn thấy. | Hàng hóa chở trên thùng là một phần thể tích nhìn thấy gắn liền của xe tải (`truck`) trong cảnh giao thông; hai xe buýt (1 tầng và 2 tầng) đều thuộc lớp `bus`. |
