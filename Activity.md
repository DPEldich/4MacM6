# Sơ đồ Activity — Hệ thống quản lý cửa hàng laptop

## 1. Phạm vi và quy ước

Tài liệu chọn **12 luồng nghiệp vụ chính**, tương ứng cách tổ chức bản `Sequence.md` cô đọng, phù hợp đồ án môn học. Các thao tác liên quan được lồng ghép; không cần một sơ đồ cho từng use case nhỏ. Mã UC dùng để liên hệ với `DoAn.docx`.

Mã sử dụng **Mermaid `flowchart TD` để biểu diễn luồng hoạt động**: nút tròn là bắt đầu/kết thúc, hình chữ nhật là hành động, hình thoi là quyết định và các cạnh có nhãn là điều kiện rẽ nhánh. Đây là cách biểu diễn activity bằng Mermaid, không phải cú pháp UML Activity bản địa. Không dùng thanh fork/join vì các luồng được chọn không có bằng chứng cần xử lý song song.

Trừ đăng nhập, tiền điều kiện là người dùng đã đăng nhập và mở được phân hệ. Nhân viên thao tác thay mặt khách hàng/nhà cung cấp. Các bước nạp biểu mẫu, kết nối CSDL và ánh xạ DTO được rút gọn; kết thúc một sơ đồ là kết thúc lần xử lý đang mô tả, không nhất thiết thoát ứng dụng. Các nhánh hủy đóng vai trò kết thúc hoặc trở về màn hình phù hợp, không mặc định hoàn tác dữ liệu đã ghi.

## 2. Danh sách sơ đồ

| Mã | Nghiệp vụ | Use case liên quan |
|---|---|---|
| AD01 | Đăng nhập | UC01 |
| AD02 | Quản lý sản phẩm | UC24, UC25, UC27 |
| AD03 | Quản lý sản phẩm của nhà cung cấp | UC12 |
| AD04 | Lập phiếu nhập | UC35 |
| AD05 | Hoàn thành phiếu nhập | UC64 |
| AD06 | Lập đơn bán hàng, khách hàng và khuyến mãi | UC41, UC15, UC42 |
| AD07 | Tra cứu tồn kho và serial | UC30, UC31 |
| AD08 | Tiếp nhận và cập nhật bảo hành | UC21, UC22 |
| AD09 | Quản lý chương trình khuyến mãi | UC50, UC51, UC53 |
| AD10 | Cấp và cập nhật tài khoản | UC55, UC56 |
| AD11 | Tạo vai trò và thiết lập quyền | UC72, UC59 |
| AD12 | Thống kê và xuất báo cáo | UC60, UC61, UC62 |

## 3. Mã Mermaid

### AD01. Đăng nhập

**Liên hệ:** UC01.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Nhập tên đăng nhập và mật khẩu]
    A --> B{Đã nhập đủ thông tin?}
    B -- Không --> E[Thông báo thiếu thông tin]
    E --> A
    B -- Có --> C[Tìm tài khoản đang hoạt động]
    C --> D{Tìm thấy tài khoản?}
    D -- Không --> F[Thông báo sai thông tin đăng nhập]
    D -- Có --> G[Kiểm tra mật khẩu bằng BCrypt]
    G --> H{Mật khẩu đúng?}
    H -- Không --> F
    F --> A
    H -- Có --> I[Lưu tài khoản vào phiên làm việc]
    I --> J[Thông báo thành công và mở Dashboard]
    J --> T((Kết thúc))
```

**Nguồn:** `GUI/JavaFX/LoginView.java`; `Bus/AccountBUS.java`; `DAO/AccountDAO.java`.

### AD02. Quản lý sản phẩm

**Liên hệ:** UC24, UC25, UC27.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn thêm, sửa hoặc xóa sản phẩm]
    A --> B{Có quyền tương ứng?}
    B -- Không --> C[Thông báo không có quyền]
    C --> T((Kết thúc))
    B -- Có --> D{Thao tác đã chọn?}
    D -- Thêm hoặc sửa --> E[Hiển thị biểu mẫu sản phẩm]
    E --> F[Nhập hoặc điều chỉnh thông tin]
    F --> G{Lưu hay hủy?}
    G -- Hủy --> T
    G -- Lưu --> H{Dữ liệu hợp lệ?}
    H -- Không --> I[Thông báo lỗi nhập liệu]
    I --> F
    H -- Có --> J[Ghi và kiểm tra thông tin sản phẩm]
    J --> K{Lưu thành công?}
    K -- Không --> L[Thông báo lỗi lưu]
    L --> F
    K -- Có --> M[Nạp lại danh sách sản phẩm]
    M --> T
    D -- Xóa --> N[Chọn sản phẩm cần xóa]
    N --> O{Đồng ý xác nhận xóa?}
    O -- Không --> T
    O -- Có --> P{Sản phẩm đã có chi tiết nhập?}
    P -- Có --> Q[Thông báo không được xóa]
    Q --> T
    P -- Không --> R[Thực hiện xóa và thông báo kết quả]
    R --> M
```

**Lưu ý:** Khi thêm, biểu mẫu còn cập nhật giá bán bằng JDBC sau khi tạo sản phẩm. Nếu một bước ghi lỗi, không mặc định toàn bộ thao tác đã rollback.

**Nguồn:** `GUI/JavaFX/ProductView.java`; `GUI/JavaFX/ProductEditDialogFx.java`; `DAO/SanPhamDAO.java`.

### AD03. Quản lý sản phẩm của nhà cung cấp

**Liên hệ:** UC12.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn nhà cung cấp và quản lý sản phẩm]
    A --> B{Có quyền SUPPLIER.UPDATE?}
    B -- Không --> C[Thông báo không có quyền]
    C --> T((Kết thúc))
    B -- Có --> D[Hiển thị danh sách sản phẩm liên kết]
    D --> E{Thao tác?}
    E -- Thêm --> F[Chọn sản phẩm chưa liên kết]
    F --> G[Kiểm tra và lưu liên kết cung ứng]
    E -- Gỡ liên kết --> H[Chọn sản phẩm liên kết]
    H --> I{Xác nhận gỡ liên kết?}
    I -- Không --> D
    I -- Có --> J[Xóa liên kết cung ứng]
    G --> K[Thông báo kết quả và cập nhật danh sách]
    J --> K
    K --> D
    E -- Đóng --> T
```

**Lưu ý:** Gỡ liên kết không xóa sản phẩm khỏi hệ thống và không tạo phiếu nhập.

**Nguồn:** `GUI/JavaFX/SupplierProductsDialogFx.java`; `Bus/supplierProductsBUS.java`; `DAO/supplierProductsDAO.java`.

### AD04. Lập phiếu nhập

**Liên hệ:** UC35.

```mermaid
flowchart TD
S((Bắt đầu)) --> A{Có quyền RECEIPT.ADD?}
    A -- Không --> B[Thông báo không có quyền]
    B --> T((Kết thúc))
    A -- Có --> C[Chọn nhà cung cấp]
    C --> D[Hiển thị sản phẩm liên kết với nhà cung cấp]
    D --> E[Thêm sản phẩm, nhập giá và tháng bảo hành]
    E --> F[Nhập hoặc sinh danh sách serial cho từng dòng]
    F --> G[Tính số lượng từ số serial và tổng tiền phiếu]
    G --> H{Lưu hay hủy?}
    H -- Hủy --> T
    H -- Lưu --> I{Phiếu và các serial hợp lệ?}
    I -- Không --> J[Thông báo lỗi và yêu cầu sửa]
    J --> E
    I -- Có --> K[Tạo phiếu nhập chờ xác nhận]
    K --> L[Lưu chi tiết nhập, tính lại giá vốn và lưu serial]
    L --> M{Các bước lưu thành công?}
    M -- Có --> N[Thông báo thành công và nạp lại danh sách]
    N --> T
    M -- Không --> O[Thông báo lỗi, giữ cửa sổ nhập]
    O --> T
```

**Lưu ý:** Phiếu chờ chưa cộng stock bằng trigger hoàn thành. Các DAO ghi riêng; khi lỗi, một phần dữ liệu có thể đã lưu. Nút kết thúc ở nhánh lỗi kết thúc lần xử lý lưu, không có nghĩa đóng cửa sổ.

**Nguồn:** `GUI/JavaFX/ReceiptEditDialogFx.java`; `DAO/PhieuNhapDAO.java`; `DAO/ChiTietPhieuNhapDAO.java`; `DAO/SerialDAO.java`.

### AD05. Hoàn thành phiếu nhập

**Liên hệ:** UC64.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Mở chi tiết phiếu nhập đang chờ]
    A --> B[Chọn hoàn thành phiếu]
    B --> C{Đồng ý xác nhận?}
    C -- Không --> T((Kết thúc))
    C -- Có --> D[Cập nhật trạng thái phiếu thành hoàn thành]
    D --> E{Cập nhật và kiểm tra trigger thành công?}
    E -- Không --> F[Thông báo không cập nhật được]
    F --> T
    E -- Có --> G[Trigger tăng tồn kho theo các dòng đang hoạt động]
    G --> H[Lấy tổng tiền và gọi ghi khoản PURCHASE âm]
    H --> I[Trả kết quả cập nhật trạng thái về giao diện]
    I --> J[Thông báo hoàn thành và nạp lại dữ liệu]
    J --> T
```

**Lưu ý:** Ghi khoản chi là lời gọi riêng. Code không kiểm tra kết quả boolean của bước này, nên giao diện vẫn có thể báo hoàn thành dù sổ quỹ ghi lỗi; không vẽ một nhánh cảnh báo mà chương trình chưa có.

**Nguồn:** `GUI/JavaFX/ReceiptDetailDialogFx.java`; `DAO/PhieuNhapDAO.java`; `DAO/CashLedgerDAO.java`; `data/sql.sql`.

### AD06. Lập đơn bán hàng, khách hàng và khuyến mãi

**Liên hệ:** UC41, UC15, UC42.

```mermaid
flowchart TD
S((Bắt đầu)) --> A{Có quyền ORDER.ADD?}
    A -- Không --> B[Thông báo không có quyền]
    B --> T((Kết thúc))
    A -- Có --> C[Tạo đơn tạm và nạp dữ liệu bán hàng]
    C --> D[Chọn sản phẩm và serial]
    D --> E[Nhập số điện thoại khách hàng]
    E --> F{Tìm thấy khách hàng cũ?}
    F -- Có --> G[Nạp thông tin khách hàng]
    F -- Không --> H[Tạo khách tạm và nhập tên khách hàng]
    G --> I{Chọn khuyến mãi?}
    H --> I
    I -- Không --> K[Chọn phương thức thanh toán và lưu đơn]
    I -- Có --> J[Áp dụng BILL hoặc chọn serial quà GIFT]
    J --> K
    K --> L{Giỏ hàng, khách hàng và serial hợp lệ?}
    L -- Không --> M[Thông báo lỗi, giữ biểu mẫu để sửa]
    M --> D
    L -- Có --> N[Bắt đầu transaction lưu đơn và chi tiết]
    N --> O[Đánh dấu serial, giảm tồn kho và ghi SALE]
    O --> P{Các bước ghi thành công và đủ tồn kho?}
    P -- Không --> Q[Rollback transaction và báo lưu thất bại]
    Q --> T
    P -- Có --> R[Commit transaction đơn hàng]
    R --> U{Khách hàng mới?}
    U -- Không --> X[Thông báo bán hàng thành công]
    U -- Có --> V[Cập nhật tên và số điện thoại khách tạm]
    V --> W{Cập nhật khách thành công?}
    W -- Có --> X
    W -- Không --> Y[Báo đơn đã lưu nhưng cập nhật khách thất bại]
    X --> T
    Y --> T
```

**Lưu ý:** PERCENT đã phản ánh trong giá sản phẩm. Cập nhật khách mới nằm sau commit; lỗi này không rollback đơn đã lưu. Sơ đồ tập trung vào nhánh lưu, lược bỏ việc parent dọn dữ liệu tạm khi đóng cửa sổ chưa hoàn tất.

**Nguồn:** `GUI/JavaFX/OrderCreateDialogFx.java`; `GUI/JavaFX/OrderView.java`; `DAO/OrderDAO.java`; `Bus/CustomerBUS.java`.

### AD07. Tra cứu tồn kho và serial

**Liên hệ:** UC30, UC31.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Mở màn hình tồn kho]
    A --> B[Nạp và hiển thị tồn kho theo lô]
    B --> C[Nhập hoặc chọn tiêu chí lọc]
    C --> D[Hiển thị các lô phù hợp]
    D --> E{Xem serial của một lô?}
    E -- Không --> T((Kết thúc))
    E -- Có --> F[Chọn lô và mở danh sách serial]
    F --> G[Tra cứu serial và mã đơn bán liên quan]
    G --> H[Hiển thị serial, trạng thái và mã đơn]
    H --> I{Tiếp tục lọc serial?}
    I -- Có --> J[Lọc theo serial, mã đơn hoặc tình trạng bán]
    J --> H
    I -- Không --> T
```

**Lưu ý:** Trong cửa sổ serial, nhãn Còn hàng/Đã bán được suy ra từ có hoặc không có mã đơn liên quan.

**Nguồn:** `GUI/JavaFX/InventoryView.java`; `GUI/JavaFX/SerialDetailDialogFx.java`; `DAO/khoDAO.java`; `DAO/SerialDAO.java`.

### AD08. Tiếp nhận và cập nhật bảo hành

**Liên hệ:** UC21, UC22.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn tạo phiếu hoặc sửa phiếu bảo hành]
    A --> B{Có quyền tương ứng?}
    B -- Không --> C[Thông báo không có quyền]
    C --> T((Kết thúc))
    B -- Có --> D{Tạo mới hay cập nhật?}
    D -- Tạo mới --> E[Nhập serial và tra cứu đơn bán]
    E --> F{Tìm thấy đơn bán?}
    F -- Không --> G[Thông báo và yêu cầu tra cứu lại]
    G --> E
    F -- Có --> H[Nạp sản phẩm, hạn bảo hành và các phiếu hiện có]
    H --> I[Khóa lựa chọn hết hạn hoặc có phiếu status từ 1 đến 3]
    I --> J[Chọn máy, nhập mô tả, ngày trả và trạng thái]
    D -- Cập nhật --> K[Mở phiếu đã chọn và sửa thông tin xử lý]
    J --> L{Lưu hay hủy?}
    K --> L
    L -- Hủy --> T
    L -- Lưu --> M{Dữ liệu hợp lệ theo chế độ?}
    M -- Không --> N[Thông báo lỗi, giữ biểu mẫu]
    N --> T
    M -- Có --> O[Thêm hoặc cập nhật phiếu bảo hành]
    O --> P[Thông báo kết quả, nạp lại khi thành công]
    P --> T
```

**Lưu ý:** Khi cập nhật, serial và sản phẩm được giữ cố định. Code có bốn trạng thái nhưng không ép thứ tự chuyển trạng thái; không thêm bước “đã trả khách” riêng.

**Nguồn:** `GUI/JavaFX/WarrantyEditDialogFx.java`; `GUI/JavaFX/WarrantyDetailDialogFx.java`; `DAO/warrantyCardDAO.java`.

### AD09. Quản lý chương trình khuyến mãi

**Liên hệ:** UC50, UC51, UC53.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn thêm hoặc sửa chương trình]
    A --> B{Có quyền tương ứng?}
    B -- Không --> C[Thông báo không có quyền]
    C --> T((Kết thúc))
    B -- Có --> D[Nhập tên, loại, thời gian và điều kiện khuyến mãi]
    D --> E{Dữ liệu hợp lệ?}
    E -- Không --> F[Thông báo lỗi để sửa]
    F --> D
    E -- Có --> G[Lưu chương trình khuyến mãi]
    G --> H{Lưu thành công?}
    H -- Không --> I[Thông báo lưu thất bại]
    I --> T
    H -- Có --> J{Cấu hình sản phẩm PERCENT hoặc GIFT?}
    J -- Không --> T
    J -- Có --> K{Có quyền PROMOTION.UPDATE?}
    K -- Không --> C
    K -- Có --> L[Chọn sản phẩm, mức giảm hoặc sản phẩm quà]
    L --> M{Cấu hình hợp lệ và không xung đột?}
    M -- Không --> N[Thông báo lỗi cấu hình]
    N --> L
    M -- Có --> O[Lưu cấu hình sản phẩm khuyến mãi]
    O --> P[Thông báo kết quả và cập nhật danh sách]
    P --> T
```

**Lưu ý:** BILL cấu hình giảm tổng hóa đơn; PERCENT/GIFT cấu hình thêm ở cấp sản phẩm. Hai lần lưu chương trình và cấu hình không được coi là một transaction chung.

**Nguồn:** `GUI/JavaFX/PromotionEditDialogFx.java`; `GUI/JavaFX/PromotionItemEditDialogFx.java`; `Bus/promotionsBUS.java`.

### AD10. Cấp và cập nhật tài khoản

**Liên hệ:** UC55, UC56.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn thêm hoặc sửa tài khoản]
    A --> B{Có quyền tương ứng?}
    B -- Không --> C[Thông báo không có quyền]
    C --> T((Kết thúc))
    B -- Có --> D[Hiển thị biểu mẫu, nhân viên và vai trò]
    D --> E[Nhập thông tin tài khoản]
    E --> F{Lưu hay hủy?}
    F -- Hủy --> T
    F -- Lưu --> G{Đủ thông tin và username mới không trùng?}
    G -- Không --> H[Thông báo lỗi nhập liệu]
    H --> E
    G -- Có --> I{Có mật khẩu mới?}
    I -- Có --> J[Băm mật khẩu bằng BCrypt]
    I -- Không, đang sửa --> K[Giữ mật khẩu cũ]
    J --> L[Thêm hoặc cập nhật tài khoản]
    K --> L
    L --> M{Lưu thành công?}
    M -- Không --> N[Thông báo lỗi lưu]
    N --> E
    M -- Có --> O[Thông báo và nạp lại danh sách]
    O --> T
```

**Lưu ý:** Khi thêm bắt buộc có mật khẩu; khi sửa, username chỉ đọc và để trống mật khẩu sẽ giữ mật khẩu cũ. Chức năng sửa này không thay đổi trạng thái hoạt động.

**Nguồn:** `GUI/JavaFX/AccountEditDialogFx.java`; `DAO/AccountDAO.java`; `DTO/PasswordUtil.java`.

### AD11. Tạo vai trò và thiết lập quyền

**Liên hệ:** UC72, UC59.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn tạo vai trò hoặc sửa quyền]
    A --> B{Có quyền tương ứng?}
    B -- Không --> C[Thông báo không có quyền]
    C --> T((Kết thúc))
    B -- Có --> D{Tạo vai trò mới?}
    D -- Có --> E[Nhập tên vai trò và chọn Tạo]
    E --> F{Tên hợp lệ và chưa trùng?}
    F -- Không --> G[Thông báo lỗi tên vai trò]
    G --> E
    F -- Có --> H[Tạo role và RBAC ban đầu trong transaction]
    H --> I{Tạo thành công?}
    I -- Không --> J[Thông báo không tạo được vai trò]
    J --> T
    I -- Có --> K[Nạp ma trận quyền của vai trò]
    D -- Không --> K
    K --> L[Bật hoặc tắt các quyền]
    L --> M{Lưu quyền hay đóng?}
    M -- Đóng --> T
    M -- Lưu --> N[Cập nhật lần lượt các quyền có ID hợp lệ]
    N --> O{Tất cả cập nhật thành công?}
    O -- Có --> P[Thông báo thành công và đóng cửa sổ]
    O -- Không --> Q[Thông báo lỗi hoặc lưu chưa hoàn tất]
    P --> T
    Q --> T
```

**Lưu ý:** Đóng sau khi tạo role không xóa role đã tạo. Lưu ma trận có thể thành công một phần và không tự tải lại mọi PermissionManager của phiên hiện tại.

**Nguồn:** `GUI/JavaFX/RolePermissionsDialogFx.java`; `DAO/RoleDAO.java`; `DAO/RolePermissionsDAO.java`.

### AD12. Thống kê và xuất báo cáo

**Liên hệ:** UC60, UC61, UC62.

```mermaid
flowchart TD
S((Bắt đầu)) --> A[Chọn đơn bán hoặc phiếu nhập]
    A --> B[Chọn tiêu chí, đối tượng và khoảng ngày]
    B --> C[Chọn Xuất thống kê]
    C --> D{Ngày và đối tượng hợp lệ?}
    D -- Không --> E[Thông báo, yêu cầu sửa lựa chọn]
    E --> B
    D -- Có --> F[Truy vấn số tổng, số bản ghi và dữ liệu trang]
    F --> G[Hiển thị báo cáo hoặc danh sách rỗng]
    G --> H{Xuất file?}
    H -- Không --> T((Kết thúc))
    H -- Có --> I{Định dạng?}
    I -- Excel --> J[Lấy dữ liệu trang hiện tại]
    I -- PDF --> K[Lấy toàn bộ dữ liệu theo bộ lọc hợp lệ]
    J --> L{Có dữ liệu để xuất?}
    K --> L
    L -- Không --> M[Thông báo không có dữ liệu]
    M --> T
    L -- Có --> N[Chọn nơi lưu file]
    N --> O{Hủy chọn nơi lưu?}
    O -- Có --> T
    O -- Không --> P[Tạo và ghi file]
    P --> Q[Thông báo thành công hoặc lỗi ghi file]
    Q --> T
```

**Lưu ý:** Tiêu chí Nhân viên/Sản phẩm cần đối tượng cụ thể. Biểu đồ cơ cấu phương thức thanh toán chỉ hiển thị ở thống kê đơn bán với tiêu chí Tiền.

**Nguồn:** `GUI/JavaFX/StatisticsView.java`; `Bus/ThongKeBUS.java`; `DAO/ThongKeDAO.java`.

## 4. Ghi chú sử dụng

Các sơ đồ mô tả nghiệp vụ theo mã nguồn và phần phân tích đã thực hiện, không phải bằng chứng chạy thử. Nhánh lỗi chỉ biểu diễn ở mức cần thiết để hiểu luồng. Không tự bổ sung transaction, kiểm tra quyền hay bước xử lý mà chương trình chưa có. Khi đưa vào báo cáo, có thể kết xuất từng khối Mermaid và dùng tên AD01–AD12 làm tên hình.
