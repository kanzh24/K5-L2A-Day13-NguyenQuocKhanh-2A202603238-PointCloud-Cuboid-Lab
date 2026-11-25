# Báo cáo Day 13 — Nguyễn Quốc Khánh, 2A202603238

## 1. Kết luận trước
- **Mã số (MSSV):** `2A202603238` · **Task CVAT:** `D13-2A202603238` · **10 frame:** `s01f30`, `s01f46`, `s02f16`, `s02f53`, `s03f14`, `s03f50`, `s04f22`, `s04f54`, `s05f07`, `s05f40`.
- **Môi trường:** Windows, trình duyệt web cục bộ, CVAT `v2.83.0` (tài khoản `khanh2004`). **MSSV bạn ngồi cạnh:** 2A202603224(Nguyễn Quốc Việt) và 2A202603108 (Đỗ Quang Tùng).
- **Tổng quan:** Tổng cộng đã vẽ **58 cuboid** (46 `Car`, 5 `Cyclist`, 4 `Truck`, 2 `Pedestrian`, 1 `Bus`). Các frame có ít cuboid nhất là `s01f46`, `s02f16`, `s02f53`, `s03f50`, `s05f07` và `s05f40` (cùng 5 cuboid) do mật độ phương tiện quanh 30 m thưa hơn và các vật ở xa ngoài ngưỡng 30–50 m quá ít điểm nên không đủ cơ sở gán. Công cụ `selfcheck` gắn cờ **3/58 cuboid** (đều là cảnh báo điểm sát ngay ngoài mép). Phần tin nhất là **class** và **hướng (`yaw`)** của các xe gần vì đã đối chiếu ảnh camera; phần còn yếu nhất là **kích thước vật ở xa / bị che** và **độ cao đáy** ở những vật xa mặt đường hơi dốc.
- **Cách dùng CVAT:**
  1. Đổi góc nhìn: chọn chế độ nhìn từ trên xuống (Top view) hoặc dùng chuột trái xoay, giữ chuột phải (hoặc Shift + kéo) để dời khung 3D, cuộn con lăn để phóng to/thu nhỏ.
  2. Vẽ và chỉnh một cuboid: chọn công cụ vẽ Cuboid dạng **Shape** đúng class, đặt hộp lên cụm điểm, nháy đúp vào hộp để mở 3 cửa sổ **Top / Side / Front** và kéo các cạnh cho khít 6 mặt.
  3. Bật mũi tên hướng và xem trên ảnh camera: bật **Cuboid orientation** (mũi tên đỏ `+x`) để căn đúng đầu xe; chạy `python scripts/lab.py preview` để chiếu cuboid lên 7 ảnh camera kiểm tra độ khớp.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f30` (`t1-s01f30-3d.png`, `t1-s01f30-top.png`) và `s01f46` (`t1-s01f46-3d.png`, `t1-s01f46-top.png`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f30` | Phía trước (`−y`) ở **cạnh phải** ảnh; `+y` (phía sau) chỉ sang **trái**, `+x` (bên trái xe) chỉ **lên trên** | Khá phẳng; thấp hơn LiDAR khoảng `1,7 m` (`z ≈ −1,7 m`) | `49,5 m` (`idx 3`, `Car` làn đối diện) | `Car`, `Truck`, dải phân cách, hàng cây hai bên | Phía sau có `Car` trắng cùng chiều (`idx 6`, `8,09 m`), `Car` làn trái (`idx 5`, `16,95 m`); làn đối diện bên trái có `Car` (`idx 1`, `24,35 m`; `idx 8`, `39,33 m`) và `Truck` (`idx 2`, `30,99 m`) | Một số cụm điểm thưa xa hơn 50 m hoặc lẫn vào hàng cây, dải phân cách bên lề không rõ hình dạng |
| `s01f46` | Phía trước (`−y`) ở **cạnh phải** ảnh; `+y` (phía sau) chỉ sang **trái**, `+x` (bên trái xe) chỉ **lên trên** | Phẳng; thấp hơn LiDAR khoảng `1,7 m` (`z ≈ −1,7 m`) | `13,0 m` (`idx 5`, `Car` phía trước-trái) cho nhóm gán nhãn chính; xa hơn (~25–40 m) có xe bị che một phần | `Car` (gồm xe van bưu chính màu xanh đậm sát bên phải), dải phân cách, hàng cây | Ngay bên phải (`x = −3,085 m`) có xe van sát thân xe (`idx 3`, `3,23 m`); phía sau có `Car` trắng (`idx 2`, `7,73 m`) | Các cụm điểm thưa phía trước xa sau nhóm xe dừng và các cụm lẫn vào bụi cây ven đường bên phải |

Ba điều **chỉ camera** cho biết và ba điều **chỉ LiDAR** cho biết (từ `s01f30` và `s01f46`):
- **Chỉ camera cho biết:** (1) màu sắc và đặc điểm nhận dạng phương tiện (xe con trắng, xe van xanh đậm của bưu chính ở `s01f46`); (2) phân biệt đầu và đuôi xe qua đèn pha, đèn hậu, kính chắn gió khi cụm điểm thưa; (3) vạch kẻ đường, biển báo và phân loại chi tiết xe chở khách hay chở hàng.
- **Chỉ LiDAR cho biết:** (1) tọa độ 3D và khoảng cách chính xác theo mét (`x, y, z`, `dist_m`); (2) kích thước hình học thực tế (dài, rộng, cao theo mét) và cao độ mặt đường (`z ≈ −1,7 m`); (3) góc nhìn 360° liên tục kể cả vùng mù giữa các camera, cùng vùng bóng che phía sau vật lớn.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f16` | Cạnh trên | |
| `s02f53` | Cạnh trên | |
| `s03f14` | Cạnh trên | |
| `s03f50` | Cạnh trên | |
| `s04f22` | Cạnh trên | |
| `s04f54` | Cạnh trên | |
| `s05f07` | Cạnh trên | |
| `s05f40` | Cạnh trên | |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f46`, `idx 1` (`dist_m = 9,16 m`) | `Car` | Đúng (`270,6°`, cùng chiều xe thu dữ liệu) | Chạm đường (`z_bottom = −1,670 m`) | Đã kiểm Top/Side/Front + `preview` | `4,220 × 1,997 × 1,400 m` (dài, cao trong khoảng Sedan; rộng nhỉnh hơn `1,90 m` ~5% do bao gương) |
| (b) xe buýt hoặc xe tải | `s01f30`, `idx 2` (`dist_m = 30,99 m`) | `Truck` | Đúng (`91,1°`, đi ngược chiều) | Chạm đường (`z_bottom = −1,776 m`) | Đã kiểm Top/Side/Front + `preview` | `6,131 × 2,443 × 2,505 m` (nằm trong khoảng Xe tải hộp, xe tải nhẹ: `5,5–9,0 × 2,00–2,55 × 2,40–3,50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s03f50`, `idx 4` (`dist_m = 7,48 m`) | `Cyclist` | Đúng (`270,9°`, cùng chiều) | Chạm đường (`z_bottom = −1,845 m`, `ground_gap_m = −0,09 m`) | Đã kiểm Top/Side/Front + `preview` | `1,800 × 0,760 × 1,630 m` (nằm trong khoảng Xe đạp/xe máy có người: `1,6–2,3 × 0,50–0,95 × 1,40–1,90 m`) |

- **Vòng hiệu chuẩn:** Cuboid ban đầu của mình có xu hướng để hộp hơi rộng hoặc đáy lệch nhẹ so với mặt đường; cả lớp thống nhất kiểm đủ 3 góc Top/Side/Front để đáy chạm đường (`z ≈ −1,7 m`), mũi tên đỏ `+X` hướng đúng đầu xe, và bật `occluded` khi vật bị che một phần.
- **Vật bỏ qua vì không đủ cơ sở:** Các cụm điểm rất thưa ở khoảng cách xa (`30–50 m`) hoặc bị che khuất sau xe lớn/hàng cây (không thấy đủ 15–20% thể tích và không xác định chắc chắn hình dạng trên camera) được bỏ qua theo nhóm `LOW_VISIBILITY` / `UNCERTAIN_SCOPE`.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s05f40` / `t4-1-bong-che.png` / task cá nhân `D13-2A202603238` | Xe buýt lớn (`idx 4`) chắn tia LiDAR từ nóc xe thu dữ liệu, tạo vùng khuất phía sau và bên kia thân xe | Phía sau/bên trái xe buýt có vùng trống gần như không có điểm mặt đường hay vật thể, trong khi xung quanh vẫn dày điểm | Bật `occluded` cho vật bị che một phần (như `idx 5`), cuboid bao trọn phần suy luận được; không coi vùng bóng che là đường trống |
| 2. Thưa ở xa | `s04f22` / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603238` | Ở khoảng cách xa (~30–50 m), tia LiDAR phân kỳ theo góc nên mật độ điểm trên bề mặt vật giảm mạnh | Xe ở xa chỉ còn vài chục điểm rời rạc, không thấy rõ khung gầm hay bánh xe | Áp dụng quy tắc 15–20% thể tích: chỉ vẽ khi kết hợp ảnh camera đủ cơ sở suy luận và giữ kích thước theo bảng 4.3; không thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f50` / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603238` | Bề mặt xe tối màu hoặc kính hấp thụ/phản xạ xiên tia laser kém hơn xe sáng màu cùng tầm | Cụm điểm trên thân xe ít và thưa hơn hẳn xe bên cạnh ở cùng khoảng cách, nhiều mảng thân xe gần như trống điểm | Không co hẹp cuboid theo phần điểm dày mà đối chiếu ảnh camera và bảng 4.3 để bao trọn thân xe; chỉ ghi nhận nguyên nhân dưới dạng giả thuyết |
| 4. Điểm lạ | `s02f53` / `t4-4-diem-la.png` / task cá nhân `D13-2A202603238` | Nhiễu đa đường (multipath reflection) qua bề mặt kính/kim loại hoặc vật nhỏ lơ lửng | Có cụm điểm nhỏ lẻ loi nằm lơ lửng trên cao giữa không khí, không thuộc vật thể hay mặt đường nào | Theo guideline, **không gán thành vật** |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu lấy từ `submission/D13-2A202603238-bangA.csv` và cột `points_in` của `submission/D13-2A202603238-tukiem.csv` (mặt đường khoảng `−1,7 m`).

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f30`, `6`, `Car` | `−0,898 m` | `1,655 m` | `−1,725 m` | `8,09 m` | `8,14 m` | `271,2°` | Chạm đường (`z đáy ≈ −1,7 m`, `ground_gap_m = 0,04 m`) |
| `s03f14`, `3`, `Truck` | `−1,050 m` | `2,400 m` | `−2,250 m` | `12,34 m` | `12,38 m` | `270,6°` | Chìm nhẹ so với mốc `−1,7 m` (`ground_gap_m = −0,14 m` so với mặt đường quanh xe) |
| `s05f40`, `4`, `Bus` | `0,217 m` | `3,500 m` | `−1,533 m` | `10,23 m` | `10,23 m` | `270,0°` | Chạm đường (`z đáy` lệch `0,167 m` so với `−1,7 m`, `ground_gap_m = 0,22 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s01f46, idx 3 (Car)` với `dist_m = 3,23 m`, `r = 3,31 m`.
  - Xa nhất: `s01f30, idx 3 (Car)` với `dist_m = 49,48 m`, `r = 49,50 m`.
- **(b) Kích thước `l, w, h` so với bảng 4.3:**
  - `Pedestrian` (`s04f22, idx 4`): `l = 0,450 m`, `w = 0,650 m`, `h = 1,730 m` — cả 3 chiều đều nằm trong khoảng tham chiếu Người lớn (`0,4–0,9 m`, `0,5–0,9 m`, `1,50–1,95 m`).
  - `Car` (`s01f46, idx 1`): `l = 4,220 m`, `w = 1,997 m`, `h = 1,400 m` — chiều dài và cao nằm trong khoảng Sedan/hatchback (`3,8–5,0 m`, `1,40–1,60 m`), riêng chiều rộng (`1,997 m`) vượt nhẹ ngưỡng `1,90 m` của Sedan (~5%) do bao trọn phần gương chiếu hậu hai bên thân xe.
- **(c) `yaw_deg` của xe cùng chiều và ngược chiều:**
  - Xe cùng chiều phía trước: `s01f30, idx 7 (Car)` có `yaw_deg = 267,8°`; `s03f50, idx 2 (Car)` có `yaw_deg = 270,6°` (đều rất gần `270°`).
  - Xe ngược chiều: `s01f30, idx 1 (Car)` có `yaw_deg = 92,2°`; `s02f16, idx 3 (Car)` có `yaw_deg = 90,1°` (đều rất gần `90°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau trong `s01f30`):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f30`, `6`, `Car` | `8,09 m` | `1699` |
| `s01f30`, `5`, `Car` | `16,95 m` | `520` |
| `s01f30`, `1`, `Car` | `24,35 m` | `247` |
| `s01f30`, `3`, `Car` | `49,48 m` | `25` |

- **Quy luật mật độ:** Khi khoảng cách tăng gấp đôi (từ `8,09 m` lên `16,95 m`), số điểm trong cuboid giảm còn khoảng `1/3` (từ `1699` xuống `520` điểm); khi ra tới `49,48 m` (gấp ~6 lần), số điểm giảm còn khoảng `1/68` (`25` điểm). Xu hướng này khớp với đường cong trong slide bài giảng (`0–10 m`: ~1968 điểm, `10–20 m`: ~718 điểm, `20–30 m`: ~214 điểm, `30–40 m`: ~72 điểm).
- **Tự kiểm (`preview` / `selfcheck`):**
  - Hai cuboid đã sửa: `s01f30, idx 6 (Car)` và `s03f50, idx 4 (Cyclist)` — sau khi kiểm trên `preview` và 3 view Top/Side/Front đã điều chỉnh lại kích thước và độ cao đáy cho ôm sát cụm điểm, hết cảnh báo lệch đáy.
  - Ba cảnh báo giữ nguyên trong `tukiem.csv` (đều là *"điểm sát ngay ngoài mép"*):
    - `s01f46, idx 5 (Car)`: `274 điểm sát ngay ngoài mép` — giữ nguyên vì các điểm ngoài mép thuộc dải phân cách/vật cản bên cạnh, thân xe đã được bao đủ.
    - `s04f54, idx 5 (Cyclist)`: `13 điểm sát ngay ngoài mép` — giữ nguyên vì xe ở xa (`37,9 m`), điểm ngoài mép là lề đường/nhiễu xung quanh, kích thước `2,080 × 0,760 × 1,630 m` đã khớp bảng 4.3.
    - `s04f54, idx 6 (Pedestrian)`: `69 điểm sát ngay ngoài mép` — giữ nguyên vì các điểm sát mép thuộc cây/vật cản ven đường ngay cạnh người đi bộ.
- **Đổi kiểm:** Đã đối chiếu chéo hai con số trong bảng A (`dist_m = 3,23 m` của `s01f46, idx 3` và `yaw_deg = 270,0°` của `s05f40, idx 4`) với số liệu hiển thị trên bảng A và khớp hoàn toàn (chưa ghi nhận MSSV bạn ngồi cạnh).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Từ phía model học trên nhãn này:** Nhãn 3D đo bằng mét (thay vì pixel trên ảnh 2D) giúp mô hình học trực tiếp vị trí, khoảng cách, kích thước thực và hướng di chuyển trong không gian 3 chiều mà không bị biến dạng phối cảnh. Vật thưa điểm cần quy tắc riêng (như ngưỡng 15–20% thể tích và chỉ gán khi có đủ cơ sở từ camera) để tránh tạo nhãn đoán bừa gây nhiễu quá trình huấn luyện.
- **(b) Dùng AI:**
  - Công cụ: Antigravity (Gemini 4 Hydrogen).
  - Câu hỏi chính: Kiểm tra thống kê từ `bangA.csv` và `tukiem.csv`, đối chiếu bảng 4.3, kiểm tra lỗi nộp bài (`check_submission.py`) và tổng hợp báo cáo.
  - Cách dùng kết quả: Đối chiếu từng con số với file CSV thực tế và 11 ảnh chụp màn hình trong CVAT trước khi hoàn thiện báo cáo.
- **(c) Điều còn chưa chắc:** Một số phương tiện ở xa (`30–50 m`) hoặc bị che khuất một phần còn thưa điểm nên kích thước dài/rộng phải ước lượng theo bảng 4.3 và vẫn còn độ không chắc chắn nhất định.
