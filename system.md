# Thiết kế hệ thống
https://devopscube.com/aws-client-vpn/

## 1. Kiến trúc logic

Hệ thống được tổ chức thành bốn lớp chính:

1. **Lớp người dùng và thiết bị (User & Endpoint Layer)**  
   - Thiết bị người dùng (laptop, PC, điện thoại) kết nối Internet từ mạng bên ngoài.  
   - Cài đặt VPN client để thiết lập đường hầm an toàn tới hạ tầng cloud/on‑prem.[1]

2. **Lớp kết nối VPN và biên mạng (VPN & Perimeter Layer)**  
   - **Remote Access VPN Gateway** (ví dụ AWS Client VPN endpoint hoặc OpenVPN server):  
     - Thiết lập tunnel mã hóa (SSL/IPsec) giữa thiết bị và mạng nội bộ/cloud.[2][1]
     - Xác thực bước đầu (username/password, certificate…).  
   - **Firewall / Security Group**:  
     - Giới hạn traffic từ subnet VPN chỉ đến các dịch vụ được phép, đặc biệt là **Spring Boot Access Gateway**.  
     - Áp dụng nguyên tắc “deny by default, allow by exception”.[3][2]

3. **Lớp Zero Trust Access Gateway (Application Access Layer)**  
   - **Spring Boot Zero Trust Access Gateway**:  
     - Cung cấp endpoint HTTP/HTTPS cho người dùng sau khi đã vào VPN.  
     - Tích hợp với **Identity Provider (IdP)** qua OIDC/SAML để thực hiện xác thực và MFA.[4][5]
     - Triển khai **policy engine** để đánh giá yêu cầu truy cập dựa trên:  
       - Identity (user, role, group).  
       - Ngữ cảnh (IP, thời gian, loại thiết bị, mức độ nhạy cảm của tài nguyên).  
       - Trạng thái phiên (session, token còn hạn hay không).  
     - Quyết định cho phép/từ chối hoặc yêu cầu tăng cường xác thực (step‑up MFA).  
     - Ghi **audit log** cho mọi sự kiện truy cập.  

   - **Policy Store / Configuration DB**:  
     - Cơ sở dữ liệu lưu: người dùng, vai trò, nhóm, danh sách ứng dụng, và ma trận quyền truy cập.  
     - Lưu log và sự kiện phục vụ phân tích và kiểm toán.

4. **Lớp ứng dụng và dữ liệu nội bộ (Internal Applications & Data Layer)**  
   - Các **Internal Business Applications** (REST API, web app…) chỉ chấp nhận truy cập từ gateway hoặc thông qua cơ chế token đã được xác thực.  
   - CSDL nội bộ (VD: PostgreSQL/MySQL) lưu trữ dữ liệu nghiệp vụ.  
   - Hệ thống logging/monitoring (ELK, Prometheus/Grafana, CloudWatch,…) thu thập log từ VPN gateway, Spring Boot gateway và ứng dụng.[6][7]

Mô hình này phản ánh đúng tinh thần Zero Trust: **không tin cậy mặc định bất cứ kết nối nào**, mọi truy cập đều đi qua lớp kiểm soát trung tâm, sử dụng **least‑privilege** và **continuous verification**.[8][9][4]

## 2. Use case 1: Nhân viên truy cập hợp lệ vào ứng dụng nội bộ

### Mục tiêu

Nhân viên A làm việc từ xa muốn truy cập vào ứng dụng quản lý đơn hàng nội bộ một cách an toàn, có MFA và được giới hạn đúng phạm vi công việc.

### Luồng xử lý

1. **Thiết lập kết nối VPN**  
   - Nhân viên A mở VPN client, nhập thông tin đăng nhập/certificate.  
   - VPN Gateway xác thực và thiết lập tunnel mã hóa, cấp cho thiết bị một địa chỉ IP trong VPN subnet.[1]

2. **Truy cập Gateway**  
   - A mở trình duyệt, truy cập URL của Spring Boot Zero Trust Access Gateway (`https://gateway.internal`).  
   - Yêu cầu HTTP đi qua tunnel VPN, firewall, tới gateway.

3. **Xác thực và MFA**  
   - Gateway phát hiện chưa có phiên hợp lệ → redirect đến IdP.  
   - A đăng nhập username/password; IdP yêu cầu **MFA** (OTP/app/push) theo chính sách.  
   - Sau khi thành công, IdP cấp **ID token và Access token (JWT)** trả lại cho gateway.[9][4]

4. **Đánh giá policy Zero Trust**  
   - Gateway phân tích token: user = A, role = SALES, group = SALES_TEAM.  
   - Lấy thêm ngữ cảnh: IP thuộc VPN subnet, truy cập trong giờ hành chính, từ quốc gia được phép.  
   - Policy engine kiểm tra:  
     - Role SALES được phép truy cập ứng dụng “Order Management”.  
     - Không được truy cập ứng dụng “Finance”.  
   - Nếu điều kiện thỏa, gateway tạo session và lưu thông tin phiên trong DB/cache.

5. **Truy cập ứng dụng nội bộ**  
   - A chọn menu “Quản lý đơn hàng”.  
   - Request tới resource `/orders/**` được gửi tới gateway kèm token/session.  
   - Gateway kiểm tra lại token + policy cho resource cụ thể, sau đó forward request tới ứng dụng Order App (kèm thông tin user đã được chuẩn hóa).  
   - Order App xử lý nghiệp vụ và trả kết quả về cho gateway, rồi gateway trả cho A.

6. **Ghi log và giám sát**  
   - Mọi sự kiện (login, MFA thành công, truy cập resource) được log với timestamp, user, IP, resource.  
   - Log được gửi sang hệ thống monitoring để phục vụ phân tích, phát hiện bất thường và audit sau này.[3][6]

Kết quả: Nhân viên A truy cập được đúng ứng dụng cần thiết, qua kênh mã hóa và xác thực mạnh, nhưng không có quyền “đi lang thang” trong toàn bộ mạng nội bộ.

## 3. Use case 2: Truy cập bị từ chối do vi phạm chính sách

### Mục tiêu

Tài khoản B (nhân viên bình thường) cố truy cập ứng dụng tài chính nhạy cảm nhưng bị chặn bởi chính sách Zero Trust.

### Luồng xử lý

1. **Đăng nhập tương tự use case 1**  
   - B kết nối VPN, login qua IdP + MFA thành công, nhận token và thiết lập session ở gateway.

2. **Yêu cầu truy cập ứng dụng tài chính**  
   - B cố truy cập vào URL tương ứng với Finance App trên gateway.  
   - Request đến gateway kèm token/session hiện tại.

3. **Đánh giá policy và từ chối**  
   - Gateway phân tích token: user = B, role = STAFF, không thuộc nhóm FINANCE_ADMIN.  
   - Policy engine kiểm tra bảng quyền:  
     - Ứng dụng “Finance” chỉ được phép cho nhóm FINANCE_ADMIN.  
   - Do không thỏa điều kiện, gateway trả về HTTP 403 Forbidden và trang thông báo “Bạn không có quyền truy cập ứng dụng này”.  

4. **Ghi nhận sự kiện bảo mật**  
   - Sự kiện truy cập bị từ chối được ghi log kèm thông tin user, IP, target resource.  
   - Nếu B cố gắng truy cập nhiều lần, hệ thống có thể kích hoạt rule cảnh báo hoặc tạm thời khóa tài khoản theo cấu hình nâng cao.

Use case này minh họa khả năng **granular access control** và **least‑privilege** của kiến trúc: dù đã qua VPN và MFA, người dùng vẫn không thể truy cập vượt quá phạm vi được gán, đúng với triết lý “never trust, always verify”.[5][10][4]

## 4. Tóm tắt đặc điểm kiến trúc

- Kết hợp **Remote Access VPN** (bảo mật kênh truyền) với **Zero Trust Access Gateway** (kiểm soát ở tầng ứng dụng).  
- Mọi quyết định truy cập được đưa ra dựa trên **identity + ngữ cảnh**, không chỉ dựa vào việc “đã ở trong mạng nội bộ”.[11][3]
- Sử dụng **MFA** và **continuous verification** để giảm thiểu rủi ro account bị đánh cắp hoặc thiết bị bị xâm nhập.  
- Dễ mở rộng: có thể thêm nhiều ứng dụng nội bộ, policy chi tiết hơn, hoặc tích hợp thêm hệ thống SIEM/IDS/IPS trong tương lai.