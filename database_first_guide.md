# 🔄 CHUYỂN ĐỔI SANG DATABASE FIRST - Website Đặt Sân Bóng Đá

## 🎯 **SO SÁNH CODE FIRST vs DATABASE FIRST**

| Aspect | Code First | Database First |
|--------|------------|----------------|
| **Workflow** | Models → Database | Database → Models |
| **Control** | Code controls DB | Database controls code |
| **Team** | Developer-led | DBA-led |
| **Versioning** | Migrations | SQL Scripts |
| **Flexibility** | High | Medium |
| **Enterprise** | Good | Excellent |

---

## 📋 **BƯỚC 1: TẠO DATABASE TRƯỚC**

### **1.1 Chạy SQL Script để tạo Database hoàn chỉnh**

```sql
-- Tạo database
CREATE DATABASE DatSanBongDa;
GO

USE DatSanBongDa;
GO

-- 1. Bảng Roles
CREATE TABLE Roles (
    IDrole INT IDENTITY(1,1) PRIMARY KEY,
    Role_name NVARCHAR(50) NOT NULL UNIQUE,
    Description NVARCHAR(255)
);

-- 2. Bảng LoaiSanBong
CREATE TABLE LoaiSanBong (
    IDLoaiSan INT IDENTITY(1,1) PRIMARY KEY,
    LoaiSan NVARCHAR(50) NOT NULL UNIQUE,
    min_players INT NOT NULL CHECK (min_players > 0),
    max_players INT NOT NULL,
    CONSTRAINT CK_LoaiSanBong_PlayerCount CHECK (max_players >= min_players)
);

-- 3. Bảng TienIchSanBong
CREATE TABLE TienIchSanBong (
    IDTienIch INT IDENTITY(1,1) PRIMARY KEY,
    TenTienIch NVARCHAR(100) NOT NULL UNIQUE,
    MoTa NVARCHAR(255)
);

-- 4. Bảng PhuongThucThanhToan
CREATE TABLE PhuongThucThanhToan (
    IDPT INT IDENTITY(1,1) PRIMARY KEY,
    TenPT NVARCHAR(50) NOT NULL UNIQUE,
    MoTa NVARCHAR(255)
);

-- 5. Bảng TrangThaiDonDat
CREATE TABLE TrangThaiDonDat (
    IDstatus INT IDENTITY(1,1) PRIMARY KEY,
    Tenstatus NVARCHAR(50) NOT NULL UNIQUE,
    MoTa NVARCHAR(255)
);

-- 6. Bảng NguoiDung
CREATE TABLE NguoiDung (
    IDuser INT IDENTITY(1,1) PRIMARY KEY,
    username NVARCHAR(50) UNIQUE,
    email NVARCHAR(100) NOT NULL UNIQUE,
    pass_hash NVARCHAR(255) NOT NULL,
    fullname NVARCHAR(100),
    sdt NVARCHAR(20) UNIQUE,
    DiaChi NVARCHAR(255),
    trangthaiTK BIT DEFAULT 1,
    last_log DATETIME,
    create_at DATETIME DEFAULT GETDATE(),
    update_at DATETIME DEFAULT GETDATE(),
    IDrole INT NOT NULL,
    FOREIGN KEY (IDrole) REFERENCES Roles(IDrole) ON DELETE NO ACTION ON UPDATE CASCADE
);

-- 7. Bảng SanBong
CREATE TABLE SanBong (
    IDSanBong INT IDENTITY(1,1) PRIMARY KEY,
    TenSanBong NVARCHAR(100) NOT NULL,
    DiaChi NVARCHAR(255) NOT NULL,
    MoTa NVARCHAR(MAX),
    GiaThue DECIMAL(10,2) NOT NULL CHECK (GiaThue > 0),
    TrangThaiSan_ BIT DEFAULT 1,
    HinhAnh NVARCHAR(255),
    AverageDanhGia DECIMAL(3,2) DEFAULT 0 CHECK (AverageDanhGia >= 0 AND AverageDanhGia <= 5),
    TongLuotDanhGia INT DEFAULT 0 CHECK (TongLuotDanhGia >= 0),
    created_at DATETIME DEFAULT GETDATE(),
    updated_at DATETIME DEFAULT GETDATE(),
    Latitude DECIMAL(10,8),
    Longitude DECIMAL(11,8),
    IDLoaiSan INT NOT NULL,
    FOREIGN KEY (IDLoaiSan) REFERENCES LoaiSanBong(IDLoaiSan) ON DELETE NO ACTION ON UPDATE CASCADE
);

-- 8. Bảng DonDatSan
CREATE TABLE DonDatSan (
    IDBooking INT IDENTITY(1,1) PRIMARY KEY,
    BookingCode NVARCHAR(50) UNIQUE,
    NgayBooking DATE NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    TongTien DECIMAL(10,2) NOT NULL CHECK (TongTien > 0),
    TTThanhToan BIT DEFAULT 0,
    GhiChu NVARCHAR(MAX),
    LyDoHuy NVARCHAR(MAX),
    admin_notes NVARCHAR(MAX),
    created_at DATETIME DEFAULT GETDATE(),
    updated_at DATETIME DEFAULT GETDATE(),
    IDuser INT NOT NULL,
    IDSanBong INT NOT NULL,
    IDPT INT NOT NULL,
    IDstatus INT NOT NULL,
    FOREIGN KEY (IDuser) REFERENCES NguoiDung(IDuser) ON DELETE NO ACTION ON UPDATE NO ACTION,
    FOREIGN KEY (IDSanBong) REFERENCES SanBong(IDSanBong) ON DELETE NO ACTION ON UPDATE NO ACTION,
    FOREIGN KEY (IDPT) REFERENCES PhuongThucThanhToan(IDPT) ON DELETE NO ACTION ON UPDATE NO ACTION,
    FOREIGN KEY (IDstatus) REFERENCES TrangThaiDonDat(IDstatus) ON DELETE NO ACTION ON UPDATE NO ACTION,
    CONSTRAINT CK_DonDatSan_Time CHECK (start_time < end_time)
);

-- 9. Bảng Reviews
CREATE TABLE Reviews (
    IDreview INT IDENTITY(1,1) PRIMARY KEY,
    rating TINYINT NOT NULL CHECK (rating >= 1 AND rating <= 5),
    comment NVARCHAR(1000),
    created_at DATETIME DEFAULT GETDATE(),
    updated_at DATETIME,
    is_approved BIT DEFAULT 1,
    IDuser INT NOT NULL,
    IDSanBong INT NOT NULL,
    IDBooking INT NOT NULL UNIQUE,
    FOREIGN KEY (IDuser) REFERENCES NguoiDung(IDuser) ON DELETE NO ACTION ON UPDATE NO ACTION,
    FOREIGN KEY (IDSanBong) REFERENCES SanBong(IDSanBong) ON DELETE NO ACTION ON UPDATE NO ACTION,
    FOREIGN KEY (IDBooking) REFERENCES DonDatSan(IDBooking) ON DELETE NO ACTION ON UPDATE NO ACTION
);

-- 10. Bảng ThongBao
CREATE TABLE ThongBao (
    IDThongBao INT IDENTITY(1,1) PRIMARY KEY,
    TieuDe NVARCHAR(200) NOT NULL,
    NoiDung NVARCHAR(MAX) NOT NULL,
    LoaiThongBao NVARCHAR(50) DEFAULT 'info',
    TrangThaiDoc BIT DEFAULT 0,
    created_at DATETIME DEFAULT GETDATE(),
    IDuser INT NOT NULL,
    FOREIGN KEY (IDuser) REFERENCES NguoiDung(IDuser) ON DELETE CASCADE ON UPDATE NO ACTION
);

-- 11. Bảng SuCo
CREATE TABLE SuCo (
    IDSuCo INT IDENTITY(1,1) PRIMARY KEY,
    LoaiSuCo NVARCHAR(100) NOT NULL,
    TrangThai NVARCHAR(50) DEFAULT 'Mới',
    resolution_notes NVARCHAR(MAX),
    reported_at DATETIME DEFAULT GETDATE(),
    resolved_at DATETIME,
    MoTa NVARCHAR(MAX) NOT NULL,
    IDuser INT,
    IDBooking INT,
    IDSanBong INT,
    IDAdmin INT,
    user_backup_info NVARCHAR(500),
    booking_backup_info NVARCHAR(500),
    sanBong_backup_info NVARCHAR(500),
    admin_backup_info NVARCHAR(500),
    FOREIGN KEY (IDuser) REFERENCES NguoiDung(IDuser) ON DELETE SET NULL ON UPDATE NO ACTION,
    FOREIGN KEY (IDBooking) REFERENCES DonDatSan(IDBooking) ON DELETE SET NULL ON UPDATE NO ACTION,
    FOREIGN KEY (IDSanBong) REFERENCES SanBong(IDSanBong) ON DELETE SET NULL ON UPDATE NO ACTION,
    FOREIGN KEY (IDAdmin) REFERENCES NguoiDung(IDuser) ON DELETE NO ACTION ON UPDATE NO ACTION
);

-- 12. Bảng PhanCongNhanVien
CREATE TABLE PhanCongNhanVien (
    IDPhanCong INT IDENTITY(1,1) PRIMARY KEY,
    NgayBatDau DATE NOT NULL,
    NgayKetThuc DATE,
    GhiChu NVARCHAR(MAX),
    created_at DATETIME DEFAULT GETDATE(),
    IDuser INT NOT NULL,
    IDSanBong INT NOT NULL,
    FOREIGN KEY (IDuser) REFERENCES NguoiDung(IDuser) ON DELETE CASCADE ON UPDATE NO ACTION,
    FOREIGN KEY (IDSanBong) REFERENCES SanBong(IDSanBong) ON DELETE CASCADE ON UPDATE NO ACTION,
    CONSTRAINT CK_PhanCongNhanVien_Date CHECK (NgayKetThuc IS NULL OR NgayKetThuc >= NgayBatDau)
);

-- 13. Bảng SanBong_TienIch (Many-to-Many)
CREATE TABLE SanBong_TienIch (
    IDSanBong INT NOT NULL,
    IDTienIch INT NOT NULL,
    PRIMARY KEY (IDSanBong, IDTienIch),
    FOREIGN KEY (IDSanBong) REFERENCES SanBong(IDSanBong) ON DELETE CASCADE ON UPDATE NO ACTION,
    FOREIGN KEY (IDTienIch) REFERENCES TienIchSanBong(IDTienIch) ON DELETE CASCADE ON UPDATE NO ACTION
);

-- 14. Bảng PasswordResetTokens (cho forgot password)
CREATE TABLE PasswordResetTokens (
    IDToken INT IDENTITY(1,1) PRIMARY KEY,
    IDuser INT NOT NULL,
    Token NVARCHAR(255) NOT NULL UNIQUE,
    ExpiryDate DATETIME NOT NULL,
    IsUsed BIT DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (IDuser) REFERENCES NguoiDung(IDuser) ON DELETE CASCADE
);

-- Tạo các indexes để tối ưu hiệu suất
CREATE NONCLUSTERED INDEX IX_DonDatSan_NgayBooking ON DonDatSan (NgayBooking);
CREATE NONCLUSTERED INDEX IX_DonDatSan_IDstatus ON DonDatSan (IDstatus);
CREATE NONCLUSTERED INDEX IX_Reviews_IDSanBong ON Reviews (IDSanBong);
CREATE NONCLUSTERED INDEX IX_Reviews_is_approved ON Reviews (is_approved);
CREATE NONCLUSTERED INDEX IX_SuCo_TrangThai ON SuCo (TrangThai);
CREATE NONCLUSTERED INDEX IX_SuCo_reported_at ON SuCo (reported_at);
CREATE NONCLUSTERED INDEX IX_NguoiDung_email ON NguoiDung (email);
CREATE NONCLUSTERED INDEX IX_NguoiDung_username ON NguoiDung (username);

-- Insert dữ liệu mẫu
INSERT INTO Roles (Role_name, Description) VALUES 
('Admin', 'Quản trị viên hệ thống'),
('Staff', 'Nhân viên quản lý sân'),
('Customer', 'Khách hàng đặt sân');

INSERT INTO LoaiSanBong (LoaiSan, min_players, max_players) VALUES 
('Sân 5', 5, 10),
('Sân 7', 7, 14),
('Sân 11', 11, 22);

INSERT INTO TienIchSanBong (TenTienIch, MoTa) VALUES 
('Đèn chiếu sáng', 'Hệ thống đèn LED chất lượng cao'),
('Phòng thay đồ', 'Phòng thay đồ sạch sẽ, rộng rãi'),
('Bãi đỗ xe', 'Bãi đỗ xe miễn phí'),
('Canteen', 'Quầy bán nước uống và đồ ăn nhẹ'),
('WiFi', 'Internet miễn phí tốc độ cao'),
('Camera an ninh', 'Hệ thống camera giám sát 24/7');

INSERT INTO PhuongThucThanhToan (TenPT, MoTa) VALUES 
('Tiền mặt', 'Thanh toán bằng tiền mặt tại sân'),
('Chuyển khoản', 'Chuyển khoản ngân hàng'),
('VNPay', 'Thanh toán qua VNPay'),
('MoMo', 'Thanh toán qua ví MoMo');

INSERT INTO TrangThaiDonDat (Tenstatus, MoTa) VALUES 
('Chờ duyệt', 'Đơn đặt sân đang chờ duyệt'),
('Đã duyệt', 'Đơn đặt sân đã được duyệt'),
('Đang chơi', 'Khách hàng đang sử dụng sân'),
('Hoàn thành', 'Đã hoàn thành sử dụng sân'),
('Đã hủy', 'Đơn đặt sân đã bị hủy');
```

---

## 📋 **BƯỚC 2: CẤU HÌNH PROJECT CHO DATABASE FIRST**

### **2.1 Cài đặt NuGet Packages**

```xml
<!-- packages.config -->
<packages>
  <package id="EntityFramework" version="6.4.4" targetFramework="net48" />
  <package id="Microsoft.AspNet.Mvc" version="5.2.9" targetFramework="net48" />
  <package id="BCrypt.Net-Next" version="4.0.3" targetFramework="net48" />
  <package id="Microsoft.AspNet.SignalR" version="2.4.3" targetFramework="net48" />
  <package id="PagedList.Mvc" version="4.5.0" targetFramework="net48" />
  <package id="Newtonsoft.Json" version="13.0.3" targetFramework="net48" />
  <!-- Remove Code First specific packages -->
</packages>
```

### **2.2 Cập nhật web.config**

```xml
<connectionStrings>
  <add name="DatSanBongDaEntities" 
       connectionString="Data Source=server_name;Initial Catalog=DatSanBongDa;Integrated Security=True"
       providerName="System.Data.SqlClient" />
</connectionStrings>

<appSettings>
  <!-- Email configuration -->
  <add key="FromEmail" value="noreply@footballfield.com" />
  <add key="FromPassword" value="your-email-password" />
  <add key="SMTPHost" value="smtp.gmail.com" />
  <add key="SMTPPort" value="587" />
  
  <!-- Security settings -->
  <add key="PasswordResetTokenExpiryHours" value="24" />
  <add key="MaxLoginAttempts" value="5" />
  
  <!-- Other settings -->
  <add key="webpages:Version" value="3.0.0.0" />
  <add key="webpages:Enabled" value="false" />
  <add key="ClientValidationEnabled" value="true" />
  <add key="UnobtrusiveJavaScriptEnabled" value="true" />
</appSettings>
```

---

## 📋 **BƯỚC 3: TẠO ENTITY DATA MODEL**

### **3.1 Add ADO.NET Entity Data Model**

1. **Right-click** vào project → **Add** → **New Item**
2. Chọn **ADO.NET Entity Data Model** → Name: `FootballFieldModel.edmx`
3. Chọn **EF Designer from database**
4. Chọn connection string đã tạo
5. Chọn **Tables**, **Views**, **Stored Procedures** cần thiết
6. **Finish**

### **3.2 Cấu hình Entity Framework Context**

```csharp
// Data/DatSanBongDaEntities.cs (Auto-generated, có thể customize)
public partial class DatSanBongDaEntities : DbContext
{
    public DatSanBongDaEntities()
        : base("name=DatSanBongDaEntities")
    {
        // Disable lazy loading nếu muốn
        this.Configuration.LazyLoadingEnabled = false;
        
        // Enable proxy creation
        this.Configuration.ProxyCreationEnabled = true;
        
        // Validate on save
        this.Configuration.ValidateOnSaveEnabled = true;
    }

    public virtual DbSet<Role> Roles { get; set; }
    public virtual DbSet<LoaiSanBong> LoaiSanBongs { get; set; }
    public virtual DbSet<TienIchSanBong> TienIchSanBongs { get; set; }
    public virtual DbSet<PhuongThucThanhToan> PhuongThucThanhToans { get; set; }
    public virtual DbSet<TrangThaiDonDat> TrangThaiDonDats { get; set; }
    public virtual DbSet<NguoiDung> NguoiDungs { get; set; }
    public virtual DbSet<SanBong> SanBongs { get; set; }
    public virtual DbSet<DonDatSan> DonDatSans { get; set; }
    public virtual DbSet<Review> Reviews { get; set; }
    public virtual DbSet<ThongBao> ThongBaos { get; set; }
    public virtual DbSet<SuCo> SuCos { get; set; }
    public virtual DbSet<PhanCongNhanVien> PhanCongNhanViens { get; set; }
    public virtual DbSet<PasswordResetToken> PasswordResetTokens { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Custom configurations nếu cần
        modelBuilder.Entity<DonDatSan>()
            .Property(e => e.TongTien)
            .HasPrecision(10, 2);

        modelBuilder.Entity<SanBong>()
            .Property(e => e.GiaThue)
            .HasPrecision(10, 2);

        modelBuilder.Entity<SanBong>()
            .Property(e => e.AverageDanhGia)
            .HasPrecision(3, 2);

        modelBuilder.Entity<SanBong>()
            .Property(e => e.Latitude)
            .HasPrecision(10, 8);

        modelBuilder.Entity<SanBong>()
            .Property(e => e.Longitude)
            .HasPrecision(11, 8);

        base.OnModelCreating(modelBuilder);
    }
}
```

---

## 📋 **BƯỚC 4: TẠO PARTIAL CLASSES CHO VALIDATION**

Vì Database First tạo ra các entity classes tự động, ta cần tạo **partial classes** để thêm validation attributes:

### **4.1 Models/Partials/NguoiDung.Partial.cs**

```csharp
using System.ComponentModel.DataAnnotations;

namespace YourProject.Models
{
    [MetadataType(typeof(NguoiDungMetadata))]
    public partial class NguoiDung
    {
        // Có thể thêm custom properties hoặc methods
        public string RoleName 
        { 
            get { return this.Role?.Role_name; } 
        }
    }

    public class NguoiDungMetadata
    {
        [Required(ErrorMessage = "Email là bắt buộc")]
        [EmailAddress(ErrorMessage = "Email không hợp lệ")]
        [StringLength(100, ErrorMessage = "Email không được quá 100 ký tự")]
        public string email { get; set; }

        [StringLength(50, ErrorMessage = "Username không được quá 50 ký tự")]
        public string username { get; set; }

        [Required(ErrorMessage = "Mật khẩu là bắt buộc")]
        [StringLength(255, ErrorMessage = "Mật khẩu đã hash không được quá 255 ký tự")]
        public string pass_hash { get; set; }

        [StringLength(100, ErrorMessage = "Họ tên không được quá 100 ký tự")]
        [Display(Name = "Họ và tên")]
        public string fullname { get; set; }

        [StringLength(20, ErrorMessage = "Số điện thoại không được quá 20 ký tự")]
        [Display(Name = "Số điện thoại")]
        public string sdt { get; set; }

        [StringLength(255, ErrorMessage = "Địa chỉ không được quá 255 ký tự")]
        [Display(Name = "Địa chỉ")]
        public string DiaChi { get; set; }
    }
}
```

### **4.2 Models/Partials/SanBong.Partial.cs**

```csharp
using System.ComponentModel.DataAnnotations;

namespace YourProject.Models
{
    [MetadataType(typeof(SanBongMetadata))]
    public partial class SanBong
    {
        public string LoaiSanName 
        { 
            get { return this.LoaiSanBong?.LoaiSan; } 
        }
        
        public string AverageRatingDisplay
        {
            get { return this.AverageDanhGia?.ToString("F1") ?? "0.0"; }
        }
    }

    public class SanBongMetadata
    {
        [Required(ErrorMessage = "Tên sân bóng là bắt buộc")]
        [StringLength(100, ErrorMessage = "Tên sân không được quá 100 ký tự")]
        [Display(Name = "Tên sân bóng")]
        public string TenSanBong { get; set; }

        [Required(ErrorMessage = "Địa chỉ là bắt buộc")]
        [StringLength(255, ErrorMessage = "Địa chỉ không được quá 255 ký tự")]
        [Display(Name = "Địa chỉ")]
        public string DiaChi { get; set; }

        [Display(Name = "Mô tả")]
        public string MoTa { get; set; }

        [Required(ErrorMessage = "Giá thuê là bắt buộc")]
        [Range(0.01, double.MaxValue, ErrorMessage = "Giá thuê phải lớn hơn 0")]
        [Display(Name = "Giá thuê (VNĐ/giờ)")]
        public decimal GiaThue { get; set; }

        [Display(Name = "Hình ảnh")]
        public string HinhAnh { get; set; }

        [Range(-90, 90, ErrorMessage = "Latitude phải trong khoảng -90 đến 90")]
        public decimal? Latitude { get; set; }

        [Range(-180, 180, ErrorMessage = "Longitude phải trong khoảng -180 đến 180")]
        public decimal? Longitude { get; set; }
    }
}
```

### **4.3 Models/Partials/DonDatSan.Partial.cs**

```csharp
using System.ComponentModel.DataAnnotations;

namespace YourProject.Models
{
    [MetadataType(typeof(DonDatSanMetadata))]
    public partial class DonDatSan
    {
        public string StatusName 
        { 
            get { return this.TrangThaiDonDat?.Tenstatus; } 
        }
        
        public string PaymentMethodName
        {
            get { return this.PhuongThucThanhToan?.TenPT; }
        }
        
        public TimeSpan Duration
        {
            get { return this.end_time - this.start_time; }
        }
    }

    public class DonDatSanMetadata
    {
        [Display(Name = "Mã đặt sân")]
        public string BookingCode { get; set; }

        [Required(ErrorMessage = "Ngày đặt là bắt buộc")]
        [Display(Name = "Ngày đặt")]
        [DataType(DataType.Date)]
        public System.DateTime NgayBooking { get; set; }

        [Required(ErrorMessage = "Giờ bắt đầu là bắt buộc")]
        [Display(Name = "Giờ bắt đầu")]
        [DataType(DataType.Time)]
        public System.TimeSpan start_time { get; set; }

        [Required(ErrorMessage = "Giờ kết thúc là bắt buộc")]
        [Display(Name = "Giờ kết thúc")]
        [DataType(DataType.Time)]
        public System.TimeSpan end_time { get; set; }

        [Required(ErrorMessage = "Tổng tiền là bắt buộc")]
        [Range(0.01, double.MaxValue, ErrorMessage = "Tổng tiền phải lớn hơn 0")]
        [Display(Name = "Tổng tiền")]
        public decimal TongTien { get; set; }

        [Display(Name = "Ghi chú")]
        public string GhiChu { get; set; }
    }
}
```

---

## 📋 **BƯỚC 5: CẬP NHẬT CONTROLLERS CHO DATABASE FIRST**

### **5.1 Cập nhật Base Controller**

```csharp
public abstract class BaseController : Controller
{
    protected DatSanBongDaEntities db = new DatSanBongDaEntities();

    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            db.Dispose();
        }
        base.Dispose(disposing);
    }

    protected ActionResult HandleDbUpdateException(DbUpdateException ex)
    {
        // Log the exception
        // Return appropriate error response
        TempData["ErrorMessage"] = "Có lỗi xảy ra khi cập nhật dữ liệu. Vui lòng thử lại.";
        return RedirectToAction("Index");
    }
}
```

### **5.2 Cập nhật AccountController**

```csharp
public class AccountController : BaseController
{
    // GET: Account/Login
    [HttpGet]
    public ActionResult Login()
    {
        return View();
    }

    // POST: Account/Login
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Login(LoginViewModel model)
    {
        if (ModelState.IsValid)
        {
            // Tìm user trong database
            var user = db.NguoiDungs
                .Include(u => u.Role) // Eager loading
                .FirstOrDefault(u => 
                    (u.email == model.EmailOrUsername || u.username == model.EmailOrUsername) 
                    && u.trangthaiTK == true);

            if (user != null && SecurityHelper.VerifyPassword(model.Password, user.pass_hash))
            {
                // Cập nhật last login
                user.last_log = DateTime.Now;
                db.SaveChanges();

                // Set authentication cookie
                AuthenticationHelper.SetAuthCookie(user.IDuser.ToString(), user.Role.Role_name, false);

                // Redirect theo role
                switch (user.Role.Role_name)
                {
                    case "Admin":
                        return RedirectToAction("Dashboard", "Admin");
                    case "Staff":
                        return RedirectToAction("Dashboard", "Staff");
                    default:
                        return RedirectToAction("Index", "Home");
                }
            }
            else
            {
                ModelState.AddModelError("", "Email/Username hoặc mật khẩu không đúng");
            }
        }

        return View(model);
    }

    // GET: Account/Register
    [HttpGet]
    public ActionResult Register()
    {
        return View();
    }

    // POST: Account/Register
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Register(RegisterViewModel model)
    {
        if (ModelState.IsValid)
        {
            // Kiểm tra email đã tồn tại
            if (db.NguoiDungs.Any(u => u.email == model.Email))
            {
                ModelState.AddModelError("Email", "Email này đã được sử dụng");
                return View(model);
            }

            // Kiểm tra username đã tồn tại
            if (!string.IsNullOrEmpty(model.Username) && 
                db.NguoiDungs.Any(u => u.username == model.Username))
            {
                ModelState.AddModelError("Username", "Username này đã được sử dụng");
                return View(model);
            }

            // Tạo user mới
            var customerRole = db.Roles.First(r => r.Role_name == "Customer");
            
            var user = new NguoiDung
            {
                username = model.Username,
                email = model.Email,
                pass_hash = SecurityHelper.HashPassword(model.Password),
                fullname = model.FullName,
                sdt = model.PhoneNumber,
                trangthaiTK = true,
                create_at = DateTime.Now,
                update_at = DateTime.Now,
                IDrole = customerRole.IDrole
            };

            db.NguoiDungs.Add(user);
            db.SaveChanges();

            TempData["SuccessMessage"] = "Đăng ký thành công! Vui lòng đăng nhập.";
            return RedirectToAction("Login");
        }

        return View(model);
    }

    // Other actions remain similar but use db context instead of ApplicationDbContext
}
```

---

## 📋 **BƯỚC 6: CẬP NHẬT HELPERS**

### **6.1 Cập nhật ScheduleHelper**

```csharp
public static class ScheduleHelper
{
    public static bool CheckAvailability(int fieldId, DateTime date, TimeSpan startTime, TimeSpan endTime, int? excludeBookingId = null)
    {
        using (var db = new DatSanBongDaEntities())
        {
            var query = db.DonDatSans
                .Where(b => b.IDSanBong == fieldId &&
                           DbFunctions.TruncateTime(b.NgayBooking) == date.Date &&
                           b.IDstatus != 5); // Không tính đơn đã hủy

            if (excludeBookingId.HasValue)
            {
                query = query.Where(b => b.IDBooking != excludeBookingId.Value);
            }

            var conflictingBookings = query
                .Where(b => b.start_time < endTime && b.end_time > startTime)
                .Any();

            return !conflictingBookings;
        }
    }

    public static List<TimeSlotViewModel> GetAvailableTimeSlots(int fieldId, DateTime date)
    {
        var timeSlots = new List<TimeSlotViewModel>();
        var openTime = new TimeSpan(6, 0, 0); // 6:00 AM
        var closeTime = new TimeSpan(22, 0, 0); // 10:00 PM

        using (var db = new DatSanBongDaEntities())
        {
            var field = db.SanBongs.Find(fieldId);
            if (field == null) return timeSlots;

            var existingBookings = db.DonDatSans
                .Where(b => b.IDSanBong == fieldId &&
                           DbFunctions.TruncateTime(b.NgayBooking) == date.Date &&
                           b.IDstatus != 5)
                .Select(b => new { b.start_time, b.end_time })
                .ToList();

            for (var time = openTime; time < closeTime; time = time.Add(new TimeSpan(1, 0, 0)))
            {
                var endTime = time.Add(new TimeSpan(1, 0, 0));
                var isAvailable = !existingBookings.Any(b => b.start_time < endTime && b.end_time > time);

                timeSlots.Add(new TimeSlotViewModel
                {
                    StartTime = time,
                    EndTime = endTime,
                    IsAvailable = isAvailable,
                    Price = field.GiaThue,
                    DisplayTime = $"{time:hh\\:mm} - {endTime:hh\\:mm}"
                });
            }
        }

        return timeSlots;
    }

    public static decimal CalculateTotal(decimal pricePerHour, TimeSpan duration)
    {
        var hours = (decimal)duration.TotalHours;
        return Math.Round(pricePerHour * hours, 2);
    }
}
```

---

## 📋 **BƯỚC 7: REPOSITORY PATTERN (OPTIONAL)**

Nếu muốn sử dụng Repository Pattern với Database First:

### **7.1 Generic Repository Interface**

```csharp
public interface IRepository<T> where T : class
{
    IQueryable<T> GetAll();
    T GetById(int id);
    void Insert(T entity);
    void Update(T entity);
    void Delete(int id);
    void Delete(T entity);
    void Save();
}
```

### **7.2 Generic Repository Implementation**

```csharp
public class Repository<T> : IRepository<T> where T : class
{
    private DatSanBongDaEntities context;
    private DbSet<T> dbSet;

    public Repository(DatSanBongDaEntities context)
    {
        this.context = context;
        this.dbSet = context.Set<T>();
    }

    public virtual IQueryable<T> GetAll()
    {
        return dbSet;
    }

    public virtual T GetById(int id)
    {
        return dbSet.Find(id);
    }

    public virtual void Insert(T entity)
    {
        dbSet.Add(entity);
    }

    public virtual void Update(T entity)
    {
        dbSet.Attach(entity);
        context.Entry(entity).State = EntityState.Modified;
    }

    public virtual void Delete(int id)
    {
        T entityToDelete = dbSet.Find(id);
        Delete(entityToDelete);
    }

    public virtual void Delete(T entityToDelete)
    {
        if (context.Entry(entityToDelete).State == EntityState.Detached)
        {
            dbSet.Attach(entityToDelete);
        }
        dbSet.Remove(entityToDelete);
    }

    public virtual void Save()
    {
        context.SaveChanges();
    }
}
```

### **7.3 Unit of Work Pattern**

```csharp
public class UnitOfWork : IDisposable
{
    private DatSanBongDaEntities context = new DatSanBongDaEntities();
    private Repository<SanBong> sanBongRepository;
    private Repository<DonDatSan> donDatSanRepository;
    private Repository<NguoiDung> nguoiDungRepository;

    public Repository<SanBong> SanBongRepository
    {
        get
        {
            if (this.sanBongRepository == null)
            {
                this.sanBongRepository = new Repository<SanBong>(context);
            }
            return sanBongRepository;
        }
    }

    public Repository<DonDatSan> DonDatSanRepository
    {
        get
        {
            if (this.donDatSanRepository == null)
            {
                this.donDatSanRepository = new Repository<DonDatSan>(context);
            }
            return donDatSanRepository;
        }
    }

    public Repository<NguoiDung> NguoiDungRepository
    {
        get
        {
            if (this.nguoiDungRepository == null)
            {
                this.nguoiDungRepository = new Repository<NguoiDung>(context);
            }
            return nguoiDungRepository;
        }
    }

    public void Save()
    {
        context.SaveChanges();
    }

    private bool disposed = false;

    protected virtual void Dispose(bool disposing)
    {
        if (!this.disposed)
        {
            if (disposing)
            {
                context.Dispose();
            }
        }
        this.disposed = true;
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

---

## 📋 **BƯỚC 8: MIGRATION TỪNG BƯỚC**

### **8.1 Checklist Migration**

- [ ] **Database Schema**: Tạo database với SQL script
- [ ] **Entity Model**: Generate EDMX từ database
- [ ] **Partial Classes**: Thêm validation attributes
- [ ] **Update Controllers**: Thay đổi từ ApplicationDbContext sang DatSanBongDaEntities
- [ ] **Update Helpers**: Cập nhật các helper classes
- [ ] **Test**: Kiểm tra tất cả chức năng
- [ ] **Performance**: Tối ưu queries với Include()

### **8.2 Những điểm cần lưu ý**

1. **Navigation Properties**: Kiểm tra các relationships được tạo đúng
2. **Lazy Loading**: Quyết định enable/disable lazy loading
3. **Include()**: Sử dụng eager loading để tránh N+1 queries
4. **Connection String**: Đảm bảo connection string đúng
5. **Validation**: Sử dụng MetadataType cho validation
6. **Custom Logic**: Thêm business logic vào partial classes

---

## 🎯 **KẾT LUẬN**

### **✅ Ưu điểm của Database First:**
- **Kiểm soát database** tốt hơn với DBA
- **Performance** được tối ưu từ database level
- **Enterprise-ready** cho các dự án lớn
- **Team collaboration** tốt hơn với database specialists

### **✅ Nhược điểm:**
- **Phức tạp hơn** khi thay đổi schema
- **Regenerate models** khi database thay đổi
- **Validation** phải làm riêng với partial classes

### **🚀 Recommendation:**
Database First **phù hợp** với:
- Dự án **enterprise** lớn
- Team có **DBA chuyên nghiệp**
- Database **đã tồn tại** trước
- Cần **performance** cao và **kiểm soát** tốt

**Website đặt sân bóng đá của bạn sẽ hoạt động hoàn hảo với cả hai approaches! 🏆**