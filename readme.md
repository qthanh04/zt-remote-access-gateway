# Zero Trust Remote Access with VPN & MFA

Hệ thống mẫu triển khai **truy cập từ xa an toàn** cho ứng dụng nội bộ doanh nghiệp, sử dụng **VPN remote access**, **Multi‑Factor Authentication (MFA)** và một lớp **Spring Boot Zero Trust Access Gateway** để kiểm soát truy cập ở tầng ứng dụng.

## Tính năng chính

- **VPN Remote Access**: Thiết lập kênh kết nối được mã hóa từ thiết bị người dùng tới mạng nội bộ/cloud (ví dụ AWS Client VPN hoặc OpenVPN).  
- **Xác thực đa yếu tố (MFA)**: Tích hợp với Identity Provider (IdP) hỗ trợ OIDC/SAML để bắt buộc MFA khi đăng nhập (OTP, app, push…).[web:115][web:82]  
- **Zero Trust Access Gateway (Spring Boot)**:  
  - Kiểm tra token, danh tính, role, group và ngữ cảnh truy cập.  
  - Áp dụng chính sách least‑privilege, chỉ cho truy cập vào đúng ứng dụng/dịch vụ được phép.[web:118][web:121]  
  - Ghi log chi tiết các sự kiện truy cập để phục vụ audit và phát hiện bất thường.  
- **Internal Applications**: Một hoặc nhiều ứng dụng nội bộ giả lập (REST API/ứng dụng web) chỉ có thể truy cập thông qua gateway.  

## Kiến trúc tổng quan

Luồng truy cập cơ bản:

1. Người dùng kết nối VPN từ thiết bị cá nhân đến **VPN Gateway**, được cấp IP trong subnet VPN.  
2. Người dùng truy cập URL của **Spring Boot Zero Trust Access Gateway**.  
3. Gateway redirect người dùng đến IdP để **login + MFA**, nhận lại JWT token sau khi xác thực thành công.[web:112][web:120]  
4. Gateway đánh giá chính sách (user, role, IP, thời gian, loại tài nguyên…) và tạo phiên truy cập nếu hợp lệ.  
5. Mọi request tới ứng dụng nội bộ phải đi qua gateway, nơi token và quyền được kiểm tra lại trước khi forward.

Kiến trúc này kết hợp ưu điểm của **VPN (mã hóa, tunneling)** và **Zero Trust (kiểm soát theo ứng dụng, không tin cậy theo mạng)**.[web:118][web:119]

## Công nghệ sử dụng (gợi ý)

- **Backend**: Spring Boot (Java), Spring Security, Spring Data.  
- **Auth & MFA**: IdP hỗ trợ OIDC/SAML (AWS Cognito, Keycloak, Okta,…).  
- **VPN**: AWS Client VPN / OpenVPN / tương đương.  
- **Database**: PostgreSQL/MySQL cho lưu trữ cấu hình policy, user mapping, log.  
- **Infra**: Docker, docker‑compose hoặc AWS (VPC, Subnet, Security Group,…).  

## Đối tượng sử dụng

- Môi trường lab của sinh viên/nhà nghiên cứu để tìm hiểu Zero Trust remote access.  
- Doanh nghiệp vừa và nhỏ muốn tham khảo mô hình secure remote access hiện đại trước khi triển khai thực tế.[web:111][web:118]

---