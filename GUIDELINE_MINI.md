# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Quy ước ID lớp

Bản nhãn ban đầu dùng thứ tự ID theo README/`data.yaml` gốc: `0 car, 1 truck, 2 bus, 3 van`.

Khi chấm với `gold_train_labels/`, tôi phát hiện id bị lệch so với README và data.yaml. Nhãn eval có sẵn trong repo (`dataset/labels/eval/`) cũng dùng cùng quy ước với gold.

**Quyết định:** để nhãn train nhất quán với gold train và nhãn eval dùng khi train/đánh giá, tôi chuyển toàn bộ nhãn của mình sang quy ước của gold và cập nhật `data.yaml` cho khớp:

| ID | Lớp (quy ước mới, theo gold) | ID cũ theo README |
| ---: | --- | ---: |
| 0 | `car` | 0 |
| 1 | `van` | 3 |
| 2 | `truck` | 1 |
| 3 | `bus` | 2 |


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
| `drive_012` | Xe khách cỡ nhỏ màu kem: thân hộp, đầu xe phẳng, có hàng cửa sổ khách dọc thân nhưng ngắn hơn xe buýt thường. Dễ nhầm với `van`. | `bus` | Schema ghi rõ `bus` gồm "minibus chở khách". Xe có hàng cửa sổ hành khách, cửa lên xuống riêng, dài hơn hẳn van cỡ xe con, nên là minibus, không phải van. |
| `drive_014` | Xe van đỏ mui cao : cao và dài hơn xe con, có cửa sổ thân. Dễ nhầm với `bus`. | `van` | Một khối hộp liền, capo ngắn, chiều dài gần bằng xe con lớn; không có hàng ghế khách dài như minibus. Schema xếp xe van chở người vào `van`.|
| `drive_014` | Xe cứu hộ có cần cẩu phía sau. Không phải xe tải chở hàng thông thường. | `truck` | Cabin tài xế tách riêng với phần sau (sàn kéo + cần cẩu), đúng đặc điểm phân biệt `truck` trong guideline. Không chở khách nên không phải `bus`/`van`. Gold cũng ghi `truck`. |
| `drive_049` | Dãy xe đỗ dọc lề trái và xe rất nhỏ ở xa cuối đường. README ghi "xe đang đỗ vẫn là car", nhưng gold không gán các xe này. | Không gán xe đỗ ngoài lòng đường chính và xe quá nhỏ ở xa | Bản đầu tôi gán theo README. Sau gold QC, tôi thống nhất phạm vi với gold: chỉ gán xe trong vùng đường chính đủ gần camera |
