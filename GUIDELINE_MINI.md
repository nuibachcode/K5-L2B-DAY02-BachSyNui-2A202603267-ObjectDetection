# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Phương tiện giao thông cá nhân/gia đình cỡ nhỏ từ 4-7 chỗ, bao gồm sedan, hatchback, SUV, CUV, taxi và xe bán tải (pickup truck) cabin nhỏ. | Xe 4-7 chỗ thông dụng (Vios, Camry, CX-5, SantaFe), taxi đô thị, xe bán tải nhỏ (Ranger, Hilux). Phân biệt: không gán cho xe tải lớn, xe khách hay xe van thân hộp. |
| `truck` | Phương tiện chuyên dụng chở hàng hóa có kết cấu cabin tách rời thùng hàng hoặc sàn chở hàng; bao gồm xe tải thùng kín, xe tải hở, xe ben, xe bồn, xe đầu kéo container. | Xe tải chở hàng lớn, xe đầu kéo, xe chuyên chở vật liệu. Phân biệt: xe bán tải pickup nhỏ tính là `car`, không tính vào `truck`. |
| `bus` | Phương tiện giao thông công cộng hoặc chuyên chở hành khách số lượng lớn (thường từ 24 chỗ trở lên), bao gồm xe buýt nội đô, xe khách liên tỉnh, xe buýt khớp nối (articulated bus). | Xe buýt công cộng 2 tầng hoặc đơn, xe khách giường nằm, xe buýt sân bay dài. Phân biệt: xe van/minivan 7-16 chỗ cỡ nhỏ thuộc `van`. |
| `van` | Xe thân hộp đơn khối (monospace / 1-box design) từ 7-16 chỗ, mui cao, thân phẳng, dùng để chở hàng hóa bưu kiện hoặc chở người thương mại/gia đình. | Ford Transit, Toyota HiAce, Hyundai Starex, xe van chuyển phát bưu chính. Phân biệt: không gán cho xe buýt lớn (`bus`) hay xe SUV/sedan (`car`). |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là**: Bounding box 2D ôm khít mép ngoài của các phần thân xe nhìn thấy được (thân, bánh xe, gương chiếu hậu, mui xe). Tuyệt đối không lấy lấn khoảng trống mặt đường, bầu trời, bóng đổ (shadow) hay xe bên cạnh; không gộp nhiều xe vào một box duy nhất.
- **Vật thể bị che khuất**: Chỉ vẽ box ôm trọn phần thân xe **thực sự nhìn thấy được** (visible parts), không ước lượng hay vẽ bù phần bị khuất sau xe khác, cột đèn, cây cối, hoặc người đi bộ. Nếu vật thể bị che khuất quá nhiều (trên 80-90%) không còn nhận diện rõ đặc trưng lớp thì bỏ qua không gán.
- **Vật thể bị cắt bởi rìa ảnh**: Bounding box phải chạm sát mép ảnh tại vị trí cắt (`x=0, 1` hoặc `y=0, 1`). Không mở rộng box ra ngoài biên ảnh giả định phần xe ngoài khung.
- **Vật thể rất nhỏ/mơ hồ**: Nếu vật thể quá xa ở cuối trục đường (dưới ~12-15 pixels) hoặc mờ nhòe nghiêm trọng do chuyển động/thời tiết không thể xác định chắc chắn thuộc lớp nào thì **không tự đoán** để tránh gây nhiễu cho mô hình.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe ô tô con màu trắng đi ở làn giữa bị thanh ngang của cột kim loại (gantry) chắn ngang capo/kính trước; đồng thời có xe đỗ trong bãi sau hàng rào bên trái. | 1. Với xe trắng bị chắn: Vẽ 1 box `car` ôm trọn vùng nhìn thấy của thân xe (tính cả phần trên và dưới thanh chắn).<br>2. Với các xe đang đỗ bên lề: Vẫn gán `car` nếu nhìn rõ thân xe. | Schema định nghĩa loại phương tiện, không phụ thuộc trạng thái di chuyển (xe đỗ vẫn là `car`). Vật thể bị che khuất một phần vẫn gán nhãn bao quanh phần nhìn thấy vì nhận diện rõ lớp. |
| `drive_015.jpg` | 1. Xe tải nhỏ cabin đơn chở bao hàng cao sau thùng hở (làn trong).<br>2. Xe buýt công cộng cực dài có khớp nối cao su ở giữa (làn ngoài).<br>3. Xe ở mép đáy ảnh bị cắt chỉ thấy phần mui. | 1. Gán `truck` (ID 1) cho xe chở hàng thùng hở.<br>2. Gán 1 box duy nhất `bus` (ID 2) bao trọn toàn bộ xe buýt khớp nối.<br>3. Gán `car` cho xe ở đáy ảnh, box chạm đúng mép dưới ảnh. | Xe có cabin tách rời sàn chở hàng chuyên dụng thuộc định nghĩa `truck`. Xe buýt khớp nối là một khối phương tiện chở khách công cộng thống nhất. Xe cắt rìa phải gán theo phần thấy và chạm đúng biên ảnh. |
| `drive_026.jpg` | 1. Hàng loạt xe buýt lớn xếp nối đuôi nhau che lấp một phần đầu/đuôi của nhau.<br>2. Có xe minivan/MPV màu tối di chuyển cạnh xe buýt.<br>3. Người đi bộ băng qua đường cạnh các phương tiện. | 1. Mỗi xe buýt vẽ một box `bus` riêng biệt ôm sát phần nhìn thấy (không gộp).<br>2. Gán `van` (ID 3) cho xe dạng thân hộp 1 khoang.<br>3. Hoàn toàn không gán nhãn cho người đi bộ. | Mỗi phương tiện là 1 đối tượng độc lập. Lớp `van` phân biệt rõ với `bus` (kích thước lớn) và `car` (sedan/SUV). Người đi bộ ngoài scope của schema 4 lớp. |
| `drive_003.jpg` | Giao lộ ùn tắc nghiêm trọng, cụm xe ở rất xa dưới chân cầu vượt bị che lấp dày đặc, chỉ còn nhìn thấy 1 mảnh mui xe nhỏ vài pixel. | Chỉ gán box cho các xe còn nhận diện được đặc trưng loại xe (thấy từ ~30% thân xe trở lên). Những mảnh mui xe quá nhỏ/mờ ở xa bị che khuất gần hết thì bỏ qua không gán. | Tránh tự đoán khi vật thể không đủ thông tin nhận diện lớp hoặc không thể xác định ranh giới box, bảo đảm chất lượng ground truth. |
