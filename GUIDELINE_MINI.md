# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người, thân thấp, có nắp capo và cốp tách rõ (hoặc đuôi SUV): sedan, hatchback, SUV, taxi, xe bán tải cabin nhỏ. | Phân biệt với `van`: car thấp hơn, kính hông nhỏ, nóc dốc về sau. Xe bán tải dù có thùng hàng vẫn là `car`. |
| `truck` | Xe chuyên chở hàng hóa, có cabin tách biệt với thùng hàng/rơ-moóc: xe tải thùng, xe ben, xe đầu kéo, xe bồn. | Phân biệt với `car`: cabin cao, thùng hàng lớn phía sau, nhiều trục bánh. Hàng chở trên thùng (đất, cát) tính vào box. |
| `bus` | Xe chở khách cỡ lớn/trung: xe buýt thành phố, xe buýt nối toa, xe khách, minibus chở khách. | Thân dài hình hộp, nhiều cửa sổ dọc hai bên, thường có biển số tuyến/đèn LED tuyến. Phân biệt với `van`: dài hơn rõ rệt, có từ 2 hàng cửa sổ trở lên. |
| `van` | Xe thân hộp một khối (không tách capo/cốp rõ), nóc cao, cửa lùa, dùng chở hàng hoặc chở người: van, minivan. | Phân biệt với `car`: nóc cao và phẳng, đuôi gần như thẳng đứng. Phân biệt với `bus`: ngắn, kích thước gần xe con. |

## Quy tắc chung

- **Box bám sát vật thể** nghĩa là: 4 cạnh box chạm mép ngoài cùng của phần thân xe nhìn thấy (nóc, đáy bánh xe, đầu, đuôi); không lấy thêm mặt đường, bóng đổ, trời hay xe bên cạnh. Gương chiếu hậu không bắt buộc tính vào box. Mỗi xe một box riêng, kể cả xe nối toa (một phương tiện → một box).
- **Vật thể bị che khuất**: vẽ box quanh phần nhìn thấy được nếu vẫn nhận ra lớp; không đoán phần bị che. Hai xe chồng lên nhau → hai box riêng, không gộp.
- **Vật thể bị cắt bởi rìa ảnh**: cạnh box chạm đúng rìa ảnh, không kéo ra ngoài và không đoán phần nằm ngoài khung. Vẫn gán nếu phần nhìn thấy đủ nhận ra lớp.
- **Vật thể rất nhỏ/mơ hồ**: xe ở xa vẫn gán nếu nhìn rõ đó là xe và xác định được lớp (zoom để vẽ chính xác). Nếu chỉ là vài pixel hoặc không chắc lớp → không tự đoán, ghi vào bảng dưới và hỏi giảng viên.
- **Không gán**: mô tô/xe máy, xe đạp, người đi bộ, biển báo, đèn giao thông, xe trong gương/kính phản chiếu, hình xe trên bảng quảng cáo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_008` | Xe buýt nối toa vàng-trắng ở giữa ảnh: hai toa nối bằng khớp xếp | 1 box `bus` ôm cả hai toa | Đây là một phương tiện duy nhất; quy tắc "mỗi xe một box" — tách 2 box sẽ thành 2 vật thể sai |
| `drive_008` | Xe trắng thân cao hình hộp phía trên, sau xe buýt đỏ, bị che một phần | `van` | Nóc cao phẳng, đuôi thẳng, không tách capo/cốp như sedan → theo định nghĩa `van`, không phải `car` |
| `drive_008` | Xe tải ben chở đất nhô cao khỏi thùng | 1 box `truck`, tính cả phần đất trên thùng | Hàng hóa là phần nhìn thấy gắn liền với xe; box bám phần nhìn thấy của vật thể |
| `drive_008` | Xe màu đỏ sẫm bị cắt ở rìa phải ảnh, chỉ thấy một dải hẹp thân xe và bánh xe | `car`, box chạm rìa phải ảnh | Phần nhìn thấy (thân, bánh, kích thước tương đương xe con bên cạnh) đủ nhận ra xe con; quy tắc cắt khung: box chạm rìa, không đoán phần ngoài ảnh. Nếu giảng viên coi là quá ít thông tin thì bỏ box này |
