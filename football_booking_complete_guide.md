# Lộ trình phát triển Website đặt sân bóng đá - ASP.NET MVC 5 (Hoàn chỉnh)

## Giai đoạn 5: Phát triển giao diện với Bootstrap 5

### Bước 5.1: Layout chính
#### Views/Shared/_Layout.cshtml
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - Đặt Sân Bóng Đá</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="~/Content/Site.css" rel="stylesheet" type="text/css" />
    <link rel="manifest" href="~/manifest.json">
    <meta name="theme-color" content="#28a745">
</head>
<body>
    <!-- Navigation -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-success">
        <div class="container">
            <a class="navbar-brand fw-bold" href="@Url.Action("Index", "Home")">
                <i class="fas fa-futbol me-2"></i>FootballField
            </a>
            
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item">
                        <a class="nav-link" href="@Url.Action("Index", "Home")">
                            <i class="fas fa-home me-1"></i>Trang chủ
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="@Url.Action("Search", "Home")">
                            <i class="fas fa-search me-1"></i>Tìm sân
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="@Url.Action("About", "Home")">
                            <i class="fas fa-info-circle me-1"></i>Giới thiệu
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="@Url.Action("Contact", "Home")">
                            <i class="fas fa-phone me-1"></i>Liên hệ
                        </a>
                    </li>
                </ul>
                
                <ul class="navbar-nav">
                    @if (Request.IsAuthenticated)
                    {
                        <!-- Notification Bell -->
                        <li class="nav-item dropdown me-3">
                            <a class="nav-link position-relative" href="#" id="notificationDropdown" role="button" data-bs-toggle="dropdown">
                                <i class="fas fa-bell"></i>
                                <span class="notification-badge badge bg-danger" id="notificationCount" style="display: none;">0</span>
                            </a>
                            <ul class="dropdown-menu dropdown-menu-end notification-dropdown">
                                <li><h6 class="dropdown-header">Thông báo</h6></li>
                                <li><hr class="dropdown-divider"></li>
                                <div id="notificationList">
                                    <li><span class="dropdown-item-text text-muted">Không có thông báo mới</span></li>
                                </div>
                                <li><hr class="dropdown-divider"></li>
                                <li><a class="dropdown-item text-center" href="@Url.Action("Index", "Notification")">Xem tất cả</a></li>
                            </ul>
                        </li>
                        
                        <!-- User Menu -->
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                                <i class="fas fa-user-circle me-1"></i>@User.Identity.Name
                            </a>
                            <ul class="dropdown-menu dropdown-menu-end">
                                <li><a class="dropdown-item" href="@Url.Action("Profile", "Account")">
                                    <i class="fas fa-user me-2"></i>Thông tin cá nhân
                                </a></li>
                                <li><a class="dropdown-item" href="@Url.Action("MyBookings", "Booking")">
                                    <i class="fas fa-calendar-alt me-2"></i>Lịch đặt sân
                                </a></li>
                                @if (User.IsInRole("Admin"))
                                {
                                    <li><hr class="dropdown-divider"></li>
                                    <li><a class="dropdown-item" href="@Url.Action("Dashboard", "Admin")">
                                        <i class="fas fa-tachometer-alt me-2"></i>Quản trị
                                    </a></li>
                                }
                                else if (User.IsInRole("Staff"))
                                {
                                    <li><hr class="dropdown-divider"></li>
                                    <li><a class="dropdown-item" href="@Url.Action("Dashboard", "Staff")">
                                        <i class="fas fa-tasks me-2"></i>Quản lý
                                    </a></li>
                                }
                                <li><hr class="dropdown-divider"></li>
                                <li><a class="dropdown-item" href="@Url.Action("Logout", "Account")">
                                    <i class="fas fa-sign-out-alt me-2"></i>Đăng xuất
                                </a></li>
                            </ul>
                        </li>
                    }
                    else
                    {
                        <li class="nav-item">
                            <a class="nav-link" href="@Url.Action("Login", "Account")">
                                <i class="fas fa-sign-in-alt me-1"></i>Đăng nhập
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link btn btn-outline-light ms-2" href="@Url.Action("Register", "Account")">
                                <i class="fas fa-user-plus me-1"></i>Đăng ký
                            </a>
                        </li>
                    }
                </ul>
            </div>
        </div>
    </nav>
    
    <!-- Main Content -->
    <main>
        @if (TempData["SuccessMessage"] != null)
        {
            <div class="alert alert-success alert-dismissible fade show m-3" role="alert">
                <i class="fas fa-check-circle me-2"></i>@TempData["SuccessMessage"]
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        }
        
        @if (TempData["ErrorMessage"] != null)
        {
            <div class="alert alert-danger alert-dismissible fade show m-3" role="alert">
                <i class="fas fa-exclamation-circle me-2"></i>@TempData["ErrorMessage"]
                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
            </div>
        }
        
        @RenderBody()
    </main>
    
    <!-- Footer -->
    <footer class="bg-dark text-light mt-5 py-4">
        <div class="container">
            <div class="row">
                <div class="col-md-4">
                    <h5 class="mb-3">
                        <i class="fas fa-futbol me-2"></i>FootballField
                    </h5>
                    <p>Hệ thống đặt sân bóng đá trực tuyến hàng đầu Việt Nam. Dễ dàng, nhanh chóng, tiện lợi.</p>
                    <div>
                        <a href="#" class="text-light me-3"><i class="fab fa-facebook-f"></i></a>
                        <a href="#" class="text-light me-3"><i class="fab fa-twitter"></i></a>
                        <a href="#" class="text-light me-3"><i class="fab fa-instagram"></i></a>
                        <a href="#" class="text-light"><i class="fab fa-youtube"></i></a>
                    </div>
                </div>
                <div class="col-md-2">
                    <h6 class="mb-3">Liên kết</h6>
                    <ul class="list-unstyled">
                        <li><a href="@Url.Action("Index", "Home")" class="text-light text-decoration-none">Trang chủ</a></li>
                        <li><a href="@Url.Action("Search", "Home")" class="text-light text-decoration-none">Tìm sân</a></li>
                        <li><a href="@Url.Action("About", "Home")" class="text-light text-decoration-none">Giới thiệu</a></li>
                        <li><a href="@Url.Action("Contact", "Home")" class="text-light text-decoration-none">Liên hệ</a></li>
                    </ul>
                </div>
                <div class="col-md-3">
                    <h6 class="mb-3">Hỗ trợ</h6>
                    <ul class="list-unstyled">
                        <li><a href="#" class="text-light text-decoration-none">Câu hỏi thường gặp</a></li>
                        <li><a href="#" class="text-light text-decoration-none">Hướng dẫn đặt sân</a></li>
                        <li><a href="#" class="text-light text-decoration-none">Chính sách</a></li>
                        <li><a href="#" class="text-light text-decoration-none">Điều khoản</a></li>
                    </ul>
                </div>
                <div class="col-md-3">
                    <h6 class="mb-3">Liên hệ</h6>
                    <p><i class="fas fa-map-marker-alt me-2"></i>123 Đường ABC, Quận XYZ, TP.HCM</p>
                    <p><i class="fas fa-phone me-2"></i>0123 456 789</p>
                    <p><i class="fas fa-envelope me-2"></i>contact@footballfield.vn</p>
                </div>
            </div>
            <hr class="my-4">
            <div class="text-center">
                <p>&copy; @DateTime.Now.Year FootballField. All rights reserved.</p>
            </div>
        </div>
    </footer>
    
    <!-- Scripts -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="~/Scripts/site.js"></script>
    
    @RenderSection("scripts", required: false)
    
    <!-- PWA Install Button -->
    <button id="installBtn" class="btn btn-primary position-fixed bottom-0 end-0 m-3" style="display: none;">
        <i class="fas fa-download me-2"></i>Cài đặt App
    </button>
    
    <script>
        // PWA Install
        let deferredPrompt;
        const installBtn = document.getElementById('installBtn');
        
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installBtn.style.display = 'block';
        });
        
        installBtn.addEventListener('click', (e) => {
            installBtn.style.display = 'none';
            deferredPrompt.prompt();
            deferredPrompt.userChoice.then((result) => {
                if (result.outcome === 'accepted') {
                    console.log('User accepted the install prompt');
                }
                deferredPrompt = null;
            });
        });
        
        // Notification polling
        @if (Request.IsAuthenticated)
        {
            <text>
            function loadNotifications() {
                $.get('@Url.Action("GetUnreadCount", "Notification")', function(data) {
                    const count = data.count;
                    const badge = $('#notificationCount');
                    if (count > 0) {
                        badge.text(count).show();
                    } else {
                        badge.hide();
                    }
                });
            }
            
            // Load notifications on page load and every 30 seconds
            loadNotifications();
            setInterval(loadNotifications, 30000);
            </text>
        }
    </script>
</body>
</html>
```

### Bước 5.2: Trang chủ hoàn chỉnh
#### Views/Home/Index.cshtml
```html
@model List<SanBong>
@{
    ViewBag.Title = "Trang chủ";
}

<!-- Hero Section -->
<section class="hero-section bg-gradient-success text-white py-5">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6">
                <h1 class="display-4 fw-bold mb-4">Đặt Sân Bóng Đá Online</h1>
                <p class="lead mb-4">Tìm và đặt sân bóng đá dễ dàng, nhanh chóng với hệ thống hiện đại nhất. Hơn 1000+ sân bóng chất lượng cao đang chờ bạn.</p>
                <div class="d-grid d-md-flex gap-3">
                    <a href="@Url.Action("Search", "Home")" class="btn btn-light btn-lg">
                        <i class="fas fa-search me-2"></i>Tìm sân ngay
                    </a>
                    <a href="@Url.Action("About", "Home")" class="btn btn-outline-light btn-lg">
                        <i class="fas fa-info-circle me-2"></i>Tìm hiểu thêm
                    </a>
                </div>
            </div>
            <div class="col-lg-6 text-center">
                <img src="~/Images/hero-football.png" alt="Football" class="img-fluid" style="max-height: 400px;">
            </div>
        </div>
    </div>
</section>

<!-- Quick Stats -->
<section class="py-5 bg-light">
    <div class="container">
        <div class="row text-center">
            <div class="col-md-3 mb-4">
                <div class="card border-0 h-100">
                    <div class="card-body">
                        <div class="text-success mb-3">
                            <i class="fas fa-futbol fa-3x"></i>
                        </div>
                        <h3 class="fw-bold text-success">@ViewBag.TotalFields</h3>
                        <p class="mb-0">Sân bóng chất lượng</p>
                    </div>
                </div>
            </div>
            <div class="col-md-3 mb-4">
                <div class="card border-0 h-100">
                    <div class="card-body">
                        <div class="text-primary mb-3">
                            <i class="fas fa-calendar-check fa-3x"></i>
                        </div>
                        <h3 class="fw-bold text-primary">@ViewBag.TotalBookings</h3>
                        <p class="mb-0">Lượt đặt sân thành công</p>
                    </div>
                </div>
            </div>
            <div class="col-md-3 mb-4">
                <div class="card border-0 h-100">
                    <div class="card-body">
                        <div class="text-warning mb-3">
                            <i class="fas fa-users fa-3x"></i>
                        </div>
                        <h3 class="fw-bold text-warning">@ViewBag.TotalUsers</h3>
                        <p class="mb-0">Người dùng tin tưởng</p>
                    </div>
                </div>
            </div>
            <div class="col-md-3 mb-4">
                <div class="card border-0 h-100">
                    <div class="card-body">
                        <div class="text-info mb-3">
                            <i class="fas fa-star fa-3x"></i>
                        </div>
                        <h3 class="fw-bold text-info">4.8/5</h3>
                        <p class="mb-0">Đánh giá trung bình</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- Quick Search -->
<section class="py-5">
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-lg-8">
                <div class="card shadow-lg border-0">
                    <div class="card-body p-4">
                        <h4 class="text-center mb-4">
                            <i class="fas fa-search text-success me-2"></i>Tìm sân nhanh
                        </h4>
                        <form action="@Url.Action("Search", "Home")" method="get">
                            <div class="row g-3">
                                <div class="col-md-6">
                                    <input type="text" name="keyword" class="form-control form-control-lg" 
                                           placeholder="Tên sân hoặc địa chỉ...">
                                </div>
                                <div class="col-md-3">
                                    <select name="loaiSan" class="form-select form-select-lg">
                                        <option value="">Tất cả loại sân</option>
                                        <option value="1">Sân 5 người</option>
                                        <option value="2">Sân 7 người</option>
                                        <option value="3">Sân 11 người</option>
                                    </select>
                                </div>
                                <div class="col-md-3">
                                    <button type="submit" class="btn btn-success btn-lg w-100">
                                        <i class="fas fa-search me-2"></i>Tìm kiếm
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- Featured Fields -->
<section class="py-5 bg-light">
    <div class="container">
        <div class="text-center mb-5">
            <h2 class="fw-bold">Sân Nổi Bật</h2>
            <p class="lead text-muted">Những sân bóng được đánh giá cao nhất</p>
        </div>
        
        <div class="row">
            @foreach (var field in Model)
            {
                <div class="col-lg-4 col-md-6 mb-4">
                    <div class="card h-100 shadow-sm border-0 card-hover">
                        <div class="position-relative">
                            <img src="@(string.IsNullOrEmpty(field.AnhSan) ? "/Images/default-field.jpg" : field.AnhSan)" 
                                 class="card-img-top" style="height: 250px; object-fit: cover;" 
                                 alt="@field.TenSanBong">
                            <div class="position-absolute top-0 end-0 m-3">
                                <span class="badge bg-success fs-6">@field.LoaiSanBong.LoaiSan</span>
                            </div>
                        </div>
                        
                        <div class="card-body d-flex flex-column">
                            <h5 class="card-title fw-bold">@field.TenSanBong</h5>
                            <p class="card-text text-muted mb-2">
                                <i class="fas fa-map-marker-alt me-1"></i>@field.DiaChi
                            </p>
                            
                            <div class="mb-2">
                                <div class="d-flex align-items-center">
                                    <div class="text-warning me-2">
                                        @for (int i = 1; i <= 5; i++)
                                        {
                                            <i class="fa@(i <= field.AverageDanhGia ? "s" : "r") fa-star"></i>
                                        }
                                    </div>
                                    <small class="text-muted">(@field.TongLuotDanhGia đánh giá)</small>
                                </div>
                            </div>
                            
                            <p class="card-text">
                                <span class="h5 text-success fw-bold">@field.GiaThue.ToString("N0") VNĐ</span>
                                <small class="text-muted">/giờ</small>
                            </p>
                            
                            <div class="mt-auto">
                                <div class="d-grid gap-2">
                                    <a href="@Url.Action("Details", "Booking", new { id = field.IDSanBong })" 
                                       class="btn btn-outline-success">
                                        <i class="fas fa-info-circle me-2"></i>Xem chi tiết
                                    </a>
                                    <a href="@Url.Action("Book", "Booking", new { id = field.IDSanBong })" 
                                       class="btn btn-success">
                                        <i class="fas fa-calendar-plus me-2"></i>Đặt sân ngay
                                    </a>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            }
        </div>
        
        <div class="text-center mt-4">
            <a href="@Url.Action("Search", "Home")" class="btn btn-outline-success btn-lg">
                <i class="fas fa-eye me-2"></i>Xem tất cả sân bóng
            </a>
        </div>
    </div>
</section>

<!-- Features -->
<section class="py-5">
    <div class="container">
        <div class="text-center mb-5">
            <h2 class="fw-bold">Tại Sao Chọn Chúng Tôi?</h2>
            <p class="lead text-muted">Những lợi ích vượt trội khi sử dụng dịch vụ của chúng tôi</p>
        </div>
        
        <div class="row">
            <div class="col-md-4 mb-4">
                <div class="text-center">
                    <div class="bg-success text-white rounded-circle d-inline-flex align-items-center justify-content-center mb-3" 
                         style="width: 80px; height: 80px;">
                        <i class="fas fa-clock fa-2x"></i>
                    </div>
                    <h4>Đặt Sân 24/7</h4>
                    <p class="text-muted">Hệ thống hoạt động 24/7, bạn có thể đặt sân bất cứ lúc nào</p>
                </div>
            </div>
            <div class="col-md-4 mb-4">
                <div class="text-center">
                    <div class="bg-primary text-white rounded-circle d-inline-flex align-items-center justify-content-center mb-3" 
                         style="width: 80px; height: 80px;">
                        <i class="fas fa-shield-alt fa-2x"></i>
                    </div>
                    <h4>Thanh Toán An Toàn</h4>
                    <p class="text-muted">Nhiều phương thức thanh toán tiện lợi và bảo mật tuyệt đối</p>
                </div>
            </div>
            <div class="col-md-4 mb-4">
                <div class="text-center">
                    <div class="bg-warning text-white rounded-circle d-inline-flex align-items-center justify-content-center mb-3" 
                         style="width: 80px; height: 80px;">
                        <i class="fas fa-headset fa-2x"></i>
                    </div>
                    <h4>Hỗ Trợ Tận Tình</h4>
                    <p class="text-muted">Đội ngũ hỗ trợ khách hàng chuyên nghiệp, sẵn sàng giúp đỡ</p>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- CTA Section -->
<section class="py-5 bg-success text-white">
    <div class="container text-center">
        <h2 class="fw-bold mb-4">Bắt Đầu Đặt Sân Ngay Hôm Nay!</h2>
        <p class="lead mb-4">Tham gia cộng đồng hơn @ViewBag.TotalUsers người dùng đang tin tưởng sử dụng dịch vụ của chúng tôi</p>
        @if (!Request.IsAuthenticated)
        {
            <a href="@Url.Action("Register", "Account")" class="btn btn-light btn-lg me-3">
                <i class="fas fa-user-plus me-2"></i>Đăng ký ngay
            </a>
        }
        <a href="@Url.Action("Search", "Home")" class="btn btn-outline-light btn-lg">
            <i class="fas fa-search me-2"></i>Tìm sân bóng
        </a>
    </div>
</section>

@section scripts {
<style>
    .hero-section {
        background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
    }
    
    .card-hover {
        transition: transform 0.3s ease, box-shadow 0.3s ease;
    }
    
    .card-hover:hover {
        transform: translateY(-5px);
        box-shadow: 0 10px 25px rgba(0,0,0,0.15) !important;
    }
    
    .bg-gradient-success {
        background: linear-gradient(135deg, #28a745 0%, #20c997 100%);
    }
</style>
}
```

### Bước 5.3: Trang tìm kiếm
#### Views/Home/Search.cshtml
```html
@model PagedList.IPagedList<SanBong>
@{
    ViewBag.Title = "Tìm kiếm sân bóng";
}

<div class="container my-4">
    <!-- Search Header -->
    <div class="row">
        <div class="col-12">
            <h2 class="fw-bold mb-4">
                <i class="fas fa-search text-success me-2"></i>Tìm Kiếm Sân Bóng
            </h2>
        </div>
    </div>
    
    <!-- Advanced Search Filter -->
    <div class="row mb-4">
        <div class="col-12">
            <div class="card shadow-sm">
                <div class="card-header bg-light">
                    <h5 class="mb-0">
                        <i class="fas fa-filter me-2"></i>Bộ Lọc Tìm Kiếm
                        <button class="btn btn-sm btn-outline-secondary float-end" type="button" 
                                data-bs-toggle="collapse" data-bs-target="#searchFilter">
                            <i class="fas fa-chevron-down"></i>
                        </button>
                    </h5>
                </div>
                <div class="collapse show" id="searchFilter">
                    <div class="card-body">
                        <form method="get" action="@Url.Action("Search")">
                            <div class="row g-3">
                                <div class="col-md-3">
                                    <label class="form-label">Từ khóa</label>
                                    <input type="text" name="keyword" class="form-control" 
                                           value="@ViewBag.CurrentKeyword" 
                                           placeholder="Tên sân, địa chỉ...">
                                </div>
                                <div class="col-md-2">
                                    <label class="form-label">Loại sân</label>
                                    <select name="loaiSan" class="form-select">
                                        <option value="">Tất cả</option>
                                        @foreach (var loai in ViewBag.LoaiSanBongs as List<LoaiSanBong>)
                                        {
                                            <option value="@loai.IDLoaiSan" 
                                                    @(ViewBag.CurrentLoaiSan == loai.IDLoaiSan ? "selected" : "")>
                                                @loai.LoaiSan
                                            </option>
                                        }
                                    </select>
                                </div>
                                <div class="col-md-2">
                                    <label class="form-label">Giá từ (VNĐ)</label>
                                    <input type="number" name="giaMin" class="form-control" 
                                           value="@ViewBag.CurrentGiaMin" 
                                           placeholder="0" step="50000">
                                </div>
                                <div class="col-md-2">
                                    <label class="form-label">Giá đến (VNĐ)</label>
                                    <input type="number" name="giaMax" class="form-control" 
                                           value="@ViewBag.CurrentGiaMax" 
                                           placeholder="1000000" step="50000">
                                </div>
                                <div class="col-md-2">
                                    <label class="form-label">Đánh giá</label>
                                    <select name="rating" class="form-select">
                                        <option value="">Tất cả</option>
                                        <option value="4">4+ sao</option>
                                        <option value="3">3+ sao</option>
                                        <option value="2">2+ sao</option>
                                    </select>
                                </div>
                                <div class="col-md-1">
                                    <label class="form-label">&nbsp;</label>
                                    <div class="d-grid">
                                        <button type="submit" class="btn btn-success">
                                            <i class="fas fa-search"></i>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Results Summary -->
    <div class="row mb-3">
        <div class="col-md-6">
            <p class="text-muted">
                Tìm thấy <strong>@Model.TotalItemCount</strong> sân bóng
                @if (!string.IsNullOrEmpty(ViewBag.CurrentKeyword))
                {
                    <text>cho từ khóa "<strong>@ViewBag.CurrentKeyword</strong>"</text>
                }
            </p>
        </div>
        <div class="col-md-6 text-end">
            <div class="btn-group" role="group">
                <input type="radio" class="btn-check" name="sortBy" id="sortDefault" checked>
                <label class="btn btn-outline-secondary" for="sortDefault">Phổ biến</label>
                
                <input type="radio" class="btn-check" name="sortBy" id="sortPrice">
                <label class="btn btn-outline-secondary" for="sortPrice">Giá thấp</label>
                
                <input type="radio" class="btn-check" name="sortBy" id="sortRating">
                <label class="btn btn-outline-secondary" for="sortRating">Đánh giá</label>
            </div>
        </div>
    </div>
    
    <!-- Search Results -->
    <div class="row">
        @if (Model.Any())
        {
            @foreach (var field in Model)
            {
                <div class="col-lg-4 col-md-6 mb-4">
                    <div class="card h-100 shadow-sm border-0 field-card">
                        <div class="position-relative">
                            <img src="@(string.IsNullOrEmpty(field.AnhSan) ? "/Images/default-field.jpg" : field.AnhSan)" 
                                 class="card-img-top" style="height: 200px; object-fit: cover;" 
                                 alt="@field.TenSanBong">
                            <div class="position-absolute top-0 start-0 m-2">
                                <span class="badge bg-success">@field.LoaiSanBong.LoaiSan</span>
                            </div>
                            <div class="position-absolute top-0 end-0 m-2">
                                @if (field.AverageDanhGia >= 4)
                                {
                                    <span class="badge bg-warning">
                                        <i class="fas fa-star"></i> @field.AverageDanhGia.ToString("F1")
                                    </span>
                                }
                            </div>
                        </div>
                        
                        <div class="card-body d-flex flex-column">
                            <h5 class="card-title fw-bold">@field.TenSanBong</h5>
                            <p class="card-text text-muted mb-2">
                                <i class="fas fa-map-marker-alt me-1"></i>
                                @field.DiaChi
                            </p>
                            
                            <div class="mb-2">
                                <div class="text-warning">
                                    @for (int i = 1; i <= 5; i++)
                                    {
                                        <i class="fa@(i <= field.AverageDanhGia ? "s" : "r") fa-star"></i>
                                    }
                                    <small class="text-muted ms-1">(@field.TongLuotDanhGia)</small>
                                </div>
                            </div>
                            
                            @if (!string.IsNullOrEmpty(field.MoTaSan))
                            {
                                <p class="card-text small">
                                    @(field.MoTaSan.Length > 80 ? field.MoTaSan.Substring(0, 80) + "..." : field.MoTaSan)
                                </p>
                            }
                            
                            <div class="mt-auto">
                                <div class="d-flex justify-content-between align-items-center mb-2">
                                    <span class="h5 text-success fw-bold mb-0">
                                        @field.GiaThue.ToString("N0") VNĐ/giờ
                                    </span>
                                    <div>
                                        <a href="@Url.Action("Details", "Booking", new { id = field.IDSanBong })" 
                                           class="btn btn-sm btn-outline-success me-1">
                                            <i class="fas fa-info"></i>
                                        </a>
                                        <a href="@Url.Action("Book", "Booking", new { id = field.IDSanBong })" 
                                           class="btn btn-sm btn-success">
                                            <i class="fas fa-calendar-plus"></i>
                                        </a>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            }
        }
        else
        {
            <div class="col-12">
                <div class="text-center py-5">
                    <div class="mb-4">
                        <i class="fas fa-search fa-4x text-muted"></i>
                    </div>
                    <h3>Không tìm thấy sân bóng phù hợp</h3>
                    <p class="text-muted">Vui lòng thử lại với từ khóa khác hoặc điều chỉnh bộ lọc</p>
                    <a href="@Url.Action("Search")" class="btn btn-success">
                        <i class="fas fa-redo me-2"></i>Tìm kiếm lại
                    </a>
                </div>
            </div>
        }
    </div>
    
    <!-- Pagination -->
    @if (Model.PageCount > 1)
    {
        <div class="row">
            <div class="col-12">
                <nav aria-label="Search results pagination">
                    <ul class="pagination justify-content-center">
                        @if (Model.HasPreviousPage)
                        {
                            <li class="page-item">
                                <a class="page-link" href="@Url.Action("Search", new { page = Model.PageNumber - 1, keyword = ViewBag.CurrentKeyword, loaiSan = ViewBag.CurrentLoaiSan, giaMin = ViewBag.CurrentGiaMin, giaMax = ViewBag.CurrentGiaMax })">
                                    <i class="fas fa-chevron-left"></i>
                                </a>
                            </li>
                        }
                        
                        @for (int i = Math.Max(1, Model.PageNumber - 2); i <= Math.Min(Model.PageCount, Model.PageNumber + 2); i++)
                        {
                            <li class="page-item @(i == Model.PageNumber ? "active" : "")">
                                <a class="page-link" href="@Url.Action("Search", new { page = i, keyword = ViewBag.CurrentKeyword, loaiSan = ViewBag.CurrentLoaiSan, giaMin = ViewBag.CurrentGiaMin, giaMax = ViewBag.CurrentGiaMax })">
                                    @i
                                </a>
                            </li>
                        }
                        
                        @if (Model.HasNextPage)
                        {
                            <li class="page-item">
                                <a class="page-link" href="@Url.Action("Search", new { page = Model.PageNumber + 1, keyword = ViewBag.CurrentKeyword, loaiSan = ViewBag.CurrentLoaiSan, giaMin = ViewBag.CurrentGiaMin, giaMax = ViewBag.CurrentGiaMax })">
                                    <i class="fas fa-chevron-right"></i>
                                </a>
                            </li>
                        }
                    </ul>
                </nav>
            </div>
        </div>
    }
</div>

@section scripts {
<style>
    .field-card {
        transition: transform 0.2s ease, box-shadow 0.2s ease;
    }
    
    .field-card:hover {
        transform: translateY(-2px);
        box-shadow: 0 8px 25px rgba(0,0,0,0.1) !important;
    }
</style>

<script>
    // Sort functionality
    $('input[name="sortBy"]').change(function() {
        const sortBy = $(this).attr('id').replace('sort', '').toLowerCase();
        const url = new URL(window.location);
        url.searchParams.set('sortBy', sortBy);
        url.searchParams.set('page', '1');
        window.location = url.toString();
    });
</script>
}
```

## Giai đoạn 6: Phân quyền và Dashboard

### Bước 6.1: Admin Dashboard
#### Controllers/AdminController.cs
```csharp
[CustomAuthorize(AllowedRoles = new[] { "Admin" })]
public class AdminController : Controller
{
    private ApplicationDbContext db = new ApplicationDbContext();
    
    protected override void Dispose(bool disposing)
    {
        if (disposing)
        {
            db.Dispose();
        }
        base.Dispose(disposing);
    }
    
    // GET: Admin/Dashboard
    public ActionResult Dashboard()
    {
        var today = DateTime.Today;
        var thisMonth = new DateTime(today.Year, today.Month, 1);
        
        var model = new AdminDashboardViewModel
        {
            // Thống kê tổng quan
            TotalFields = db.SanBongs.Count(),
            TotalBookings = db.DonDatSans.Count(),
            TotalUsers = db.NguoiDungs.Count(),
            TotalRevenue = db.DonDatSans.Where(b => b.TTThanhToan).Sum(b => (decimal?)b.TongTien) ?? 0,
            
            // Thống kê hôm nay
            TodayBookings = db.DonDatSans.Count(b => DbFunctions.TruncateTime(b.NgayBooking) == today),
            TodayRevenue = db.DonDatSans
                .Where(b => DbFunctions.TruncateTime(b.NgayBooking) == today && b.TTThanhToan)
                .Sum(b => (decimal?)b.TongTien) ?? 0,
            
            // Thống kê tháng này
            ThisMonthBookings = db.DonDatSans.Count(b => b.NgayBooking >= thisMonth),
            ThisMonthRevenue = db.DonDatSans
                .Where(b => b.NgayBooking >= thisMonth && b.TTThanhToan)
                .Sum(b => (decimal?)b.TongTien) ?? 0,
            
            // Đơn đặt gần đây
            RecentBookings = db.DonDatSans
                .Include(b => b.NguoiDung)
                .Include(b => b.SanBong)
                .Include(b => b.TrangThaiDonDat)
                .OrderByDescending(b => b.created_at)
                .Take(10)
                .ToList(),
                
            // Top sân bóng
            TopFields = db.SanBongs
                .OrderByDescending(s => s.AverageDanhGia)
                .ThenByDescending(s => s.TongLuotDanhGia)
                .Take(5)
                .ToList(),
                
            // Pending bookings
            PendingBookings = db.DonDatSans
                .Where(b => b.IDstatus == 1)
                .Include(b => b.NguoiDung)
                .Include(b => b.SanBong)
                .OrderBy(b => b.NgayBooking)
                .Take(10)
                .ToList()
        };
        
        return View(model);
    }
    
    // GET: Admin/ManageFields
    public ActionResult ManageFields(int page = 1, string search = "")
    {
        var query = db.SanBongs.Include(s => s.LoaiSanBong).AsQueryable();
        
        if (!string.IsNullOrEmpty(search))
        {
            query = query.Where(s => s.TenSanBong.Contains(search) || s.DiaChi.Contains(search));
        }
        
        var fields = query.OrderBy(s => s.TenSanBong).ToPagedList(page, 20);
        
        ViewBag.CurrentSearch = search;
        ViewBag.LoaiSanBongs = db.LoaiSanBongs.ToList();
        ViewBag.TienIchs = db.TienIchSanBongs.ToList();
        
        return View(fields);
    }
    
    // GET: Admin/CreateField
    public ActionResult CreateField()
    {
        ViewBag.LoaiSanBongs = new SelectList(db.LoaiSanBongs, "IDLoaiSan", "LoaiSan");
        ViewBag.TienIchs = db.TienIchSanBongs.ToList();
        return View();
    }
    
    // POST: Admin/CreateField
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult CreateField(SanBong model, List<int> selectedAmenities, HttpPostedFileBase imageFile)
    {
        if (ModelState.IsValid)
        {
            // Xử lý upload hình ảnh
            if (imageFile != null && imageFile.ContentLength > 0)
            {
                var fileName = Path.GetFileName(imageFile.FileName);
                var path = Path.Combine(Server.MapPath("~/Images/Fields/"), fileName);
                imageFile.SaveAs(path);
                model.AnhSan = "/Images/Fields/" + fileName;
            }
            
            model.created_at = DateTime.Now;
            model.updated_at = DateTime.Now;
            
            db.SanBongs.Add(model);
            db.SaveChanges();
            
            // Thêm tiện ích
            if (selectedAmenities != null)
            {
                foreach (var amenityId in selectedAmenities)
                {
                    var amenity = db.TienIchSanBongs.Find(amenityId);
                    if (amenity != null)
                    {
                        model.TienIchs.Add(amenity);
                    }
                }
                db.SaveChanges();
            }
            
            TempData["SuccessMessage"] = "Thêm sân bóng thành công";
            return RedirectToAction("ManageFields");
        }
        
        ViewBag.LoaiSanBongs = new SelectList(db.LoaiSanBongs, "IDLoaiSan", "LoaiSan");
        ViewBag.TienIchs = db.TienIchSanBongs.ToList();
        return View(model);
    }
    
    // GET: Admin/EditField/5
    public ActionResult EditField(int id)
    {
        var field = db.SanBongs
            .Include(s => s.TienIchs)
            .FirstOrDefault(s => s.IDSanBong == id);
            
        if (field == null)
            return HttpNotFound();
            
        ViewBag.LoaiSanBongs = new SelectList(db.LoaiSanBongs, "IDLoaiSan", "LoaiSan", field.IDLoaiSan);
        ViewBag.TienIchs = db.TienIchSanBongs.ToList();
        ViewBag.SelectedAmenities = field.TienIchs.Select(t => t.IDTienIch).ToList();
        
        return View(field);
    }
    
    // POST: Admin/EditField/5
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult EditField(SanBong model, List<int> selectedAmenities, HttpPostedFileBase imageFile)
    {
        if (ModelState.IsValid)
        {
            var field = db.SanBongs
                .Include(s => s.TienIchs)
                .FirstOrDefault(s => s.IDSanBong == model.IDSanBong);
                
            if (field == null)
                return HttpNotFound();
            
            // Cập nhật thông tin
            field.TenSanBong = model.TenSanBong;
            field.DiaChi = model.DiaChi;
            field.MoTaKichThuoc = model.MoTaKichThuoc;
            field.GiaThue = model.GiaThue;
            field.MoTaSan = model.MoTaSan;
            field.TrangThaiSan_ = model.TrangThaiSan_;
            field.IDLoaiSan = model.IDLoaiSan;
            field.updated_at = DateTime.Now;
            
            // Xử lý upload hình ảnh mới
            if (imageFile != null && imageFile.ContentLength > 0)
            {
                var fileName = Path.GetFileName(imageFile.FileName);
                var path = Path.Combine(Server.MapPath("~/Images/Fields/"), fileName);
                imageFile.SaveAs(path);
                field.AnhSan = "/Images/Fields/" + fileName;
            }
            
            // Cập nhật tiện ích
            field.TienIchs.Clear();
            if (selectedAmenities != null)
            {
                foreach (var amenityId in selectedAmenities)
                {
                    var amenity = db.TienIchSanBongs.Find(amenityId);
                    if (amenity != null)
                    {
                        field.TienIchs.Add(amenity);
                    }
                }
            }
            
            db.SaveChanges();
            
            TempData["SuccessMessage"] = "Cập nhật sân bóng thành công";
            return RedirectToAction("ManageFields");
        }
        
        ViewBag.LoaiSanBongs = new SelectList(db.LoaiSanBongs, "IDLoaiSan", "LoaiSan", model.IDLoaiSan);
        ViewBag.TienIchs = db.TienIchSanBongs.ToList();
        return View(model);
    }
    
    // POST: Admin/DeleteField/5
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult DeleteField(int id)
    {
        var field = db.SanBongs.Find(id);
        if (field == null)
            return Json(new { success = false, message = "Không tìm thấy sân bóng" });
        
        // Kiểm tra có booking nào đang active không
        var hasActiveBookings = db.DonDatSans.Any(b => b.IDSanBong == id && 
            (b.IDstatus == 1 || b.IDstatus == 2 || b.IDstatus == 3));
            
        if (hasActiveBookings)
        {
            return Json(new { success = false, message = "Không thể xóa sân có lịch đặt đang hoạt động" });
        }
        
        field.TrangThaiSan_ = false; // Soft delete
        db.SaveChanges();
        
        return Json(new { success = true, message = "Xóa sân bóng thành công" });
    }
    
    // GET: Admin/ManageUsers
    public ActionResult ManageUsers(int page = 1, string search = "", string role = "")
    {
        var query = db.NguoiDungs.Include(u => u.Role).AsQueryable();
        
        if (!string.IsNullOrEmpty(search))
        {
            query = query.Where(u => u.fullname.Contains(search) || 
                               u.email.Contains(search) || 
                               u.username.Contains(search));
        }
        
        if (!string.IsNullOrEmpty(role))
        {
            query = query.Where(u => u.Role.Role_name == role);
        }
        
        var users = query.OrderBy(u => u.fullname).ToPagedList(page, 20);
        
        ViewBag.Roles = db.Roles.ToList();
        ViewBag.CurrentSearch = search;
        ViewBag.CurrentRole = role;
        
        return View(users);
    }
    
    // POST: Admin/ToggleUserStatus
    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult ToggleUserStatus(int userId)
    {
        var user = db.NguoiDungs.Find(userId);
        
        if (user == null)
            return Json(new { success = false, message = "Không tìm thấy người dùng" });
            
        var currentUserId = AuthenticationHelper.GetCurrentUserId();
        if (user.IDuser == currentUserId)
            return Json(new { success = false, message = "Không thể khóa chính mình" });
            
        user.trangthaiTK = !user.trangthaiTK;
        user.update_at = DateTime.Now;
        
        db.SaveChanges();
        
        string status = user.trangthaiTK ? "kích hoạt" : "khóa";
        return Json(new { 
            success = true, 
            message = $"{(user.trangthaiTK ? "Kích hoạt" : "Khóa")} tài khoản thành công",
            newStatus = user.trangthaiTK 
        });
    }
    
    // GET: Admin/ManageBookings
    public ActionResult ManageBookings(int page = 1, string search = "", int? status = null)
    {
        var query = db.DonDatSans
            .Include(b => b.NguoiDung)
            .Include(b => b.SanBong)
            .Include(b => b.TrangThaiDonDat)
            .Include(b => b.PhuongThucThanhToan)
            .AsQueryable();
        
        if (!string.IsNullOrEmpty(search))
        {
            query = query.Where(b => b.SanBong.TenSanBong.Contains(search) || 
                               b.NguoiDung.fullname.Contains(search) ||
                               b.NguoiDung.email.Contains(search));
        }
        
        if (status.HasValue)
        {
            query = query.Where(b => b.IDstatus == status.Value);
        }
        
        var bookings = query.OrderByDescending(b => b.created_at).ToPagedList(page, 20);
        
        ViewBag.TrangThaiDonDats = db.TrangThaiDonDats.ToList();
        ViewBag.CurrentSearch = search;
        ViewBag.CurrentStatus = status;
        
        return View(bookings);
    }
    
    // AJAX: Get Dashboard Stats
    [HttpGet]
    public ActionResult GetDashboardStats()
    {
        var today = DateTime.Today;
        var thisMonth = new DateTime(today.Year, today.Month, 1);
        var lastMonth = thisMonth.AddMonths(-1);
        
        var stats = new
        {
            todayBookings = db.DonDatSans.Count(b => DbFunctions.TruncateTime(b.NgayBooking) == today),
            todayRevenue = db.DonDatSans
                .Where(b => DbFunctions.TruncateTime(b.NgayBooking) == today && b.TTThanhToan)
                .Sum(b => (decimal?)b.TongTien) ?? 0,
            thisMonthRevenue = db.DonDatSans
                .Where(b => b.NgayBooking >= thisMonth && b.TTThanhToan)
                .Sum(b => (decimal?)b.TongTien) ?? 0,
            lastMonthRevenue = db.DonDatSans
                .Where(b => b.NgayBooking >= lastMonth && b.NgayBooking < thisMonth && b.TTThanhToan)
                .Sum(b => (decimal?)b.TongTien) ?? 0,
            pendingBookings = db.DonDatSans.Count(b => b.IDstatus == 1),
            totalUsers = db.NguoiDungs.Count(u => u.trangthaiTK)
        };
        
        return Json(stats, JsonRequestBehavior.AllowGet);
    }
}
```

### Bước 6.2: AdminDashboardViewModel
#### ViewModels/AdminDashboardViewModel.cs
```csharp
public class AdminDashboardViewModel
{
    // Thống kê tổng quan
    public int TotalFields { get; set; }
    public int TotalBookings { get; set; }
    public int TotalUsers { get; set; }
    public decimal TotalRevenue { get; set; }
    
    // Thống kê hôm nay
    public int TodayBookings { get; set; }
    public decimal TodayRevenue { get; set; }
    
    // Thống kê tháng này
    public int ThisMonthBookings { get; set; }
    public decimal ThisMonthRevenue { get; set; }
    
    // Dữ liệu chi tiết
    public List<DonDatSan> RecentBookings { get; set; }
    public List<SanBong> TopFields { get; set; }
    public List<DonDatSan> PendingBookings { get; set; }
    
    // Constructor
    public AdminDashboardViewModel()
    {
        RecentBookings = new List<DonDatSan>();
        TopFields = new List<SanBong>();
        PendingBookings = new List<DonDatSan>();
    }
}
```

### Bước 6.3: Admin Dashboard View
#### Views/Admin/Dashboard.cshtml
```html
@model AdminDashboardViewModel
@{
    ViewBag.Title = "Quản trị hệ thống";
    Layout = "~/Views/Shared/_AdminLayout.cshtml";
}

<!-- Dashboard Header -->
<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pt-3 pb-2 mb-3 border-bottom">
    <h1 class="h2">
        <i class="fas fa-tachometer-alt text-primary me-2"></i>Dashboard
    </h1>
    <div class="btn-toolbar mb-2 mb-md-0">
        <div class="btn-group me-2">
            <button type="button" class="btn btn-outline-secondary">
                <i class="fas fa-download me-1"></i>Xuất báo cáo
            </button>
        </div>
        <button type="button" class="btn btn-primary" onclick="refreshDashboard()">
            <i class="fas fa-sync-alt me-1"></i>Làm mới
        </button>
    </div>
</div>

<!-- Stats Cards -->
<div class="row mb-4">
    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-primary shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-primary text-uppercase mb-1">
                            Tổng sân bóng
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">@Model.TotalFields</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-futbol fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-success shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-success text-uppercase mb-1">
                            Tổng doanh thu
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">
                            @Model.TotalRevenue.ToString("N0") VNĐ
                        </div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-dollar-sign fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-info shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-info text-uppercase mb-1">
                            Đặt sân hôm nay
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">@Model.TodayBookings</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-calendar-day fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <div class="col-xl-3 col-md-6 mb-4">
        <div class="card border-left-warning shadow h-100 py-2">
            <div class="card-body">
                <div class="row no-gutters align-items-center">
                    <div class="col mr-2">
                        <div class="text-xs font-weight-bold text-warning text-uppercase mb-1">
                            Chờ duyệt
                        </div>
                        <div class="h5 mb-0 font-weight-bold text-gray-800">@Model.PendingBookings.Count</div>
                    </div>
                    <div class="col-auto">
                        <i class="fas fa-clock fa-2x text-gray-300"></i>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Charts Row -->
<div class="row mb-4">
    <!-- Revenue Chart -->
    <div class="col-xl-8 col-lg-7">
        <div class="card shadow mb-4">
            <div class="card-header py-3 d-flex flex-row align-items-center justify-content-between">
                <h6 class="m-0 font-weight-bold text-primary">Doanh thu theo tháng</h6>
                <div class="dropdown no-arrow">
                    <a class="dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        <i class="fas fa-ellipsis-v fa-sm fa-fw text-gray-400"></i>
                    </a>
                    <div class="dropdown-menu dropdown-menu-right shadow">
                        <a class="dropdown-item" href="#">Xuất Excel</a>
                        <a class="dropdown-item" href="#">Xuất PDF</a>
                    </div>
                </div>
            </div>
            <div class="card-body">
                <div class="chart-area">
                    <canvas id="revenueChart"></canvas>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Booking Status Chart -->
    <div class="col-xl-4 col-lg-5">
        <div class="card shadow mb-4">
            <div class="card-header py-3 d-flex flex-row align-items-center justify-content-between">
                <h6 class="m-0 font-weight-bold text-primary">Trạng thái đặt sân</h6>
            </div>
            <div class="card-body">
                <div class="chart-pie pt-4 pb-2">
                    <canvas id="bookingStatusChart"></canvas>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Tables Row -->
<div class="row">
    <!-- Recent Bookings -->
    <div class="col-lg-6 mb-4">
        <div class="card shadow mb-4">
            <div class="card-header py-3">
                <h6 class="m-0 font-weight-bold text-primary">Đặt sân gần đây</h6>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table table-borderless" id="recentBookingsTable">
                        <thead>
                            <tr>
                                <th>Khách hàng</th>
                                <th>Sân</th>
                                <th>Ngày</th>
                                <th>Trạng thái</th>
                            </tr>
                        </thead>
                        <tbody>
                            @foreach (var booking in Model.RecentBookings)
                            {
                                <tr>
                                    <td>
                                        <div class="d-flex align-items-center">
                                            <div class="mr-3">
                                                <div class="icon-circle bg-primary">
                                                    <i class="fas fa-user text-white"></i>
                                                </div>
                                            </div>
                                            <div>
                                                <div class="small font-weight-bold">@booking.NguoiDung.fullname</div>
                                                <div class="small text-gray-500">@booking.NguoiDung.email</div>
                                            </div>
                                        </div>
                                    </td>
                                    <td>
                                        <div class="font-weight-bold">@booking.SanBong.TenSanBong</div>
                                        <div class="small text-gray-500">@booking.start_time.ToString(@"hh\:mm") - @booking.end_time.ToString(@"hh\:mm")</div>
                                    </td>
                                    <td>@booking.NgayBooking.ToString("dd/MM/yyyy")</td>
                                    <td>
                                        @if (booking.TrangThaiDonDat != null)
                                        {
                                            <span class="badge badge-@GetStatusClass(booking.IDstatus)">
                                                @booking.TrangThaiDonDat.Tenstatus
                                            </span>
                                        }
                                    </td>
                                </tr>
                            }
                        </tbody>
                    </table>
                </div>
                <div class="text-center">
                    <a href="@Url.Action("ManageBookings")" class="btn btn-primary">Xem tất cả</a>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Top Fields -->
    <div class="col-lg-6 mb-4">
        <div class="card shadow mb-4">
            <div class="card-header py-3">
                <h6 class="m-0 font-weight-bold text-primary">Sân bóng nổi bật</h6>
            </div>
            <div class="card-body">
                @foreach (var field in Model.TopFields)
                {
                    <div class="d-flex align-items-center mb-3">
                        <div class="mr-3">
                            <img src="@(string.IsNullOrEmpty(field.AnhSan) ? "/Images/default-field.jpg" : field.AnhSan)" 
                                 class="rounded" style="width: 60px; height: 60px; object-fit: cover;" 
                                 alt="@field.TenSanBong">
                        </div>
                        <div class="flex-grow-1">
                            <div class="font-weight-bold">@field.TenSanBong</div>
                            <div class="small text-gray-500">@field.DiaChi</div>
                            <div class="d-flex align-items-center">
                                <div class="text-warning mr-2">
                                    @for (int i = 1; i <= 5; i++)
                                    {
                                        <i class="fa@(i <= field.AverageDanhGia ? "s" : "r") fa-star"></i>
                                    }
                                </div>
                                <small>(@field.TongLuotDanhGia)</small>
                            </div>
                        </div>
                        <div class="text-right">
                            <div class="font-weight-bold text-success">@field.GiaThue.ToString("N0")</div>
                            <div class="small text-gray-500">VNĐ/giờ</div>
                        </div>
                    </div>
                }
                <div class="text-center">
                    <a href="@Url.Action("ManageFields")" class="btn btn-primary">Quản lý sân</a>
                </div>
            </div>
        </div>
    </div>
</div>

@section scripts {
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
// Revenue Chart
const revenueCtx = document.getElementById('revenueChart').getContext('2d');
const revenueChart = new Chart(revenueCtx, {
    type: 'line',
    data: {
        labels: ['Tháng 1', 'Tháng 2', 'Tháng 3', 'Tháng 4', 'Tháng 5', 'Tháng 6'],
        datasets: [{
            label: 'Doanh thu (VNĐ)',
            data: [12000000, 19000000, 15000000, 25000000, 22000000, 30000000],
            borderColor: 'rgb(75, 192, 192)',
            backgroundColor: 'rgba(75, 192, 192, 0.1)',
            tension: 0.1
        }]
    },
    options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
            y: {
                beginAtZero: true,
                ticks: {
                    callback: function(value) {
                        return new Intl.NumberFormat('vi-VN').format(value) + ' VNĐ';
                    }
                }
            }
        }
    }
});

// Booking Status Chart
const statusCtx = document.getElementById('bookingStatusChart').getContext('2d');
const statusChart = new Chart(statusCtx, {
    type: 'doughnut',
    data: {
        labels: ['Chờ duyệt', 'Đã duyệt', 'Đang chơi', 'Hoàn thành', 'Đã hủy'],
        datasets: [{
            data: [15, 25, 10, 40, 10],
            backgroundColor: [
                '#f6c23e',
                '#1cc88a',
                '#36b9cc',
                '#858796',
                '#e74a3b'
            ]
        }]
    },
    options: {
        responsive: true,
        maintainAspectRatio: false
    }
});

// Refresh dashboard function
function refreshDashboard() {
    $.get('@Url.Action("GetDashboardStats")', function(data) {
        // Update stats
        // Implementation depends on your needs
        location.reload();
    });
}

// Auto refresh every 5 minutes
setInterval(refreshDashboard, 300000);
</script>

@{
    string GetStatusClass(int? statusId)
    {
        switch (statusId)
        {
            case 1: return "warning"; // Chờ duyệt
            case 2: return "success"; // Đã duyệt  
            case 3: return "info";    // Đang chơi
            case 4: return "secondary"; // Hoàn thành
            case 5: return "danger";  // Đã hủy
            default: return "secondary";
        }
    }
}
}
```

Đây là phần đầu của hướng dẫn hoàn chỉnh. Bạn có muốn tôi tiếp tục với các phần còn lại không?