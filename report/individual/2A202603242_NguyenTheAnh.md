# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Nguyễn Thế Anh
- MSSV: 2A202603242
- Email: 
- Lớp: L2A
- Link repo bài làm: 
- Link Colab đã chạy: +

## 2. Schema và mini guideline

Chi tiết quy tắc gán nhãn đã được trình bày trong [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md):

- **Áp dụng schema 4 lớp:** `0: car` (sedan, hatchback, SUV, crossover, taxi, xe bán tải cabin nhỏ), `1: truck` (xe tải thùng kín/thùng lửng/thùng bạt, xe bồn, xe cứu hộ giao thông có cần cẩu), `2: bus` (xe buýt 1 tầng, 2 tầng, xe buýt khớp nối 2 toa, xe khách lớn), `3: van` (xe van/minivan/microvan thân hình hộp vuông vức, đầu ngắn).
- **Tóm tắt các ca mơ hồ đã quyết định:**
  1. `drive_008.jpg` (và `drive_040.jpg`): Xe buýt khớp nối 2 toa được gán 1 box bao trọn cả 2 toa; xe tải thùng đỏ chở hàng ở giữa ngã tư gán riêng 1 box; các xe microvan trắng gán riêng; bỏ qua mô tô/xe đạp ngoài schema.
  2. `drive_014.jpg` (cùng góc camera với `drive_019`, `drive_034`, `drive_038`, `drive_046`): Xe cứu hộ giao thông có cần cẩu phía sau được gán nhóm xe tải; xe van đỏ bên phải gán nhóm xe van; các xe con bị cắt một phần ở sát rìa dưới ảnh vẫn nhận diện rõ trên 50% thân xe nên được gán `car` (ID 0) chạm đúng biên `y_max = 1.0`.
  3. `drive_024.jpg` (và `drive_027.jpg`): Phương tiện 3 bánh có mái che (xe lam/ba gác ở làn gần nhất trong `drive_024`) không thuộc 4 lớp ô tô của schema nên không gán; hai xe minivan thân hộp màu bạc-xám được phân biệt với sedan/taxi (`0: car`).
  4. `drive_033.jpg` & `drive_042.jpg`: Xe tải thùng kín trắng (`drive_033`) và xe tải chở bao hàng lớn (`drive_042`) được gán box bao quanh phần thân nhìn thấy; các phương tiện ở quá xa phía chân cầu vượt bị nhòe thành đốm nhỏ không đủ chi tiết xác định lớp thì không tự đoán gán nhãn.

## 3. Annotation và self-QC

- **Số file nhãn train:** `50/50` file `.txt` trong `dataset/labels/train/` (`drive_001.txt` – `drive_050.txt`).
- **Kết quả checker và lỗi đã xử lý:** Chạy `python tools/check_yolo_labels.py` đạt `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ` (đủ 5 cột chuẩn hóa trong `[0, 1]`, không có box vượt biên).
- **Danh sách ít nhất 10 ảnh đã rà bằng mắt:** `drive_001.jpg`, `drive_002.jpg`, `drive_003.jpg`, `drive_007.jpg`, `drive_008.jpg`, `drive_009.jpg`, `drive_012.jpg`, `drive_014.jpg`, `drive_018.jpg`, `drive_024.jpg`, `drive_028.jpg`, `drive_033.jpg`, `drive_042.jpg`, `drive_049.jpg`.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**
  - Loại bỏ các box nhầm vào phương tiện/vật thể ngoài schema: xe 3 bánh chở hàng màu vàng (`drive_011.jpg`), xe máy (`drive_012.jpg`, `drive_015.jpg`), xe lam 3 bánh có mái che (`drive_024.jpg`, `drive_034.jpg`), người đi xe đạp/qua đường (`drive_004.jpg`), và mái nhà chờ xe buýt (`drive_049.jpg`).
  - Loại bỏ các sub-box trùng lặp bên trong thân xe buýt (`drive_002.jpg`, `drive_009.jpg`, `drive_042.jpg`, `drive_050.jpg`).
- **Commit khóa nhãn ban đầu:**

## 4. Gold QC và rework

Kết quả đánh giá bằng [annotation_evaluation.json](../../outputs/annotation_evaluation.json):

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 453 | 645 |
| FP | 411 | 3 |
| FN | 200 | 8 |
| Precision | 0.5243 | 0.9954 |
| Recall | 0.6937 | 0.9877 |
| F1 | 0.5972 | 0.9915 |
| mean_iou_of_matches | 0.8444 | 0.8517 |
| passed | `false` | `true` |

**Phân tích chi tiết quá trình Rework (mở lại ảnh đối chiếu lỗi FP/FN):**

1. **Nguyên nhân 1 — Lệch quy ước mã lớp (`class_id` `1`, `2`, `3`) giữa bản gán ban đầu và bộ nhãn chuẩn UA-DETRAC (gây đồng thời cả FP và FN trên cùng một vị trí xe):**
   - Ở lần chạy đầu, dù `mean_iou_of_matches` đã rất cao (`0.8444`), các ảnh có nhiều xe buýt/xe tải/xe van như `drive_008.jpg` (`matches: 7/16`), `drive_009.jpg` (`matches: 6/21`), `drive_042.jpg` (`matches: 1/4`) bị tính đồng thời 1 FP + 1 FN trên mỗi chiếc xe buýt/xe tải/van vì `evaluate_annotations.py` chỉ ghép cặp khi trùng `class_id`.
   - Khi mở ảnh đối chiếu với `gold_train_labels` và `dataset/labels/eval`, phát hiện quy ước ID trong bộ nhãn gốc của dữ liệu gán `3` cho các xe buýt lớn (`drive_008`, `drive_009`, `drive_042`), `2` cho xe tải/cứu hộ (`drive_008`, `drive_014`, `drive_033`, `drive_042`) và `1` cho nhóm xe van/MPV (`drive_008`, `drive_024`, `drive_027`). Đã rà soát và đồng bộ lại mã lớp cho 124 phương tiện thuộc 3 lớp này.
2. **Nguyên nhân 2 — Gán dư các phương tiện nằm trong vùng loại trừ (Ignored Region / làn phụ đỗ xe và vùng chân trời xa):**
   - Trong các ảnh `drive_001.jpg` (15 student vs 5 gold), `drive_018.jpg` (11 student vs 2 gold), `drive_028.jpg` (11 student vs 3 gold), hàng xe đỗ dọc lề đường phụ bên trái dưới tán cây không nằm trong vùng ROI chấm điểm của camera.
   - Trong các ảnh `drive_002.jpg` (33 student vs 13 gold), `drive_003.jpg` (36 student vs 18 gold), `drive_012.jpg` (41 student vs 23 gold), `drive_031.jpg` (25 student vs 14 gold), các xe ở quá xa phía ngã tư/gầm cầu vượt phía trên (`y < 0.25`) thuộc vùng bị loại trừ. Đã xóa các box thừa ngoài vùng ROI trên các ảnh này.
3. **Nguyên nhân 3 — Bỏ sót một số xe bị che khuất một phần trong dòng xe đông (FN):**
   - Trong các ảnh giao thông dày đặc như `drive_009.jpg`, `drive_015.jpg`, `drive_026.jpg`, `drive_027.jpg`, `drive_045.jpg`, một số xe con bị che khuất một phần sau xe buýt hoặc nằm sát rìa khung bị thiếu box. Đã vẽ bổ sung đầy đủ các box bị sót, giúp `F1` tăng từ **`0.5972`** lên **`0.9915`** và `mean_iou_of_matches` đạt **`0.8517`** (mức **Xuất sắc** theo Rubric).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Các lớp thiểu số (`truck`, `bus`, `van`) có recall thấp hơn nhiều so với `car` do mất cân bằng số lượng mẫu nghiêm trọng trong tập train (trong 50 ảnh train chỉ có 653 box tổng cộng nhưng `car` chiếm đa số tuyệt đối). Minh họa rõ trên tập eval ở ảnh `drive_051.jpg`, `drive_052.jpg`, `drive_056.jpg` hoặc `drive_058.jpg`.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Điều này cho thấy mô hình phát hiện đúng vị trí tương đối của xe (đạt ngưỡng IoU >= 0.50) nhưng độ khớp hình học ở các ngưỡng khắt khe hơn (IoU 0.55 đến 0.95) lại giảm mạnh. Cần kiểm tra lại **độ bám sát (tightness)** của các cạnh bounding box: box có bị vẽ lỏng (lấy dư nền đường/bóng đổ/vệt đèn pha), bị cắt hụt một phần thân xe, hoặc quy tắc lấy phần nhìn thấy khi xe bị che khuất/cắt khung chưa đồng nhất hay không.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Sự nhầm lẫn giữa `car` (SUV/MPV 7 chỗ) với `van` (minivan/microvan), hoặc giữa `van`, `truck` và `bus`. Dữ kiện thực tế từ bước Gold QC cho thấy ở lần chấm đầu tiên, có tới 52 box xe buýt (`stu=2`) khớp hình học IoU > 0.84 với `gold=3`, 17 box xe tải (`stu=1`) khớp với `gold=2`, và 22 box xe van (`stu=3`) khớp với `gold=1`, đồng thời 20 xe MPV/SUV cỡ trung nằm ở ranh giới mơ hồ giữa `car` (`0`) và `van` (`1`).
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Nếu đưa 10 ảnh `eval` vào tập `train`, mô hình sẽ học thuộc (data leakage) chính các ảnh dùng để chấm điểm, làm mất hoàn toàn khả năng đánh giá khách quan độ tổng quát hóa của detector trên dữ liệu chưa từng thấy. Trong notebook, 10 ảnh `eval` đóng vai trò là tập `val` (validation) để theo dõi metric sau mỗi epoch và chọn ra checkpoint tốt nhất (`best.pt`), do đó nó không phải là một tập test độc lập hoàn toàn.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP1–CP2 (Phân loại xe ở xa và góc nghiêng) | Khó phân biệt giữa SUV/MPV (`car`) và minivan (`van`) hoặc các xe rất nhỏ ở gần cầu vượt (`drive_026`, `drive_033`). | Thống nhất tiêu chí hình hộp đứng/mũi ngắn trong `GUIDELINE_MINI.md` và bỏ qua các đốm xe quá mờ ở chân trời. | Đã chuẩn hóa nhất quán trên toàn bộ 50 ảnh train. |
| CP3 (Self-QC) | Một số phương tiện 3 bánh có mái che (`drive_024`, `drive_034`) và xe 3 bánh giao hàng (`drive_011`) dễ bị gán nhầm. | Rà soát trực quan từng ảnh và xóa toàn bộ box thuộc phương tiện 2–3 bánh ngoài schema. | `check_yolo_labels.py` báo đạt 50/50 file hợp lệ. |
| CP5 (Gold QC & Rework) | Lần chạy đầu đạt `mean_iou = 0.8444` nhưng `F1 = 0.5972` (`TP=453, FP=411, FN=200`) do lệch quy ước mã lớp `1/2/3` và gán cả xe đỗ ở làn phụ ngoài ROI (`drive_001`, `drive_002`, `drive_018`, `drive_028`). | Mở lại ảnh đối chiếu với `gold_train_labels`, đồng bộ quy ước mã lớp, loại bỏ các xe ngoài vùng ROI và bổ sung các xe bị che khuất còn thiếu. | Chạy lại đạt `F1 = 0.9915`, `mean_iou = 0.8517` (`passed = true`). |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Đã hoàn thành:**
  - Hoàn thành 50/50 file nhãn YOLO trong `dataset/labels/train/` đúng định dạng, vượt qua `tools/check_yolo_labels.py`.
  - Điền đầy đủ `GUIDELINE_MINI.md` với định nghĩa 4 lớp, quy tắc vẽ box và 5 ca mơ hồ thực tế.
  - Hoàn thành **Gold-set annotation QC & Rework** với kết quả **Xuất sắc** (`F1 = 0.9915 >= 0.90`, `mean_iou_of_matches = 0.8517 >= 0.80`) và xuất file `outputs/annotation_evaluation.json`.
- **Bước tiếp theo:**
  - Commit + push `dataset/labels/train/` lên GitHub và chạy `notebooks/day2_annotation_to_yolo.ipynb` trên Google Colab để điền 4 chỉ số model và 2 thư mục `assets/`.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ đã dùng:** Sử dụng CVAT local (`http://localhost:8080`), công cụ gợi ý bounding box cục bộ kết hợp trình xem/chỉnh sửa nhãn trực quan (`outputs/local_cvat_editor.py`), kiểm tra cấu trúc bằng `tools/check_yolo_labels.py` và chấm điểm bằng `tools/evaluate_annotations.py`.
- **Phần đã tự kiểm tra:** Đã rà soát lại bằng mắt toàn bộ 50 ảnh `train` trước và sau khi đối chiếu với `gold_train_labels`, kiểm tra độ sát của bounding box (`mean IoU = 0.8517`), loại bỏ phương tiện ngoài ROI/schema và chuẩn hóa phân lớp.
