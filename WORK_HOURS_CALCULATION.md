# Cách Tính Giờ Làm Việc

Tài liệu này mô tả cách tính giờ làm việc mỗi ngày dựa trên dữ liệu chấm công. Hệ thống hỗ trợ 2 loại ca làm việc:

1. **Ca chấm công 2 lần** - Chỉ chấm đến và chấm về
2. **Ca chấm công 4 lần** - Chấm đến, chấm ra nghỉ trưa, chấm vào sau nghỉ, chấm về

---

## 1. Ca Chấm Công 2 Lần

### Điều kiện áp dụng
Ca này được áp dụng khi **không có** cấu hình đầy đủ các window nghỉ trưa (`breakOutWindowStart`, `breakOutWindowEnd`, `breakInWindowStart`, `breakInWindowEnd`).

### Các bước tính toán

#### Bước 1: Sắp xếp các lần chấm công
Tất cả các lần chấm công trong ngày được sắp xếp theo thứ tự thời gian từ sớm đến muộn.

#### Bước 2: Xác định giờ đến và giờ về

**Trường hợp A: Không có cấu hình Window**
- Nếu không có `checkInWindow` và `checkOutWindow`:
  - **Giờ đến** = Lần chấm công **đầu tiên** trong ngày
  - **Giờ về** = Lần chấm công **cuối cùng** trong ngày
  - Yêu cầu tối thiểu 2 lần chấm công

**Trường hợp B: Có cấu hình Window**
- Mỗi lần chấm công được kiểm tra xem nằm trong window nào:

| Vị trí chấm công | Window | Quy tắc chọn |
|------------------|--------|--------------|
| Giờ đến | `checkInWindowStart` - `checkInWindowEnd` | Lấy lần **sớm nhất** |
| Giờ về | `checkOutWindowStart` - `checkOutWindowEnd` | Lấy lần **muộn nhất** |

**Xử lý vùng giao nhau của window:**
- Nếu một lần chấm công nằm trong cả 2 window (check-in và check-out giao nhau):
  - Nếu **chưa có** giờ đến → Gán làm giờ đến
  - Nếu **đã có** giờ đến → Gán làm giờ về (cập nhật nếu muộn hơn)

#### Bước 3: Tính tổng giờ làm việc

```
Tổng giờ = Giờ về - Giờ đến
```

#### Bước 4: Trừ giờ nghỉ trưa (nếu có)

Nếu có cấu hình `breakStartTime` và `breakEndTime`:

```
Giờ nghỉ thực tế = phần overlap giữa [breakStart, breakEnd] và [workStart, workEnd]

overlapStart = max(breakStart, workStart)
overlapEnd = min(breakEnd, workEnd)

Nếu overlapEnd > overlapStart:
    Tổng giờ = Tổng giờ - (overlapEnd - overlapStart)
```

> **Lưu ý:** Chỉ trừ phần giờ nghỉ thực sự nằm trong khoảng giờ làm việc của nhân viên.

### Ví dụ minh họa

**Cấu hình ca:**
- Check-in window: 07:00 - 09:00
- Check-out window: 16:30 - 18:30
- Break time: 12:00 - 13:00

**Dữ liệu chấm công:**
- 08:15, 12:05, 17:30

**Tính toán:**
1. Giờ đến: 08:15 (nằm trong check-in window)
2. Giờ về: 17:30 (nằm trong check-out window)
3. Tổng giờ: 17:30 - 08:15 = 9h 15m
4. Trừ giờ nghỉ: 12:00 - 13:00 = 1h (nằm hoàn toàn trong giờ làm)
5. **Kết quả: 8h 15m**

---

## 2. Ca Chấm Công 4 Lần

### Điều kiện áp dụng
Ca này được áp dụng khi **có đầy đủ** cấu hình:
- `breakStartTime`, `breakEndTime`
- `breakOutWindowStart`, `breakOutWindowEnd`
- `breakInWindowStart`, `breakInWindowEnd`

### Các bước tính toán

#### Bước 1: Sắp xếp các lần chấm công
Tất cả các lần chấm công trong ngày được sắp xếp theo thứ tự thời gian.

#### Bước 2: Xác định 4 mốc thời gian

**Trường hợp A: Không có đủ Window**
- Lấy 4 lần chấm công theo thứ tự:
  1. **Chấm đến** = Lần thứ 1
  2. **Chấm ra nghỉ** = Lần thứ 2
  3. **Chấm vào sau nghỉ** = Lần thứ 3
  4. **Chấm về** = Lần thứ 4
- Yêu cầu tối thiểu 4 lần chấm công

**Trường hợp B: Có đầy đủ Window**

| Mốc thời gian | Window | Quy tắc chọn |
|---------------|--------|--------------|
| Chấm đến (shiftCheckIn) | `checkInWindowStart` - `checkInWindowEnd` | Lấy lần **sớm nhất** |
| Chấm ra nghỉ (breakCheckOut) | `breakOutWindowStart` - `breakOutWindowEnd` | Lấy lần **muộn nhất** |
| Chấm vào sau nghỉ (breakCheckIn) | `breakInWindowStart` - `breakInWindowEnd` | Lấy lần **sớm nhất** |
| Chấm về (shiftCheckOut) | `checkOutWindowStart` - `checkOutWindowEnd` | Lấy lần **muộn nhất** |

**Xử lý vùng giao nhau:**
- Nếu một lần chấm nằm trong nhiều window → Ưu tiên gán vào window còn thiếu
- Nếu tất cả đã có → Áp dụng logic sớm nhất/muộn nhất tương ứng

#### Bước 3: Tính tổng giờ làm việc

```
Giờ buổi sáng = Chấm ra nghỉ - Chấm đến
Giờ buổi chiều = Chấm về - Chấm vào sau nghỉ

Tổng giờ = Giờ buổi sáng + Giờ buổi chiều
```

> **Lưu ý:** Giờ nghỉ trưa được tự động loại trừ vì công thức sẽ không tính khoảng thời gian từ `breakCheckOut` đến `breakCheckIn`.

### Ví dụ minh họa

**Cấu hình ca:**
- Check-in window: 07:30 - 08:30
- Break-out window: 11:30 - 12:30
- Break-in window: 13:00 - 14:00
- Check-out window: 17:00 - 18:00

**Dữ liệu chấm công:**
- 08:00, 12:00, 13:15, 17:45

**Tính toán:**
1. Chấm đến: 08:00
2. Chấm ra nghỉ: 12:00
3. Chấm vào sau nghỉ: 13:15
4. Chấm về: 17:45
5. Giờ buổi sáng: 12:00 - 08:00 = 4h 00m
6. Giờ buổi chiều: 17:45 - 13:15 = 4h 30m
7. **Kết quả: 8h 30m**

---

## Các Trường Hợp Đặc Biệt

### Ca làm việc đặc biệt theo ngày
Hệ thống hỗ trợ cấu hình ca làm việc khác nhau cho từng ngày trong tuần thông qua `specialShiftDays`. Ví dụ: Thứ 7 có thể có ca làm ngắn hơn.

### Thiếu chấm công
- Nếu không đủ số lần chấm công cần thiết → Không tính được giờ làm việc (trả về `null`)
- Ca 2 lần: Cần tối thiểu 2 lần chấm hợp lệ
- Ca 4 lần: Cần tối thiểu 4 lần chấm hợp lệ

### Chấm công ngoài Window
- Các lần chấm công không nằm trong bất kỳ window nào sẽ bị bỏ qua
- Không ảnh hưởng đến kết quả tính toán

---

## Bảng Tóm Tắt

| Tiêu chí | Ca 2 lần | Ca 4 lần |
|----------|----------|----------|
| Số lần chấm tối thiểu | 2 | 4 |
| Công thức | `Về - Đến - Nghỉ` | `(Ra nghỉ - Đến) + (Về - Vào sau nghỉ)` |
| Xử lý giờ nghỉ | Trừ từ tổng giờ | Tự động loại (không tính) |
| Quy tắc chọn giờ đến | Sớm nhất | Sớm nhất |
| Quy tắc chọn giờ về | Muộn nhất | Muộn nhất |
