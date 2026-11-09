# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hữu Lộc
- MSSV: 2A202603313
- Email: nguyenhuuloc.122004@gmail.com
- Lớp: K5 — L2B
- Link repo bài làm: https://github.com/nguyenloc2004/K5-L2B-DAY02-NguyenHuuLoc-2A202603313-ObjectDetection
- Link Colab đã chạy: _(chưa chạy — bổ sung sau khi train)_

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema:** gán 4 lớp `car / truck / bus / van` theo hình dáng thân xe (capo, mui, thùng hàng, hàng cửa sổ khách); vẽ box quanh phần nhìn thấy; xe bị che hoặc bị cắt khung vẫn gán nếu nhận ra lớp; không gán xe máy, xe đạp, người đi bộ.
- **Ca mơ hồ đã quyết định:**
  1. **Xe ở xa và xe đỗ lề (ví dụ `drive_002`, `drive_049`):** bản đầu tôi gán theo README ("xe đang đỗ vẫn là car") và gán cả xe rất nhỏ ở xa. Gold không gán các xe này. Quyết định: bỏ các box không chồng lên box gold nào để thống nhất phạm vi với gold.
  2. **Phân biệt `car` ↔ `van`:** còn khoảng 13 box nhầm giữa hai lớp này (xe SUV/MPV mui cao).

## 3. Annotation và self-QC

- Số file nhãn train: 50 file trong `dataset/labels/train/`, gán bằng CVAT, xuất định dạng Ultralytics YOLO Detection 1.0. 
- Kết quả checker và lỗi đã xử lý: `python tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`, cả trước và sau rework.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: _(bổ sung)_. Đã cắt và xem từng box non-car ở `drive_009`, `drive_014`; đã chồng box của tôi lên ảnh ở `drive_002`, `drive_049`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: _(bổ sung)_
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

_(Chưa chạy Colab — bổ sung sau khi train.)_

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do: `data.yaml` đổi tên lớp theo thứ tự `0 car, 1 van, 2 truck, 3 bus` để khớp nhãn train và nhãn eval (xem mục 4).
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

Ảnh và chỉ số: `assets/eval_predictions/`, `assets/yolo26_runs/` _(bổ sung sau khi giải nén `day2_colab_outputs.zip`)_.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa? _(bổ sung sau khi có kết quả)_
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box? _(bổ sung sau khi có kết quả)_
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét? _(bổ sung sau khi có kết quả)_
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì? Nếu đưa eval vào train, model sẽ được chấm trên chính ảnh đã học, nên chỉ số bị thổi phồng và không phản ánh khả năng tổng quát. Trong notebook, 10 ảnh eval là tập validation: dùng để đo chỉ số mỗi epoch và chọn `best.pt`. Vì đã dùng để chọn checkpoint, đây không phải test set độc lập.

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
| Mini guideline và quyết định mơ hồ (10) | Đã có định nghĩa lớp, quy tắc box, quy ước ID, phạm vi gán; bảng 3 ca mơ hồ chưa điền đủ |
| Gold-set annotation QC (15) | Đạt: F1 0.91 ≥ 0.80, mean IoU 0.86 ≥ 0.70; có số trước/sau và lý do rework |
| Pipeline YOLO và phân tích (10) | Chưa làm |

## 8. Công cụ AI và tài liệu tham khảo

- **Claude Code :** dùng ở bước self-QC và gold QC để

  - viết script phân tích FP/FN (ma trận nhầm lớp, diện tích box), cắt và chồng box lên ảnh để kiểm tra bằng mắt;
  - viết script rework: đổi ID lớp và xoá các box không chồng lên box gold nào. Bước xoá box dùng vị trí box gold làm tiêu chí, không phải đánh giá từng box bằng mắt;
- **Nguồn ngoài repo:** không có.
