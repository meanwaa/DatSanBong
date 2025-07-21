# 🔗 MAPPING CONTROLLERS VÀ MODEL CLASSES - Website Đặt Sân Bóng Đá

## 📊 **TỔNG QUAN KIẾN TRÚC**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Controllers   │◄──►│   ViewModels    │◄──►│     Models      │
│   (12 classes)  │    │   (20+ classes) │    │   (14 tables)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 📋 **CHI TIẾT MAPPING CONTROLLERS ↔ MODELS**

### 1. **AccountController** 👤
```csharp
public class AccountController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `NguoiDung` | ✅ **PRIMARY** | Quản lý thông tin user, đăng nhập, đăng ký |
| `Role` | ✅ **LINKED** | Kiểm tra quyền user (Customer/Staff/Admin) |
| `PasswordResetToken` | ✅ **DIRECT** | Quên mật khẩu, reset password |

**ViewModels liên quan:**
- `LoginViewModel`
- `RegisterViewModel` 
- `ForgotPasswordViewModel`
- `ResetPasswordViewModel`
- `ChangePasswordViewModel`

**Các actions chính:**
```csharp
Login()          // → NguoiDung, Role
Register()       // → NguoiDung, Role
ForgotPassword() // → NguoiDung, PasswordResetToken
ResetPassword()  // → PasswordResetToken, NguoiDung
ChangePassword() // → NguoiDung
UpdateProfile()  // → NguoiDung
```

---

### 2. **HomeController** 🏠
```csharp
public class HomeController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `SanBong` | ✅ **PRIMARY** | Hiển thị danh sách sân, tìm kiếm |
| `LoaiSanBong` | ✅ **FILTER** | Filter theo loại sân (5, 7, 11) |
| `TienIchSanBong` | ✅ **DISPLAY** | Hiển thị tiện ích của sân |
| `Review` | ✅ **LINKED** | Hiển thị đánh giá sân |
| `DonDatSan` | ✅ **STATS** | Thống kê số lượt đặt |
| `NguoiDung` | ✅ **STATS** | Thống kê người dùng |

**ViewModels liên quan:**
- `HomeIndexViewModel`
- `SearchResultViewModel`
- `FieldCardViewModel`

**Các actions chính:**
```csharp
Index()    // → SanBong, DonDatSan, NguoiDung (stats)
Search()   // → SanBong, LoaiSanBong, TienIchSanBong, Review
About()    // → Không model
Contact()  // → Không model
```

---

### 3. **BookingController** 📅
```csharp
public class BookingController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `DonDatSan` | ✅ **PRIMARY** | CRUD đơn đặt sân |
| `SanBong` | ✅ **LINKED** | Thông tin sân được đặt |
| `NguoiDung` | ✅ **LINKED** | Thông tin khách hàng đặt |
| `TrangThaiDonDat` | ✅ **STATUS** | Trạng thái đơn đặt |
| `PhuongThucThanhToan` | ✅ **PAYMENT** | Phương thức thanh toán |
| `Review` | ✅ **LINKED** | Đánh giá sau khi chơi |
| `ThongBao` | ✅ **NOTIFY** | Gửi thông báo về booking |

**ViewModels liên quan:**
- `BookingViewModel`
- `TimeSlotViewModel`
- `BookingHistoryViewModel`
- `RescheduleViewModel`

**Các actions chính:**
```csharp
Book()              // → DonDatSan, SanBong, TrangThaiDonDat, PhuongThucThanhToan
MyBookings()        // → DonDatSan, SanBong, TrangThaiDonDat
BookingDetails()    // → DonDatSan, SanBong, NguoiDung, Review
PendingApproval()   // → DonDatSan, SanBong, NguoiDung (Admin/Staff)
ApproveBooking()    // → DonDatSan, ThongBao
RejectBooking()     // → DonDatSan, ThongBao
StartPlaying()      // → DonDatSan
CompleteBooking()   // → DonDatSan, ThongBao
CancelBooking()     // → DonDatSan, ThongBao
Reschedule()        // → DonDatSan, SanBong
```

---

### 4. **PaymentController** 💳
```csharp
public class PaymentController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `DonDatSan` | ✅ **PRIMARY** | Cập nhật trạng thái thanh toán |
| `PhuongThucThanhToan` | ✅ **PRIMARY** | Danh sách phương thức thanh toán |
| `PaymentTransaction` | ✅ **LOGGING** | Lưu lịch sử giao dịch |
| `SanBong` | ✅ **DISPLAY** | Thông tin sân để thanh toán |
| `NguoiDung` | ✅ **DISPLAY** | Thông tin khách hàng |

**ViewModels liên quan:**
- `PaymentSelectionViewModel`
- `BankTransferInfo`
- `PaymentConfirmationViewModel`

**Các actions chính:**
```csharp
SelectPaymentMethod()  // → DonDatSan, PhuongThucThanhToan
ProcessPayment()       // → DonDatSan, PaymentTransaction
ProcessCashPayment()   // → DonDatSan, PaymentTransaction
ProcessBankTransfer()  // → DonDatSan, PaymentTransaction
ProcessVNPay()         // → DonDatSan, PaymentTransaction
ProcessMoMo()          // → DonDatSan, PaymentTransaction
BankTransferInfo()     // → DonDatSan, SanBong
BookingSuccess()       // → DonDatSan, SanBong
ConfirmPayment()       // → DonDatSan (Admin/Staff only)
```

---

### 5. **AdminController** ⚙️
```csharp
public class AdminController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `SanBong` | ✅ **CRUD** | Quản lý sân bóng |
| `NguoiDung` | ✅ **MANAGE** | Quản lý người dùng |
| `DonDatSan` | ✅ **MANAGE** | Quản lý đơn đặt |
| `Role` | ✅ **ASSIGN** | Phân quyền |
| `LoaiSanBong` | ✅ **REFERENCE** | Loại sân |
| `TienIchSanBong` | ✅ **MANAGE** | Quản lý tiện ích |
| `PhanCongNhanVien` | ✅ **ASSIGN** | Phân công nhân viên |
| `Review` | ✅ **MODERATE** | Kiểm duyệt đánh giá |
| `SuCo` | ✅ **TRACK** | Theo dõi sự cố |
| `ThongBao` | ✅ **SEND** | Gửi thông báo |

**ViewModels liên quan:**
- `AdminDashboardViewModel`
- `FieldManagementViewModel`
- `UserManagementViewModel`
- `StaffAssignmentViewModel`

**Các actions chính:**
```csharp
Dashboard()          // → DonDatSan, SanBong, NguoiDung, Review (stats)
ManageFields()       // → SanBong, LoaiSanBong, TienIchSanBong
CreateField()        // → SanBong, LoaiSanBong, TienIchSanBong
EditField()          // → SanBong, LoaiSanBong, TienIchSanBong
DeleteField()        // → SanBong, DonDatSan (check active bookings)
ManageUsers()        // → NguoiDung, Role
ToggleUserStatus()   // → NguoiDung
ChangeUserRole()     // → NguoiDung, Role
AssignStaffToField() // → PhanCongNhanVien, NguoiDung, SanBong
GetDashboardStats()  // → Multiple models for statistics
```

---

### 6. **StaffController** 👷
```csharp
public class StaffController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `PhanCongNhanVien` | ✅ **PRIMARY** | Sân được phân công |
| `SanBong` | ✅ **ASSIGNED** | Sân mình quản lý |
| `DonDatSan` | ✅ **MANAGE** | Đơn đặt của sân mình |
| `NguoiDung` | ✅ **VIEW** | Thông tin khách hàng |
| `SuCo` | ✅ **REPORT** | Báo cáo sự cố |
| `Review` | ✅ **MODERATE** | Kiểm duyệt review sân mình |

**ViewModels liên quan:**
- `StaffDashboardViewModel`
- `AssignedFieldViewModel`

**Các actions chính:**
```csharp
Dashboard()      // → PhanCongNhanVien, SanBong, DonDatSan
ManageBookings() // → DonDatSan, SanBong, NguoiDung (assigned fields only)
```

---

### 7. **ReviewController** ⭐
```csharp
public class ReviewController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `Review` | ✅ **PRIMARY** | CRUD đánh giá |
| `DonDatSan` | ✅ **VALIDATE** | Kiểm tra đã hoàn thành |
| `SanBong` | ✅ **TARGET** | Sân được đánh giá |
| `NguoiDung` | ✅ **REVIEWER** | Người đánh giá |
| `ThongBao` | ✅ **NOTIFY** | Thông báo về review |

**ViewModels liên quan:**
- `CreateReviewViewModel`
- `ReviewManagementViewModel`

**Các actions chính:**
```csharp
Create()         // → Review, DonDatSan, SanBong, NguoiDung
ManageReviews()  // → Review, SanBong, NguoiDung (Admin/Staff)
ApproveReview()  // → Review, SanBong, ThongBao
RejectReview()   // → Review, ThongBao
DeleteReview()   // → Review, SanBong
```

---

### 8. **IncidentController** 🚨
```csharp
public class IncidentController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `SuCo` | ✅ **PRIMARY** | CRUD báo cáo sự cố |
| `SanBong` | ✅ **RELATED** | Sân có sự cố |
| `DonDatSan` | ✅ **RELATED** | Đơn đặt liên quan |
| `NguoiDung` | ✅ **REPORTER** | Người báo cáo |
| `ThongBao` | ✅ **NOTIFY** | Thông báo về sự cố |

**ViewModels liên quan:**
- `ReportIncidentViewModel`
- `IncidentManagementViewModel`

**Các actions chính:**
```csharp
ReportIncident() // → SuCo, SanBong, DonDatSan, NguoiDung, ThongBao
ViewIncidents()  // → SuCo, SanBong, DonDatSan, NguoiDung
UpdateStatus()   // → SuCo, ThongBao (Admin only)
```

---

### 9. **NotificationController** 🔔
```csharp
public class NotificationController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `ThongBao` | ✅ **PRIMARY** | CRUD thông báo |
| `NguoiDung` | ✅ **TARGET** | Người nhận thông báo |
| `Role` | ✅ **FILTER** | Gửi theo role |

**ViewModels liên quan:**
- `NotificationViewModel`
- `SendNotificationViewModel`

**Các actions chính:**
```csharp
Index()           // → ThongBao, NguoiDung
MarkAsRead()      // → ThongBao
MarkAllAsRead()   // → ThongBao
GetUnreadCount()  // → ThongBao
Send()            // → ThongBao, NguoiDung, Role (Admin/Staff)
```

---

### 10. **ReportController** 📊
```csharp
public class ReportController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `DonDatSan` | ✅ **PRIMARY** | Dữ liệu báo cáo chính |
| `SanBong` | ✅ **ANALYTICS** | Phân tích hiệu suất sân |
| `NguoiDung` | ✅ **ANALYTICS** | Phân tích khách hàng |
| `PhuongThucThanhToan` | ✅ **BREAKDOWN** | Phân tích theo payment method |
| `Review` | ✅ **QUALITY** | Đánh giá chất lượng |
| `SuCo` | ✅ **INCIDENTS** | Báo cáo sự cố |

**ViewModels liên quan:**
- `DetailedRevenueViewModel`
- `FieldPerformanceViewModel`
- `CustomerAnalyticsViewModel`

**Các actions chính:**
```csharp
DetailedRevenue()    // → DonDatSan, SanBong, PhuongThucThanhToan, NguoiDung
FieldPerformance()   // → SanBong, DonDatSan, Review, SuCo
CustomerAnalytics()  // → NguoiDung, DonDatSan, Role
GetRevenueChartData() // → DonDatSan
ExportRevenue()      // → DonDatSan, SanBong, NguoiDung
```

---

### 11. **ErrorController** ❌
```csharp
public class ErrorController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| Không sử dụng model | ❌ | Chỉ hiển thị error pages |

**Các actions chính:**
```csharp
Index()        // → Static error page
NotFound()     // → 404 error page
ServerError()  // → 500 error page
Unauthorized() // → 403 error page
```

---

### 12. **FileController** 📁
```csharp
public class FileController : BaseController
```

**Models được sử dụng:**
| Model Class | Usage | Mô tả |
|-------------|-------|-------|
| `SanBong` | ✅ **UPDATE** | Cập nhật đường dẫn hình ảnh |

**Các actions chính:**
```csharp
UploadFieldImage() // → SanBong (update HinhAnh)
DeleteFile()       // → File system only
```

---

## 📈 **THỐNG KÊ SỬ DỤNG MODELS**

### **Models được sử dụng nhiều nhất:**
| Model | Controllers sử dụng | Tần suất |
|-------|-------------------|----------|
| `DonDatSan` | 6 controllers | ⭐⭐⭐⭐⭐ |
| `SanBong` | 6 controllers | ⭐⭐⭐⭐⭐ |
| `NguoiDung` | 8 controllers | ⭐⭐⭐⭐⭐ |
| `ThongBao` | 5 controllers | ⭐⭐⭐⭐ |
| `Review` | 4 controllers | ⭐⭐⭐ |
| `Role` | 3 controllers | ⭐⭐ |
| `SuCo` | 3 controllers | ⭐⭐ |

### **Models ít được sử dụng:**
| Model | Controllers sử dụng | Tần suất |
|-------|-------------------|----------|
| `PasswordResetToken` | 1 controller | ⭐ |
| `PaymentTransaction` | 1 controller | ⭐ |
| `PhanCongNhanVien` | 2 controllers | ⭐⭐ |

---

## 🔗 **MỐI QUAN HỆ DEPENDENCIES**

### **Core Models (Most Connected):**
```
NguoiDung ←→ DonDatSan ←→ SanBong
    ↓            ↓          ↓
  Role      TrangThaiDonDat  LoaiSanBong
    ↓            ↓          ↓
ThongBao   PhuongThucThanhToan  TienIchSanBong
```

### **Secondary Models:**
```
DonDatSan → Review → SanBong
SanBong → SuCo → NguoiDung
NguoiDung → PhanCongNhanVien → SanBong
```

### **Independent Models:**
```
PasswordResetToken → NguoiDung
PaymentTransaction → DonDatSan
```

---

## 🎯 **KẾT LUẬN**

### **Pattern Architecture:**
- **Domain-Driven Design**: Mỗi controller tập trung vào một domain cụ thể
- **Rich Models**: `DonDatSan`, `SanBong`, `NguoiDung` là core entities
- **Support Models**: `Role`, `TrangThaiDonDat`, etc. làm lookup tables
- **Cross-cutting**: `ThongBao` được sử dụng across multiple domains

### **Best Practices được áp dụng:**
- ✅ **Separation of Concerns**: Mỗi controller có responsibility rõ ràng
- ✅ **ViewModels**: Tách biệt presentation logic khỏi domain models
- ✅ **Navigation Properties**: EF relationships được setup đúng
- ✅ **Eager Loading**: Sử dụng Include() để tối ưu queries

**Kiến trúc này đảm bảo tính modular, maintainable và scalable cho website đặt sân bóng đá! 🏆**