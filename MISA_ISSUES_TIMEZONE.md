# Vấn đề múi giờ khi sử dụng hệ thống chấm công MISA

## Tổng quan

Tài liệu này mô tả các vấn đề gặp phải khi sử dụng hệ thống chấm công MISA ở múi giờ không phải Việt Nam (ví dụ: Úc, UTC+10/+11).

### Nguyên nhân gốc rễ

Khi thiết lập ca làm việc trong MISA, admin cần nhập:
- Giờ bắt đầu và kết thúc ca
- Khoảng thời gian cho phép chấm công

**⚠️ Vấn đề cốt lõi**: MISA luôn lấy tham chiếu thời gian theo múi giờ Việt Nam (UTC+7), bất kể nhân viên đang ở múi giờ nào.

Ngoài ra, API của MISA chỉ trả về thời gian theo chuẩn ISO 8601 (UTC+0) mà không cung cấp thông tin về múi giờ của nhân viên, dẫn đến việc có lấy API về cũng không thể xác định đúng thời gian địa phương.

---

## Các vấn đề chính

### 1. Thời gian nhắc chấm công bị sai lệch

**Mô tả**: Notification nhắc chấm công trên thiết bị di động bị sai lệch do tính toán dựa trên múi giờ Việt Nam.

**Ví dụ**:
- Ca làm việc thiết lập: 9:00 AM (check-in)
- Múi giờ Úc: UTC+10 (chênh lệch +3 giờ so với Việt Nam UTC+7)
- Thời gian nhận notification tại Úc: **1:00 PM** (thay vì 9:00 AM)

### 2. Không thể chấm công được

**Mô tả**: Nhân viên không thể chấm công do khoảng thời gian cho phép được tính theo múi giờ Việt Nam.

**Ví dụ**:
- Khoảng thời gian cho phép chấm công: 7:00 AM - 11:00 AM (giờ VN)
- Quy đổi sang giờ Úc: 10:00 AM - 2:00 PM
- Nhân viên cố gắng chấm công lúc 9:00 AM (giờ Úc) ≈ 6:00 AM (giờ VN)
- **Kết quả**: Hệ thống từ chối vì nằm ngoài khoảng thời gian cho phép

---

## Giải pháp tạm thời và hạn chế

### Phương pháp hiện tại

**Cách xử lý**: Điều chỉnh giờ phân ca bằng cách trừ đi chênh lệch múi giờ (ví dụ: -4 giờ cho Úc do chênh lệch thực tế là +3-4 giờ tùy giờ mùa hè).

**Ví dụ cụ thể**:

| Thực tế (giờ Úc) | Thiết lập trên MISA | Hiển thị trên MISA |
|------------------|---------------------|-------------------|
| 9:00 AM - 5:00 PM | 5:00 AM - 1:00 PM | 5:00 AM - 1:00 PM |

### ✅ Ưu điểm

- Nhân viên có thể chấm công bình thường
- Notification nhắc chấm công đúng giờ theo múi giờ địa phương
- Không ảnh hưởng đến trải nghiệm chấm công hàng ngày

### ❌ Nhược điểm

#### A. Dữ liệu chấm công bị sai lệch

Giờ chấm công trong báo cáo không phản ánh thời gian thực tế:

| Thời gian thực tế (Úc) | Ghi nhận trên MISA | Chênh lệch |
|------------------------|-------------------|------------|
| 9:00 AM | 5:00 AM | -4 giờ |
| 5:00 PM | 1:00 PM | -4 giờ |

#### B. Vấn đề khi tạo đơn (Leave/WFH)

**Tình huống**: Khi tạo đơn xin nghỉ/WFH, UI tự động điền:
- Giờ bắt đầu = Giờ bắt đầu ca (5:00 AM thay vì 9:00 AM)
- Giờ kết thúc = Giờ kết thúc ca (1:00 PM thay vì 5:00 PM)

**Hậu quả**:
- Nhân viên phải nhớ trừ 4 giờ khi điền thông tin đơn
- Dễ gây nhầm lẫn, đặc biệt khi tạo nhiều đơn trong ngày
- Đã có training nhưng vẫn là điểm bất tiện lớn

**Ví dụ**:
```
Xin nghỉ cả ngày:
- Mong đợi: 9:00 AM - 5:00 PM
- Phải điền: 5:00 AM - 1:00 PM ❌
```

#### C. Bảo trì và mở rộng phức tạp

1. **Giờ mùa hè**: Admin phải điều chỉnh giờ phân ca thủ công mỗi khi có thay đổi Daylight Saving Time

2. **Nhân viên ở múi giờ khác**: 
   - Phải tính toán lại giờ phân ca cho từng múi giờ
   - Cần tạo ca làm việc riêng cho mỗi múi giờ
   - Tăng độ phức tạp quản lý

Các vấn đề trên vẫn có thể xử lý được bằng việc lấy API và điều chỉnh lại, tuy nhiên vẫn tồn tại những vấn đề nghiêm trọng không thể giải quyết chỉ bằng cách này.

---

## Vấn đề nghiêm trọng không thể giải quyết

### 🚨 Mất đồng bộ giữa phân ca và đơn xin nghỉ

**Mô tả**: Phân ca và đơn xin nghỉ/WFH không đồng bộ với nhau, gây ảnh hưởng nghiêm trọng đến trải nghiệm người dùng.

**Kịch bản lỗi**:

1. **Ngày thứ 5 (Thursday)**: 
   - Nhân viên tạo đơn xin nghỉ vào ngày này
   - Đơn được phê duyệt

2. **Nhân viên khi sử dụng MISA**:
   - Vào ngày thứ 5 (Thursday) hệ thống Misa vẫn ghi nhận nhân viên đi làm ngày đó, nhắc nhân viên chấm công ❌
   - Không phản ánh đúng trạng thái nghỉ phép

3. **Ngày thứ 6 (Friday)**:
   - Hệ thống lại hiểu hôm nay nhân viên nghỉ nên không cho phép chấm công ❌

**Feedback từ người dùng (Cristina)**:
> "So say I put in an application for leave/WFH on Thursday, it will appear on the shift assignment as Thursday, which means on Friday, it will either make me sign in/out with the timekeeping button and not the QR code OR if I took leave on Thursday mean that I can't sign in on Friday?"

**Kiểm chứng điều này bằng việc xem trên Misa app**:
- Phát hiện một số trường hợp vẫn chấm công được bình thường dù đã có đơn nghỉ phép trước đó.
    - Ảnh 1: https://prnt.sc/UM_0Hi7-QxlA
    - Ảnh 2: https://prnt.sc/XcIjzssmjykq
- Có thể điều này xảy ra do khung window chấm công của các ca khác nhau dẫn đến việc chấm công vẫn được phép.

### 🚨 Sai lệch ngày trên giao diện Admin

**Điều kiện**: Admin mở giao diện quản lý ở múi giờ Úc

**Hiện tượng**:

| Module | Trạng thái |
|--------|-----------|
| **Ca làm việc → Bảng phân ca tổng hợp** | ❌ Dữ liệu ngày trước nhảy sang ngày kế tiếp |
| **Chấm công → Bảng chấm công chi tiết** | ✅ Hiển thị đúng ngày |

**Ví dụ**:
- Phân ca thứ 6 hiển thị ở ngày thứ 7
- Mặc dù ngày thứ 7 không được phân ca

**Lưu ý**: 
- Nếu mở ở múi giờ Việt Nam → Không có lỗi
- Chứng tỏ MISA không xử lý nhất quán múi giờ giữa các module

**Thông tin bổ sung**
- Bảng chấm công chi tiết = Bảng phân ca tổng hợp + Dữ liệu chấm công thực tế
- Ảnh lỗi: 
    - Bảng phân ca tổng hợp: https://prnt.sc/ZeL2TQIMppFs (Nguồn: Cristina)
    - Bảng chấm công chi tiết: https://prnt.sc/PZNBdKmVoMgl (Nguồn: Cristina)

---

## Phương án có thể xử lý bằng API

Một số vấn đề có thể khắc phục bằng cách:
- Lấy dữ liệu qua MISA API
- Xử lý và chuyển đổi múi giờ bên ngoài hệ thống
- Hiển thị dữ liệu đã được điều chỉnh trên giao diện tùy chỉnh

**Tuy nhiên**, các vấn đề sau **không thể giải quyết** qua API:
- ❌ Mất đồng bộ giữa phân ca và đơn xin nghỉ
- ❌ Sai lệch ngày giữa các module khác nhau
- ❌ Logic xử lý múi giờ không nhất quán trong hệ thống
- ❌ Nhân viên tạo đơn hôm trước dẫn đến hôm sau không thể chấm công, không có dữ liệu chấm công thì không thể tính toán

---

## Kết luận

Các vấn đề trên **chỉ có thể được khắc phục từ phía MISA** thông qua:

1. Hỗ trợ múi giờ (timezone) cho từng nhân viên
2. Đồng bộ hóa logic xử lý thời gian giữa các module
3. Cập nhật API để trả về thông tin múi giờ
4. Xử lý nhất quán ngày/giờ trên toàn bộ hệ thống

**Tác động**: Những hạn chế này ảnh hưởng nghiêm trọng đến trải nghiệm người dùng và tính chính xác của dữ liệu chấm công khi triển khai MISA tại các quốc gia có múi giờ khác Việt Nam.

---

## Metadata

- **Người viết**: Hòa
- **Ngày cập nhật**: 13/02/2026
