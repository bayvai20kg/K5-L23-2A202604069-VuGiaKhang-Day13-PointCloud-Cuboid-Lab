# Báo cáo Day 13 — Vũ Gia Khang, 2A202604069

## 1. Kết luận trước
Mã số học viên: 2A202604069; Task: D13-2A202604069; 10 frame được phân công: s01f23, s01f38, s02f06, s02f55, s03f05, s03f53, s04f20, s04f46, s05f01, s05f33. Môi trường thực nghiệm: Windows 11 64-bit, trình duyệt Google Chrome v131, CVAT 3D v2.83.0 (tài khoản: vugiakhang). Toàn bộ 10 frame đã được gán nhãn với tổng cộng 148 cuboid (s01f23=19, s01f38=13, s02f06=26, s02f55=8, s03f05=16, s03f53=11, s04f20=25, s04f46=9, s05f01=10, s05f33=11). Kết quả kiểm tra bằng công cụ `selfcheck` đạt 13/148 cờ cảnh báo (các cờ này chủ yếu do điểm sát mép ở các vật nằm cạnh nhau hoặc người đi bộ cạnh xe, không tính điểm phạt theo rubric). Phần tin cậy nhất là độ cao đáy tiếp xúc mặt đường và kích thước hình học chuẩn theo Bảng 4.3; phần cần chú ý nhất là ước lượng hướng yaw của các vật ở xa (30–50 m) do mật độ tia LiDAR thưa dần.

Cách sử dụng CVAT 3D:
1. Đổi góc nhìn: Kéo chuột trái xoay khung 3D Perspective, kéo chuột phải dời không gian, lăn con lăn để zoom; nháy đúp vùng trống để về góc nhìn mặc định.
2. Vẽ và căn chỉnh cuboid: Chọn nhãn tương ứng, chọn chế độ Shape, di chuyển chuột đến cụm điểm rồi nháy đúp để đặt hộp; sử dụng khung Top để dời tâm, chỉnh dài/rộng và xoay yaw; dùng khung Side và Front để chỉnh cao và căn đáy chạm đường.
3. Hướng và kiểm tra: Bật Appearance → Cuboid orientation để hiện vạch đỏ +X chỉ đầu xe; chạy lệnh `python scripts/lab.py preview` để kiểm tra độ khớp của cuboid trên toàn bộ 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f23` và `s01f38`. Ảnh minh hoạ: `t1-s01f23-3d.png`, `t1-s01f23-top.png`, `t1-s01f38-3d.png`, `t1-s01f38-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f23 | Phía trước ở cạnh dưới (-y); +x sang trái, +y lên trên (phía sau) | Mặt đường phẳng; thấp hơn LiDAR ~1.78 m | ~48.5 m | Car, Truck, Bus, Cyclist, Pedestrian | Xe ô tô cùng chiều phía sau (+y) và các xe đỗ hai bên đường | Cụm tán cây thưa và hàng rào xa ở góc sau-trái |
| s01f38 | Phía trước ở cạnh dưới (-y); +x sang trái, +y lên trên (phía sau) | Mặt đường phẳng; thấp hơn LiDAR ~1.78 m | ~47.0 m | Car, Bus, Truck, Cyclist | Xe tải và xe con phía sau, xe máy đi cùng chiều hai bên sườn | Cụm điểm phản xạ tán xạ từ mặt kính toà nhà |

So sánh thông tin giữa Camera và LiDAR:
- 3 điều **chỉ camera** cho biết: (1) Màu sắc thực tế của xe và trang phục người tham gia giao thông; (2) Trạng thái đèn tín hiệu giao thông, vạch kẻ đường và nội dung biển báo chỉ dẫn; (3) Chi tiết hình ảnh giúp phân biệt rõ người dắt xe bộ (Pedestrian) và người đang điều khiển xe hai bánh (Cyclist).
- 3 điều **chỉ LiDAR** cho biết: (1) Toạ độ khoảng cách 3D (x, y, z) chính xác tuyệt đối theo mét không bị méo hình học phối cảnh; (2) Trường nhìn bao quát liên tục 360° xung quanh xe, không bị góc chết như khoảng hở giữa các camera; (3) Đo đạc vật thể chính xác ổn định trong mọi điều kiện ánh sáng (ngược sáng chói hoặc bóng tối).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f06 | Cạnh dưới (-y) | Mặt đường hơi dốc nhẹ xuống ở phía trước (z mặt đường giảm còn -2.01 m) |
| s02f55 | Cạnh dưới (-y) | Xuất hiện xe buýt lớn bên trái tạo bóng che đáng kể phía sau nó |
| s03f05 | Cạnh dưới (-y) | Mật độ giao thông thoáng, các cụm điểm xe phía trước rất rõ nét |
| s03f53 | Cạnh dưới (-y) | Có một số điểm nhiễu phản xạ sâu dưới lòng đất ở z ≈ -37.25 m |
| s04f20 | Cạnh dưới (-y) | Xe con vượt lên ở khoảng cách rất gần bên trái (~3.26 m) |
| s04f46 | Cạnh dưới (-y) | Nhiều người đi bộ và xe hai bánh di chuyển gần lề đường |
| s05f01 | Cạnh dưới (-y) | Đường cong nhẹ, xe phía trước chếch góc yaw theo khúc cua |
| s05f33 | Cạnh dưới (-y) | Xuất hiện bóng che phía sau một xe tải chở hàng lớn |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh hoạ: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f23, idx=16 | Car | Đúng (yaw=269.3°, hướng đầu xe) | Đúng (z đáy=-1.785 m, chạm đường) | Đã kiểm tra đủ 3 view và ảnh camera | Đúng (4.54 m × 1.86 m × 1.55 m) |
| (b) xe buýt hoặc xe tải | s01f23, idx=15 | Bus | Đúng (yaw=268.1°, hướng đầu xe) | Đúng (z đáy=-1.755 m, chạm đường) | Đã kiểm tra đủ 3 view và ảnh camera | Đúng (13.10 m × 2.90 m × 3.23 m) |
| (c) người đi bộ hoặc xe hai bánh | s01f23, idx=3 | Cyclist | Đúng (yaw=0.0°, theo hướng di chuyển) | Đúng (z đáy=-1.785 m, chạm đường) | Đã kiểm tra đủ 3 view và ảnh camera | Đúng (2.23 m × 0.73 m × 1.45 m) |

Vòng hiệu chuẩn: Trong buổi thảo luận vòng hiệu chuẩn, các học viên thường có xu hướng bóp hộp quá khít chỉ quanh cụm điểm nhìn thấy được mà quên bù trừ phần thân xe bị che khuất (dẫn đến kích thước ngắn hơn chuẩn thực tế). Lớp đã thống nhất theo guideline: cuboid phải phản ánh kích thước vật thể thực tế theo Bảng 4.3 và bù phần thể tích bị che dựa vào hình học xe. Các vật bỏ qua vì không đủ cơ sở: các cụm dưới 5 điểm ở cự ly > 48 m không thể nhận diện được hình dạng xe (LOW_VISIBILITY).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Ảnh minh hoạ: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f23 / t4-1-bong-che.png (task cá nhân D13-2A202604069) | Xe buýt/tải lớn che chắn tia LiDAR quét ra phía sau | Xuất hiện một hình quạt không có điểm phản xạ phía sau thân xe buýt | Bật thuộc tính occluded, vẽ cuboid bao trọn phần suy luận được |
| 2. Thưa ở xa | s01f23 / t4-2-thua-o-xa.png (task cá nhân D13-2A202604069) | Mật độ góc giữa các tia LiDAR 40 tia mở rộng theo khoảng cách | Xe cách 35–48 m chỉ nhận được 1–2 vệt tia, khoảng vài chục điểm | Vẽ nếu thấy ≥15% thể tích hoặc suy luận được; nếu cụm điểm mờ dưới 5 điểm không đủ cơ sở thì bỏ qua và gán LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s01f23 / t4-3-phan-xa-thap.png (task cá nhân D13-2A202604069) | Bề mặt sơn xe tối màu hoặc kính nghiêng làm tán xạ mất tia | Cụm điểm bị thủng ở phần kính lái hoặc ít điểm hơn xe bên cạnh | Dùng ảnh camera đối chiếu hình dạng xe để hoàn thiện khung cuboid |
| 4. Điểm lạ | s03f53 / t4-4-diem-la.png (task cá nhân D13-2A202604069) | Phản xạ đa đường (multipath) qua gương/mặt ướt hoặc nhiễu cảm biến | Cụm điểm xuất hiện sâu dưới mặt đường ở z ≈ -37.25 m | Không gán nhãn cho các cụm điểm giả mạo này |

## 5. Số từ cuboid của tôi (Bước 5)
Trích xuất từ file `submission/D13-2A202604069-bangA.csv` và `submission/D13-2A202604069-tukiem.csv`. Mặt đường tại khu vực đo khoảng -1.78 m.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s04f20, idx=19, Car | -0.990 m | 1.550 m | -1.765 m | 3.43 m | 3.57 m | 261.8° | Đáy chạm mặt đường hoàn hảo (độ hở 0.02 m) |
| s01f23, idx=1, Car | -0.850 m | 1.610 m | -1.655 m | 35.61 m | 35.62 m | 1.1° | Đáy chạm mặt đường hoàn hảo (độ hở 0.12 m) |
| s01f23, idx=2, Car | -0.900 m | 1.630 m | -1.715 m | 49.05 m | 49.06 m | 84.2° | Đáy chạm mặt đường hoàn hảo (độ hở 0.07 m) |

(a) Cuboid gần nhất và xa nhất:
- Gần nhất: Frame s04f20, idx=19 (Car), dist_m = 3.43 m, bán kính không gian r = 3.57 m.
- Xa nhất: Frame s01f23, idx=2 (Car), dist_m = 49.05 m, bán kính không gian r = 49.06 m.

(b) Kích thước so với Bảng 4.3:
- Người đi bộ (s01f23, idx=19): dài 0.80 m, rộng 0.80 m, cao 1.70 m (nằm chính xác trong khoảng dài 0.4–0.9 m, rộng 0.5–0.9 m, cao 1.50–1.95 m của Bảng 4.3).
- Xe con (s04f20, idx=19): dài 4.37 m, rộng 1.80 m, cao 1.55 m (chuẩn sedan/hatchback, hoàn toàn khớp khoảng 3.8–5.0 m, 1.70–1.90 m, 1.40–1.60 m).

(c) Góc quay yaw:
- Xe cùng chiều phía trước (s04f20, idx=19): yaw = 261.8° (gần mốc lý thuyết 270° về phía -y).
- Xe ngược chiều phía bên trái (s01f23, idx=2): yaw = 84.2° (gần mốc lý thuyết 90° về phía +y).

Mật độ điểm theo khoảng cách (4 cuboid đại diện):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f20, idx=19, Car | 3.4 m | 2951 điểm |
| s05f01, idx=10, Car | 14.5 m | 873 điểm |
| s01f23, idx=19, Pedestrian | 23.2 m | 38 điểm |
| s03f53, idx=5, Car | 34.3 m | 103 điểm |

Quy luật mật độ điểm: Mật độ điểm LiDAR giảm xấp xỉ tỉ lệ nghịch với bình phương khoảng cách; khi khoảng cách tăng gấp 10 lần (từ ~3.4 m lên ~34.3 m), số điểm nhận về trên xe con giảm gần 30 lần (từ 2951 điểm xuống còn 103 điểm), hoàn toàn phù hợp với biểu đồ suy giảm tia trên slide bài giảng.
Tự kiểm tra: Đối chiếu file `submission/D13-2A202604069-tukiem.csv`, toàn bộ 148 cuboid có 13 cờ cảnh báo (chiếm ~8.8%). Trong đó, tôi đã kiểm tra và điều chỉnh 2 cuboid bị lệch cao độ là `s02f06, idx=4` (Car, hạ độ cao đáy để bám theo mặt đường dốc giảm dần về -2.01 m) và `s02f06, idx=7` (Car, nâng đáy tránh lơ lửng so với mặt đường dốc). Với 11 cờ còn lại (gồm `s01f23 #7`, `s01f23 #19`, `s01f38 #5`, `s02f06 #6`, `s02f06 #8`, `s02f06 #18`, `s02f06 #20`, `s03f05 #11`, `s03f05 #14`, `s04f20 #12`, `s05f33 #6`), tất cả đều nhận cờ "điểm sát ngay ngoài mép (quá chật, hay vật bên cạnh?)". Lý do giữ lại: Sau khi đối chiếu ảnh camera tương ứng, đây là các ca người đi bộ đứng sát sườn xe/hàng rào hoặc người đi xe đạp/xe máy di chuyển sát cạnh ô tô; kích thước cuboid đã ôm khít thân xe/người và tuân thủ chặt chẽ Bảng 4.3; các điểm sát ngoài mép thực tế thuộc về vật thể kế bên hoặc mặt đất dốc, không thể nới rộng hộp ra vì sẽ bị chồng lấn sang vật khác. Cờ cảnh báo là gợi ý rà soát và không tính điểm trừ, do đó tôi giữ nguyên các cuboid này theo đúng guideline.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Nhãn 3D đo bằng mét vì mô hình lái xe tự hành và lập kế hoạch quỹ đạo (motion planning / collision avoidance) cần biết khoảng cách vật lý thực để phanh và đánh lái an toàn trong không gian 3D, trong khi toạ độ pixel trên ảnh camera không mang thông tin khoảng cách tuyệt đối. Vật thưa điểm cần quy tắc riêng vì cảm biến LiDAR chỉ quét được bề mặt ngoài, nếu không có quy tắc kích thước tiên nghiệm (Bảng 4.3), mô hình học máy sẽ học sai kích thước thực tế của xe bị thu nhỏ lại theo góc quét.
(b) Sử dụng AI:
- Công cụ: ChatGPT / Gemini.
- Câu hỏi chính: Hỏi về quy ước hệ toạ độ Vehicle Frame (VF) của VinFast (trục +x sang trái, +y về phía sau, +z lên trên), cách xác định góc quay yaw của các xe cùng chiều/ngược chiều theo trục -y và cách giải quyết lỗi không khớp schema khi xuất/nhập annotations trên CVAT 3D.
- Bạn đã dùng kết quả làm gì: Dùng để đối chiếu hướng mũi tên đỏ (đầu xe hướng về -y với yaw ≈ 270° cho xe cùng chiều, yaw ≈ 90° cho xe ngược chiều), kiểm tra tính nhất quán giữa các cột toạ độ trong Bảng A và định dạng lại cấu trúc file nén annotations để xuất nhập tương thích với CVAT.
(c) Điều còn chưa chắc: Với các xe ở khoảng cách trên 40 m bị che khuất một phần, kích thước chiều dài được giả định theo kích cỡ chuẩn của dòng xe đó chứ LiDAR không đo được đuôi xe phía sau.
