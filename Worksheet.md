# Nhóm: C401-01

## Thành viên
- Vũ Việt Dũng (2A202600444)
- Vũ Tiến Thành (2A202600443)
- Phan Thị Mai Phương (2A202600281)
- Phạm Minh Trí (2A202600264)
- Nguyễn Mậu Lân (2A202600400)

---

# I. Worksheet 0: Learning Timeline

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

# II. Worksheet 1: Enterprise Deployment Clinic

## a) Xác định bối cảnh tổ chức/khách hàng sử dụng hệ thống

- **Tổ chức:**  
  Ngân hàng thương mại triển khai AI assistant để chuẩn hóa và tăng tốc xử lý nghiệp vụ. 

- **Khách hàng (End-user):**
  - Giao dịch viên tại quầy  
  - Nhân viên Back-office  
  - Bộ phận Compliance  

---

## b) Liệt kê dữ liệu mà hệ thống sẽ động đến

-	Policy (Chính sách) nội bộ của ngân hàng.
-	Biểu mẫu, sổ tay hướng dẫn nghiệp vụ tín dụng (SOP).
-	Hồ sơ khách hàng cá nhân/doanh nghiệp.
-	Tài liệu về các tiêu chuẩn Compliance (Tuân thủ) và KYC (Nhận biết khách hàng).
 

---

## c) Mức độ nhạy cảm dữ liệu

- **Mức độ:** Cực kỳ nhạy cảm (Very High)

- **Dẫn chứng:**
  - Chứa PII (CCCD, lịch sử giao dịch, hạn mức tín dụng)
  - Vi phạm có thể dẫn tới:
    - Thiệt hại kinh tế  
    - Vi phạm pháp lý (Nghị định 13/2023/NĐ-CP)  
    - Vi phạm quy định Ngân hàng Nhà nước  

---

## d) 3 ràng buộc enterprise lớn nhất

-	Không được đưa toàn bộ dữ liệu lên Public Cloud: Dữ liệu lõi phải nằm trong vùng an toàn của ngân hàng.
-	Bắt buộc có Audit Trail: Phải truy vết được mọi hành động của AI và người dùng.
-	Tích hợp hệ thống cũ (Legacy Systems): Phải kết nối được với hệ thống Core Banking hoặc CRM nội bộ đang có.
  

---

## e) Lựa chọn kiến trúc

- **Mô hình:** Hybrid  

- **Lưu ý thiết kế:**
  - Áp dụng Clean Architecture  
  - Tách biệt:
    - Domain Layer (On-premise)  
    - Infrastructure Layer (Cloud LLM API)  

---

## f) Lý do chọn Hybrid

- **Lý do 1 – Bảo mật & tuân thủ:**
  - Vector DB + Data Masking nằm On-premise  
  - Không đưa dữ liệu thô ra ngoài  

- **Lý do 2 – Tối ưu chi phí:**
  - Không cần đầu tư GPU lớn  
  - Dùng Cloud LLM qua Private Link (có cam kết không lưu trữ dữ liệu của doanh nghiệp) 

---

# III. Worksheet 2: Cost Anatomy Lab

## a) Traffic ước lượng:

- Khoảng 500 nhân sự (bao gồm Giao dịch viên và Chuyên viên tín dụng).  
- ~10 requests/user → **5,000 requests/ngày**  
- Peak: **150–200 req/phút**

---

## b) Ước lượng Tokens (Khi sử dụng LLM API Enterprise):
Quy định nội bộ và SOP tín dụng của ngân hàng thường rất dài. Kể cả khi hệ thống đã áp dụng kỹ thuật tiền xử lý (semantic chunking) tốt để cắt nhỏ tài liệu, lượng ngữ cảnh đưa vào vẫn khá lớn.

- **Input:** ~4,000 tokens/request (Bao gồm System prompt + Câu hỏi + Các đoạn SOP trích xuất)
- **Output:** ~300 tokens/request (Câu trả lời ngắn gọn, đi thẳng vào quy định).

---

## c) Cost Layers:

- **Token**: Chi phí gọi Cloud LLM.
- **Compute**: Server chạy Backend (ví dụ: các service viết bằng Go/Python), hệ thống Embedding model, và Worker xử lý Message Queue.
Storage: Chi phí cho Vector Database và PostgreSQL lưu trữ dữ liệu người dùng.
- **Human review**: Nguồn lực chuyên gia Compliance để kiểm duyệt ngẫu nhiên chất lượng câu trả lời.
- **Logging**: Hệ thống lưu trữ Audit Log bảo mật cao (ví dụ: ELK stack hoặc Splunk).
- **Maintenance**: Chi phí duy trì Data Pipeline để liên tục cập nhật SOP mới. 

---

## d) Cost MVP (1 tháng)

| Thành phần | Phân tích | Mức giá |
|----------|--------|--------|
| Token API | 100k requests (~400M input tokens và 30M output tokens)| $1,500 - $2,000 |
| Compute & Storage | Cụm server Backend và Vector Database chạy On-premise | $400 - $500 |
| Logging | Lưu trữ log định danh cho 100k requests với chuẩn bảo mật cao | $150 - $200 |
| Human Review | 1 SME part-time | $500 |
| **Tổng** |  | **~$2,550 - $3,200** |

---

## e) Khi scale 5x–10x

- Khi lưu lượng tăng lên 50k request/ngày, token cost tăng mạnh nhất theo cấp số nhân
- Storage (log) tăng theo thời gian, vì ngân hàng bắt buộc phải lưu trữ lịch sử truy vấn trong nhiều năm mà không được xóa  
- Compute tăng ít hơn nếu scale tốt  

---

## Câu hỏi phân tích

### 1. Cost driver lớn nhất của hệ thống là gì?

- **Token Input API**: Các câu hỏi về pháp chế ngân hàng đòi hỏi độ chính xác tuyệt đối, buộc hệ thống RAG phải nhồi một lượng ngữ cảnh (context) rất dài và chi tiết vào prompt để tránh hiện tượng ảo giác (hallucination).

---

### 2. Hidden cost (Chi phí ẩn) nào dễ bị quên nhất?

- **Data Pipeline Maintenance**: Tài liệu ngân hàng cập nhật liên tục (biểu mẫu mới, lãi suất mới). Việc bóc tách file PDF scan, làm sạch dữ liệu, chia nhỏ (chunking) và vector hóa lại tốn rất nhiều tài nguyên Compute và công sức của Data Engineer. Nếu không tổ chức mã nguồn theo chuẩn (như Clean Architecture) để tách biệt lớp xử lý dữ liệu ngay từ đầu, nợ kỹ thuật sẽ sinh ra chi phí bảo trì khổng lồ.

- **Human-in-the-loop**: AI không tự ra quyết định tín dụng. Ngân hàng bắt buộc phải duy trì nguồn lực con người để phê duyệt và đối soát các kết quả do AI gợi ý.

---

### 3. Đội có chỗ nào đang ước lượng quá lạc quan không?

- **Data Quality**
  - Kỹ sư thường nghĩ rằng chỉ cần ném file tài liệu nội bộ vào hệ thống là AI sẽ đọc được. Thực tế, quy định ngân hàng thường chứa nhiều bảng biểu phức tạp, file PDF dạng scan hoặc có chữ ký nháy đè lên chữ. Việc xử lý (parsing) các định dạng này chuẩn xác cực kỳ khó khăn và tốn chi phí phát triển lớn.

- **Cache Hit Rate**
  - Nhóm có thể nghĩ rằng việc dùng Semantic Caching sẽ tiết kiệm 50% tiền API. Nhưng trong nghiệp vụ tín dụng, mỗi hồ sơ có các thông số (hạn mức, tài sản đảm bảo) khác nhau, tỷ lệ câu hỏi lặp lại y hệt (Cache Hit) thực tế sẽ thấp hơn kỳ vọng rất nhiều. 

---

# IV. Worksheet 3: Cost Optimization Debate

Việc chọn chiến lược phải đảm bảo cân bằng giữa bài toán chi phí và rủi ro tuân thủ (Compliance). Dưới đây là 3 chiến lược phù hợp nhất được lựa chọn và phân tích chi tiết:

## a) Model Routing

Chiến lược này áp dụng tư duy thiết kế hệ thống đa tác tử (multi-agent). Ta sẽ có một model đóng vai trò "Giám sát" (Supervisor) ở cổng vào để điều phối các yêu cầu.
- **Tiết kiệm**: Cắt giảm mạnh chi phí Token API. Không dùng dùng model đắt tiền cho mọi câu hỏi.
- **Lợi ích**: Phân luồng hiệu quả. Supervisor sẽ phân tích ý định (intent):
  - Nếu nhân viên hỏi xin link tải biểu mẫu -> Điều phối về Rule-based flow hoặc model nhỏ giá rẻ.
  - Nếu nhân viên yêu cầu phân tích rủi ro hồ sơ tín dụng -> Điều phối lên model suy luận cao (đắt tiền hơn).
- **Trade-off (Đánh đổi)**: Tăng độ phức tạp của kiến trúc hệ thống. Sẽ có một độ trễ nhỏ (latency) ở nhịp phân loại đầu tiên trước khi trả kết quả.
- **Thời điểm áp dụng**: Khi lượng request đủ lớn và các luồng nghiệp vụ bắt đầu phân nhánh rõ ràng.


---

## b) Semantic Caching

Khác với cache truyền thống (phải gõ đúng y hệt từng chữ), Semantic Cache lưu trữ câu trả lời dựa trên ý nghĩa của câu hỏi.

-	**Tiết kiệm**: Cắt giảm cả chi phí Token (không phải gọi lại LLM) và Compute (không phải chạy lại luồng RAG truy xuất Vector DB) cho các câu hỏi trùng lặp.
-	**Lợi ích**: Tốc độ phản hồi (latency) gần như ngay lập tức. Rất hữu ích vào mùa cao điểm (ví dụ: cuối tháng chốt KPI, nhiều nhân viên cùng hỏi một quy định SOP).
-	**Trade-off (Đánh đổi)**: Rủi ro lưu trữ dữ liệu cũ (Stale Data). Trong ngân hàng, nếu chính sách thay đổi (ví dụ: cập nhật hạn mức vay mới) mà bộ phận Ops chưa kịp xóa cache, AI sẽ tư vấn sai số liệu cũ, dẫn đến hậu quả nghiêm trọng.
-	**Thời điểm áp dụng**: Cần thiết kế ngay từ đầu, nhưng chỉ áp dụng cho các luồng thông tin ít biến động (như quy định PCCC, sổ tay văn hóa).


---

## c) Smaller/Self-hosted model khi volume đủ lớn 

Đưa các mô hình mã nguồn mở (như Llama 3 8B) về chạy trực tiếp trên máy chủ nội bộ của ngân hàng.

-	**Tiết kiệm**: Triệt tiêu hoàn toàn chi phí biến đổi (Token API) trả cho các bên thứ ba. Càng nhiều user hỏi, chi phí trung bình trên mỗi request càng giảm.
-	**Lợi ích**: Đảm bảo bảo mật 100%. Dữ liệu nghiệp vụ nhạy cảm tuyệt đối không rời khỏi hạ tầng (On-premise) của ngân hàng.
-	**Trade-off (Đánh đổi)**: Cần chi phí đầu tư ban đầu (Capex) cực lớn để mua máy chủ GPU. Đòi hỏi đội ngũ kỹ sư nội bộ cứng tay để duy trì, tối ưu hiệu suất và tinh chỉnh (fine-tune) mô hình.
-	**Thời điểm áp dụng**: Khi hệ thống đã vượt qua pha thử nghiệm (MVP) và bước vào giai đoạn scale mạnh (Growth), lượng traffic ổn định ở mức rất cao.


---

##  Lộ trình

### Làm ngay (Giai đoạn MVP):
- Model Routing: Cần làm ngay từ đầu bằng các quy tắc (rule-based) đơn giản để phân loại intent, định hình cấu trúc hệ thống rõ ràng.
-	Semantic Caching: Áp dụng ngay với các câu hỏi FAQ đóng, giúp giảm tải hệ thống.
###	Để sau (Giai đoạn Scale):
- Self-hosted model: Chỉ triển khai khi dự án đã chứng minh được giá trị (ROI), ngân hàng đồng ý rót vốn đầu tư hạ tầng máy chủ và lượng user tăng gấp 5 - 10 lần. Dùng Cloud LLM ở pha đầu để đi nhanh hơn.
 

---

# V. Worksheet 4: Scaling & Reliability Tabletop

## a) Phân tích 3 tình huống

### Tình huống A: Traffic tăng đột biến (Lưu lượng tăng vọt)

- **Bối cảnh:**  
  Giai đoạn chốt hồ sơ cuối tháng hoặc khi có chính sách lãi suất mới, hàng loạt giao dịch viên cùng truy vấn hệ thống.

- **Tác động tới User:**  
  Hệ thống bị nghẽn cổ chai (bottleneck) tại Vector Database On-premise, người dùng bị văng ra ngoài hoặc web bị treo. Quy trình giải ngân bị đình trệ.

- **Phản ứng ngắn hạn:**  
  Áp dụng hệ thống Queue (Hàng đợi). Các câu hỏi sẽ được đưa vào hàng đợi. Người dùng nhận được thông báo:  
  > "Hệ thống đang xử lý hồ sơ, vui lòng chờ trong giây lát..."  
  thay vì báo lỗi sập trang.

- **Giải pháp dài hạn:**  
  Thiết lập cơ chế Auto-scaling (Tự động mở rộng) cho các node Backend xử lý RAG khi CPU hoặc Queue Depth (độ sâu hàng đợi) vượt ngưỡng 70%.

---

### Tình huống B: Provider timeout (Cloud LLM Provider bị sập/lỗi)

- **Bối cảnh:**  
  Dịch vụ API của nhà cung cấp LLM (ví dụ: OpenAI, Anthropic) gặp sự cố diện rộng.

- **Tác động tới User:**  
  Giao dịch viên gửi câu hỏi nhưng hệ thống quay vòng vòng rồi báo lỗi 504 Gateway Timeout. Trải nghiệm cực kỳ tệ.

- **Phản ứng ngắn hạn:**  
  Kích hoạt cơ chế Circuit Breaker (Cầu dao tự động). Khi phát hiện API lỗi 3 lần liên tiếp, hệ thống ngắt kết nối ngay lập tức để tránh treo server nội bộ và hiển thị thông báo lỗi chuẩn mực.

- **Giải pháp dài hạn:**  
  Thiết lập cấu hình Multi-Provider (Đa nhà cung cấp). Nếu Provider A sập, hệ thống tự động định tuyến (route) lệnh gọi API sang Provider B để đảm bảo dịch vụ xuyên suốt.

---

### Tình huống C: Response chậm (Phản hồi quá lâu)

- **Bối cảnh:**  
  Câu hỏi phức tạp đòi hỏi RAG pipeline phải quét qua cuốn cẩm nang tín dụng dài 500 trang, mất hơn 30 giây để trả lời.

- **Tác động tới User:**  
  Giao dịch viên mất kiên nhẫn, tắt tab và quay lại thói quen cũ là gọi điện thoại hỏi trực tiếp quản lý (phá vỡ mục tiêu của dự án).

- **Phản ứng ngắn hạn:**  
  Sử dụng kỹ thuật Streaming Response (Trả lời từng chữ) giống ChatGPT để user thấy hệ thống đang làm việc.

- **Giải pháp dài hạn:**  
  Tối ưu hóa lại thuật toán Search trong Vector Database (chuyển từ Exact k-NN sang Approximate Nearest Neighbor - ANN) và tăng cường Semantic Caching.

---

## b) Metric cần monitor:

Bộ phận IT/Ops của ngân hàng cần thiết lập Dashboard (ví dụ: Grafana, Datadog) để theo dõi các chỉ số sinh tồn sau:

- **Latency (Độ trễ):**  
  Theo dõi p95 và p99 latency (thời gian phản hồi của 95% và 99% request). Nếu p95 > 5 giây, hệ thống đang có vấn đề.

- **Error Rate (Tỷ lệ lỗi):**  
  Theo dõi mã lỗi 4xx (lỗi từ user/phân quyền) và 5xx (lỗi hệ thống/Provider).

- **Queue Depth (Độ sâu hàng đợi):**  
  Số lượng request đang bị kẹt lại chưa được xử lý.

- **Token Usage / Cost API:**  
  Giám sát chi phí theo thời gian thực để chặn ngay nếu có dấu hiệu bị spam request.

---

## c) Fallback Proposal

### Real-time vs Async:

- **Real-time (Thời gian thực):**  
  Các câu hỏi tra cứu SOP ngắn gọn, kiểm tra rule KYC cơ bản  
  (Ví dụ: "Hạn mức chuyển tiền không OTP là bao nhiêu?").

- **Async (Bất đồng bộ):**  
  Các yêu cầu tổng hợp, đối chiếu hồ sơ tín dụng lớn  
  (Ví dụ: "Phân tích rủi ro hồ sơ công ty X dựa trên BCTC năm ngoái").  
  Sẽ trả về thông báo:  
  > "Agent đang tổng hợp dữ liệu, báo cáo sẽ được gửi qua email nội bộ sau 5 phút".

---

### Queue, Circuit Breaker, Retry Policy:

- Bắt buộc dùng cả 3.  
  - Dùng Queue để hứng traffic spike.  
  - Dùng Circuit Breaker để bảo vệ server khi API ngoài lỗi.  
  - Dùng Exponential Backoff Retry (thử lại với độ trễ tăng dần) khi gặp lỗi mạng chập chờn tạm thời.

---

### Mô hình Fallback 3 Lớp (Graceful Degradation):

- **Lớp 1 (Primary):**  
  Gọi Cloud LLM đắt tiền (Chất lượng tốt nhất).

- **Lớp 2 (Fallback 1 - Model khác):**  
  Nếu LLM chính sập → Gọi API dự phòng  
  (Ví dụ chuyển từ GPT-4o sang Claude 3).

- **Lớp 3 (Fallback 2 - Rule-based):**  
  Nếu rớt mạng hoàn toàn → Trả về thanh công cụ tìm kiếm từ khóa (Elasticsearch) thông thường, cung cấp link PDF gốc để nhân viên tự đọc.

- **Lớp Đặc biệt (Human Escalation):**  
  Nếu câu hỏi chứa rủi ro pháp lý cao hoặc AI chấm điểm "Confidence Score" (Độ tự tin) thấp → Tự động chuyển tiếp (escalate) câu hỏi dưới dạng Ticket (phiếu hỗ trợ) cho chuyên viên Compliance xử lý. Tuyệt đối không để AI tự đoán bừa.