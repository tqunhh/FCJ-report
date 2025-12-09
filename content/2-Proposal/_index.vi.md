---
title: "Bản đề xuất"
date: "2025-10-10"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# Travel Journal
## Giải pháp AWS Serverless cho nhật ký du lịch

### 1. Tóm tắt điều hành  
Travel Journal Web ra đời với mong muốn giúp mỗi người lưu giữ hành trình của cuộc đời — không chỉ là những chuyến đi, mà còn là ký ức, cảm xúc và câu chuyện đằng sau từng bức ảnh. Ứng dụng kết nối con người với trải nghiệm, biến mỗi chuyến đi thành một phần trong “bản đồ ký ức” của riêng họ.

Người dùng có thể tải lên hình ảnh, ghi chú địa điểm, và hệ thống tự động nhận diện loại cảnh (biển, núi, thành phố…) nhờ Amazon Rekognition. Dữ liệu hành trình được hiển thị trực quan trên bản đồ thời gian thực bằng Amazon Location Service, mang đến trải nghiệm sinh động và gắn kết.

Nền tảng tận dụng sức mạnh của AWS Serverless Architecture (Lambda, API Gateway, S3, DynamoDB, Cognito), đảm bảo hiệu năng cao, bảo mật mạnh mẽ và khả năng mở rộng linh hoạt 

### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  
Nhiều người yêu du lịch muốn lưu giữ hành trình của mình, nhưng các nền tảng hiện nay chỉ cho phép đăng ảnh hoặc ghi chú rời rạc, thiếu sự kết nối trực quan giữa cảm xúc, hình ảnh và địa điểm thực tế. Việc sắp xếp kỷ niệm trở nên khó khăn, không thể xem lại hành trình trên bản đồ hay thống kê được số điểm đến, thời gian và trải nghiệm theo từng chuyến đi. Bên cạnh đó, các nền tảng lưu trữ đám mây phổ biến lại không cá nhân hóa trải nghiệm cho từng người dùng. 

*Giải pháp*  
Travel Journal Web được xây dựng dưới dạng ứng dụng web, tận dụng kiến trúc AWS Serverless để tối ưu hiệu năng và chi phí.

Hệ thống sử dụng Amazon S3 lưu trữ hình ảnh và dữ liệu tĩnh, được phân phối toàn cầu qua Amazon CloudFront. Amazon Cognito quản lý đăng nhập an toàn, trong khi API Gateway và AWS Lambda xử lý logic phía máy chủ. Hình ảnh tải lên S3 được phân tích bởi Amazon Rekognition, lưu kết quả và vị trí vào Amazon DynamoDB, và hiển thị trực quan qua Amazon Location Service.

Toàn bộ hệ thống được giám sát bằng Amazon CloudWatch, theo dõi thông lượng, lỗi, độ trễ và dung lượng cơ sở dữ liệu; cảnh báo được gửi qua SNS. Các dữ liệu và khóa bảo mật được quản lý bởi AWS KMS và Secrets Manager.

Giải pháp này mang đến một ứng dụng du lịch thông minh, bảo mật và tiết kiệm, giúp người dùng dễ dàng lưu giữ và xem lại hành trình của mình mọi lúc, mọi nơi.

*Lợi ích và hoàn vốn đầu tư (ROI)* 
Giải pháp giúp người dùng dễ dàng lưu trữ và chia sẻ hành trình du lịch, đồng thời tạo nền tảng dữ liệu để mở rộng thành ứng dụng du lịch cộng đồng trong tương lai. Với chi phí chỉ khoảng 14.55 USD/tháng, ứng dụng có thể phục vụ 100–200 người dùng mà không cần máy chủ vật lý. Thời gian hoàn vốn ước tính 6–8 tháng, nhờ tiết kiệm chi phí vận hành, bảo trì và lưu trữ ảnh tập trung.  

### 3. Kiến trúc giải pháp  
Hệ thống Travel Journal Web được xây dựng hoàn toàn trên kiến trúc AWS Serverless, tối ưu hiệu năng, bảo mật và khả năng mở rộng. Giao diện web tĩnh được lưu trữ trên Amazon S3, phân phối toàn cầu qua Amazon CloudFront và bảo vệ bởi AWS WAF, ACM và Route 53. Người dùng xác thực thông qua Amazon Cognito, trong khi Amazon API Gateway kết hợp AWS Lambda đảm nhiệm xử lý nghiệp vụ phía máy chủ. Ảnh người dùng tải lên được lưu tại Amazon S3 và xử lý tự động qua hàng đợi Amazon SQS, AWS Lambda, Amazon Rekognition và Amazon Location Service. Kết quả được lưu trữ trong Amazon DynamoDB và phân phối lại qua S3. Hệ thống hỗ trợ cơ chế retry, DLQ, gửi thông báo bằng Amazon SNS, giám sát tập trung bằng Amazon CloudWatch và AWS X-Ray, đồng thời bảo mật dữ liệu bằng AWS IAM, KMS và Secrets Manager.

![Travel journal Architecture](/images/travel_journal_architecture.jpg)


*Dịch vụ AWS sử dụng*  
- Amazon Route 53: Quản lý tên miền và định tuyến truy cập toàn cầu.
- AWS Certificate Manager: Cấp phát và quản lý chứng chỉ SSL/TLS cho các endpoint bảo mật.
- Amazon CloudFront: Phân phối nội dung tĩnh và động với độ trễ thấp.
- AWS WAF: Bảo vệ ứng dụng khỏi các mối đe dọa web phổ biến.
- AWS Lambda: Xử lý sự kiện và logic phía máy chủ mà không cần quản lý hạ tầng.
- Amazon API Gateway: Trung gian giữa frontend và backend, tiếp nhận yêu cầu từ người dùng và chuyển đến AWS Lambda.
- Amazon S3: Lưu trữ hình ảnh, dữ liệu người dùng, và nhật ký hoạt động.
- Amazon DynamoDB: Lưu trữ dữ liệu phi quan hệ về hành trình, địa điểm và thông tin bài viết, tối ưu tốc độ truy cập.
- Amazon Cognito: Quản lý xác thực và quyền truy cập người dùng.
- Amazon Rekognition: Phân tích, nhận diện hình ảnh.
- Amazon Location Service: Cung cấp dịch vụ định vị và bản đồ.
- Amazon SNS: Gửi thông báo đến người dùng và quản trị viên.
- Amazon SQS (Main Queue): Đệm các yêu cầu xử lý ảnh được gửi từ S3 trước khi kích hoạt Lambda.
- Dead Letter Queue (DLQ): Lưu trữ các thông điệp lỗi hoặc sự kiện thất bại từ SQS để phục vụ xử lý và giám sát sau này.
- Amazon CloudWatch: Giám sát hoạt động, log và hiệu năng dịch vụ.
- AWS IAM: Quản lý quyền truy cập, cấp vai trò cho Lambda, API Gateway và dịch vụ AWS khác.
- AWS KMS: Mã hóa dữ liệu ở trạng thái lưu trữ và truyền tải, tăng cường bảo mật.
- AWS Secrets Manager: Lưu trữ và mã hóa thông tin bí mật
- AWS CodeBuild: Biên dịch, kiểm thử và đóng gói mã nguồn tự động.
- AWS CodePipeline: Tự động hóa toàn bộ quy trình CI/CD — từ commit, build, test đến triển khai ứng dụng lên môi trường AWS.


*Thiết kế thành phần*  
- Xác thực người dùng: Amazon Cognito đảm nhiệm đăng nhập, quản lý token và phân quyền.
- Xử lý logic ứng dụng: AWS Lambda tiếp nhận yêu cầu từ API Gateway để lưu hành trình, tải ảnh và phân tích dữ liệu.
- Quản lý dữ liệu: Amazon DynamoDB lưu thông tin chuyến đi, OpenSearch hỗ trợ tìm kiếm và truy vấn nhanh.
- Xử lý hàng đợi: Amazon SQS (Main Queue) tiếp nhận các yêu cầu xử lý ảnh từ S3 trước khi gọi Lambda; Dead Letter Queue (DLQ) lưu các thông điệp thất bại để xử lý sau.
- Phân tích hình ảnh: Amazon Rekognition nhận diện nội dung và gắn nhãn tự động.
- Dữ liệu bản đồ & định vị: Amazon Location Service theo dõi vị trí và hiển thị bản đồ.
- Lưu trữ nội dung: Amazon S3 lưu trữ ảnh, dữ liệu người dùng và tệp tĩnh; nội dung được phân phối toàn cầu thông qua Amazon CloudFront (được bảo vệ bởi AWS WAF, SSL/TLS qua ACM, và định tuyến bởi Route 53).
- Giám sát & thông báo: Amazon CloudWatch giám sát log và hiệu năng; SNS gửi cảnh báo và thông báo đến người dùng.
- Phân phối & bảo mật web: AWS IAM quản lý quyền truy cập giữa các dịch vụ; AWS Secrets Manager và KMS bảo vệ thông tin nhạy cảm.
- Triển khai & CI/CD: AWS CodeBuild và CodePipeline tự động hóa quy trình build, test và triển khai ứng dụng.

### 4. Triển khai kỹ thuật  
*Các giai đoạn triển khai*  
Dự án được chia thành hai phần chính — phát triển ứng dụng web và tích hợp hạ tầng AWS — với bốn giai đoạn triển khai cụ thể:

1. Phân tích yêu cầu và thiết kế kiến trúc: Nghiên cứu các dịch vụ AWS phù hợp (CloudFront, WAF, Cognito, DynamoDB , Lambda, API Gateway, S3...) và vẽ sơ đồ kiến trúc tổng thể (Tháng 1).
2. Tính toán chi phí và mô phỏng hệ thống: Ước tính chi phí từng dịch vụ bằng AWS Pricing Calculator, thử nghiệm quy trình xử lý ảnh và vị trí để kiểm tra tính khả thi. Tối ưu kiến trúc và tài nguyên: Giảm số request, tận dụng cache CloudFront và tối ưu dung lượng S3 để giảm chi phí; tinh chỉnh Lambda và API Gateway cho hiệu năng cao (Tháng 2).
3. Phát triển, kiểm thử và triển khai: Lập trình ứng dụng web, kiểm thử bảo mật bằng WAF và giám sát hoạt động bằng CloudWatch (Tháng 3).

*Yêu cầu kỹ thuật*  
- Frontend: Ứng dụng web xây dựng bằng React.js, triển khai tĩnh trên Amazon S3, phân phối toàn cầu qua CloudFront giúp tăng tốc độ và giảm tải backend.
- Bảo mật & truy cập: AWS WAF chống tấn công web; AWS Certificate Manager (ACM) cung cấp chứng chỉ SSL/TLS, Amazon Cognito quản lý xác thực người dùng (email, OAuth2).
- Backend: API Gateway tiếp nhận yêu cầu, chuyển đến AWS Lambda xử lý nghiệp vụ như ghi nhật ký, tải ảnh, truy vấn dữ liệu.
- Xử lý hàng đợi: Amazon SQS (Main Queue) đệm các yêu cầu xử lý ảnh được kích hoạt từ S3, trong khi Dead Letter Queue (DLQ) lưu trữ thông điệp lỗi để xử lý sau.
- Cơ sở dữ liệu: Amazon DynamoDB lưu trữ dữ liệu hành trình, bài viết, và thông tin người dùng
- Phân tích ảnh: Amazon Rekognition nhận diện cảnh vật, khuôn mặt, gợi ý nhãn ảnh.
- Định vị & bản đồ: Amazon Location Service trực quan hóa tọa độ GPS trên bản đồ tương tác.
- Giám sát hệ thống: CloudWatch thu thập log, AWS Secrets Manager vàKMS bảo vệ thông tin nhạy cảm.
- Thông báo: Amazon SNS gửi cảnh báo khi có lỗi hệ thống hoặc người dùng mới.
- Triển khai & CI/CD: AWS CodeBuild và AWS CodePipeline tự động hóa quy trình build, test và triển khai ứng dụng.

### 5. Lộ trình & Mốc triển khai  
- *Thực tập (Tháng 1–3)*:  
    - Tháng 1: Học AWS và tích lũy kiến thức 
    - Tháng 2: Thiết kế, tính chi phí và điều chỉnh kiến trúc.  
    - Tháng 3: Triển khai, kiểm thử, đưa vào sử dụng.  
- *Sau triển khai*: Nghiên cứu thêm trong vòng 1 năm.  

### 6. Ước tính ngân sách  
Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=e732e6253c20390cbb8d794f05b4a951dec9d574)
  

*Chi phí hạ tầng* 
- Amazon Route 53: 0,50 USD/tháng 
- AWS Certificate Manager: 0,0 USD/tháng
- Amazon CloudFront: 0,61 USD/tháng
- AWS WAF: 0,6 USD/ tháng
- AWS Lambda: 0,01 USD/tháng
- Amazon API Gateway: 0,45 USD/tháng
- Amazon S3: 1,47 USD/tháng
- Amazon DynamoDB: 16,35 USD/tháng
- Amazon Cognito: 5,00 USD/tháng
- Amazon Rekognition: 10,08 USD/tháng
- Amazon Location Service: 4,35 USD/tháng
- Amazon SNS: 2,58 USD/tháng
- Amazon SQS (Main Queue): 3,1 USD/tháng
- Amazon CloudWatch: 6,87 USD/tháng
- AWS IAM: 0,00 USD/tháng
- AWS Secrets Manager: 0,4 USD/tháng
- AWS KMS: 2,3 USD/tháng
- AWS CodeBuild: 0,8 USD/tháng
- AWS CodePipeline: 0,00 USD/tháng

*Tổng*: 61,78 USD/tháng, 946,56 USD/năm


### 7. Đánh giá rủi ro  
*Ma trận rủi ro*  
- Vi phạm bảo mật hoặc mất quyền truy cập người dùng: Ảnh hưởng cao, xác suất rất thấp.
- Tăng chi phí do Rikognition: Ảnh hưởng cao, xác suất trung bình
- Sai lệch dữ liệu vị trí: Ảnh hưởng cao, xác suất thấp

*Chiến lược giảm thiểu*  
- Mạng: Sử dụng Amazon CloudFront để phân phối nội dung nhanh và ổn định, hạn chế phụ thuộc vào kết nối khu vực.
- Hạ tầng: Tận dụng cơ chế tự động khởi động lại hàm Lambda và lưu cache trên trình duyệt để giảm gián đoạn dịch vụ.
- Bảo mật: Áp dụng xác thực đa lớp qua Amazon Cognito và phân quyền truy cập chặt chẽ cho tài nguyên S3.
- Chi phí: Thiết lập cảnh báo ngân sách qua AWS Budgets, tối ưu hóa Lambda và S3 theo truy cập thực tế.

*Kế hoạch dự phòng*  
- Tích hợp AWS SNS & CloudWatch Alerts để gửi thông báo ngay khi hệ thống gặp sự cố (ví dụ: lỗi Lambda, quá tải API Gateway, vượt ngân sách).
- Sử dụng CloudFormation để khôi phục nhanh toàn bộ cấu hình dịch vụ khi gặp sự cố nghiêm trọng.
- Lưu trữ bản sao dữ liệu hình ảnh và nhật ký trên S3 phiên bản sao lưu (S3 Versioning) để tránh mất mát.  

### 8. Kết quả kỳ vọng  
*Cải tiến kỹ thuật*:Ứng dụng giúp người dùng tự động lưu trữ, phân tích và hiển thị hành trình du lịch trên bản đồ thay vì ghi chép thủ công.  
*Giá trị dài hạn*:Tạo ra kho dữ liệu hành trình, hình ảnh và cảm xúc du lịch phong phú — nền tảng cho các ứng dụng gợi ý địa điểm, cá nhân hóa trải nghiệm. 