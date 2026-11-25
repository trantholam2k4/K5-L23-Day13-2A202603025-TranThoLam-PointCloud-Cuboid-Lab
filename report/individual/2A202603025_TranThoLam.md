# Báo cáo Day 13 Lab — Đọc point cloud VF và vẽ cuboid 3D

- **Họ và tên:** Trần Thọ Lâm
- **MSSV:** 2A202603025
- **Lớp / Nhóm:** K5-L2
- **Task CVAT:** `D13-2A202603025`

---

## §1. Tổng quan kết quả gán nhãn

- **Tổng số frame thực hiện:** 10 frame (`s01f01`, `s01f20`, `s02f26`, `s02f54`, `s03f14`, `s03f34`, `s04f12`, `s04f28`, `s05f05`, `s05f32`).
- **Tổng số cuboid đã lưu và đồng bộ (`bangA.csv`):** **53 cuboid**, phân bố theo class:
  - `Car`: 47 cuboid
  - `Truck`: 1 cuboid
  - `Cyclist`: 2 cuboid
  - `Pedestrian`: 3 cuboid
  - `Bus`: 0 cuboid
- **Phân bố số lượng theo từng frame:** `s01f01` (9), `s01f20` (5), `s02f26` (5), `s02f54` (5), `s03f14` (5), `s03f34` (5), `s04f12` (8), `s04f28` (5), `s05f05` (2), `s05f32` (4).
- **Frame có ít cuboid nhất:** `s05f05` với **2 cuboid**.
- **Kết quả phiếu tự kiểm (`tukiem.csv`):** **43 / 53 cuboid** có cờ cảnh báo gợi ý (chủ yếu do giữ chiều cao hộp mặc định `2.385 m` ở class `Car` và một số hộp có độ cao đáy lệch mặt đường ước tính).

---

## §2. Đọc frame năm bước

### 2.1. Hai frame đọc kỹ

#### Frame 1: `s01f01`
- **Hệ trục và hướng quan sát (ảnh `t1-s01f01-3d.png`, `t1-s01f01-top.png`):**
  - Gốc tọa độ đặt tại cảm biến LiDAR trên nóc xe thu dữ liệu.
  - Trục **$-y$** hướng về **phía trước** đầu xe (tương ứng góc nhìn chính diện của `cam01`), trục **$+y$** hướng về **phía sau** đuôi xe (`cam06`).
  - Trục **$+x$** hướng sang **bên trái** xe thu dữ liệu (`cam05`), trục **$-x$** hướng sang **bên phải**, và trục **$+z$** hướng thẳng đứng **lên trên**.
- **Các vật thể và cụm điểm nhận diện được:**
  - Khu vực ngã tư rộng; phía trước ($-y$) và bên trái ($+x$) có nhiều xe con (`Car`) di chuyển cùng chiều (như cuboid `#1` tại `x = 3.211 m, y = -2.924 m`, khoảng cách `4.34 m`) và ngược chiều (`#5` tại `x = 19.637 m, y = -3.456 m`).
  - Phía sau ($+y$) có các xe con chạy phía sau như `#2` (`y = 7.782 m`), `#3` (`y = 14.013 m`), `#6` (`y = 16.910 m`).
  - Phía bên phải ($-x$) gần khu vực lề/vỉa hè có cụm điểm của người đi xe hai bánh (`#4 Cyclist` tại `x = -10.589 m, y = 6.038 m`) và người đi bộ (`#8`, `#9 Pedestrian` quanh `x ≈ -14 m, y ≈ -8.2 m`).
- **Cụm điểm chưa đọc rõ / không suy đoán:**
  - Các cụm điểm ở rất xa ngoài phạm vi `50 m` hoặc bị che khuất một phần phía sau hàng xe và dải phân cách nên mật độ tia thưa, không đủ cơ sở xác định hình dạng đầy đủ nên bỏ qua theo guideline.

#### Frame 2: `s01f20`
- **Hệ trục và hướng quan sát (ảnh `t1-s01f20-3d.png`, `t1-s01f20-top.png`):**
  - Phía trước xe thu dữ liệu là trục **$-y$** (`cam01`), phía sau là trục **$+y$** (`cam06`), bên trái là **$+x$** (`cam05`) và bên phải là **$-x$** (`cam07`).
- **Các vật thể và cụm điểm nhận diện được:**
  - Phía trước bên phải ($-x, -y$) có xe con chạy cùng chiều (`#1 Car` tại `x = -4.678 m, y = -5.231 m`, cách `7.02 m`).
  - Phía sau bên phải ($-x, +y$) có các xe con di chuyển cùng hướng như `#2 Car` (`x = -4.144 m, y = 9.775 m`, cách `10.62 m`), `#3 Car` (`x = -7.650 m, y = 10.148 m`, cách `12.71 m`) và `#4 Car` (`x = -4.048 m, y = 20.056 m`, cách `20.46 m`).
  - Dọc hai bên đường có dải cây xanh/hàng rào tạo thành các vệt điểm kéo dài (không phải đối tượng giao thông cần gán nhãn).
- **Cụm điểm chưa đọc rõ / không suy đoán:**
  - 1 cụm ô tô bên trái (cùng các cụm điểm xa/khuất ở rìa tầm quét chưa đủ rõ biên dạng nên không suy đoán thêm).

### 2.2. Tám frame đọc lướt
Trên cả 8 frame còn lại (`s01f20`, `s02f26`, `s02f54`, `s03f14`, `s03f34`, `s04f12`, `s04f28`, `s05f05`), hướng **phía trước** của xe thu dữ liệu đều cố định theo trục **$-y$** của hệ tọa độ LiDAR VF (khớp với hướng camera trước `cam01`), phía sau là **$+y$**, bên trái là **$+x$** và bên phải là **$-x$**.

---

## §3. Ba cuboid có hướng dẫn

| Nhóm mẫu | Frame & `idx` | Class | Tọa độ tâm $(x, y, z)$ (m) | Kích thước $(l, w, h)$ (m) | Hướng `yaw` | Độ cao đáy `z_bottom` & `ground_gap` | Ảnh minh họa |
|---|---|---|---|---|---|---|---|
| 1. Xe con | `s01f01` `#2` | `Car` | `(5.493, 7.782, -1.078)` | `4.780 × 2.212 × 1.708` | `269.4°` (`4.7021 rad`) | `z_bottom = -1.932 m` (`ground_gap = -0.14 m`, sát mặt đường) | `t2-a-car.png` |
| 2. Xe tải / Buýt | `s01f20` `#2` | `Truck` | `(6.387, -15.144, -0.056)` | `10.482 × 3.016 × 2.755` | `87.4°` (`1.5260 rad`) | `z_bottom = -1.434 m` (`ground_gap = +0.54 m`) | `t2-b-bus-truck.png` |
| 3. Xe hai bánh / Người | `s03f34` `#2` | `Cyclist` | `(-6.738, -9.286, -0.886)` | `1.764 × 1.234 × 1.879` | `269.4°` (`4.7021 rad`) | `z_bottom = -1.825 m` (`ground_gap = -0.05 m`, chạm mặt đường) | `t2-c-ped-cyclist.png` |

- **Ghi chú đối tượng bỏ qua:** Các cụm điểm nằm xa quá `50 m` hoặc chỉ có vài điểm rời rạc sau vật cản, không đủ 15–20% thể tích vật thể để xác định kích thước và hướng nên được bỏ qua theo nguyên tắc không suy đoán.

---

## §4. Bốn hiện tượng chất lượng dữ liệu

1. **Bóng che (Occlusion) — Ảnh `t4-1-bong-che.png`:**
   - **Vị trí / đối tượng:** Cụm phương tiện bị xe khác che khuất một phần trên CVAT trong các frame giao thông đông đúc (như khu vực có xe lớn/nhiều xe ở `s01f20` / `s02f54`).
   - **Nguyên nhân (giả thuyết):** Tia laser của LiDAR phát ra từ một điểm đặt trên nóc xe thu dữ liệu và truyền theo đường thẳng, nên khi gặp vật thể lớn ở gần (xe tải/ô tô phía trước) thì vùng không gian phía sau bị chắn tia, tạo ra vùng khuyết điểm.
   - **Hệ quả theo guideline:** Nếu vật thể vẫn quan sát được khoảng `15–20%` thể tích hoặc suy luận được biên dạng từ camera thì vẽ cuboid bao trọn phần suy luận được; nếu là track nhiều frame thì bật `occluded = true` (tối đa 6 keyframe), còn nếu khuất hoàn toàn không đủ cơ sở thì bỏ qua.

2. **Điểm thưa ở xa (Sparsity at distance) — Ảnh `t4-2-thua-o-xa.png`:**
   - **Vị trí / đối tượng:** Cuboid `s04f12` `#7` (`Car` ở khoảng cách `41.42 m`, chỉ có `50` điểm) và `s03f34` `#5` (`Car` ở `59.23 m`, chỉ có `54` điểm).
   - **Nguyên nhân (giả thuyết):** LiDAR 40 tia có góc phân kỳ dọc và ngang cố định giữa các tia; khi khoảng cách tăng lên trên `40–50 m`, khoảng cách giữa hai vòng quét liền kề giãn rộng khiến số tia trúng vào thân xe giảm mạnh.
   - **Hệ quả theo guideline:** Cần đối chiếu kỹ với ảnh camera để xác nhận loại xe và dùng bảng tham chiếu kích thước (mục 4.3) làm cơ sở dựng hộp; với vật vượt quá ngưỡng phạm vi (`> 50 m` trong lab này) hoặc quá thưa không đủ `15–20%` thể tích thì không cố đoán (`UNCERTAIN_SCOPE` / `LOW_VISIBILITY`).

3. **Phản xạ thấp hoặc ít điểm bất thường — Ảnh `t4-3-phan-xa-thap.png`:**
   - **Vị trí / đối tượng:** Các phương tiện có bề mặt thân xe tối màu, kính chắn gió hoặc góc nghiêng bất lợi khiến số điểm phản xạ thưa hơn bình thường trên CVAT.
   - **Nguyên nhân (giả thuyết):** Các bề mặt sơn tối màu, kính chắn gió hoặc bề mặt nghiêng góc lớn hấp thụ năng lượng tia laser hoặc phản xạ gương lệch khỏi đầu thu LiDAR, làm giảm số điểm trả về trên thân xe so với các xe cùng khoảng cách.
   - **Hệ quả theo guideline:** Không thu nhỏ cuboid theo phần điểm bị khuyết mà phải mở rộng hộp bao trọn kích thước thực tế ước lượng được từ phần khung xe còn lại và ảnh camera (`LOW_VISIBILITY`).

4. **Điểm lạ / nhiễu (Anomalous / ghost points) — Ảnh `t4-4-diem-la.png`:**
   - **Vị trí / đối tượng:** Khu vực quanh hộp `s03f34` `#4` (`x = -2.596 m, y = -55.765 m, z = -24.747 m`, chỉ có `6` điểm ở tọa độ `z` âm sâu bất thường) hoặc các điểm nhiễu lơ lửng trên mặt đường/cạnh bề mặt bóng.
   - **Nguyên nhân (giả thuyết):** Tia laser phản xạ đa đường (multipath) qua bề mặt kính, kim loại bóng hoặc vũng nước/bụi trong không khí làm sai lệch thời gian bay (ToF), tạo ra các điểm ảo nằm dưới mặt đường hoặc lơ lửng ngoài vật thể thật.
   - **Hệ quả theo guideline:** Tuyệt đối không gán nhãn điểm lạ thành đối tượng giao thông; nếu cuboid lỡ đặt nhầm vào cụm điểm nhiễu/rỗng (như `s01f20 #5` có `0` điểm hoặc `s03f34 #4` ở `z = -24.747 m`) cần kiểm tra lại và loại bỏ.

---

## §5. Phân tích số liệu từ cuboid của bạn

### 5.1. Độ cao đáy cuboid ($z_{\text{đáy}} = z - h/2$)
1. **`s01f01` `#2` (`Car`):**
   $$z_{\text{đáy}} = -1.078 - \frac{1.708}{2} = -1.932\text{ m}$$
   - **Nhận xét:** Giá trị `ground_gap_m = -0.14 m`, đáy hộp nằm sát mặt đường thực tế (chạm đường tốt).
2. **`s01f20` `#2` (`Truck`):**
   $$z_{\text{đáy}} = -0.056 - \frac{2.755}{2} = -1.4335 \approx -1.434\text{ m}$$
   - **Nhận xét:** Giá trị `ground_gap_m = +0.54 m`, đáy hộp cao hơn mặt đường ước tính (hơi lơ lửng do gầm xe tải cao hoặc chưa hạ đáy hộp sát vệt bánh xe).
3. **`s02f54` `#3` (`Car`):**
   $$z_{\text{đáy}} = -1.576 - \frac{2.385}{2} = -2.7685 \approx -2.769\text{ m}$$
   - **Nhận xét:** Giá trị `ground_gap_m = -1.08 m`, đáy hộp thấp hơn mặt đường ước tính (bị chìm dưới mặt đường do tâm $z$ đặt thấp và chiều cao hộp lớn).

### 5.2. Khoảng cách cuboid gần nhất và xa nhất
- **Cuboid gần nhất:** `s05f05` `#2` (`Car`) tại $(x, y) = (-3.508\text{ m}, 0.260\text{ m})$:
  $$r = \sqrt{(-3.508)^2 + 0.260^2} = \sqrt{12.3061 + 0.0676} \approx 3.52\text{ m} \quad (\text{`dist\_m`} = 3.52\text{ m})$$
- **Cuboid xa nhất (toàn bảng A):** `s03f34` `#5` (`Car`) tại $(x, y) = (-7.639\text{ m}, 58.740\text{ m})$:
  $$r = \sqrt{(-7.639)^2 + 58.740^2} = \sqrt{58.3543 + 3450.3876} \approx 59.23\text{ m} \quad (\text{`dist\_m`} = 59.23\text{ m})$$
  *(Nếu chỉ xét trong ngưỡng chấm điểm $\le 50\text{ m}$, cuboid xa nhất là `s04f12` `#8` `Car` tại $(3.084\text{ m}, -47.186\text{ m})$ với $r \approx 47.29\text{ m}$, `dist_m = 47.29 m`.)*

### 5.3. So sánh kích thước với Bảng 4.3
- **Xe con — `s01f01` `#1` (`Car`):**
  - Kích thước đo được: $l = 4.780\text{ m}$, $w = 2.273\text{ m}$, $h = 2.385\text{ m}$.
  - So với Bảng 4.3 (Sedan/hatchback: dài `3,8–5,0 m`, rộng `1,70–1,90 m`, cao `1,40–1,60 m`; SUV/minivan: dài `4,4–5,3 m`, rộng `1,80–2,10 m`, cao `1,65–2,10 m`): chiều dài (`4.780 m`) nằm đúng trong khoảng chuẩn, còn chiều rộng (`2.273 m`) và chiều cao (`2.385 m`) nằm ngoài khoảng vì giữ kích thước hộp mẫu mặc định chưa thu hẹp sát nóc và hai bên hông xe.
- **Xe hai bánh — `s03f34` `#2` (`Cyclist`):**
  - Kích thước đo được: $l = 1.764\text{ m}$, $w = 1.234\text{ m}$, $h = 1.879\text{ m}$.
  - So với Bảng 4.3 (Xe đạp có người: dài `1,6–1,9 m`, rộng `0,50–0,70 m`, cao `1,60–1,90 m`; Xe máy có người: dài `1,8–2,3 m`, rộng `0,70–0,95 m`, cao `1,40–1,80 m`): chiều dài (`1.764 m`) và chiều cao (`1.879 m`) nằm trong khoảng hợp lý, riêng chiều rộng (`1.234 m`) hơi vượt ngưỡng trên do bao dư khoảng trống hai bên hoặc góc xoay hộp chưa ôm sát thân xe.

### 5.4. Mật độ điểm theo khoảng cách
| Frame & `idx` | Class | Khoảng cách `dist_m` (m) | Số điểm trong hộp (`points_in`) |
|---|---|---:|---:|
| `s03f14` `#5` | `Car` | `5.12` | `5800` |
| `s01f01` `#2` | `Car` | `9.53` | `1712` |
| `s02f26` `#3` | `Car` | `21.04` | `372` |
| `s04f12` `#7` | `Car` | `41.42` | `50` |

- **Quy luật:** Khi khoảng cách từ LiDAR tới vật thể tăng lên, mật độ điểm trên vật thể giảm mạnh theo cấp số (từ hàng nghìn điểm ở khoảng cách dưới `10 m` xuống chỉ còn vài chục điểm ở trên `40 m`) do các tia quét của LiDAR phân kỳ rộng hơn khi đi xa.

### 5.5. Góc `yaw` của xe cùng chiều và ngược chiều
- **Xe chạy cùng chiều với xe thu dữ liệu (hướng về phía trước $-y$ của LiDAR VF):** ví dụ `s01f01` `#1` (`Car`) có `yaw_deg = 269.4°` (`4.7021 rad`) và `s05f32` `#2` (`Car`) có `yaw_deg = 272.1°` (`4.7483 rad`), xấp xỉ `270°` (trùng hướng trục $-y$).
- **Xe chạy ngược chiều (hướng về phía sau $+y$ của LiDAR VF):** ví dụ `s01f20` `#3` (`Car`) có `yaw_deg = 91.1°` (`1.5891 rad`) và `s04f28` `#5` (`Car`) có `yaw_deg = 90.3°` (`1.5756 rad`), xấp xỉ `90°` (ngược `180°` so với xe cùng chiều).

### 5.6. Đổi kiểm với bạn cùng lớp
- Làm bài cá nhân, không có bạn đổi kiểm trực tiếp; tự đối chiếu chéo số liệu giữa `bangA.csv` và `tukiem.csv` (ví dụ `s05f05 #2` có `dist_m = 3.52 m` và `s03f34 #5` có `dist_m = 59.23 m`).

---

## §6. Tổng kết & Kỷ luật dữ liệu

- **Hai câu "để làm gì" từ phía mô hình học máy (3D Object Detection):**
  1. Dữ liệu cuboid 3D chuẩn hóa 7 tham số $(x, y, z, l, w, h, \theta)$ giúp mô hình nhận diện 3D học được cách ước lượng chính xác vị trí không gian, kích thước vật lý và hướng di chuyển (heading) của các phương tiện xung quanh từ đám mây điểm LiDAR thưa thớt.
  2. Việc tuân thủ quy tắc không đoán mò khi bị che khuất hoặc quá xa và gắn cờ thuộc tính rõ ràng giúp mô hình tránh học phải nhãn nhiễu (false positives), từ đó giảm các dự đoán ảo nguy hiểm khi xe tự hành lập kế hoạch quỹ đạo.
- **Các cuboid còn chưa chắc chắn (báo cáo trung thực):**
  - `s01f20` `#5` (`Car`, `dist_m = 27.84 m`): hộp có `0` điểm bên trong (`z = 1.351 m`), có khả năng đặt nhầm vào vùng trống hoặc cụm điểm đã dịch chuyển.
  - `s03f34` `#4` (`Car`, `dist_m = 55.83 m`): nằm ngoài phạm vi `50 m`, tọa độ `z = -24.747 m` và kích thước `5.026 × 5.026 × 3.025 m` bất thường (nghi do bắt nhầm cụm điểm nhiễu dưới mặt phẳng đường).
  - `s02f54` `#3` và `#4` (`Car`, quanh `32.6–32.9 m`): hai cuboid đang giao nhau trên cùng một vùng cụm điểm ở xa.
  - `s01f01` `#8` và `#9` (`Pedestrian`, quanh `16.1–16.6 m`): hai người đi bộ đứng sát nhau nên biên hai cuboid bị chồng lấn nhẹ.