# Nhóm: C401-01

## Thành viên
- Vũ Việt Dũng (2A202600444)
- Vũ Tiến Thành (2A202600443)
- Phan Thị Mai Phương (2A202600281)
- Phạm Minh Trí (2A202600264)
- Nguyễn Mậu Lân (2A202600400)

---

## I. Project Overview

- **Tên dự án:** Bank Operations Copilot  
- **Bài toán thực tế:**  
  Hỗ trợ nhân viên tra cứu nhanh các quy định nội bộ, quy trình KYC, SOP tín dụng và các bước xử lý hồ sơ.  
  Giải quyết tình trạng nghẽn cổ chai trong khâu thẩm định và tuân thủ.  

- **Người dùng chính (End-users):**  
  - Giao dịch viên  
  - Nhân viên back-office  
  - Bộ phận compliance (tuân thủ)  

- **Dữ liệu lõi:**  
  - Policy nội bộ  
  - Biểu mẫu tín dụng  
  - Hồ sơ khách hàng  
  - Tài liệu compliance  

---

## II. Enterprise Context

Ngành ngân hàng đòi hỏi tiêu chuẩn vận hành ở mức cao nhất. Hệ thống phải giải quyết 4 ràng buộc cốt lõi:

- **Bảo mật tối đa:**  
  Dữ liệu hồ sơ khách hàng và policy là thông tin cực kỳ nhạy cảm.  
  Không được phép đưa toàn bộ dữ liệu này lên public cloud.

- **Truy vết (Audit Trail):**  
  Bắt buộc phải có audit trail.  
  Mọi truy vấn của giao dịch viên phải được lưu log định danh để phục vụ thanh tra.

- **Tích hợp hệ thống:**  
  Phải kết nối được với các hệ thống legacy (hệ thống cũ) như Core Banking hoặc CRM nội bộ.

- **Kiểm duyệt nghiệp vụ:**  
  AI không được tự quyết định.  
  Cần xác định rõ ở đâu bắt buộc phải có human review (con người kiểm duyệt).

---

## III. Deployment Choice

- **Lựa chọn:**  
  Triển khai theo mô hình Hybrid (Lai) kết hợp Data Anonymization (Ẩn danh dữ liệu).

### Kiến trúc hệ thống

- **On-premise (Nội bộ):**  
  - Vector Database  
  - Hệ thống phân quyền  
  - Bộ lọc ẩn danh dữ liệu (Data Masking / PII Redaction)  

  → Các dữ liệu nhạy cảm (Tên khách hàng, CCCD, số dư) tuyệt đối không đưa vào prompt.

- **Cloud (Đám mây):**  
  - Chỉ gửi các đoạn text đã được ẩn danh  
  - Ví dụ:  
    > "Khách hàng hạng A, thu nhập X, cần vay Y theo SOP nào?"  

  → Gửi lên Private Cloud LLM có cam kết Enterprise để xử lý logic và sinh câu trả lời.

---

## IV. Cost Anatomy

- **Chi phí MVP (Thử nghiệm với nhóm nhỏ):**  
  - Thiết lập hạ tầng bảo mật On-premise  
  - Tích hợp API với hệ thống legacy  
  - Xây dựng luồng RAG  
  - Chi phí token LLM thấp  

- **Chi phí Growth (Scale toàn hệ thống):**
  - **Cost driver lớn nhất:**  
    - Nâng cấp server/phần cứng On-premise  
    - Xử lý mã hóa/giải mã dữ liệu liên tục  
    - Lưu trữ log Audit Trail trong nhiều năm  

  - **Hidden cost:**  
    - Chi phí nhân sự (SME)  
    - Review và tinh chỉnh câu trả lời AI liên quan đến SOP tín dụng  

---

## V. Optimization Plan

- **Semantic Caching:**  
  Lưu cache cho các truy vấn phổ biến về KYC không chứa thông tin cá nhân.

- **Model Routing (Định tuyến phân tầng):**  
  - Dùng model nhỏ (self-hosted, Python/Go)  
  - Kiểm tra intent và rule bảo mật  
  - Chỉ route các câu hỏi phức tạp lên LLM lớn  

- **Selective Inference:**  
  - Giới hạn context trong prompt  
  - RAG chỉ trích xuất 2–3 điều khoản liên quan nhất  
  - Giảm chi phí token  

---

## VI. Reliability Plan

- **Human-in-the-loop:**  
  - AI chỉ đóng vai trò Copilot  
  - Gợi ý và trích dẫn quy định  
  - Bắt buộc có bước **Approve** từ chuyên viên compliance  

- **Fallback khi Provider lỗi (Timeout):**  
  - Tự động chuyển sang hệ thống tra cứu rule-based  
  - Đảm bảo không gián đoạn dịch vụ  

- **Scaling:**  
  - Áp dụng Clean Architecture  
  - Tách biệt Core Domain và Infrastructure  
  - Scale service RAG qua Queue  
  - Không ảnh hưởng Core Banking  

---

## VII. Track Recommendation + Next Step

- **Track đề xuất:** AI Engineering / Application  

- **Phase 2 – Multi-agent System:**

  - Xây dựng **Supervisor Agent**  
    → Tiếp nhận và phân tích câu hỏi  

  - Điều phối đến các **Worker Agents**:
    - **KYC Agent:** kiểm tra rule định danh  
    - **Credit SOP Agent:** tính toán và đối chiếu hạn mức  
    - **Compliance Agent:** rà soát từ khóa cấm  

  → Thiết kế giúp:
  - Dễ bảo trì  
  - Dễ debug  
  - Cô lập rủi ro bảo mật tốt hơn  