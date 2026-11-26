# Báo cáo Day 13 — Trinh Xuan Toan, 2A202603121

## 1. Kết luận trước
- Mã số học viên: 2A202603121
- Task: D13-2A202603121
- 10 frame được giao: s01f20, s01f48, s02f13, s02f35, s03f17, s03f43, s04f10, s04f59, s05f09, s05f25
- Môi trường: Windows 11, trình duyệt Google Chrome, CVAT v2.83.0.
- MSSV bạn ngồi cạnh đổi kiểm: 2A202603120.
- Tổng số cuboid đã gán nhãn: 135 cuboid. Frame có số lượng cuboid ít nhất là s04f59 (9 cuboid) do đoạn đường thông thoáng, mật độ phương tiện thưa; frame nhiều nhất là s05f25 (26 cuboid) do ngã tư đông đúc nhiều xe và người đi bộ.
- Kết quả tự kiểm (selfcheck): 30/135 cuboid có cảnh báo (~22%), chủ yếu là các vật thể ở xa >50 m bị thưa điểm và người đi bộ trên vỉa hè có cao độ đáy cao hơn lòng đường.
- Đánh giá bản thân: Tin tưởng nhất ở hướng xe (yaw) và độ khớp ở khung chiếu Top/Side/Front cho các vật trong bán kính 30 m; phần yếu nhất là ước lượng chiều dài thùng xe tải ở cự ly xa >45 m khi mật độ điểm bị suy giảm.

Cách dùng CVAT:
- Đổi góc nhìn: Kéo chuột trái để xoay không gian 3D, kéo chuột phải để tịnh tiến/dời khung nhìn, lăn con lăn chuột để phóng to/thu nhỏ; nháy đúp chỗ trống để trở về góc nhìn mặc định.
- Vẽ và chỉnh cuboid: Chọn công cụ Cuboid trên thanh công cụ trái, chọn đúng Label, bấm Shape (không dùng Track), di chuột tới tâm cụm điểm và nháy đúp để đặt; sau đó điều chỉnh tâm, kích thước dài/rộng và góc xoay yaw trên view Top, chỉnh cao độ và đáy chạm mặt đường trên view Side và Front (không xoay ở Side/Front).
- Bật mũi tên hướng và kiểm tra: Vào Appearance bật Cuboid orientation để thấy trục đỏ (+X cuboid) chỉ đúng đầu xe; sau khi lưu (Ctrl+S), chạy lệnh preview để đối chiếu hình chiếu cuboid trên 7 góc ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f20 và s01f48.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f20 | Phía trước ở cạnh trên của ảnh (hướng −y); +x chỉ sang bên trái, +y chỉ về phía sau xe | Mặt đường phẳng, cao độ z khoảng −1,7 m đến −1,8 m (thấp hơn LiDAR nóc xe ~1,7 m) | ~99,56 m (xe tải #10 ở xa phía sau) | Car, Truck, Pedestrian, Cyclist | Có xe tải #4 (x=15,1 m, y=26,7 m), xe con #11 (x=3,1 m, y=16,6 m) phía sau; người đi bộ #6 bên trái | Cụm điểm phản xạ thưa ở góc xa sau bên trái (>70 m) |
| s01f48 | Phía trước ở cạnh trên của ảnh (hướng −y); +x chỉ sang bên trái, +y chỉ về phía sau xe | Mặt đường tương đối bằng phẳng, cao độ z trung bình −1,7 m (thấp hơn LiDAR ~1,7 m) | ~86,37 m (xe tải #6 phía trước xa) | Car, Truck, Pedestrian, Cyclist | Có người đi bộ #1, #7, #9, #11 bên trái; xe hai bánh #10 (y=85,3 m phía sau) | Cụm điểm thưa ở lề đường bên phải bị che khuất một phần |

3 điều chỉ camera cho biết:
1. Màu sơn thực tế của xe và đặc điểm nhận dạng bên ngoài (ví dụ: xe taxi, xe công an, xe cứu thương).
2. Biển số xe, nhãn hiệu/logo và chữ viết trên thân xe/biển hiệu.
3. Đèn tín hiệu giao thông (xanh/đỏ/vàng), vạch kẻ đường và đèn phanh/đèn xi-nhan của phương tiện.

3 điều chỉ LiDAR cho biết:
1. Khoảng cách vật lý chính xác theo mét từ cảm biến tới từng điểm trên bề mặt vật thể (đo trực tiếp bằng thời gian bay ToF).
2. Kích thước 3 chiều thực tế (chiều dài, chiều rộng, chiều cao tính theo mét) mà không phụ thuộc vào tiêu cự ống kính hay góc nhìn.
3. Tọa độ không gian 3D chính xác của vật thể trong vùng 360° xung quanh xe, bất kể điều kiện ánh sáng ngày hay đêm.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f13 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Mật độ xe con đi cùng chiều và ngược chiều khá dày đặc ở hai bên đường |
| s02f35 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Có một cụm điểm nhiễu nằm sâu dưới mặt đất (z ≈ −13,5 m) do phản xạ kính/nước |
| s03f17 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Một xe tải dài ở cự ly gần phía trước tạo bóng che lớn về phía sau |
| s03f43 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Nhiều xe tải và xe con đỗ dọc lề đường phía xa |
| s04f10 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Biển báo bên đường có độ phản xạ cường độ (intensity) rất cao |
| s04f59 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Đường tương đối vắng vẻ, số lượng phương tiện ít nhất trong 10 frame |
| s05f09 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Xe lưu thông đan xen cả phía trước và phía sau |
| s05f25 | Cạnh trên của ảnh nhìn từ trên xuống (hướng −y) | Khu vực nút giao có mật độ phương tiện và người đi bộ đông đúc nhất (26 cuboid) |

## 3. Ba cuboid có hướng dẫn (Bước 3)

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f20, idx 7 | Car | Đúng (yaw=274,1°, vạch đỏ hướng về phía trước theo hướng xe chạy) | Đúng (z đáy=−1,815 m, ground_gap=0,03 m) | Đã kiểm đủ 3 view và ảnh cam01 | Dài 4,651 m, rộng 1,851 m, cao 1,546 m (chuẩn sedan: 3,8–5,0 m; 1,70–1,90 m; 1,40–1,60 m) |
| (b) xe buýt hoặc xe tải | s01f20, idx 16 | Truck | Đúng (yaw=269,7°, vạch đỏ chỉ dọc theo thân xe về phía trước) | Đúng (z đáy=−1,913 m, ground_gap=0,03 m) | Đã kiểm đủ 3 view và ảnh cam01, cam07 | Dài 7,630 m, rộng 2,758 m, cao 3,228 m (chuẩn xe tải nhẹ/hộp: 5,5–9,0 m; 2,00–2,55 m; 2,40–3,50 m) |
| (c) người đi bộ hoặc xe hai bánh | s01f20, idx 6 | Pedestrian | Đúng (yaw=350,6°, hướng theo mặt/chiều di chuyển của người) | Đúng (z đáy=−1,376 m, đứng trên lề đường/vỉa hè) | Đã kiểm đủ 3 view và ảnh cam06 | Dài 0,589 m, rộng 0,640 m, cao 1,748 m (chuẩn người lớn: 0,4–0,9 m; 0,5–0,9 m; 1,50–1,95 m) |

Vòng hiệu chuẩn:
- Khi so sánh với các bạn trong lớp, ban đầu một số bài bị lệch góc xoay yaw 180° do nhầm lẫn giữa đầu và đuôi xe khi quan sát từ khung nhìn Top; một số bài gán thiếu phần cản trước/sau của xe khiến chiều dài cuboid bị ngắn hơn xe thực tế.
- Cả lớp đã thống nhất: Bắt buộc phải đối chiếu ảnh camera (cam01 nhìn trước, cam09 nhìn sau) để xác định chính xác đầu xe trước khi xoay trục đỏ +X; đồng thời căn chỉnh đáy cuboid ở cả Side và Front để đáy nằm đúng mặt phẳng đường, không được để lơ lửng.
- Các vật bỏ qua vì không đủ cơ sở: Tại frame s01f20, một cụm điểm ở góc sau trái cách xa trên 85 m chỉ có 3 điểm phản xạ rời rạc, không thể nhận dạng hình dạng và camera không soi tới, đã bỏ qua theo diện UNCERTAIN_SCOPE và LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f20 / t4-1-bong-che.png / bài cá nhân | Thân xe tải lớn phía trước chắn các tia laser LiDAR quét ra phía sau | Vùng trống hình rẻ quạt ngay phía sau đuôi xe tải hoàn toàn không có điểm LiDAR nào, trong khi hai bên vẫn có điểm mặt đường | Bật thuộc tính occluded = 1 cho các xe/vật thể bị che khuất một phần phía sau; kích thước cuboid bao trọn cả phần bị che suy luận được |
| 2. Thưa ở xa | s01f20 / t4-2-thua-o-xa.png / bài cá nhân (xe hai bánh #1 cách 64,97 m) | Góc phân kỳ chùm tia LiDAR tăng theo khoảng cách, chùm tia bị loãng nên vật ở xa chỉ nhận được rất ít tia quét | Vật thể cách >50 m chỉ nhận được từ 4 đến 6 điểm phản xạ rải rác | Chỉ gán cuboid khi nhận biết được ít nhất 15–20% thể tích và có ảnh camera hỗ trợ; nếu quá ít điểm và không rõ hình dạng thì bỏ qua và phân loại LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s01f20 / t4-3-phan-xa-thap.png / bài cá nhân (xe con #12 tối màu) | Bề mặt sơn sẫm màu và kính xe hấp thụ ánh sáng hồng ngoại bước sóng laser LiDAR khiến tín hiệu phản xạ yếu | Xe ở cự ly trung bình (~18,8 m) nhưng số điểm phản xạ trên thân xe thưa hơn rõ rệt so với xe màu sáng cùng khoảng cách, cường độ intensity thấp | Ghi nhận đây là giả thuyết vật lý, không khẳng định chắc chắn; căn cứ thêm vào ảnh camera để giữ đúng kích thước chuẩn của xe thay vì bóp nhỏ cuboid theo cụm điểm |
| 4. Điểm lạ | s02f35 / t4-4-diem-la.png / bài cá nhân (cụm điểm z ≈ −13,5 m) | Hiện tượng tán xạ/phản xạ nhiều lần (multipath) qua mặt gương hoặc vũng nước trên mặt đường, hoặc xung nhiễu cảm biến | Một cụm điểm lơ lửng nằm sâu dưới mặt đường khoảng 11 m đến 12 m khi xoay nghiêng khung 3D | Tuyệt đối không gán nhãn hay tạo cuboid cho cụm điểm này |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ submission/D13-2A202603121-bangA.csv và submission/D13-2A202603121-tukiem.csv.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f20, idx 7, Car | −1,042 m | 1,546 m | −1,815 m | 3,63 m | 3,77 m | 274,1° | Chạm mặt đường (mặt đường phẳng quanh −1,8 m, khoảng hở ground_gap = 0,03 m) |
| s01f20, idx 16, Truck | −0,299 m | 3,228 m | −1,913 m | 14,55 m | 14,55 m | 269,7° | Chạm mặt đường (khoảng hở ground_gap = 0,03 m chuẩn xác) |
| s01f20, idx 6, Pedestrian | −0,502 m | 1,748 m | −1,376 m | 12,64 m | 12,65 m | 350,6° | Đứng trên vỉa hè/lề đường cao hơn lòng đường xe chạy khoảng 0,35 m |

(a) Khoảng cách:
- Cuboid gần nhất: s03f17, idx 7 (Car), khoảng cách ngang dist_m = 3,17 m, khoảng cách không gian 3D r = 3,26 m.
- Cuboid xa nhất: s01f20, idx 10 (Truck), khoảng cách ngang dist_m = 99,56 m, khoảng cách không gian 3D r = 99,56 m.

(b) Kích thước so với bảng 4.3:
- Người đi bộ s01f20, idx 6 (Pedestrian): dài 0,589 m, rộng 0,640 m, cao 1,748 m. Hoàn toàn nằm gọn trong khoảng kích thước chuẩn của người lớn trong bảng 4.3 (dài 0,4–0,9 m; rộng 0,5–0,9 m; cao 1,50–1,95 m).
- Xe con s01f20, idx 7 (Car): dài 4,651 m, rộng 1,851 m, cao 1,546 m. Nằm hoàn toàn trong khoảng chuẩn xe sedan/hatchback trong bảng 4.3 (dài 3,8–5,0 m; rộng 1,70–1,90 m; cao 1,40–1,60 m). Không có chiều nào bị nằm ngoài khoảng.

(c) Góc yaw:
- Xe cùng chiều phía trước: s01f20, idx 7 (Car) có yaw_deg = 274,1° (rất gần 270°, vạch đỏ ngắn hướng thẳng về phía trước theo chiều −y của hệ VF).
- Xe đi ngược chiều: s01f20, idx 12 (Car) có yaw_deg = 93,1° (rất gần 90°, xe đi ngược về phía sau xe thu dữ liệu).

Mật độ theo khoảng cách (4 cuboid xe con ở các cự ly khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (points_in) |
|---|---|---|
| s01f20, idx 7, Car | 3,63 m | 2738 |
| s01f20, idx 11, Car | 16,93 m | 586 |
| s01f48, idx 8, Car | 20,90 m | 60 |
| s01f20, idx 13, Car | 46,24 m | 27 |

Quy luật mật độ:
- Khi khoảng cách tăng lên gấp đôi thì mật độ điểm rơi trên thân xe giảm khoảng 3 đến 4 lần (suy giảm theo quy luật tỉ lệ nghịch với bình phương khoảng cách).
- Kết quả này hoàn toàn tương đồng với đường cong phân bố trung vị ở slide bài giảng: ở khoảng cách gần dưới 10 m xe con nhận tới ~2000–2700 điểm; từ 10–20 m giảm còn ~600 điểm; ra cự ly 20–30 m chỉ còn khoảng vài chục đến 200 điểm; và ngoài 40 m chỉ còn khoảng 20–30 điểm.

Tự kiểm và điều chỉnh:
- Hai cuboid đã chỉnh sửa sau khi chạy preview và selfcheck:
  1. s05f25, idx 1 (Truck): Ban đầu đáy hộp bị đặt hơi cao so với mặt đường cục bộ quanh đó (cờ lơ lửng), đã hạ độ cao đáy xuống sát mặt đường ở view Front/Side.
  2. s01f20, idx 4 (Truck): Ban đầu thành hộp hơi hẹp bị cờ 39 điểm nằm sát mép ngoài, đã nới nhẹ chiều rộng thêm 0,15 m trên view Top để bao trọn toàn bộ cụm điểm của thùng xe.
- Các cảnh báo giữ nguyên: Các cờ "rất ít điểm" đối với các phương tiện ở xa ngoài 50 m (như s01f48, idx 10) giữ nguyên vì ở cự ly xa mật độ tia quét vật lý tự nhiên bị thưa; cờ của s01f20, idx 6 (Pedestrian) giữ nguyên vì người đi bộ thực tế đang đứng trên vỉa hè cao hơn lòng đường xe chạy.

Đổi kiểm chéo:
- Đã thực hiện đổi kiểm chéo với bạn ngồi cạnh mang mã số 2A202603120.
- Hai con số đã đối chiếu: Khoảng cách ngang dist_m = 3,63 m của cuboid s01f20, idx 7 và cao độ đáy z_bottom = −1,913 m của cuboid s01f20, idx 16. Kết quả trên Bảng A mở trên máy đối chiếu hoàn toàn khớp 100%, không bị lệch số nào.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa từ phía mô hình học máy:
- Nhãn 3D bắt buộc phải đo bằng mét trong không gian vật lý thực thay vì pixel, bởi vì hệ thống điều khiển tự hành cần khoảng cách và kích thước thực tế để lập kế hoạch di chuyển, tính toán khoảng cách phanh an toàn và thời gian tới va chạm (TTC), trong khi kích thước pixel trên ảnh 2D thay đổi phụ thuộc vào tiêu cự ống kính và góc nghiêng của camera.
- Vật thể thưa điểm cần có quy tắc tối thiểu (như thấy được 15–20% thể tích hoặc suy luận được) để tránh việc người gán nhãn đoán mò chủ quan, đưa dữ liệu nhiễu (label noise) vào tập huấn luyện, khiến mô hình học sai kích thước vật thể hoặc sinh ra các cảnh báo phanh giả (false positive).

(b) Cách sử dụng AI:
- Công cụ: Trợ lý AI Antigravity.
- Câu hỏi chính: Hỗ trợ cú pháp PowerShell, cấu hình dữ liệu lab.ini, kiểm tra định dạng và các quy tắc kiểm tra của check_submission.py.
- Cách dùng kết quả: Tham khảo cú pháp dòng lệnh kiểm tra bài nộp và đóng gói ZIP.

(c) Điều còn chưa chắc:
- Đối với các xe tải lớn ở cự ly ngoài 45 m nằm sát ranh giới 50 m, chùm tia quét LiDAR chỉ phản xạ được ở phần đầu xe, còn phần đuôi xe bị thưa điểm nên ranh giới chiều dài phía sau hộp được ước lượng dựa trên kích thước xe tham chiếu và ảnh camera, độ chính xác có thể còn dung sai nhỏ.
