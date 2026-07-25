# 📋 HƯỚNG DẪN THAO TÁC - BÀI TẬP INVENTORY MANAGEMENT (23/07/2026)

> **Khóa học**: YonSuite Partner Implementation Training - SCM Inventory Management
> **Giảng viên**: James Cheng
> **Product Version**: YonSuite Professional v20260515

---

## 🏗️ TRƯỚC KHI LÀM: Chuẩn bị hệ thống (System Preparation)

Cần kiểm tra các thiết lập sau đã có sẵn chưa:

### 3.1 Master Data

| # | Mục | Đường dẫn menu | Ghi chú |
|---|-----|---------------|---------|
| 1 | **Business Unit** | `Yon Application > Digital Modeling > Organization Management > Organization` | Bật **Purchasing Organization** và **Inventory Organization** |
| 2 | **Department** | `Yon Application > Digital Modeling > Organization Management > Organization` | Chọn Administrative Org → New Sibling → Tạo phòng ban |
| 3 | **Warehouse (Kho)** | `Yon Application > Master Data > Shipping Information > Warehouse Information` | **Calculate Cost = Yes** → phải cấu hình Cost Area. Nếu cần Bin Management → bật **Storage Bin Management = Yes** |
| 4 | **Storage Bin** | `Yon Application > Master Data > Shipping Information > Warehouse Information > Storage Bin` | Không bắt buộc, thiết lập Bin theo cấu trúc nhiều tầng nếu cần |
| 5 | **Supplier (NCC)** | `Yon Application > Master Data > Suppliers Information` | Thiết lập tax rate, currencies, payment terms mặc định |
| 6 | **Customer (KH)** | `Yon Application > Master Data > Customers Information` | Tương tự Supplier |
| 7 | **Material (Vật liệu)** | `Yon Application > Master Data > Material` | Đánh dấu **Purchasable**. Value Management Mode: **Inventory Accounting** (vật liệu kho), Expense (dịch vụ/tiêu hao), Fixed Assets (tài sản cố định). Cấu hình Batch Management, Safety Stock nếu cần |

### 3.2 Basic Settings

| # | Mục | Đường dẫn menu | Ghi chú |
|---|-----|---------------|---------|
| 8 | **Transaction Type** | `Yon Application > Business Model Management > Business Model Management > Object Modeling > Transaction Type` | Dùng mặc định hoặc tạo mới theo yêu cầu |
| 9 | **Business Process** | `Yon Application > Digital Modeling > Business Process > Business Process > Business Process Design` | Kéo thả để thiết kế flow |
| 10 | **Work Flow** | `Yon Application > Digital Modeling > Workflow > Workflow` | Kéo thả, định nghĩa multi-level approval |
| 11 | **Warehouse Material Relationship** | `SCM Cloud > Material Management > Inventory Management > Settings > Warehouse Material Relationship` | Định nghĩa quan hệ Kho-Vật liệu. Có thể để Prompt hoặc Strict Control |
| 12 | **Storage Bin Material Cross-Reference** | `Yon Application > Master Data > Shipping Information > Warehouse Information > Storage Bin Material Cross-Reference` | Gán Bin với Material, có thể định nghĩa độ ưu tiên |
| 13 | **Inventory Status** | `SCM Cloud > Supply Chain Public > Settings > Inv Status` | Qualified / Pending Inspection / Frozen / Unqualified / Scrap |
| 14 | **Available Quantity Management** | `SCM Cloud > Material Management > Inventory Management > Settings > Calculation Rules of Available Quantity` | Cấu hình công thức tính SL khả dụng |
| 15 | **Available Quantity Check Rule** | `SCM Cloud > Material Management > Inventory Management > Settings > Available Quantity Check Rule` | Strict Control / Do Not Check, thời điểm Save hoặc Approve |
| 16 | **Available Quantity Rule Allocation** | `SCM Cloud > Material Management > Inventory Management > Settings > Available Quantity Rule Allocation` | Phân bổ rule cho từng đơn vị |

### 3.3 Inventory Parameters

| # | Mục | Đường dẫn menu | Ghi chú |
|---|-----|---------------|---------|
| 17 | **Inventory Parameters** | `SCM Cloud > Material Management > Inventory Management > Inventory Business Parameters` | Cấu hình: Allow Issue Quantity Exceed, Receipt/Issue Method (One-step/Two-step), Stocktaking params |
| 18 | **Issue/Receipt Method** | *(Trong Inventory Parameters)* | **One Step**: Lưu → cập nhật tồn kho ngay. **Two Step**: Lưu → Confirm mới cập nhật |
| 19 | **Stocktaking Params** | *(Trong Inventory Parameters)* | Bật/tắt: VMI stock, hiển thị Book Qty, tự động hiển thị từ Plan |

### 3.4 Financial Data & Opening Data

| # | Mục | Đường dẫn menu | Ghi chú |
|---|-----|---------------|---------|
| 20 | **Account Book** | `Finance Cloud > Intelligent Accounting Middle Platform > Accounting Common Settings > Accounting Book Center` | Mở kỳ: GL, Inventory Accounting, A/R, A/P, Fixed Assets |
| 21 | **Cost Area** | `Finance Cloud > Financial Accounting > Inventory Accounting > Basic Settings` | Gán Cost Area cho Inventory Organization và Warehouse |
| 22 | **Opening Inventory** | `SCM Cloud > Material Management > Inventory Management > Inv Receipt/Issue > Opening Inventory` | Nhập tồn kho đầu kỳ (từng dòng hoặc import Excel) |

---

## 📝 BÀI 1: TRANSFORMATION (Chuyển đổi vật liệu)

> **Mục tiêu**: Thực hiện chuyển đổi vật liệu và submit **Document Flow**.
> **Process Code**: S2P-P05-b02-01
> **Process**: Form Conversion → Inventory Adjustment

### 3 Loại Chuyển Đổi

| Loại | Mô tả | Quy tắc chi phí |
|------|-------|----------------|
| **One-to-One** | VL A → VL B. Có thể đổi số lượng, kho, batch | Cost B = Cost A |
| **Many-to-One** (Assembly) | VL A,B,C... → BOM A+ | Cost A+ = Σ(A,B,C...) |
| **One-to-Many** (Disassembly) | BOM A+ → A,B,C... (kèm trọng số) | Phân bổ chi phí A+ cho A,B,C theo trọng lượng |

### Quy trình thao tác từng bước:

| Bước | Thao tác | Chi tiết |
|------|---------|---------|
| **1** | 🖥️ Vào menu | `SCM Cloud > Material Management > Inventory Management > Inventory Adjustment > Transformation` |
| **2** | ➕ Tạo mới | Click **"Add"** / **"New"** |
| **3** | ⚙️ Chọn loại | Chọn **Transaction Type** và **Conversion Type**: `One-to-One` / `Many-to-One` / `One-to-Many` |
| **4** | 📦 Add Group | Click **"Add Group"** để tạo nhóm chuyển đổi (mỗi nhóm = 1 cặp trước/sau) |
| **5** | ⬅️ Nhập Before | **Dòng trên**: Mã VL nguồn → Số lượng → Kho xuất → Batch (nếu có) |
| **6** | ➡️ Nhập After | **Dòng dưới**: Mã VL đích → Số lượng → Kho nhận → Batch mới |
| **7** | ⚖️ Trọng số (nếu 1-Nhiều) | Nhập **Allocated Weight** để phân bổ chi phí |
| **8** | 💾 Lưu | Click **"Save"** |
| **9** | ✅ Phê duyệt | Click **"Submit"** / **"Approve"** |
| **10** | 🔍 Kiểm tra kết quả | Hệ thống tự động sinh: |

### Kết quả mong đợi:

```
Transformation Document (đã phê duyệt)
    ├── 🔵 Other Goods Issue (xuất VL cũ)
    ├── 🟢 Other Goods Receipt (nhập VL mới)
    └── 📊 Cặp bút toán kế toán (Pairwise Issue/Receipt Accounting)
```

### ✅ Checklist nộp bài:
- [ ] Chụp màn hình **Document Flow** (luồng chứng từ)
- [ ] Đảm bảo hiển thị đủ: Transformation → Goods Issue + Goods Receipt + Accounting

---

## 📝 BÀI 2: STOCK TAKING (Kiểm kê cuối năm)

> **Mục tiêu**: Hoàn thành quy trình kiểm kê và submit **Document Flow**.
> **Process Code**: S2P-P05-b05-01
> **Process**: Inventory Task → Inventory Record → Inventory Discrepancy → Inventory Gain/Inventory Loss

### Quy trình thao tác từng bước:

| Bước | Actor | Thao tác | Chi tiết |
|------|-------|---------|---------|
| **1** | 👔 Phòng TC | 🖥️ Vào menu | `SCM Cloud > Material Management > Inventory Management > Stocktaking > Closing Stocktaking` |
| **2** | 👔 Phòng TC | ➕ Tạo Plan | Click **"Add"** → Tạo **Review Plan** (Kế hoạch kiểm kê) |
| **3** | 👔 Phòng TC | ⚙️ Chọn phạm vi | Chọn **Transaction Type**, **Warehouse**, và **Stocktaking Range**: |
| | | | • 📦 **Whole Warehouse** - Toàn bộ kho |
| | | | • 📂 **Specified Category** - Theo danh mục |
| | | | • 🏷️ **Specified Materials** - Theo vật liệu cụ thể |
| **4** | 👔 Phòng TC | 📸 Chụp Snapshot | Click **"Generate Snapshot"** → Hệ thống ghi nhận **Book Quantity** tại thời điểm kiểm kê |
| **5** | 👷 Thủ kho | 🔢 Nhập SL thực tế | Vào tab **"Input Physical Inventory"** → Nhập số lượng kiểm đếm thực tế từng dòng |
| **6** | ⚙️ Hệ thống | 📊 Tính chênh lệch | Hệ thống tự động tính: |
| | | | `Chênh lệch = SL Thực tế - SL Sổ sách - SL Giao dịch PS` |
| **7** | 👔 Phòng TC | 🔍 Xem xét | Vào tab **"Counting Review"** → Xem báo cáo Gain/Loss |
| **8** | 👔 Phòng TC | ✅ Xác nhận | Click **"Result Confirmation"** → Phê duyệt kết quả kiểm kê |
| **9** | ⚙️ Hệ thống | 🔍 Tự động sinh | Hệ thống tự động sinh chứng từ điều chỉnh |

### Kết quả mong đợi:

```
Stocktaking Plan (Review Plan)
    ├── 📸 Inventory Snapshot (Book Quantity)
    ├── 🔢 Physical Inventory (Actual Count)
    ├── 📊 Counting Review (Gain/Loss Discrepancy)
    ├── ✅ Result Confirmation
    │       ├── 🟢 Hàng THỪA (Gain) → Other Goods Receipt
    │       └── 🔵 Hàng THIẾU (Loss) → Other Goods Issue
    └── 📊 Bút toán kế toán điều chỉnh
```

### 🔑 Các điểm cần lưu ý:

| Tham số | Ý nghĩa |
|---------|---------|
| **Comp Stock on Book = Yes** | Thủ kho **thấy** Book Qty khi nhập số liệu kiểm kê |
| **Comp Stock on Book = No** | Thủ kho **không thấy** Book Qty → nhập mù (kiểm soát chặt hơn) |
| **Calculate Period Transactions = Yes** | Hệ thống tự tính bù trừ giao dịch phát sinh trong thời gian kiểm kê |
| **Calculate Period Transactions = No** | Mọi giao dịch phải **đóng băng** trong thời gian kiểm kê |

### So sánh 2 loại kiểm kê:

| | Closing Stocktaking (Cuối năm) | Daily Stocktaking (Hàng ngày) |
|---|---|---|
| **Menu** | Stocktaking > Closing Stocktaking | Stocktaking > Daily Stocktaking |
| **Cần Plan?** | ✅ Có - Phòng TC tạo trước | ❌ Không - Thủ kho tạo trực tiếp |
| **Actor chính** | Phòng TC + Thủ kho | Thủ kho / QL Kho |
| **Phạm vi** | Toàn bộ kho / Danh mục / VL cụ thể | VL cụ thể, khu vực nhỏ |

### ✅ Checklist nộp bài:
- [ ] Chụp màn hình **Document Flow** (luồng chứng từ)
- [ ] Đảm bảo hiển thị đủ: Plan → Physical Inventory → Counting Review → Gain/Loss Adjustment

---

## 📝 BÀI 3: SCRAP - SCENARIO 2 (Xử lý phế liệu - Hủy trực tiếp)

> **Mục tiêu**: Hoàn thành Scrap Scenario 2 (Issue Only) và submit **Document Flow**.
> **Process Code**: S2P-P05-b03-01
> **Process**: Scrap Material Order → Material Outbound → Accounting Adjustment

### Quy trình thao tác từng bước:

| Bước | Thao tác | Chi tiết |
|------|---------|---------|
| **1** | 🖥️ Vào menu | `SCM Cloud > Material Management > Inventory Management > Inventory Adjustment > Scrap` |
| **2** | ➕ Tạo mới | Click **"Add"** / **"New"** |
| **3** | ⚙️ Chọn loại GD | Chọn **Transaction Type = "Scrap (Issue Only)"** |
| | | ⚠️ Trong cấu hình Transaction Type: Chọn **"Yes"** cho `Only Generate Scrap Issue After Approval` |
| **4** | 🏢 Chọn tổ chức | Chọn **Inventory Organization** |
| **5** | 🏭 Chọn VL | Chọn **Warehouse** (kho nguồn), tìm và chọn **Material** cần xử lý phế liệu |
| **6** | 🔢 Nhập SL | Nhập **Adjustment Quantity** (số lượng cần hủy) |
| **7** | 🚫 Không chọn kho đích | **Không** chọn Transfer In Warehouse (vì Scenario 2 là hủy trực tiếp) |
| **8** | 💾 Lưu | Click **"Save"** |
| **9** | ✅ Phê duyệt | Click **"Submit"** / **"Approve"** (đề xuất thiết lập workflow phê duyệt) |
| **10** | 🔍 Kiểm tra kết quả | Hệ thống chỉ sinh Other Goods Issue |

### Kết quả mong đợi:

```
Scrap Document (Issue Only - đã phê duyệt)
    ├── 🔵 Other Goods Issue (xuất hủy)
    │       └── Material B → ❌ throw away (hủy vĩnh viễn)
    └── 📊 Bút toán kế toán (xuất kho giảm giá trị)
```

### 🔄 So sánh 2 Scenario của Scrap:

| | Scenario 1 | Scenario 2 ⭐ **(Bài tập)** |
|---|---|---|
| **Transaction Type** | Scrap | Scrap (Issue Only) |
| **Only Generate Scrap Issue** | No | **Yes** |
| **Kho phế liệu đích?** | ✅ Có - chọn Transfer In Warehouse | ❌ Không |
| **Chứng từ sinh ra** | 🔵 Goods Issue + 🟢 Goods Receipt | 🔵 Chỉ Goods Issue |
| **Kết quả** | VL A → Scrap Warehouse → thanh lý sau | VL B → **throw away** (hủy ngay) |
| **Phù hợp khi** | Cần theo dõi phế liệu tồn, có giá trị thu hồi | Không cần theo dõi, hủy trực tiếp |

### ✅ Checklist nộp bài:
- [ ] Chụp màn hình **Document Flow** (luồng chứng từ)
- [ ] Đảm bảo hiển thị đủ: Scrap (Issue Only) → Goods Issue + Accounting
- [ ] Lưu ý: **Không** có Goods Receipt (đây là điểm khác Scenario 1)

---

## 📊 TÓM TẮT TOÀN BỘ NAVIGATION PATH

```
SCM Cloud > Material Management > Inventory Management
│
├── 📥 Inv Receipt/Issue
│   ├── Opening Inventory .................. Nhập tồn kho đầu kỳ
│   ├── Other Goods Issue .................. Xuất kho khác
│   └── Other Goods Receipt ................ Nhập kho khác
│
├── 🔄 Inventory Adjustment
│   ├── Transformation ............... ⭐ BÀI 1
│   ├── Scrap ........................ ⭐ BÀI 3
│   └── Stock Transfer ............... Chuyển kho nội bộ
│
├── 📋 Stocktaking
│   ├── Closing Stocktaking .......... ⭐ BÀI 2
│   └── Daily Stocktaking ............ Kiểm kê hàng ngày
│
├── ⚙️ Settings
│   ├── Warehouse Material Relationship
│   ├── Calculation Rules of Available Quantity
│   ├── Available Quantity Check Rule
│   └── Available Quantity Rule Allocation
│
├── 📊 Inventory Business Parameters

└── 📈 Reports
    ├── On-hand Stock Query ........... Truy vấn tồn kho hiện có
    ├── Stock Aging Analysis ........... Phân tích tuổi tồn kho
    └── Issue/Receipt Query ............ Truy vấn nhập/xuất
```

---

## 🎯 TỔNG KẾT 7 QUY TRÌNH INVENTORY MANAGEMENT

| # | Process Code | Quy trình | Các bước chính | Menu Path |
|---|-------------|----------|---------------|-----------|
| 1 | S2P-P05-b02-01 | **Transformation** | Form Conversion → Inventory Adjustment | `Inventory Adjustment > Transformation` |
| 2 | S2P-P05-b03-01 | **Scrap** | Scrap Order → Material Outbound → Accounting Adj. | `Inventory Adjustment > Scrap` |
| 3 | S2P-P05-b04-01 | **Stock Transfer** | Warehouse Stock Transfer Order | `Inventory Adjustment > Stock Transfer` |
| 4 | S2P-P05-b05-01 | **Year-End Stocktaking** | Task → Record → Discrepancy → Gain/Loss | `Stocktaking > Closing Stocktaking` |
| 5 | S2P-P05-b05-02 | **Daily Inventory Check** | Task → Record → Discrepancy → Gain/Loss | `Stocktaking > Daily Stocktaking` |
| 6 | S2P-P05-b07-01 | **Other Goods Issue** | Issue Order → Outbound → Financial Adj. | `Inv Receipt/Issue > Other Goods Issue` |
| 7 | S2P-P05-b07-02 | **Other Goods Receipt** | Receipt Order → Receipt → Financial Adj. | `Inv Receipt/Issue > Other Goods Receipt` |

---

## ⚠️ LƯU Ý QUAN TRỌNG

1. **Document Flow**: Khi chụp màn hình nộp bài, phải hiển thị được **luồng chứng từ** (document flow) thể hiện mối liên kết giữa các chứng từ được sinh ra tự động.

2. **Thứ tự thao tác**: Làm theo đúng thứ tự bước trong từng quy trình - không được bỏ qua bước phê duyệt.

3. **Workflow**: Đề xuất thiết lập workflow phê duyệt cho Scrap và Transformation để có document flow đầy đủ hơn.

4. **One-step vs Two-step**: Kiểm tra Inventory Parameter để biết hệ thống đang dùng phương thức nào - ảnh hưởng đến thời điểm cập nhật tồn kho.

5. **Opening Inventory**: Đảm bảo đã nhập tồn kho đầu kỳ trước khi thực hiện các nghiệp vụ điều chỉnh.

---

> 📅 **Ngày học**: July 23, 2026
> 📄 **Slide nguồn**: YonSuite Partner Implementation Training_SCM_Inventory Mangement_20260723(with Homework).pdf
> 🤖 **Generated with Claude Code** | Co-Authored-By: Claude <noreply@anthropic.com>
