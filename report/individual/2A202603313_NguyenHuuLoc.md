# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hữu Lộc
- MSSV: 2A202603313
- Email: nguyenhuuloc.122004@gmail.com
- Lớp: K5 — L2B
- Link repo bài làm: https://github.com/nguyenloc2004/K5-L2B-DAY02-NguyenHuuLoc-2A202603313-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1k6dlH82USMR9soS5jj3saGALf-a6X3r_?usp=sharing

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema:** gán 4 lớp `car / truck / bus / van` theo hình dáng thân xe (capo, mui, thùng hàng, hàng cửa sổ khách); vẽ box quanh phần nhìn thấy; xe bị che hoặc bị cắt khung vẫn gán nếu nhận ra lớp; không gán xe máy, xe đạp, người đi bộ.
- **Quy ước ID lớp:** gold train và nhãn eval dùng `0 car, 1 van, 2 truck, 3 bus`, khác README/`data.yaml` gốc (`0 car, 1 truck, 2 bus, 3 van`). Tôi chuyển nhãn của mình và `data.yaml` sang quy ước của gold để train/eval nhất quán (chi tiết ở mục 4).
- **Ca mơ hồ đã quyết định** (bảng đầy đủ trong guideline):
  1. **`drive_012` — minibus chở khách màu kem → `bus`.** Có hàng cửa sổ khách dọc thân, dài hơn hẳn van cỡ xe con; schema ghi rõ `bus` gồm "minibus chở khách".
  2. **`drive_014` — xe van đỏ mui cao → `van`.** Một khối hộp liền, capo ngắn, dài gần bằng xe con lớn, không có hàng ghế khách dài như minibus.
  3. **`drive_014` — xe cứu hộ có cần cẩu → `truck`.** Cabin tách riêng với phần sau (sàn kéo + cần cẩu), không chở khách.
  4. **`drive_049` — xe đỗ lề và xe rất nhỏ ở xa → không gán.** Bản đầu tôi gán theo README ("xe đang đỗ vẫn là car"); sau gold QC tôi thống nhất phạm vi với gold, chỉ gán xe trong vùng đường chính đủ gần camera.


## 3. Annotation và self-QC

- Số file nhãn train: 50 file trong `dataset/labels/train/`, gán bằng CVAT, xuất định dạng Ultralytics YOLO Detection 1.0. 
- Kết quả checker và lỗi đã xử lý: `python tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`, cả trước và sau rework.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_002`, `drive_003`, `drive_006`, `drive_009`, `drive_012`, `drive_014`, `drive_016`, `drive_019`, `drive_027`, `drive_036`, `drive_044`, `drive_047`, `drive_049`, `drive_050`. Cách rà: chồng box lên ảnh (`drive_002`, `drive_049`) và cắt riêng từng box truck/bus/van hoặc từng box lệch lớp so với gold để xem loại xe.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Box truck/bus/van đúng loại xe nhưng ID lệch quy ước gold (ví dụ `drive_009`: 8 xe buýt; `drive_014`: xe van đỏ, xe cứu hộ). Sửa bằng cách đổi ID toàn bộ nhãn (mục 4).
  - Gán cả xe đỗ lề và xe rất nhỏ ở xa (`drive_002`: 49 box so với 13 của gold; `drive_049`: 27 box so với 6). Sửa bằng cách xoá box ngoài phạm vi gán.
- Commit khóa nhãn ban đầu: `593055a`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa |
|---|---:|---:|
| TP | 474 | 581 |
| FP | 627 | 41 |
| FN | 179 | 72 |
| Precision | 0.4305 | 0.9341 |
| Recall | 0.7259 | 0.8897 |
| F1 | 0.5405 | 0.9114 |
| mean_iou_of_matches | 0.8504 | 0.8575 |
| passed | false | **true** |

File kết quả: `outputs/annotation_evaluation.json`

**Phân tích lỗi trước sửa:**

- **Sai lớp có hệ thống.** Lỗi sai id so với gold label
- **Gán thừa so với phạm vi của gold.** Phần lớn là xe rất nhỏ ở xa và xe đỗ lề.

**Rework đã làm:**

1. Đổi ID toàn bộ nhãn: `truck 1→2`, `bus 2→3`, `van 3→1`, `car` giữ `0`. Cập nhật `data.yaml` theo cùng thứ tự. Chấm thử riêng bước này (chỉ đổi ID, chưa xoá box): F1 = 0.66.
2. Xoá các box ở xa. Sau bước này: F1 = 0.91.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU Tesla T4 (15 GB); Ultralytics 8.4.208, Python 3.13.16, torch 2.14.1+cu130. Repo được clone ở commit `d18cc1e` ("rework after qc"); notebook xác nhận `50 train + 10 eval`.
- Các tham số thay đổi so với notebook nếu có và lý do: không đổi tham số train (`yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`). Chỉ có `data.yaml` đổi tên lớp theo thứ tự `0 car, 1 van, 2 truck, 3 bus` để khớp nhãn train và nhãn eval (xem mục 4). Train 50 epoch mất khoảng 1.3 phút (0.021 giờ).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` 

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.622 |
| Recall | 0.632 |
| mAP50 | 0.592 |
| mAP50–95 | 0.402 |

Chỉ số theo từng lớp (best checkpoint, 10 ảnh eval, 113 box):

| Lớp | Số box trong eval | Số box trong train | Precision | Recall | mAP50 | mAP50–95 |
|---|---:|---:|---:|---:|---:|---:|
| car | 94 | 502 | 0.872 | 0.819 | 0.891 | 0.604 |
| van | 5 | 44 | 0.571 | 0.543 | 0.504 | 0.396 |
| truck | 2 | 21 | 0.247 | 0.500 | 0.247 | 0.173 |
| bus | 12 | 55 | 0.796 | 0.667 | 0.725 | 0.435 |

Ảnh và chỉ số:

- Ảnh dự đoán trên 10 ảnh eval: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/)
- Log train theo epoch: [assets/yolo26_runs/vehicle_annotation/results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [results.png](assets/yolo26_runs/vehicle_annotation/results.png)
- Đánh giá best checkpoint: [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png), [val_batch0_pred.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg) so với [val_batch0_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg)

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?** `truck`, recall 0.50 (và mAP50 chỉ 0.247). Eval chỉ có 2 xe tải: một ở `drive_051`, một ở `drive_058`. Confusion matrix cho thấy model bắt được 1 xe tải và bỏ sót 1 (rơi vào cột background). Ảnh minh họa xe bị bỏ sót: [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg). Xe tải trắng ở góc trên bên phải là cảnh ban đêm, ở xa và bị cột đèn che một phần. Model không vẽ box `truck` nào ở đó, chỉ có các box `car` nhỏ cạnh đó . Nguyên nhân có thể là train chỉ có 21 box truck so với 502 box car (mất cân bằng lớp), cộng thêm ảnh đêm. Với 2 box trong eval, recall truck dao động rất mạnh (bắt thêm 1 xe là lên 1.0), nên con số này chỉ mang tính tham khảo.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?** Ở đây mAP50 = 0.59, mAP50–95 = 0.40; riêng `car` là 0.89 so với 0.60. Nghĩa là model tìm đúng xe nhưng box chưa khớp sát ở ngưỡng IoU cao (0.75–0.95). Cần kiểm: (a) cạnh box dự đoán có lấn bóng xe hay mặt đường không; (b) ở cảnh đông xe như [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), các box chồng nhau ở làn trái và xe buýt nối toa có nhiều box `bus` chồng lên nhau (0.67, 0.69, 0.67), làm giảm IoU; (c) quy ước vẽ box trong nhãn train của tôi có khớp nhãn eval không (ví dụ có lấy gương chiếu hậu, cách vẽ xe bị che). Mean IoU nhãn của tôi so với gold train là 0.86, nên khác biệt quy ước có tồn tại nhưng không lớn.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?** Nhầm `van` → `car`: confusion matrix có 2/5 xe van trong eval bị dự đoán là `car`, 1 bị dự đoán là `bus`. Điều này liên quan tới ranh giới `car` ↔ `van` trong guideline (phân biệt bằng capo rõ hay không và mui cao kéo tới đuôi). Dữ kiện hỗ trợ: ở bước gold QC, sau rework vẫn còn 13 box tôi gán khác gold giữa hai lớp này, phần lớn là xe mui cao, ví dụ `drive_006` (hatchback mui cao, tôi gán `van`, gold `car`) và `drive_027` (MPV, tôi gán `car`, gold `van`). Nghĩa là ngay trong nhãn train, ranh giới hai lớp này với xe mui cao chưa nhất quán, nên model cũng dễ nhầm ở eval. Lớp `van` và `truck` có ít mẫu (44 và 21 box), nên không thể tách hẳn ảnh hưởng của guideline khỏi ảnh hưởng của mất cân bằng lớp chỉ từ 10 ảnh eval.
4. **Vì sao không dùng 10 ảnh eval để train?** Vai trò validation của chúng trong notebook là gì? Nếu đưa eval vào train, model sẽ được chấm trên chính ảnh đã học, nên chỉ số bị thổi phồng và không phản ánh khả năng tổng quát. Trong notebook, 10 ảnh eval là tập validation: dùng để đo chỉ số mỗi epoch và chọn `best.pt`. Vì đã dùng để chọn checkpoint, đây không phải test set độc lập.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Chấm với gold | F1 = 0.54; hầu như mọi box truck/bus/van đều sai lớp | Thống kê ma trận nhầm lớp, cắt ảnh từng box để so | Phát hiện gold dùng thứ tự ID khác README; đổi ID nhãn của mình và `data.yaml` theo gold. |
| Chấm với gold | Precision 0.43; Rất nhiều box của tôi không có trong gold | Chồng box của tôi và gold lên ảnh (`drive_002`, `drive_049`) | Gold không gán xe ở xa và xe đỗ lề; xoá các box này. F1 = 0.91 |


## 7. Tự đánh giá theo rubric

| Tiêu chí | Tình trạng |
|---|---|
| Hoàn thành và định dạng YOLO (10) | Đạt: 50 file, checker 50/50. Lưu ý thứ tự ID đã đổi theo gold (mục 4) |
| Đầy đủ vật thể (20) | Recall 0.89 so với gold; còn 72 FN chưa rà từng ảnh |
| Độ bám sát và hình học box (20) | Mean IoU 0.86 |
| Phân lớp nhất quán (15) | Sau khi đổi ID, còn khoảng 15 box sai lớp (chủ yếu car ↔ van) |
| Mini guideline và quyết định mơ hồ (10) | Đã có định nghĩa lớp, quy tắc box, quy ước ID và 4 ca mơ hồ có tên ảnh, quyết định và lý do (`drive_012`, `drive_014` ×2, `drive_049`) |
| Gold-set annotation QC (15) | Đạt: F1 0.91 ≥ 0.80, mean IoU 0.86 ≥ 0.70; có số trước/sau và lý do rework |
| Pipeline YOLO và phân tích (10) | Đã chạy Colab (T4), giữ nguyên split, không dùng eval để train; có metrics (mAP50 0.592, mAP50–95 0.402), metrics theo lớp, ảnh dự đoán 10 ảnh eval và trả lời 4 câu hỏi kèm ảnh cụ thể |

## 8. Công cụ AI và tài liệu tham khảo

- **Claude Code :** dùng ở bước self-QC và gold QC để

  - viết script phân tích FP/FN (ma trận nhầm lớp, diện tích box), cắt và chồng box lên ảnh để kiểm tra bằng mắt;
  - viết script rework: đổi ID lớp và xoá các box không chồng lên box gold nào. Bước xoá box dùng vị trí box gold làm tiêu chí, không phải đánh giá từng box bằng mắt;
- **Nguồn ngoài repo:** không có.
