# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Lê Tiến Bình
- MSSV: 2A202603346
- Email: binh04092004@gmail.com
- Lớp: K5 – L2B
- Link repo bài làm: https://github.com/Cily49/K5-L2B-Day02-LeTienBinh-2A202603346-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1NZ-lhza97fJK7Whw60XFYUOuKV2rpZvG

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema áp dụng đúng `data.yaml`: `car=0, truck=1, bus=2, van=3`; lớp mô tả loại xe, không mô tả trạng thái; box bám phần nhìn thấy; xe bị cắt khung thì box chạm rìa ảnh; không gán mô tô, xe đạp, người.

Tóm tắt các ca mơ hồ đã quyết định (đều lấy từ `drive_008`):

1. **Xe buýt nối toa** → 1 box `bus` ôm cả hai toa, vì là một phương tiện.
2. **Xe trắng thân hộp, nóc cao, bị che một phần** → `van`, không phải `car` (nóc phẳng, đuôi thẳng, không tách capo/cốp).
3. **Xe tải ben chở đất nhô khỏi thùng** → 1 box `truck`, tính cả phần hàng nhìn thấy.
4. **Xe đỏ sẫm bị cắt ở rìa phải, chỉ thấy dải hẹp thân + bánh** → `car`, box chạm rìa.

Phần cần giảng viên làm rõ: (a) xe ở xa/rất nhỏ có cần gán không; (b) xe bị che vẽ theo phần nhìn thấy hay theo toàn bộ thân xe; (c) thứ tự ID lớp trong gold (xem mục 4).

## 3. Annotation và self-QC

- Công cụ: CVAT (chạy local), export định dạng YOLO 1.1, thứ tự lớp `car, truck, bus, van` khớp `data.yaml`.
- Số file nhãn train: **50/50**, tổng **1113 box** (car 912, truck 21, bus 104, van 76). Không có ảnh nào rỗng.
- Kết quả checker và lỗi đã xử lý: lần chạy đầu `KHÔNG ĐẠT – Thiếu 50 file nhãn` do bản export CVAT bị giải nén lồng trong `dataset/labels/train/yolo-v1/labels/train/`. Đã chuyển 50 file `.txt` ra đúng `dataset/labels/train/`, bỏ metadata export (`data.yaml`, `train.txt`). Chạy lại: `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt (mở lại trong CVAT): `drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_011`, `drive_012`, `drive_014`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - `drive_008` (rà trong lúc gán nhãn): lần vẽ đầu chỉ có 11 box, bỏ sót xe bị xe buýt che, xe cắt ở rìa phải/góc dưới phải và hàng xe phía xa → bổ sung, lên 28 box. Xe trắng thân hộp sau xe buýt đỏ gán nhầm `car` → đổi thành `van`. Chỉnh lại cạnh box xe buýt nối toa và các box nhỏ ở xa.
  - `drive_005` / `drive_006` (rà sau khi khóa nhãn): cùng một xe trắng sọc đỏ ở giữa ảnh nhưng `drive_005` gán `truck`, `drive_006` gán `van` → **không nhất quán class**. Theo định nghĩa trong guideline (thân hộp một khối, không có thùng hàng tách rời) xe này nên là `van`. Phát hiện sau khi đã khóa nhãn nên giữ nguyên bản khóa, ghi nhận ở đây.
  - Các ảnh còn lại (`drive_003`, `004`, `007`, `009`, `011`, `012`, `014`): đúng định dạng, mỗi xe một box; xe bị che và xe cắt khung đã có box theo phần nhìn thấy. Không phát hiện thêm lỗi class rõ ràng.
- Commit khóa nhãn ban đầu: `a2fc5b8` — *feat: Initial labels (locked) - 50 train labels + mini guideline* (commit và push trước khi chạy đánh giá với gold).

## 4. Gold QC và rework

Lệnh: `python3 tools/evaluate_annotations.py --student-labels dataset/labels/train --gold-labels gold_train_labels --output outputs/annotation_evaluation.json`. Kết quả ở [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 473 | không rework |
| FP | 640 | — |
| FN | 180 | — |
| Precision | 0.425 | — |
| Recall | 0.724 | — |
| F1 | 0.536 | — |
| mean_iou_of_matches | 0.860 | — |
| passed | false | — |

**Phân tích (đã mở lại ảnh, không chỉ dựa vào FP/FN):**

- `mean_iou = 0.86` đạt ngưỡng → box đã ghép được đều sát. Không đạt nằm ở F1, chủ yếu do precision (1113 box của tôi so với 653 box gold).
- **Thứ tự ID lớp trong gold lệch so với schema.** Đối chiếu các box trùng vị trí (IoU ≥ 0.5) nhưng khác lớp, lệch có hệ thống theo một hoán vị:

  | Tôi gán (theo schema) | Gold gán | Số box | Ví dụ `drive_008` |
  |---|---|---:|---|
  | `bus` (2) | 3 (`van` theo schema) | 52 | xe buýt nối toa vàng–trắng |
  | `van` (3) | 1 (`truck` theo schema) | 37 | hai xe van trắng |
  | `truck` (1) | 2 (`bus` theo schema) | 19 | xe tải ben đỏ chở đất |

  Gold có vẻ dùng thứ tự `car, van, truck, bus` thay vì `car, truck, bus, van` của `data.yaml`. Chỉ riêng điểm này tạo ≈ 110 FP và ≈ 110 FN.
- **Gold không gán một số nhóm vật thể mà GUIDE yêu cầu gán**, quan sát khi vẽ chồng gold và nhãn của tôi lên `drive_008`: hàng xe phía xa ở nửa trên ảnh, xe bị cắt ở rìa phải và góc dưới phải, xe buýt đỏ–trắng ở góc trên trái. Ngoài ra gold vẽ box **toàn thân** cho xe bị xe buýt che (box chồng lên thân xe buýt), trong khi GUIDE yêu cầu box theo phần nhìn thấy. Trong 640 FP, khoảng 509 box nằm ở vị trí gold không có box nào; box FP nhỏ hơn hẳn box gold (trung vị chiều cao 33 px so với 57 px).
- Còn 42 FN là bỏ sót hoàn toàn (không có box nào của tôi gần đó); phần lớn FN còn lại có box gần nhưng khác lớp hoặc lệch, trùng với nhóm lệch ID ở trên.

**Quyết định:** không rework. Nhãn của tôi đúng schema trong README/GUIDE; muốn vượt ngưỡng phải đổi ID lớp sai schema và bỏ những xe nhìn rõ, tức là sửa để khớp gold chứ không phải sửa lỗi annotation. Theo RULES/CHECKPOINTS: không sửa gold, báo giảng viên. Chưa kịp báo giảng viên trước hạn nộp. Bằng chứng về việc lệch ID lớp và khác quy ước đã ghi đầy đủ ở trên để giảng viên kiểm tra khi chấm.

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU (`device: 0`). Phiên bản Ultralytics và Torch: 8.4.208 2.14.1+cu130 Tesla T4
- Tham số: giữ nguyên notebook — `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1` (auto), `optimizer=auto`, `lr0=0.01`, `seed=20260909`. Không thay đổi tham số nào.
- Best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không commit `.pt`).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.407 |
| Recall | 0.234 |
| mAP50 | 0.274 |
| mAP50–95 | 0.202 |

Nguồn: [`assets/model_eval_metrics.json`](assets/model_eval_metrics.json). Log train: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv), [`results.png`](assets/yolo26_runs/vehicle_annotation/results.png). Đánh giá: [confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [PR curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png). Ảnh dự đoán 10 ảnh eval: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/).

**Ví dụ cụ thể — [`drive_058`](assets/eval_predictions/visuals/drive_058.jpg):** model phát hiện tốt các xe con ở làn gần (`car` 0.66–0.80) và dự đoán đúng về mặt hình ảnh xe buýt vàng bên trái là `bus` (0.81), xe buýt phía trên là `bus` (0.47). Tuy nhiên nhãn eval gán các xe buýt này class `3` (tên `van` trong `data.yaml`), nên dự đoán `bus` bị tính là sai lớp. Confusion matrix xác nhận: 11 vật thể nhãn eval `van` bị model dự đoán `bus`, 0 vật thể `van` được dự đoán đúng.

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   `van`: 12 vật thể trong eval, model không bắt đúng cái nào (confusion matrix: 11 bị dự đoán `bus`, 1 bị bỏ sót). Minh họa: `drive_058` — 6 box class 3 trong eval đều là xe buýt nhìn rõ, model dự đoán `bus`. Recall thấp của `van` vì thế phản ánh lệch ID lớp giữa nhãn eval và schema mà tôi dùng để train, không phải model không thấy xe. `car` có recall cao nhất (84/94 ở ngưỡng của confusion matrix).

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   Kiểm độ sát của box: model tìm đúng vật thể (IoU ≥ 0.5) nhưng cạnh box lệch nên rớt ở các ngưỡng IoU cao 0.75–0.95. Cần xem box có lỏng, hụt phần đầu/đuôi xe hoặc không nhất quán giữa ảnh không. Ở bài này mAP50 = 0.274 và mAP50–95 = 0.202, khoảng cách không lớn; nguyên nhân chính của điểm thấp là sai lớp/thiếu dữ liệu hơn là box lệch. Riêng xe bị che, quy ước khác nhau (tôi vẽ phần nhìn thấy, nhãn eval vẽ toàn thân, ví dụ xe sau xe buýt trong `drive_053`) sẽ làm IoU thấp ở ngưỡng cao.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ?**
   - Lệch `bus`/`van`/`truck` giữa nhãn của tôi và gold/eval: cùng một hoán vị ở cả gold train (52 bus→3, 37 van→1, 19 truck→2) và nhãn eval (xe buýt = 3 trong `drive_058`, `drive_053`), và confusion matrix (11 `van`→`bus`). Đây là khác biệt quy ước ID lớp chứ không phải mơ hồ hình ảnh.
   - Phân biệt `van`/`car` thật sự phụ thuộc guideline: xe trắng thân hộp trong `drive_008` ban đầu tôi gán `car`, sau đó sửa `van` theo định nghĩa nóc cao/đuôi thẳng. Model chỉ có 76 box `van` và 21 box `truck` để học, so với 912 `car`, nên dễ đẩy các xe mơ hồ về `car` (confusion matrix: 4 truck → car).
   - Model dự đoán rất nhiều box ở vùng "background" (867 car, 1608 van ở ngưỡng confidence thấp của confusion matrix) — một phần do tôi gán xe ở xa/nhỏ mà nhãn eval không gán, nên dự đoán đúng vật thể vẫn bị tính FP.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng?**
   Nếu train trên eval, model đã thấy chính ảnh dùng để chấm nên chỉ số bị thổi phồng, không đo được khả năng tổng quát. Trong notebook, 10 ảnh eval là tập validation: dùng để tính metric mỗi epoch và chọn `best.pt`. Vì đã dùng để chọn checkpoint nên đây không phải test set độc lập; kết quả trên đó vẫn hơi lạc quan.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP2 – gán nhãn `drive_008` | Bỏ sót xe bị che, xe ở rìa, xe ở xa (11 box) | Zoom từng vùng, rà từ trên xuống dưới | 28 box; sửa 1 xe `car` → `van` |
| CP3 – checker | `Thiếu 50 file nhãn` | Phát hiện export CVAT lồng trong `yolo-v1/`, chuyển file ra đúng thư mục | Checker đạt 50/50 |
| CP4 – khóa nhãn | Suýt chạy gold trước khi commit bản ban đầu | Commit + push bản khóa `a2fc5b8` | Có mốc truy lại |
| CP5 – gold QC | F1 0.536 < 0.80 dù IoU 0.86 | Vẽ chồng gold/nhãn, thống kê box cùng vị trí khác lớp | Phát hiện lệch ID lớp + khác quy ước gán xe xa/cắt khung/bị che; báo giảng viên, không sửa gold |
| CP6 – YOLO | Recall thấp (0.234), `van` recall 0 | Đối chiếu confusion matrix và nhãn eval | Phần lớn do nhãn eval dùng cùng quy ước lệch ID; thêm nguyên nhân class imbalance |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):

- **Đã có:** guideline với định nghĩa lớp, quy tắc box và 4 ca mơ hồ; 50 file nhãn đạt checker; commit khóa nhãn trước gold; JSON đánh giá annotation; train/eval trên Colab với đủ metrics, ảnh dự đoán và log; trả lời 4 câu hỏi kèm ảnh cụ thể.
- **Chưa đạt/chưa kiểm được:** chưa qua ngưỡng annotation (F1 0.536). Tôi cho rằng nguyên nhân chính là khác quy ước giữa gold và schema, nhưng chưa có xác nhận của giảng viên nên chưa thể khẳng định. Khi tự rà 10 ảnh, phát hiện 1 lỗi không nhất quán class (`drive_005` truck / `drive_006` van) chưa sửa vì đã khóa nhãn.

## 8. Công cụ AI và tài liệu tham khảo

- **Claude Code (Anthropic)**: dùng để đọc và tóm tắt tài liệu lab; góp ý cách gán nhãn; soạn nháp `GUIDELINE_MINI.md`; chạy `check_yolo_labels.py`; viết script phân tích FP/FN và vẽ chồng gold/nhãn để tìm nguyên nhân F1 thấp; soạn nháp báo cáo này. Việc gán nhãn 50 ảnh do tôi tự làm trên CVAT; các số liệu trong báo cáo lấy từ output thật của script và Colab, tôi đã kiểm tra lại.
- Tài liệu: README, GUIDE, RULES, CHECKPOINTS, RUBRIC, SUBMISSION trong repo; tài liệu Ultralytics YOLO. Không dùng nguồn ngoài để tìm nhãn.
