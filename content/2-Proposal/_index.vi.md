---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# ĐỀ XUẤT KIẾN TRÚC HỆ THỐNG (SYSTEM ARCHITECTURE PROPOSAL)
## DỰ ÁN: AI E-COMMERCE PLATFORM — AWS SERVERLESS ARCHITECTURE

---

### 1. INTRODUCTION & PROBLEM STATEMENT (Giới thiệu & Đặt vấn đề)

#### 1.1. Xu hướng công nghệ Cloud-native và Decoupled Architecture
Bước sang năm 2026, ngành thương mại điện tử (TMĐT) toàn cầu đang chứng kiến sự chuyển dịch mạnh mẽ từ các kiến trúc nguyên khối (Monolithic) truyền thống sang các mô hình **Cloud-native** và **kiến trúc phân rã (Decoupled/Event-Driven Architecture)**. Sự cạnh tranh khốc liệt đòi hỏi các nền tảng bán lẻ trực tuyến phải phản hồi cực nhanh trước thay đổi của thị trường, có khả năng mở rộng không giới hạn vào các dịp mua sắm cao điểm và giảm thiểu tối đa thời gian chết (downtime). 

Kiến trúc phân rã (Decoupled) giúp tách biệt hoàn toàn giữa lớp hiển thị (Frontend) và logic nghiệp vụ xử lý ở Backend. Kết hợp với các mô hình thiết kế hướng sự kiện (Event-Driven), hệ thống có khả năng xử lý bất đồng bộ các tác vụ nặng, giảm tải cho luồng giao tiếp đồng bộ và tăng cường khả năng chịu lỗi tối đa.

#### 1.2. Các bài toán khó khăn của mô hình truyền thống
Trong quá trình vận hành các hệ thống TMĐT sử dụng mô hình truyền thống (chạy trên máy chủ ảo tự quản lý hoặc cơ sở dữ liệu quan hệ nguyên khối), các doanh nghiệp thường đối mặt với các thách thức lớn sau:
- **Quá tải kết nối khi traffic tăng đột biến:** Trong các chương trình Flash Sale hoặc chiến dịch khuyến mãi lớn, lượng người dùng truy cập tăng vọt khiến máy chủ Web/API và cơ sở dữ liệu nhanh chóng chạm ngưỡng giới hạn kết nối (Connection Limit), gây ra tình trạng nghẽn cổ chai và sập hệ thống.
- **Chi phí vận hành và lãng phí tài nguyên rỗi cao:** Hệ thống phải duy trì cấu hình phần cứng lớn để dự phòng cho các đợt cao điểm, dẫn đến việc tài nguyên máy chủ bị bỏ trống (idle) vào những khung giờ thấp điểm, gây lãng phí ngân sách vận hành rất lớn.
- **Rủi ro thanh toán trùng lặp (Double Charging):** Khi mạng chập vượt ngưỡng hoặc người dùng bấm nút thanh toán nhiều lần, hệ thống thiếu cơ chế kiểm soát xử lý trùng lập (Idempotency), dẫn đến việc gửi nhiều yêu cầu thanh toán giống nhau lên cổng thanh toán, làm tổn hại nghiêm trọng đến trải nghiệm khách hàng và uy tín doanh nghiệp.
- **Lỗ hổng bảo mật và tấn công mạng:** Việc tự quản lý hạ tầng tăng nguy cơ bị tấn công từ chối dịch vụ (DDoS), tấn công ứng dụng web (SQL Injection, XSS) và rò rỉ dữ liệu nhạy cảm do thiếu các lớp lọc traffic chuyên dụng và phân quyền IAM không chặt chẽ.
- **Quản lý trạng thái và đồng bộ đơn hàng bất đồng bộ phức tạp:** Trong mô hình kết nối đồng bộ trực tiếp (tight coupling), nếu dịch vụ thông báo (Notification) hoặc xử lý đơn hàng gặp sự cố, toàn bộ luồng Checkout sẽ bị chặn đứng, ảnh hưởng trực tiếp đến tỷ lệ chuyển đổi đơn hàng.

---

### 2. PROJECT OBJECTIVES (Mục tiêu dự án)

Dự án được xây dựng nhằm giải quyết triệt để các bài toán khó khăn trên thông qua việc thiết kế và phát triển thành công một nền tảng **AI E-Commerce Platform** tối ưu với các mục tiêu cụ thể:
- **Khả năng tự động mở rộng (Autoscaling):** Đảm bảo hệ thống có khả năng tự co giãn tài nguyên linh hoạt theo thời gian thực để đáp ứng lưu lượng truy cập lớn mà không cần quản trị viên can thiệp thủ công.
- **Tối ưu hóa chi phí vận hành (Pay-as-you-go):** Áp dụng mô hình chi phí tính theo mức độ sử dụng thực tế của AWS Serverless, giảm chi phí vận hành xuống mức tối thiểu, loại bỏ hoàn toàn chi phí cho tài nguyên rỗi.
- **Đảm bảo các nghiệp vụ cốt lõi vững chắc:**
  - Hiển thị danh mục sản phẩm và hình ảnh nhanh chóng từ Amazon S3 thông qua CDN.
  - Quản lý định danh, đăng ký, đăng nhập và phân quyền người dùng bảo mật cao bằng Amazon Cognito (cấp phát JWT token).
  - Quy trình tạo và quản lý đơn hàng diễn ra bất đồng bộ thông qua hàng đợi tin nhắn, giảm tải tối đa cho hệ thống frontend.
  - Tích hợp cổng thanh toán trực tuyến Stripe Sandbox an toàn tuyệt đối, áp dụng cơ chế chống thanh toán trùng lập (`Idempotency-Key`) và xác minh chữ ký Webhook (`Webhook Signature Verification`).
  - Hỗ trợ trải nghiệm mua sắm thông minh thông qua Chatbot tích hợp AI sử dụng Amazon Lex và hệ thống thông báo đa kênh (Email/Push/In-app) bất đồng bộ.

---

### 3. PROPOSED SYSTEM ARCHITECTURE (Kiến trúc hệ thống đề xuất)

#### 3.1. Lý do lựa chọn mô hình AWS Serverless
Thay vì sử dụng các mô hình Container (như ECS, EKS) hoặc Microservices truyền thống đòi hỏi nỗ lực quản lý và bảo trì hạ tầng lớn, dự án đề xuất kiến trúc **AWS Serverless nâng cao (v3)** vì các ưu điểm vượt trội:
- **Giảm bán kính ảnh hưởng của sự cố (Blast Radius):** Mỗi dịch vụ nghiệp vụ (Lambda function) hoạt động độc lập và chỉ thực hiện một nhiệm vụ chuyên biệt. Sự cố xảy ra tại một thành phần (ví dụ: lỗi gửi thông báo) sẽ bị cô lập hoàn toàn và không ảnh hưởng đến các luồng nghiệp vụ khác (luồng đặt hàng, thanh toán vẫn chạy bình thường).
- **Tăng tốc trải nghiệm nhà phát triển (Developer Experience - DX):** Lập trình viên tập trung 100% vào phát triển logic nghiệp vụ thay vì cấu hình mạng, nâng cấp hệ điều hành, hay thiết lập cân bằng tải. Việc sử dụng hạ tầng dưới dạng mã nguồn (IaC) thông qua AWS CDK (Cloud Development Kit) giúp quản lý tài nguyên đồng bộ, nhất quán và dễ dàng tự động hóa CI/CD.
- **Độ tin cậy cao:** Dịch vụ tích hợp sẵn tính năng High Availability và Multi-AZ trên toàn bộ các thành phần như API Gateway, Lambda, SQS, EventBridge và DynamoDB.

#### 3.2. Sơ đồ kiến trúc hệ thống chính thức (v3)

![AWS Serverless E-commerce Architecture v3](/images/2-Proposal/architecture.png)

> **Lưu ý triển khai:** AWS WAF được cấu hình ở tầng biên để bảo vệ ứng dụng, liên kết trực tiếp với CloudFront (phân phối bởi AWS Amplify) và REST endpoints của Amazon API Gateway.

#### 3.3. Mô tả chi tiết luồng dữ liệu (Data Flow)
Hệ thống vận hành theo 7 luồng dữ liệu chính được thiết kế tối ưu:
1. **Luồng 1 - Truy cập Website:** Người dùng truy cập domain của hệ thống -> Route 53 phân giải DNS -> Web Application Firewall (AWS WAF) kiểm tra và lọc request dựa trên các Managed Rules và Rate Limit -> Yêu cầu hợp lệ đi tới AWS Amplify trả về Next.js SSR/Static Content từ CDN.
2. **Luồng 2 - Xác thực người dùng:** Người dùng chọn đăng ký/đăng nhập qua Cognito Hosted UI hoặc Cognito SDK -> Trả về mã bảo mật JWT sau khi xác thực thành công -> Client đính kèm JWT vào header `Authorization: Bearer <token>` gửi đến API Gateway -> API Gateway sử dụng Cognito Authorizer để giải mã và xác minh JWT trước khi chuyển tiếp yêu cầu đến backend.
3. **Luồng 3 - Tra cứu sản phẩm:** Frontend gọi `GET /products` hoặc `GET /products/{id}` -> API Gateway chuyển tiếp đến Product Service (Lambda) -> Truy vấn metadata sản phẩm từ DynamoDB và lấy hình ảnh trực tiếp từ Amazon S3 qua Pre-signed URL an toàn.
4. **Luồng 4 - Tạo và xử lý đơn hàng:** Người dùng gửi yêu cầu đặt hàng -> API Gateway -> Order Service (Lambda) ghi nhận trạng thái khởi tạo vào DynamoDB -> Đẩy tác vụ vào **SQS Order Queue** để xử lý bất đồng bộ -> Consumer Lambda xử lý đơn hàng và xóa tin nhắn khỏi hàng đợi khi thành công -> Nếu lỗi vượt quá số lần retry cho phép (`maxReceiveCount`), tin nhắn tự động chuyển vào **Order DLQ (Dead-Letter Queue)** để bảo toàn dữ liệu và phục vụ điều tra lỗi.
5. **Luồng 5 - Thanh toán:** Client gọi `POST /checkout/payment-intent` -> Checkout Service (Lambda) kiểm tra giỏ hàng và tương tác với Stripe API -> Tạo Stripe Payment Intent sử dụng `Idempotency-Key` duy nhất nhằm chống thanh toán lặp -> Lưu thông tin thanh toán vào DynamoDB -> Nhận phản hồi trạng thái từ Stripe Webhook (được xác minh chữ ký bảo mật từ Stripe) -> Checkout Service phát bản tin sự kiện `order.placed` lên Amazon EventBridge.
6. **Luồng 6 - Kiến trúc hướng sự kiện (Event-Driven) & Thông báo:** EventBridge Custom Bus nhận event `order.placed` và phân phối đến các mục tiêu dựa trên Rules:
   - Rule 1: Kích hoạt Order Service cập nhật trạng thái đơn hàng thành "Paid" trong DynamoDB.
   - Rule 2: Gửi sự kiện đến **Notification SQS Queue** -> Kích hoạt Notification Service (Lambda) gửi thông báo (Email/Push) tới người dùng -> Tin nhắn lỗi trong luồng gửi thông báo được đưa vào **Notification DLQ**.
7. **Luồng 7 - Chatbot AI:** Người dùng đặt câu hỏi trên UI chatbot -> API Gateway chuyển tiếp đến Chatbot Backend (Lambda) -> Lambda giao tiếp với Amazon Lex để phân tích ý định (Intent) và trả về phản hồi thông minh, hỗ trợ tra cứu đơn hàng/sản phẩm bằng cách gọi API nội bộ bằng quyền IAM bảo mật.

#### 3.4. Các giải pháp bảo mật tích hợp
- **Cognito Authorizer:** Chặn đứng và lọc các request chưa xác thực ngay từ lớp API Gateway, ngăn chặn tấn công khai thác API backend.
- **Mã hóa dữ liệu:** Sử dụng AWS KMS để mã hóa toàn bộ dữ liệu lưu trữ tại DynamoDB, S3 và các cấu hình nhạy cảm.
- **Phát hiện mối đe dọa:** Amazon GuardDuty liên tục phân tích log từ CloudTrail, VPC Flow Logs và DNS Logs để phát hiện các hoạt động đáng ngờ hoặc tấn công xâm nhập hệ thống.
- **Kiểm soát quyền truy cập chặt chẽ (Least Privilege):** Mỗi Lambda function được cấu hình một IAM Role riêng biệt, chỉ cấp quyền truy cập tối thiểu đến tài nguyên cần thiết.
- **Quản lý Secrets:** Stripe API Keys và các thông tin nhạy cảm khác được lưu trữ tập trung tại AWS Secrets Manager, không hard-code trong mã nguồn.

---

### 4. TEAM DIVISION & WORKLOAD (Phân công công việc)

Nhóm gồm 05 thành viên được phân chia vai trò rõ ràng, cân bằng và công bằng nhằm tối ưu hóa năng lực cá nhân và đảm bảo tiến độ dự án:

| Thành viên | Vai trò | Trách nhiệm chính | Sản phẩm bàn giao (Deliverables) |
|---|---|---|---|
| **Thành viên 1** | **Frontend & Identity Engineer** | - Phát triển giao diện Web bằng Next.js trên AWS Amplify.<br>- Tích hợp Amazon Cognito Hosted UI và SDK cho luồng xác thực.<br>- Xây dựng giao diện giỏ hàng, Checkout, và Chatbot UI.<br>- Quản lý Route Protection (phân quyền client-side). | - Mã nguồn ứng dụng Frontend.<br>- Luồng đăng ký/đăng nhập hoàn chỉnh.<br>- Giao diện thanh toán & chatbot.<br>- Pipeline deploy frontend trên AWS Amplify. |
| **Thành viên 2** | **API & Compute Engineer** | - Cấu hình Amazon API Gateway (REST API) và Cognito Authorizer.<br>- Phát triển Product Service Lambda (CRUD sản phẩm).<br>- Phát triển API cho Order Service.<br>- Thực hiện kiểm tra tính hợp lệ của request (Request Validation). | - API Contract chi tiết.<br>- Mã nguồn các Lambda Function dịch vụ Product và Order.<br>- Cấu hình API Gateway routing & authorizer.<br>- Bộ tài liệu kiểm thử Unit Test cho API. |
| **Thành viên 3** | **Data & Messaging Engineer** | - Thiết kế cấu trúc bảng DynamoDB theo mô hình Single-Table Design.<br>- Xây dựng cấu trúc lưu trữ hình ảnh trên Amazon S3.<br>- Thiết lập kiến trúc SQS Order Queue + DLQ.<br>- Cấu hình EventBridge Custom Bus & Rules.<br>- Cấu hình chính sách sao lưu tự động AWS Backup cho DynamoDB. | - Thiết kế DynamoDB schema và chỉ mục GSI.<br>- Cấu hình hàng đợi SQS, DLQ và EventBridge rules.<br>- Script tạo tài nguyên dữ liệu và messaging.<br>- Bằng chứng cấu hình AWS Backup và quy trình restore. |
| **Thành viên 4** | **Payment, AI & Notification Engineer** | - Phát triển Checkout Lambda tích hợp Stripe API (Idempotency, Webhook).<br>- Cấu hình Amazon Lex (Intents, Utterances, Slots) cho Chatbot AI.<br>- Xây dựng Notification Service (Lambda consumer từ Notification Queue) để gửi email/tin nhắn. | - Mã nguồn Checkout Lambda và Stripe Webhook handler.<br>- Mô hình Amazon Lex chatbot hoàn chỉnh.<br>- Mã nguồn Notification Lambda.<br>- Bộ tài liệu kiểm thử luồng thanh toán và thông báo. |
| **Thành viên 5** | **DevSecOps & Observability Engineer (Team Lead / PM)** | - Quản trị kiến trúc hệ thống tổng thể, viết IaC sử dụng AWS CDK.<br>- Thiết lập phân quyền IAM Role theo nguyên tắc Least Privilege.<br>- Cấu hình AWS WAF rules, Route 53 và AWS Secrets Manager.<br>- Xây dựng CI/CD Pipeline (GitHub Actions).<br>- Thiết lập hệ thống giám sát: CloudWatch dashboards/alarms, X-Ray tracing, và Amazon GuardDuty.<br>- Quản lý tiến độ dự án (PM), điều phối họp nhóm và chuẩn bị demo. | - Mã nguồn hạ tầng AWS CDK.<br>- Quy trình CI/CD hoàn chỉnh (GitHub Workflows).<br>- Dashboard giám sát CloudWatch, cấu hình X-Ray trace.<br>- Báo cáo bảo mật GuardDuty & cấu hình WAF.<br>- Tài liệu dự án (Architecture Docs, Runbook, Release Plan). |

---

### 5. BUDGET ESTIMATION (Ước tính chi phí sơ bộ trên AWS)

Bảng dưới đây ước tính chi phí cho dự án ở quy mô thử nghiệm phòng lab/học tập (Lab-scale / Development) trong vòng 01 tháng nhằm chứng minh tính kinh tế vượt trội của giải pháp Serverless:

| Dịch vụ AWS | Mức sử dụng ước tính / Tháng | Cách tính chi phí | Chi phí ước tính / Tháng |
|---|---|---|---|
| **AWS Amplify** | - Build & Deploy: 100 phút/tháng<br>- Hosting: 1 GB lưu trữ, 5 GB băng thông | - Build: $0.01/phút<br>- Hosting: $0.02/GB lưu trữ, $0.15/GB truyền tải | $1.00 - $1.75 |
| **Amazon Cognito** | - 50 người dùng hoạt động hàng tháng (MAU) | - Miễn phí cho 50,000 MAU đầu tiên (Cognito User Pools) | $0.00 |
| **Amazon API Gateway** | - 20,000 requests/tháng | - $3.50 mỗi triệu request (REST API) | $0.07 |
| **AWS Lambda** | - 30,000 executions/tháng (Thời gian chạy trung bình 500ms, cấu hình 512MB RAM) | - Nằm trong Free Tier của Lambda (1 triệu request & 400,000 GB-s mỗi tháng) | $0.00 |
| **Amazon DynamoDB** | - 2 GB dữ liệu lưu trữ<br>- 100,000 đọc/ghi (On-demand) | - $0.25/GB lưu trữ<br>- $1.25 mỗi triệu request ghi, $0.25 mỗi triệu request đọc | $0.50 |
| **Amazon S3** | - 5 GB dữ liệu hình ảnh sản phẩm<br>- 5,000 request đọc/ghi | - Standard Storage: $0.023/GB<br>- Request: $0.005 mỗi 1000 PUT, $0.0004 mỗi 1000 GET | $0.15 |
| **Amazon SQS & DLQ** | - 10,000 tin nhắn/tháng | - Miễn phí 1 triệu tin nhắn đầu tiên mỗi tháng | $0.00 |
| **Amazon EventBridge** | - 5,000 sự kiện/tháng | - Miễn phí cho các sự kiện mặc định của AWS; $1.00/triệu sự kiện tùy chỉnh | $0.00 |
| **AWS Secrets Manager** | - 1 Secret (lưu Stripe API Key) | - $0.40/secret/tháng<br>- $0.05 mỗi 10,000 API calls | $0.45 |
| **AWS WAF** | - 1 Web ACL + 2 Rules (để test/demo ngắn hạn) | - $5.00/Web ACL/tháng (pro-rated theo ngày dùng)<br>- $1.00/Rule/tháng (pro-rated) | $1.00 - $2.00 (Chỉ bật khi cần kiểm thử/demo) |
| **AWS Backup & KMS** | - Sao lưu 2 GB DynamoDB | - $0.10/GB sao lưu mỗi tháng | $0.20 |
| **Amazon GuardDuty** | - Quy mô lab, lưu lượng traffic nhỏ | - Miễn phí 30 ngày dùng thử đầu tiên cho tài khoản mới | $0.00 |
| **AWS X-Ray** | - 20,000 traces/tháng | - Miễn phí 100,000 traces đầu tiên mỗi tháng | $0.00 |
| **Stripe Sandbox** | - Tích hợp môi trường Sandbox để giả lập thanh toán | - Hoàn toàn miễn phí | $0.00 |
| **TỔNG CỘNG** | | | **~ $3.37 - $5.12 USD / Tháng** (Rất tiết kiệm và tối ưu cho dự án thực tập) |

---

### 6. RISK ASSESSMENT & MITIGATION (Đánh giá rủi ro và Giải pháp giảm thiểu)

Dự án phân tích và chuẩn bị các phương án xử lý cho các rủi ro kỹ thuật và vận hành chính:

| Rủi ro kỹ thuật | Mức độ | Ảnh hưởng | Giải pháp giảm thiểu | Kế hoạch dự phòng (Contingency) |
|---|---|---|---|---|
| **Vượt giới hạn kết nối (Connection Limit)** khi traffic tăng đột biến | Trung bình | Gây nghẽn cổ chai, treo database, mất dữ liệu đơn hàng. | Thay thế cơ sở dữ liệu quan hệ (RDS) truyền thống bằng **Amazon DynamoDB** - cơ sở dữ liệu NoSQL Serverless tự động điều chỉnh throughput (On-demand mode) và không có khái niệm giới hạn số kết nối đồng thời. | Thiết lập CloudWatch Alarm cảnh báo khi dung lượng đọc/ghi tăng cao bất thường để chủ động kiểm tra. |
| **Mất mát đơn hàng do lỗi Stripe Webhook hoặc thanh toán trùng** | Cao | Khách hàng bị trừ tiền nhưng không nhận được đơn hàng, hoặc bị trừ tiền 2 lần (double charge) cho cùng 1 đơn. | - Áp dụng **Stripe Idempotency-Key** duy nhất cho mỗi giao dịch checkout.<br>- Xác minh chữ ký số (`Stripe-Signature`) của Webhook để đảm bảo request gửi đến từ Stripe.<br>- Sử dụng **DynamoDB Conditional Writes** để chỉ cho phép cập nhật trạng thái đơn hàng khi ở trạng thái hợp lệ.<br>- Sử dụng **SQS DLQ** để lưu trữ các message webhook lỗi. | Thiết lập quy trình chạy lại (Redrive) tin nhắn lỗi từ DLQ về hàng đợi chính để xử lý lại thủ công sau khi sửa lỗi backend. |
| **Rò rỉ API Key và thông tin nhạy cảm** | Cao | Kẻ xấu chiếm quyền điều khiển tài khoản Stripe, AWS; đánh cắp thông tin khách hàng. | Tuyệt đối không commit các file cấu hình chứa key (như `.env`) lên GitHub. Sử dụng **AWS Secrets Manager** để lưu trữ và quản lý bảo mật các API keys. | Cấu hình Secrets Manager tự động xoay vòng key (Automatic Rotation) định kỳ; thu hồi quyền của API key cũ ngay khi phát hiện rò rỉ. |
| **Kiến trúc quá phức tạp dẫn đến trễ tiến độ dự án** | Cao | Không hoàn thành kịp các tính năng theo kế hoạch 12 tuần của nhà trường. | Xác định rõ phạm vi MVP (Minimum Viable Product): Ưu tiên hoàn thiện luồng Product, Order, Checkout và Chatbot Lex trước. Dịch vụ gửi Notification có thể mock nếu thời gian quá hạn hẹp. | Tổ chức họp Daily Sync ngắn (10 phút) để phát hiện sớm các vướng mắc (blockers) kỹ thuật nhằm hỗ trợ kịp thời. |

---

### 7. ACCEPTANCE CRITERIA (Tiêu chí nghiệm thu dự án)

Dự án được nghiệm thu thành công khi đáp ứng đầy đủ các tiêu chí kỹ thuật và chức năng cụ thể dưới đây dựa trên bản thiết kế v3:

1. **Hạ tầng & Domain (Edge & DNS):**
   - [ ] Domain được phân giải thành công qua Route 53, truy cập ứng dụng bắt buộc qua giao thức HTTPS có chứng chỉ SSL hợp lệ.
   - [ ] AWS WAF được cấu hình và hoạt động ổn định: Chặn các request vi phạm rate limit (ví dụ: > 100 requests/phút từ cùng một IP) và lọc traffic dựa trên các rule thông dụng.
   - [ ] Giao diện Next.js được triển khai và cập nhật tự động thành công thông qua AWS Amplify.

2. **Xác thực & API (Identity & API):**
   - [ ] Người dùng có thể đăng ký, đăng nhập nhận mã xác thực qua email từ Amazon Cognito.
   - [ ] Đăng nhập thành công trả về JWT hợp lệ. Các route private trên API Gateway từ chối truy cập (trả về lỗi 401 Unauthorized) nếu thiếu JWT hoặc JWT không hợp lệ thông qua Cognito Authorizer.

3. **Nghiệp vụ Sản phẩm & Đơn hàng (Catalog & Order flow):**
   - [ ] API Product (`GET /products`) đọc dữ liệu thành công từ DynamoDB và hiển thị hình ảnh tải từ Amazon S3.
   - [ ] Luồng tạo đơn hàng sử dụng cơ chế bất đồng bộ: Khi gọi `POST /orders`, request được đẩy vào SQS Order Queue thành công, ghi nhận log trạng thái và xử lý thành công.
   - [ ] Kiểm thử thành công luồng xử lý lỗi: Khi giả lập lỗi xử lý đơn hàng, tin nhắn sau 3 lần retry (hoặc số lần cấu hình) được đưa vào Order DLQ thành công.

4. **Thanh toán & Hướng sự kiện (Payment & Event-driven):**
   - [ ] Luồng Checkout tạo được Stripe Payment Intent thành công trong môi trường Sandbox.
   - [ ] Sử dụng cùng một `Idempotency-Key` gửi 2 request liên tiếp lên cổng thanh toán không tạo ra giao dịch thứ 2.
   - [ ] Stripe Webhook gửi sự kiện thanh toán thành công về Backend, chữ ký Webhook được xác minh chính xác và cập nhật trạng thái đơn hàng trong DynamoDB.
   - [ ] Sự kiện `order.placed` được đẩy lên EventBridge và định tuyến thành công đến Consumer để cập nhật trạng thái và gửi thông báo qua Notification Queue.
   - [ ] Tin nhắn lỗi trong luồng Notification được chuyển vào Notification DLQ thành công.

5. **Chatbot AI (Chatbot Service):**
   - [ ] Người dùng tương tác trực tiếp được với chatbot Amazon Lex trên giao diện web.
   - [ ] Chatbot nhận diện đúng các intent mẫu (ví dụ: Tra cứu đơn hàng, Hỏi đáp sản phẩm) và phản hồi thông tin chính xác từ cơ sở dữ liệu.

6. **Giám sát & Vận hành (Observability & Ops):**
   - [ ] Dashboard CloudWatch hiển thị đầy đủ các metrics chính của API Gateway, Lambda, SQS, DynamoDB.
   - [ ] Cấu hình thành công AWS X-Ray Active Tracing: Hiển thị đầy đủ sơ đồ luồng đi (Trace Map) của request xuyên suốt từ API Gateway -> Lambda -> SQS -> EventBridge -> DynamoDB.
   - [ ] Amazon GuardDuty được kích hoạt và không cảnh báo các lỗ hổng bảo mật nghiêm trọng ở cấp độ AWS Account.
   - [ ] DynamoDB có cấu hình Backup Plan (AWS Backup), chứng minh được bằng chứng khôi phục dữ liệu (Restore Evidence) thành công từ bản sao lưu.

7. **Quản lý mã nguồn & Quy trình (CI/CD & Process):**
   - [ ] Toàn bộ hạ tầng được định nghĩa bằng AWS CDK (IaC), không cấu hình tài nguyên bằng tay trên AWS Console.
   - [ ] Pipeline CI/CD trên GitHub Actions tự động kiểm tra cú pháp (lint), build dự án và deploy lên môi trường AWS khi merge code vào nhánh chính.
   - [ ] Kho chứa mã nguồn (Repository) có đầy đủ lịch sử commit, tài liệu hướng dẫn chạy (README.md) và phân công công việc cụ thể cho từng thành viên.