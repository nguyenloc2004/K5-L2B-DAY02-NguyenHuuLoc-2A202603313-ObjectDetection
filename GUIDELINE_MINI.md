# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, xe bán tải. | Thân 3 khối (capo – cabin kính – cốp), mui thấp, capo nhìn rõ. Bán tải có cabin nhỏ và thùng hở phía sau nhưng vẫn là `car`, không phải `truck`. |
| `truck` | Xe chở hàng: xe tải thùng kín, thùng hở, xe đầu kéo kèm rơ-moóc. | Cabin tài xế tách riêng với thùng hàng (thấy khe hở giữa hai phần), thùng hàng không có cửa sổ. Khác van ở chỗ cabin và thùng là hai khối riêng. |
| `bus` | Xe chở khách nhiều chỗ: xe buýt, xe khách, minibus. | Thân hộp dài (gấp 2–4 lần xe con), đầu xe phẳng đứng, có hàng cửa sổ hành khách chạy dọc thân. Khác truck ở chỗ có cửa sổ khách, không có thùng hàng. |
| `van` | Xe van, minivan chở hàng hoặc chở người. | Một khối hộp, capo rất ngắn hoặc không có, mui cao phẳng kéo đến đuôi, kích thước gần bằng xe con. Khác car ở chỗ không có capo rõ và mui cao; khác bus ở chỗ ngắn hơn hẳn. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh box chạm đúng điểm ngoài cùng của thân xe nhìn thấy được (kể cả gương chiếu hậu nếu thấy rõ), không lấy thêm mặt đường, bóng xe, trời hay xe bên cạnh. Mỗi xe một box riêng, kể cả nhiều xe cùng lớp đứng sát nhau.
- Vật thể bị che khuất: nếu phần nhìn thấy vẫn đủ để nhận ra lớp thì vẽ box quanh **phần nhìn thấy**, không đoán phần bị che. Hai xe chồng nhau thì vẽ hai box riêng, mỗi box ôm phần nhìn thấy của xe đó.
- Vật thể bị cắt bởi rìa ảnh: vẫn gán nhãn nếu nhận ra lớp; cạnh box đặt đúng tại rìa ảnh, không kéo ra ngoài ảnh hay đoán phần nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: nếu xe ở xa hoặc chỉ còn mảnh nhỏ mà vẫn nhận ra chắc lớp thì gán; nếu không xác định chắc được lớp thì **không đoán**, ghi tên ảnh vào bảng ca mơ hồ bên dưới và hỏi giảng viên. Không gán xe máy, xe đạp, người đi bộ, xe trong gương/kính hoặc hình xe trên biển quảng cáo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_...` | | | |
| `drive_...` | | | |
| `drive_...` | | | |
