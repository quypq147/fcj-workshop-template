---
title: "Đột phá stale knowledge của AI Agent với Web Search trên Amazon Bedrock AgentCore"
date: 2026-06-30
weight: 2
draft: false
---

# ĐỘT PHÁ STALE KNOWLEDGE CỦA AI AGENT VỚI WEB SEARCH TRÊN AMAZON BEDROCK AGENTCORE

Trong làn sóng phát triển trí tuệ nhân tạo hiện nay, các AI Agent ngày càng được kỳ vọng sẽ giải quyết các tác vụ phức tạp một cách tự động và thông minh. Tuy nhiên, một trong những giới hạn lớn nhất của các mô hình ngôn ngữ lớn (LLM) và AI Agent hiện tại là hiện tượng **"Stale Knowledge" (Kiến thức bị đóng băng)**. Do kiến thức của mô hình chỉ giới hạn tại thời điểm kết thúc quá trình huấn luyện (training cutoff), AI Agent hoàn toàn "mù tịt" trước các thông tin mang tính thời gian thực (real-time) như giá cổ phiếu hôm nay, kết quả của một trận bóng đá vừa diễn ra, hay các tin tức công nghệ mới cập nhật.

Để phá vỡ rào cản này, AWS đã giới thiệu tính năng **Web Search trên Amazon Bedrock AgentCore** - một giải pháp fully managed và hoàn toàn tương thích với Model Context Protocol (MCP) của Anthropic, giúp đưa dữ liệu web thời gian thực vào các luồng xử lý của AI Agent một cách an toàn và tối ưu nhất.

{{% notice info %}}
**Sự kết hợp hoàn hảo:** Việc đưa Web Search vào Bedrock AgentCore cho phép AI Agent vừa truy xuất được thông tin tĩnh trong doanh nghiệp vừa cập nhật thông tin động từ internet toàn cầu để đưa ra quyết định chính xác nhất.
{{% /notice %}}

---

## Kiến Trúc Hệ Thống Minh Họa

Dưới đây là sơ đồ kiến trúc mô tả cách thức hoạt động của tính năng Web Search trên Amazon Bedrock AgentCore:

![Amazon Bedrock AgentCore Web Search Tool](/images/3-Blog/blog2_1-img.jpg)

---

## Các Điểm Kiến Trúc Nổi Bật của Web Search trên Bedrock AgentCore

Tính năng Web Search này không chỉ đơn thuần là việc gửi query lên một công cụ tìm kiếm thông thường, mà nó được thiết kế tối ưu sâu sắc cho AI Agent với hạ tầng mạnh mẽ của AWS:

1. **Web Index "Chính Chủ" do Amazon Vận Hành:**
   AWS sử dụng một bộ chỉ mục web khổng lồ chứa hàng chục tỷ tài liệu và được cập nhật liên tục sau mỗi vài phút. Điều này đảm bảo AI Agent luôn tiếp cận được những thông tin mới nhất trên internet mà không gặp độ trễ lớn.

2. **Tích Hợp Knowledge Graph (Đồ Thị Tri Thức):**
   Thay vì chỉ tìm kiếm từ khóa thô, Web Search kết hợp với Knowledge Graph của Amazon để ánh xạ thực thể và mối quan hệ giữa chúng, giúp trả về các thông tin thực tế (facts) có độ tin cậy và chính xác cực kỳ cao, giảm thiểu tối đa hiện tượng ảo tưởng (hallucination) của mô hình.

3. **Trích Xuất Đoạn Văn Thông Minh (Semantic Snippet Extraction):**
   Đây là điểm cộng cực lớn cho hiệu năng và chi phí. Thay vì ném toàn bộ mã nguồn raw HTML cồng kềnh của trang web vào prompt (gây lãng phí context window và tăng chi phí token), hệ thống sẽ tự động trích xuất các đoạn văn bản chứa ngữ nghĩa phù hợp nhất (semantic snippets) để gửi cho mô hình.

4. **Bảo Mật Tuyệt Đối (Private by Design):**
   Trong môi trường doanh nghiệp, bảo mật dữ liệu là ưu tiên hàng đầu. Mọi câu truy vấn (queries) từ AI Agent đến Web Search Tool đều được xử lý hoàn toàn trong hạ tầng mạng nội bộ bảo mật của AWS, cam kết không đẩy dữ liệu ra bất kỳ bên thứ ba nào nằm ngoài kiểm soát của doanh nghiệp.

---

## Tóm Tắt Chi Phí & Đánh Giá Cá Nhân

Amazon Bedrock áp dụng mô hình tính phí vô cùng linh hoạt và dễ tiếp cận:
* **Mô hình Pay-as-you-go:** Chi phí cực kỳ tối ưu chỉ **$7 cho mỗi 1,000 queries**.
* **Không phí khởi tạo:** Không có chi phí duy trì cố định hằng tháng, dùng bao nhiêu trả bấy nhiêu.

### Đúc kết cá nhân của một thực tập sinh Cloud:
> Việc kết hợp tính năng Web Search mới này với **Amazon Bedrock Knowledge Bases (RAG)** sẽ tạo nên một bộ đôi "hủy diệt" cho doanh nghiệp. Doanh nghiệp vừa có thể dùng Knowledge Bases để truy vấn tài liệu nội bộ bảo mật (quy trình, chính sách, báo cáo tài chính nội bộ), vừa có thể dùng Web Search để cập nhật nhanh các xu hướng thị trường, đối thủ cạnh tranh bên ngoài. Sự giao thoa này giúp AI Agent hoạt động toàn diện và thông minh hơn bao giờ hết!

---
* **Link bài đăng cộng đồng:** [AWS Study Group Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2193226758108951/)
* **Hashtags:** #AWS #AmazonBedrock #AgentCore #WebSearch #AIAgents #awsstudygroup
