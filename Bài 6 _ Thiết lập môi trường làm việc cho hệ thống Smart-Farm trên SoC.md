**Bài 6 : Thiết lập môi trường làm việc cho hệ thống Smart-Farm trên SoC** 

I. NHẬT KÝ (LOG) CÁC CÂU LỆNH POWERSHELL THỰC HIỆN

Dưới đây là chuỗi lệnh thực thi tuần tự từ thư mục Home (\~) của người dùng:

Bước 1: Tạo thư mục cấu trúc bằng 1 dòng lệnh

powershell  
mkdir \~\\smart-farm\\bin, \~\\smart-farm\\config, \~\\smart-farm\\data

Hãy thận trọng khi sử dụng mã.

* Giải thích: Sử dụng dấu phẩy , giúp PowerShell hiểu ta truyền một mảng đường dẫn và khởi tạo cả 3 thư mục con bin, config, data lồng trong smart-farm cùng một lúc.

Bước 2: Tạo file ẩn .env trong thư mục config

powershell  
New-Item \-Path \~\\smart-farm\\config\\.env \-ItemType File \-Attributes Hidden

Hãy thận trọng khi sử dụng mã.

* Giải thích nguyên nhân Get-ChildItem thông thường không nhìn thấy: Theo cơ chế bảo mật và quản lý hệ thống của Windows, các file có thuộc tính ẩn (Hidden) hoặc hệ thống (System) sẽ mặc định bị bộ lọc của lệnh Get-ChildItem (hoặc lệnh ls/dir) bỏ qua để tránh người dùng phổ thông xóa nhầm file cấu hình quan trọng. Muốn nhìn thấy, bắt buộc phải truyền thêm tham số \-Force (Ví dụ: Get-ChildItem \-Force).

Bước 3: Sao chép và đổi tên file trong 1 lệnh duy nhất

Giả định file mẫu nằm tại đường dẫn chung C:\\Users\\Public\\template.ps1, câu lệnh tối ưu là:

powershell  
Copy-Item \-Path "C:\\Users\\Public\\template.ps1" \-Destination "\~\\smart-farm\\bin\\pump\_control.ps1"

Hãy thận trọng khi sử dụng mã.

* Giải thích: Lệnh Copy-Item cho phép bạn vừa sao chép vừa chỉ định một cái tên hoàn toàn mới tại tham số \-Destination.

Bước 4: Kiểm tra lại toàn bộ cấu trúc thư mục

Để kiểm tra hiển thị trực quan toàn bộ các thư mục con và bao gồm cả file ẩn .env, ta chạy lệnh:

powershell  
Get-ChildItem \-Path \~\\smart-farm \-Recurse \-Force

.

II. XỬ LÝ RÀNG BUỘC (Lỗi Quyền Truy Cập Đọc/Ghi)

Khi gặp lỗi "Access to the path is denied" ở bước sao chép, nghĩa là tài khoản hiện tại không đủ đặc quyền hệ thống để can thiệp vào thư mục đích hoặc nguồn (Public).

Cách xử lý để chạy quyền Quản trị viên (Administrator):

Cách 1 (GUI): Tắt cửa sổ PowerShell hiện tại. Nhấp chuột phải vào biểu tượng PowerShell trên thanh Taskbar/Start Menu → Chọn "Run as Administrator".

Cách 2 (Dòng lệnh trực tiếp): Nếu đang thao tác trên thiết bị SoC không có màn hình (Windows Server Core), gõ ngay lệnh sau vào Terminal hiện tại để kích hoạt một tiến trình PowerShell quyền Admin mới:  
powershell  
Start-Process powershell \-Verb RunAs

III. BẢN PHÂN TÍCH LUỒNG IPO (INPUT \- PROCESS \- OUTPUT)

Dưới đây là bảng phân tích luồng xử lý dữ liệu cho hệ thống cấu hình SoC thiết lập căn cứ số này:

| Thành phần | Đầu vào (INPUT) | Quá trình xử lý (PROCESS) | Đầu ra (OUTPUT) |
| ----- | ----- | ----- | ----- |
| Bước 1: Tạo cấu trúc cây thư mục | \- Ký tự điều hướng Home (\~) \- Tên các thư mục: bin, config, data | Nhận lệnh mkdir, phân tích mảng chuỗi đường dẫn và yêu cầu hệ thống cấp phát không gian lưu trữ đĩa. | Tạo thành công cấu trúc thư mục lồng nhau: smart-farm/ chứa 3 thư mục con. |
| Bước 2: Khởi tạo file mật khẩu cảm biến | \- Tên file .env \- Thuộc tính hệ thống \-Attributes Hidden | Lệnh New-Item ghi nhận vị trí ghi file tại bộ nhớ đĩa, đồng thời gán nhãn bit ẩn (Hidden) vào siêu dữ liệu (metadata) của file. | File cấu hình bí mật .env nằm ẩn trong thư mục config. |
| Bước 3: Cấu hình Script điều khiển bơm | \- File gốc template.ps1 ở thư mục Public. \- Đường dẫn và tên mới pump\_control.ps1. | Lệnh Copy-Item đọc luồng dữ liệu nhị phân từ file mẫu, sao chép sang vùng nhớ mới thuộc thư mục bin và đổi tên định danh. | File thực thi pump\_control.ps1 xuất hiện sẵn sàng trong thư mục bin. |
| Bước 4: Kiểm tra và Duyệt lỗi hệ thống | \- Lệnh kiểm tra quét sâu \-Recurse \-Force. \- Cơ chế xác thực quyền Admin (RunAs). | Quét toàn bộ bảng chỉ mục file (MFT), bỏ qua bộ lọc ẩn để liệt kê sạch dữ liệu. Nếu thiếu đặc quyền sẽ chuyển tiếp yêu cầu xin cấp quyền từ Kernel. |  |

