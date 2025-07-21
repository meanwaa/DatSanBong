# Lộ trình phát triển Website đặt sân bóng đá - ASP.NET MVC 5

[Previous content remains...]

## Giai đoạn 11: Xử lý thanh toán (Tiếp tục)

### Bước 11.1: Payment Controller (Tiếp tục)
```csharp
        return Json(new { 
            success = true, 
            message = isPaid ? "Xác nhận thanh toán thành công" : "Hủy xác nhận thanh toán"
        });
    }
    
    // Webhook cho VNPay
    [HttpPost]
    public ActionResult VNPayReturn()
    {
        var vnpayData = Request.QueryString;
        var bookingId = int.Parse(vnpayData["vnp_TxnRef"]);
        var responseCode = vnpayData["vnp_ResponseCode"];
        
        var booking = db.DonDatSans.Find(bookingId);
        if (booking == null)
            return View("PaymentError");
            
        if (responseCode == "00") // Thành công
        {
            booking.TTThanhToan = true;
            booking.IDstatus = 2; // Đã duyệt
            booking.admin_notes += $"\nThanh toán VNPay thành công - {DateTime.Now:dd/MM/yyyy HH:mm}";
            
            // Lưu thông tin giao dịch
            var transaction = new PaymentTransaction
            {
                IDBooking = bookingId,
                TransactionId = vnpayData["vnp_TransactionNo"],
                Amount = booking.TongTien,
                PaymentMethod = "VNPay",
                Status = "Success",
                ResponseCode = responseCode,
                CreatedAt = DateTime.Now
            };
            
            db.PaymentTransactions.Add(transaction);
            db.SaveChanges();
            
            NotificationHelper.SendNotification(booking.IDuser, 
                "Thanh toán thành công", 
                $"Thanh toán cho đơn đặt sân {booking.SanBong.TenSanBong} đã thành công");
                
            return View("PaymentSuccess", booking);
        }
        else
        {
            booking.admin_notes += $"\nThanh toán VNPay thất bại - {DateTime.Now:dd/MM/yyyy HH:mm} - Mã lỗi: {responseCode}";
            db.SaveChanges();
            
            return View("PaymentError", booking);
        }
    }
}
```

### Bước 11.2: Payment Transaction Model
```csharp
public class PaymentTransaction
{
    public int IDTransaction { get; set; }
    public int IDBooking { get; set; }
    public string TransactionId { get; set; }
    public decimal Amount { get; set; }
    public string PaymentMethod { get; set; }
    public string Status { get; set; }
    public string ResponseCode { get; set; }
    public DateTime CreatedAt { get; set; }
    public string Notes { get; set; }
    
    public virtual DonDatSan DonDatSan { get; set; }
}
```

## Giai đoạn 12: Hệ thống báo cáo nâng cao

### Bước 12.1: Advanced Report Controller
```csharp
[CustomAuthorize(AllowedRoles = new[] { "Admin", "Staff" })]
public class AdvancedReportController : Controller
{
    private ApplicationDbContext db = new ApplicationDbContext();
    
    public ActionResult Dashboard()
    {
        var today = DateTime.Today;
        var thisMonth = new DateTime(today.Year, today.Month, 1);
        var lastMonth = thisMonth.AddMonths(-1);
        
        var model = new DashboardReportViewModel
        {
            // Doanh thu
            TodayRevenue = GetRevenue(today, today),
            ThisMonthRevenue = GetRevenue(thisMonth, today),
            LastMonthRevenue = GetRevenue(lastMonth, thisMonth.AddDays(-1)),
            
            // Đặt sân
            TodayBookings = GetBookingCount(today, today),
            ThisMonthBookings = GetBookingCount(thisMonth, today),
            PendingBookings = GetPendingBookingCount(),
            
            // Sân bóng
            TotalFields = db.SanBongs.Count(s => s.TrangThaiSan_),
            FieldUsageToday = GetFieldUsageToday(),
            
            // Người dùng
            TotalUsers = db.NguoiDungs.Count(u => u.trangthaiTK),
            NewUsersThisMonth = GetNewUsersCount(thisMonth),
            
            // Biểu đồ
            RevenueChart = GetRevenueChartData(30), // 30 ngày gần nhất
            BookingChart = GetBookingChartData(30),
            FieldUsageChart = GetFieldUsageChartData()
        };
        
        return View(model);
    }
    
    public ActionResult DetailedRevenue(DateTime? fromDate, DateTime? toDate, int? fieldId, string period = "daily")
    {
        fromDate = fromDate ?? DateTime.Today.AddMonths(-1);
        toDate = toDate ?? DateTime.Today;
        
        var query = db.DonDatSans
            .Where(b => b.TTThanhToan && 
                   b.NgayBooking >= fromDate && 
                   b.NgayBooking <= toDate)
            .Include(b => b.SanBong);
            
        if (fieldId.HasValue)
        {
            query = query.Where(b => b.IDSanBong == fieldId.Value);
        }
        
        var bookings = query.ToList();
        
        var model = new DetailedRevenueViewModel
        {
            FromDate = fromDate.Value,
            ToDate = toDate.Value,
            SelectedFieldId = fieldId,
            Period = period,
            TotalRevenue = bookings.Sum(b => b.TongTien),
            TotalBookings = bookings.Count,
            AverageBookingValue = bookings.Any() ? bookings.Average(b => b.TongTien) : 0,
            
            // Group theo period
            RevenueByPeriod = GroupRevenueByPeriod(bookings, period),
            RevenueByField = bookings.GroupBy(b => b.SanBong)
                .Select(g => new FieldRevenueViewModel
                {
                    FieldId = g.Key.IDSanBong,
                    FieldName = g.Key.TenSanBong,
                    Revenue = g.Sum(b => b.TongTien),
                    BookingCount = g.Count(),
                    AverageBookingValue = g.Average(b => b.TongTien)
                })
                .OrderByDescending(x => x.Revenue)
                .ToList(),
                
            RevenueByPaymentMethod = bookings
                .Where(b => b.PhuongThucThanhToan != null)
                .GroupBy(b => b.PhuongThucThanhToan.TenPhuongThuc)
                .Select(g => new PaymentMethodRevenueViewModel
                {
                    PaymentMethod = g.Key,
                    Revenue = g.Sum(b => b.TongTien),
                    BookingCount = g.Count()
                })
                .ToList()
        };
        
        ViewBag.Fields = db.SanBongs.Where(s => s.TrangThaiSan_).ToList();
        return View(model);
    }
    
    public ActionResult CustomerAnalytics()
    {
        var model = new CustomerAnalyticsViewModel
        {
            // Top khách hàng theo doanh thu
            TopCustomersByRevenue = db.NguoiDungs
                .Where(u => u.Role.Role_name == "Customer")
                .Select(u => new CustomerRevenueViewModel
                {
                    UserId = u.IDuser,
                    FullName = u.fullname,
                    Email = u.email,
                    TotalRevenue = u.DonDatSans.Where(b => b.TTThanhToan).Sum(b => b.TongTien),
                    TotalBookings = u.DonDatSans.Count(),
                    LastBookingDate = u.DonDatSans.OrderByDescending(b => b.NgayBooking).Select(b => b.NgayBooking).FirstOrDefault()
                })
                .Where(c => c.TotalRevenue > 0)
                .OrderByDescending(c => c.TotalRevenue)
                .Take(20)
                .ToList(),
                
            // Phân tích khách hàng mới vs cũ
            NewCustomersThisMonth = GetNewCustomersThisMonth(),
            ReturningCustomersThisMonth = GetReturningCustomersThisMonth(),
            
            // Phân tích theo độ tuổi, giới tính (nếu có)
            CustomersByRegion = GetCustomersByRegion(),
            
            // Khách hàng không hoạt động
            InactiveCustomers = GetInactiveCustomers(90) // 90 ngày không đặt sân
        };
        
        return View(model);
    }
    
    public ActionResult FieldPerformance()
    {
        var today = DateTime.Today;
        var thisMonth = new DateTime(today.Year, today.Month, 1);
        
        var model = new FieldPerformanceViewModel
        {
            FieldPerformances = db.SanBongs
                .Where(s => s.TrangThaiSan_)
                .Select(f => new FieldPerformanceDetailViewModel
                {
                    FieldId = f.IDSanBong,
                    FieldName = f.TenSanBong,
                    FieldType = f.LoaiSanBong.LoaiSan,
                    PricePerHour = f.GiaThue,
                    
                    // Thống kê tháng này
                    BookingsThisMonth = f.DonDatSans.Count(b => b.NgayBooking >= thisMonth),
                    RevenueThisMonth = f.DonDatSans.Where(b => b.NgayBooking >= thisMonth && b.TTThanhToan).Sum(b => b.TongTien),
                    
                    // Đánh giá
                    AverageRating = f.AverageDanhGia,
                    TotalReviews = f.TongLuotDanhGia,
                    
                    // Tỷ lệ sử dụng
                    UtilizationRate = CalculateUtilizationRate(f.IDSanBong, thisMonth, today),
                    
                    // Peak hours
                    PeakHours = GetPeakHours(f.IDSanBong),
                    
                    // Khách hàng trung thành
                    RegularCustomers = GetRegularCustomers(f.IDSanBong)
                })
                .OrderByDescending(f => f.RevenueThisMonth)
                .ToList()
        };
        
        return View(model);
    }
    
    private decimal CalculateUtilizationRate(int fieldId, DateTime fromDate, DateTime toDate)
    {
        var totalDays = (toDate - fromDate).Days + 1;
        var operatingHoursPerDay = 14; // 6AM - 8PM = 14 hours
        var totalAvailableHours = totalDays * operatingHoursPerDay;
        
        var bookedHours = db.DonDatSans
            .Where(b => b.IDSanBong == fieldId && 
                   b.NgayBooking >= fromDate && 
                   b.NgayBooking <= toDate &&
                   (b.IDstatus == 2 || b.IDstatus == 3 || b.IDstatus == 4)) // Approved, Playing, Completed
            .ToList()
            .Sum(b => (b.end_time - b.start_time).TotalHours);
            
        return totalAvailableHours > 0 ? (decimal)(bookedHours / totalAvailableHours * 100) : 0;
    }
    
    // API endpoints cho AJAX
    [HttpGet]
    public ActionResult GetRevenueData(DateTime fromDate, DateTime toDate, string period = "daily")
    {
        var data = GetRevenueChartData(fromDate, toDate, period);
        return Json(data, JsonRequestBehavior.AllowGet);
    }
    
    [HttpGet]
    public ActionResult GetBookingStatusData()
    {
        var data = db.TrangThaiDonDats
            .Select(s => new
            {
                label = s.TenTrangThai,
                value = s.DonDatSans.Count(),
                color = GetStatusColor(s.TenTrangThai)
            })
            .ToList();
            
        return Json(data, JsonRequestBehavior.AllowGet);
    }
    
    [HttpGet]
    public ActionResult ExportRevenue(DateTime fromDate, DateTime toDate, string format = "excel")
    {
        var data = GetRevenueReportData(fromDate, toDate);
        
        if (format.ToLower() == "excel")
        {
            return ExportToExcel(data, "BaoCaoDoanhThu");
        }
        else if (format.ToLower() == "pdf")
        {
            return ExportToPdf(data, "BaoCaoDoanhThu");
        }
        
        return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
    }
}
```

## Giai đoạn 13: Tính năng tìm kiếm và lọc nâng cao

### Bước 13.1: Advanced Search Controller
```csharp
public class SearchController : Controller
{
    private ApplicationDbContext db = new ApplicationDbContext();
    
    public ActionResult Index(SearchFilterViewModel filter, int page = 1)
    {
        var query = db.SanBongs.Where(s => s.TrangThaiSan_).Include(s => s.LoaiSanBong);
        
        // Tìm kiếm theo từ khóa
        if (!string.IsNullOrEmpty(filter.Keyword))
        {
            query = query.Where(s => s.TenSanBong.Contains(filter.Keyword) || 
                               s.DiaChi.Contains(filter.Keyword) ||
                               s.MoTaSan.Contains(filter.Keyword));
        }
        
        // Lọc theo loại sân
        if (filter.FieldTypeIds != null && filter.FieldTypeIds.Any())
        {
            query = query.Where(s => filter.FieldTypeIds.Contains(s.IDLoaiSan));
        }
        
        // Lọc theo giá
        if (filter.MinPrice.HasValue)
        {
            query = query.Where(s => s.GiaThue >= filter.MinPrice.Value);
        }
        if (filter.MaxPrice.HasValue)
        {
            query = query.Where(s => s.GiaThue <= filter.MaxPrice.Value);
        }
        
        // Lọc theo đánh giá
        if (filter.MinRating.HasValue)
        {
            query = query.Where(s => s.AverageDanhGia >= filter.MinRating.Value);
        }
        
        // Lọc theo tiện ích
        if (filter.AmenityIds != null && filter.AmenityIds.Any())
        {
            query = query.Where(s => s.TienIchs.Any(t => filter.AmenityIds.Contains(t.IDTienIch)));
        }
        
        // Lọc theo khu vực
        if (!string.IsNullOrEmpty(filter.District))
        {
            query = query.Where(s => s.DiaChi.Contains(filter.District));
        }
        
        // Lọc theo thời gian trống (nếu có)
        if (filter.Date.HasValue && filter.StartTime.HasValue && filter.EndTime.HasValue)
        {
            var unavailableFieldIds = db.DonDatSans
                .Where(b => b.NgayBooking.Date == filter.Date.Value.Date &&
                       b.start_time < filter.EndTime.Value &&
                       b.end_time > filter.StartTime.Value &&
                       (b.IDstatus == 1 || b.IDstatus == 2 || b.IDstatus == 3)) // Pending, Approved, Playing
                .Select(b => b.IDSanBong)
                .ToList();
                
            query = query.Where(s => !unavailableFieldIds.Contains(s.IDSanBong));
        }
        
        // Sắp xếp
        switch (filter.SortBy?.ToLower())
        {
            case "price_asc":
                query = query.OrderBy(s => s.GiaThue);
                break;
            case "price_desc":
                query = query.OrderByDescending(s => s.GiaThue);
                break;
            case "rating":
                query = query.OrderByDescending(s => s.AverageDanhGia);
                break;
            case "name":
                query = query.OrderBy(s => s.TenSanBong);
                break;
            default:
                query = query.OrderByDescending(s => s.AverageDanhGia).ThenBy(s => s.GiaThue);
                break;
        }
        
        var results = query.ToPagedList(page, 12);
        
        // Prepare filter data for view
        ViewBag.FieldTypes = db.LoaiSanBongs.ToList();
        ViewBag.Amenities = db.TienIchSanBongs.ToList();
        ViewBag.Districts = GetDistrictList();
        ViewBag.CurrentFilter = filter;
        
        return View(results);
    }
    
    [HttpGet]
    public ActionResult QuickSearch(string term)
    {
        var suggestions = db.SanBongs
            .Where(s => s.TrangThaiSan_ && 
                   (s.TenSanBong.Contains(term) || s.DiaChi.Contains(term)))
            .Take(10)
            .Select(s => new
            {
                id = s.IDSanBong,
                label = s.TenSanBong,
                address = s.DiaChi,
                price = s.GiaThue,
                rating = s.AverageDanhGia
            })
            .ToList();
            
        return Json(suggestions, JsonRequestBehavior.AllowGet);
    }
    
    [HttpGet]
    public ActionResult GetAvailableTimeSlots(int fieldId, DateTime date)
    {
        var timeSlots = ScheduleHelper.GetAvailableTimeSlots(fieldId, date);
        return Json(timeSlots, JsonRequestBehavior.AllowGet);
    }
    
    [HttpGet]
    public ActionResult GetNearbyFields(double latitude, double longitude, double radiusKm = 5)
    {
        // Giả sử bạn có tọa độ GPS trong database
        var nearbyFields = db.SanBongs
            .Where(s => s.TrangThaiSan_)
            .ToList() // Get all fields first
            .Where(s => CalculateDistance(latitude, longitude, s.Latitude ?? 0, s.Longitude ?? 0) <= radiusKm)
            .Select(s => new
            {
                id = s.IDSanBong,
                name = s.TenSanBong,
                address = s.DiaChi,
                price = s.GiaThue,
                rating = s.AverageDanhGia,
                distance = CalculateDistance(latitude, longitude, s.Latitude ?? 0, s.Longitude ?? 0),
                image = s.AnhSan
            })
            .OrderBy(s => s.distance)
            .ToList();
            
        return Json(nearbyFields, JsonRequestBehavior.AllowGet);
    }
    
    private double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        // Haversine formula để tính khoảng cách
        var R = 6371; // Radius of the Earth in km
        var dLat = ToRadians(lat2 - lat1);
        var dLon = ToRadians(lon2 - lon1);
        var a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                Math.Cos(ToRadians(lat1)) * Math.Cos(ToRadians(lat2)) *
                Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        var c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return R * c;
    }
    
    private double ToRadians(double degrees)
    {
        return degrees * (Math.PI / 180);
    }
}
```

### Bước 13.2: Search Filter View Model
```csharp
public class SearchFilterViewModel
{
    public string Keyword { get; set; }
    public List<int> FieldTypeIds { get; set; }
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public decimal? MinRating { get; set; }
    public List<int> AmenityIds { get; set; }
    public string District { get; set; }
    public DateTime? Date { get; set; }
    public TimeSpan? StartTime { get; set; }
    public TimeSpan? EndTime { get; set; }
    public string SortBy { get; set; }
    public double? Latitude { get; set; }
    public double? Longitude { get; set; }
    public double? RadiusKm { get; set; }
}
```

## Giai đoạn 14: Mobile Responsive và PWA

### Bước 14.1: Responsive CSS
```css
/* Content/Site.css */

/* Mobile First Approach */
.hero-section {
    padding: 2rem 0;
}

.hero-section h1 {
    font-size: 2rem;
}

.card-img-top {
    height: 200px;
    object-fit: cover;
}

/* Booking form enhancements */
.booking-form {
    background: #f8f9fa;
    border-radius: 15px;
    padding: 2rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.time-slot {
    border: 2px solid #e9ecef;
    border-radius: 8px;
    padding: 0.5rem 1rem;
    margin: 0.25rem;
    cursor: pointer;
    transition: all 0.3s ease;
}

.time-slot.available:hover {
    border-color: #28a745;
    background-color: #f8fff9;
}

.time-slot.selected {
    border-color: #28a745;
    background-color: #28a745;
    color: white;
}

.time-slot.unavailable {
    background-color: #f8d7da;
    border-color: #f5c6cb;
    cursor: not-allowed;
    opacity: 0.6;
}

/* Search filters */
.filter-section {
    background: white;
    border-radius: 10px;
    padding: 1.5rem;
    margin-bottom: 2rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.price-range-slider {
    margin: 1rem 0;
}

/* Notification styles */
.notification-bell {
    position: relative;
}

.notification-badge {
    position: absolute;
    top: -8px;
    right: -8px;
    background: #dc3545;
    color: white;
    border-radius: 50%;
    width: 20px;
    height: 20px;
    font-size: 0.75rem;
    display: flex;
    align-items: center;
    justify-content: center;
}

.notification-dropdown {
    width: 350px;
    max-height: 400px;
    overflow-y: auto;
}

.notification-item {
    border-bottom: 1px solid #e9ecef;
    padding: 1rem;
    transition: background-color 0.2s;
}

.notification-item:hover {
    background-color: #f8f9fa;
}

.notification-item.unread {
    background-color: #e7f3ff;
    border-left: 4px solid #007bff;
}

/* Tablet styles */
@media (min-width: 768px) {
    .hero-section {
        padding: 4rem 0;
    }
    
    .hero-section h1 {
        font-size: 3rem;
    }
    
    .card-img-top {
        height: 250px;
    }
}

/* Desktop styles */
@media (min-width: 1200px) {
    .hero-section {
        padding: 6rem 0;
    }
    
    .hero-section h1 {
        font-size: 3.5rem;
    }
}

/* Print styles */
@media print {
    .navbar, .footer, .btn, .pagination {
        display: none !important;
    }
    
    .container {
        width: 100% !important;
        max-width: none !important;
    }
}
```

### Bước 14.2: PWA Manifest
```json
// wwwroot/manifest.json
{
    "name": "Football Field Booking",
    "short_name": "FieldBook",
    "description": "Đặt sân bóng đá trực tuyến",
    "start_url": "/",
    "display": "standalone",
    "background_color": "#ffffff",
    "theme_color": "#28a745",
    "orientation": "portrait",
    "icons": [
        {
            "src": "/Images/icons/icon-72x72.png",
            "sizes": "72x72",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-96x96.png",
            "sizes": "96x96",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-128x128.png",
            "sizes": "128x128",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-144x144.png",
            "sizes": "144x144",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-152x152.png",
            "sizes": "152x152",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-192x192.png",
            "sizes": "192x192",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-384x384.png",
            "sizes": "384x384",
            "type": "image/png"
        },
        {
            "src": "/Images/icons/icon-512x512.png",
            "sizes": "512x512",
            "type": "image/png"
        }
    ]
}
```

### Bước 14.3: Service Worker
```javascript
// wwwroot/sw.js
const CACHE_NAME = 'football-field-v1';
const urlsToCache = [
    '/',
    '/Content/Site.css',
    '/Scripts/jquery-3.6.0.min.js',
    '/Scripts/bootstrap.min.js',
    '/Images/default-field.jpg',
    '/manifest.json'
];

self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then((cache) => {
                return cache.addAll(urlsToCache);
            })
    );
});

self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request)
            .then((response) => {
                // Return cached version or fetch from network
                return response || fetch(event.request);
            })
    );
});

// Push notification handling
self.addEventListener('push', (event) => {
    const options = {
        body: event.data ? event.data.text() : 'Bạn có thông báo mới',
        icon: '/Images/icons/icon-192x192.png',
        badge: '/Images/icons/icon-72x72.png',
        vibrate: [200, 100, 200],
        data: {
            dateOfArrival: Date.now(),
            primaryKey: '2'
        },
        actions: [
            {
                action: 'explore',
                title: 'Xem chi tiết',
                icon: '/Images/icons/checkmark.png'
            },
            {
                action: 'close',
                title: 'Đóng',
                icon: '/Images/icons/xmark.png'
            }
        ]
    };

    event.waitUntil(
        self.registration.showNotification('Football Field Booking', options)
    );
});

self.addEventListener('notificationclick', (event) => {
    event.notification.close();

    if (event.action === 'explore') {
        event.waitUntil(
            clients.openWindow('/Notification')
        );
    }
});
```

## Giai đoạn 15: Tính năng bảo mật và tối ưu hóa

### Bước 15.1: Security Enhancements
```csharp
// Helpers/SecurityHelper.cs
public class SecurityHelper
{
    public static string HashPassword(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password);
    }
    
    public static bool VerifyPassword(string password, string hash)
    {
        return BCrypt.Net.BCrypt.Verify(password, hash);
    }
    
    public static string GenerateRandomToken(int length = 32)
    {
        const string chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        var random = new Random();
        return new string(Enumerable.Repeat(chars, length)
            .Select(s => s[random.Next(s.Length)]).ToArray());
    }
    
    public static bool IsValidEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }
    
    public static string SanitizeHtml(string input)
    {
        if (string.IsNullOrEmpty(input))
            return string.Empty;
            
        // Remove potentially dangerous tags and scripts
        var sanitized = input;
        var dangerousTags = new[] { "script", "iframe", "object", "embed", "link", "style" };
        
        foreach (var tag in dangerousTags)
        {
            sanitized = Regex.Replace(sanitized, $"<{tag}[^>]*>.*?</{tag}>", "", RegexOptions.IgnoreCase);
            sanitized = Regex.Replace(sanitized, $"<{tag}[^>]*/>", "", RegexOptions.IgnoreCase);
        }
        
        return sanitized;
    }
}
```

### Bước 15.2: Rate Limiting
```csharp
// Attributes/RateLimitAttribute.cs
public class RateLimitAttribute : ActionFilterAttribute
{
    private readonly int _maxRequests;
    private readonly int _timeWindowSeconds;
    private readonly string _cacheKeyPrefix;
    
    public RateLimitAttribute(int maxRequests = 100, int timeWindowSeconds = 3600, string cacheKeyPrefix = "RateLimit")
    {
        _maxRequests = maxRequests;
        _timeWindowSeconds = timeWindowSeconds;
        _cacheKeyPrefix = cacheKeyPrefix;
    }
    
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        var clientId = GetClientIdentifier(filterContext.HttpContext);
        var cacheKey = $"{_cacheKeyPrefix}_{clientId}";
        
        var cache = System.Web.HttpRuntime.Cache;
        var requestCount = (int?)cache[cacheKey] ?? 0;
        
        if (requestCount >= _maxRequests)
        {
            filterContext.Result = new HttpStatusCodeResult(429, "Too Many Requests");
            return;
        }
        
        cache.Insert(cacheKey, requestCount + 1, null, 
            DateTime.Now.AddSeconds(_timeWindowSeconds), 
            System.Web.Caching.Cache.NoSlidingExpiration);
            
        base.OnActionExecuting(filterContext);
    }
    
    private string GetClientIdentifier(HttpContextBase context)
    {
        // Combine IP and User Agent for better identification
        var ip = context.Request.UserHostAddress;
        var userAgent = context.Request.UserAgent ?? "";
        return $"{ip}_{userAgent.GetHashCode()}";
    }
}
```

### Bước 15.3: Database Optimization
```csharp
// Repositories/IFieldRepository.cs
public interface IFieldRepository
{
    Task<IPagedList<SanBong>> GetFieldsAsync(SearchFilterViewModel filter, int page, int pageSize);
    Task<SanBong> GetFieldByIdAsync(int id);
    Task<List<SanBong>> GetFeaturedFieldsAsync(int count);
    Task<bool> IsFieldAvailableAsync(int fieldId, DateTime date, TimeSpan startTime, TimeSpan endTime);
}

// Repositories/FieldRepository.cs
public class FieldRepository : IFieldRepository
{
    private readonly ApplicationDbContext _context;
    
    public FieldRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<IPagedList<SanBong>> GetFieldsAsync(SearchFilterViewModel filter, int page, int pageSize)
    {
        var query = _context.SanBongs
            .Where(s => s.TrangThaiSan_)
            .Include(s => s.LoaiSanBong)
            .Include(s => s.TienIchs);
        
        // Apply filters efficiently
        if (!string.IsNullOrEmpty(filter.Keyword))
        {
            query = query.Where(s => s.TenSanBong.Contains(filter.Keyword) || 
                               s.DiaChi.Contains(filter.Keyword));
        }
        
        if (filter.FieldTypeIds?.Any() == true)
        {
            query = query.Where(s => filter.FieldTypeIds.Contains(s.IDLoaiSan));
        }
        
        if (filter.MinPrice.HasValue)
        {
            query = query.Where(s => s.GiaThue >= filter.MinPrice.Value);
        }
        
        if (filter.MaxPrice.HasValue)
        {
            query = query.Where(s => s.GiaThue <= filter.MaxPrice.Value);
        }
        
        // Optimize sorting
        switch (filter.SortBy?.ToLower())
        {
            case "price_asc":
                query = query.OrderBy(s => s.GiaThue);
                break;
            case "price_desc":
                query = query.OrderByDescending(s => s.GiaThue);
                break;
            case "rating":
                query = query.OrderByDescending(s => s.AverageDanhGia);
                break;
            default:
                query = query.OrderByDescending(s => s.AverageDanhGia).ThenBy(s => s.GiaThue);
                break;
        }
        
        return await query.ToPagedListAsync(page, pageSize);
    }
    
    public async Task<SanBong> GetFieldByIdAsync(int id)
    {
        return await _context.SanBongs
            .Include(s => s.LoaiSanBong)
            .Include(s => s.TienIchs)
            .Include(s => s.Reviews.Select(r => r.NguoiDung))
            .FirstOrDefaultAsync(s => s.IDSanBong == id);
    }
    
    public async Task<List<SanBong>> GetFeaturedFieldsAsync(int count)
    {
        return await _context.SanBongs
            .Where(s => s.TrangThaiSan_)
            .OrderByDescending(s => s.AverageDanhGia)
            .ThenByDescending(s => s.TongLuotDanhGia)
            .Take(count)
            .ToListAsync();
    }
    
    public async Task<bool> IsFieldAvailableAsync(int fieldId, DateTime date, TimeSpan startTime, TimeSpan endTime)
    {
        var conflictingBooking = await _context.DonDatSans
            .AnyAsync(b => b.IDSanBong == fieldId &&
                      b.NgayBooking.Date == date.Date &&
                      b.start_time < endTime &&
                      b.end_time > startTime &&
                      (b.IDstatus == 1 || b.IDstatus == 2 || b.IDstatus == 3));
                      
        return !conflictingBooking;
    }
}
```

## Giai đoạn 16: Testing và Deployment

### Bước 16.1: Unit Testing
```csharp
// Tests/Controllers/BookingControllerTests.cs
[TestClass]
public class BookingControllerTests
{
    private Mock<ApplicationDbContext> _mockContext;
    private BookingController _controller;
    
    [TestInitialize]
    public void Setup()
    {
        _mockContext = new Mock<ApplicationDbContext>();
        _controller = new BookingController(_mockContext.Object);
        
        // Setup mock user
        var mockIdentity = new Mock<IIdentity>();
        mockIdentity.Setup(x => x.IsAuthenticated).Returns(true);
        mockIdentity.Setup(x => x.Name).Returns("testuser");
        
        var mockPrincipal = new Mock<IPrincipal>();
        mockPrincipal.Setup(x => x.Identity).Returns(mockIdentity.Object);
        
        _controller.ControllerContext = new ControllerContext
        {
            HttpContext = new Mock<HttpContextBase>().Object
        };
        _controller.ControllerContext.HttpContext.User = mockPrincipal.Object;
    }
    
    [TestMethod]
    public void Book_ValidBooking_ReturnsSuccessView()
    {
        // Arrange
        var model = new BookingViewModel
        {
            IDSanBong = 1,
            NgayBooking = DateTime.Today.AddDays(1),
            start_time = new TimeSpan(10, 0, 0),
            end_time = new TimeSpan(12, 0, 0)
        };
        
        // Mock field availability check
        _mockContext.Setup(x => x.DonDatSans.Any(It.IsAny<Expression<Func<DonDatSan, bool>>>()))
                   .Returns(false);
        
        // Act
        var result = _controller.Book(model) as ViewResult;
        
        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual("BookingSuccess", result.ViewName);
    }
    
    [TestMethod]
    public void Book_ConflictingTime_ReturnsErrorMessage()
    {
        // Arrange
        var model = new BookingViewModel
        {
            IDSanBong = 1,
            NgayBooking = DateTime.Today.AddDays(1),
            start_time = new TimeSpan(10, 0, 0),
            end_time = new TimeSpan(12, 0, 0)
        };
        
        // Mock conflicting booking
        _mockContext.Setup(x => x.DonDatSans.Any(It.IsAny<Expression<Func<DonDatSan, bool>>>()))
                   .Returns(true);
        
        // Act
        var result = _controller.Book(model) as ViewResult;
        
        // Assert
        Assert.IsNotNull(result);
        Assert.IsTrue(_controller.ModelState.ContainsKey(""));
        Assert.AreEqual("Khung giờ này đã được đặt", _controller.ModelState[""].Errors[0].ErrorMessage);
    }
}
```

### Bước 16.2: Integration Testing
```csharp
// Tests/Integration/BookingIntegrationTests.cs
[TestClass]
public class BookingIntegrationTests
{
    private TestServer _server;
    private HttpClient _client;
    
    [TestInitialize]
    public void Setup()
    {
        var builder = new WebHostBuilder()
            .UseStartup<TestStartup>();
            
        _server = new TestServer(builder);
        _client = _server.CreateClient();
    }
    
    [TestMethod]
    public async Task GET_BookingDetails_ReturnsFieldInformation()
    {
        // Arrange
        var fieldId = 1;
        
        // Act
        var response = await _client.GetAsync($"/Booking/Details/{fieldId}");
        
        // Assert
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        Assert.IsTrue(content.Contains("TenSanBong"));
    }
    
    [TestCleanup]
    public void Cleanup()
    {
        _client?.Dispose();
        _server?.Dispose();
    }
}
```

### Bước 16.3: Deployment Configuration
```xml
<!-- Web.Release.config -->
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <connectionStrings>
    <add name="DefaultConnection"
         connectionString="Data Source=ProductionServer;Initial Catalog=DatSanBongDa_Prod;User ID=dbuser;Password=dbpassword"
         xdt:Transform="SetAttributes"
         xdt:Locator="Match(name)"/>
  </connectionStrings>
  
  <system.web>
    <compilation xdt:Transform="RemoveAttributes(debug)" />
    <customErrors mode="On" defaultRedirect="~/Error" xdt:Transform="Replace">
      <error statusCode="404" redirect="~/Error/NotFound"/>
      <error statusCode="500" redirect="~/Error/ServerError"/>
    </customErrors>
  </system.web>
  
  <system.webServer>
    <httpCompression xdt:Transform="Insert">
      <dynamicTypes>
        <add mimeType="text/*" enabled="true" />
        <add mimeType="message/*" enabled="true" />
        <add mimeType="application/javascript" enabled="true" />
        <add mimeType="application/json" enabled="true" />
        <add mimeType="*/*" enabled="false" />
      </dynamicTypes>
      <staticTypes>
        <add mimeType="text/*" enabled="true" />
        <add mimeType="message/*" enabled="true" />
        <add mimeType="application/javascript" enabled="true" />
        <add mimeType="application/json" enabled="true" />
        <add mimeType="*/*" enabled="false" />
      </staticTypes>
    </httpCompression>
    
    <urlCompression doStaticCompression="true" doDynamicCompression="true" />
    
    <staticContent>
      <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
    </staticContent>
  </system.webServer>
</configuration>
```

## Kết luận

Dự án Website đặt sân bóng đá với ASP.NET MVC 5 đã được hoàn thiện với đầy đủ các tính năng:

### Tính năng chính:
- ✅ Hệ thống xác thực và phân quyền
- ✅ Quản lý sân bóng và đặt lịch
- ✅ Hệ thống thanh toán đa dạng
- ✅ Đánh giá và phản hồi
- ✅ Thông báo real-time
- ✅ Báo cáo và thống kê
- ✅ Tìm kiếm và lọc nâng cao
- ✅ Responsive design và PWA
- ✅ Bảo mật và tối ưu hóa

### Công nghệ sử dụng:
- ASP.NET MVC 5
- Entity Framework 6
- SQL Server
- Bootstrap 5
- SignalR
- jQuery/JavaScript

### Triển khai:
1. Cấu hình IIS
2. Deploy database
3. Cấu hình SSL
4. Monitoring và backup

Website đã sẵn sàng cho việc triển khai production với khả năng mở rộng và bảo trì cao.