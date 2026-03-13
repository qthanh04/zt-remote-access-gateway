# Giới thiệu dự án

Dự án xây dựng một hệ thống **truy cập từ xa an toàn cho doanh nghiệp** dựa trên mô hình **Zero Trust**, kết hợp **VPN remote access**, **xác thực đa yếu tố (MFA)** và một lớp **Spring Boot Access Gateway** chạy trên môi trường cloud.

Mục tiêu của hệ thống là cho phép nhân viên, đối tác truy cập vào các ứng dụng nội bộ của doanh nghiệp từ bên ngoài mạng một cách an toàn, có kiểm soát chặt chẽ, đồng thời giảm thiểu rủi ro khi tài khoản hoặc thiết bị bị xâm phạm. Thay vì “tin tưởng” bất kỳ kết nối nào nằm trong mạng nội bộ, hệ thống áp dụng nguyên tắc **“never trust, always verify”**: mọi yêu cầu truy cập đều phải được xác thực, ủy quyền và kiểm tra liên tục dựa trên danh tính người dùng, ngữ cảnh truy cập và chính sách bảo mật.[web:112][web:119]

## Bối cảnh và lý do lựa chọn

Trong mô hình truyền thống, VPN thường cấp cho người dùng quyền truy cập gần như toàn bộ mạng nội bộ sau khi đăng nhập thành công, tạo điều kiện cho kẻ tấn công di chuyển ngang nếu chiếm được một tài khoản VPN.[web:106][web:123] Với xu hướng làm việc từ xa, BYOD và đa cloud, cách tiếp cận này không còn phù hợp.

Mô hình **Zero Trust Network Access (ZTNA)** và **Zero Trust Architecture (ZTA)** được các tổ chức lớn khuyến nghị như một cách tiếp cận hiện đại cho secure remote access:  
- Xác thực và ủy quyền dựa trên **identity và ngữ cảnh**, không dựa vào vị trí mạng.  
- Chỉ cấp **least‑privilege access** tới đúng ứng dụng/dịch vụ cần thiết, không mở cả mạng.  
- **Liên tục xác minh** trong suốt phiên truy cập, kết hợp MFA và giám sát hành vi.[web:113][web:118][web:121]

Dự án này hiện thực hóa các nguyên tắc đó trong một kiến trúc cụ thể, có demo chạy được, phù hợp cho môi trường doanh nghiệp vừa và nhỏ sử dụng hạ tầng cloud.

## Phạm vi và đóng góp chính

- Thiết kế kiến trúc **Zero Trust remote access** kết hợp VPN, MFA, Spring Boot gateway và các ứng dụng nội bộ.  
- Hiện thực backend **Spring Boot Access Gateway** làm lớp kiểm soát truy cập trung tâm (policy engine, tích hợp IdP/MFA, audit log).  
- Tích hợp với hạ tầng cloud (ví dụ AWS) để cung cấp đường hầm VPN được mã hóa và phân tách mạng hợp lý.  
- Xây dựng và mô tả 2 use case tiêu biểu:  
  - Truy cập hợp lệ của nhân viên vào ứng dụng nội bộ qua VPN + MFA.  
  - Yêu cầu truy cập bị từ chối do vi phạm chính sách Zero Trust (sai role, sai ngữ cảnh).

Thông qua đó, đồ án thể hiện được năng lực kết hợp giữa **kiến trúc an toàn thông tin** và **kỹ thuật phát triển phần mềm (Spring Boot, API, tích hợp cloud)** trong một bài toán thực tế của doanh nghiệp hiện đại.[web:111][web:115]
