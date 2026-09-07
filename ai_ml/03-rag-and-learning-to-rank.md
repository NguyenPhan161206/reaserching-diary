# Hệ thống RAG và Mô hình Học xếp hạng (Learning to Rank) trong Tuyển dụng

**Ngày tạo:** 2026-09-07
**Tags:** `#rag`, `#learning-to-rank`, `#lambdamart`, `#cross-encoder`, `#information-retrieval`, `#llm`
**Tài liệu tham khảo/Nguồn:** Khóa Luận Tốt Nghiệp (Võ Ngọc Trâm Anh, CNTT khoá 2022-2026)

---

## 📌 Tóm tắt ý chính (TL;DR)
- **Học xếp hạng (LTR):** Sử dụng các thuật toán như LambdaMART và Cross-Encoder để tối ưu hóa thứ tự hiển thị của các ứng viên, ưu tiên độ chính xác ở nhóm đầu (đánh giá bằng NDCG).
- **Kiến trúc RAG:** Kết hợp truy xuất thông tin (Retrieval) và LLM (Generation) để đánh giá ứng viên dựa trên ngữ cảnh thực tế, giảm thiểu nội suy vô căn cứ (hallucination).
- **Quản trị dữ liệu & Hệ thống:** Thiết kế lưu trữ 2 tầng (In-memory cho truy xuất nhanh, PostgreSQL JSONB cho lưu trữ kết quả linh hoạt) và kiến trúc xử lý bất đồng bộ (FastAPI) để tối ưu độ trễ.
- **Trích xuất đặc trưng (Feature Engineering):** Kết hợp cả tín hiệu từ vựng (BM25) và ngữ nghĩa (Vector Embedding, TF-IDF) cùng cấu trúc hồ sơ (kinh nghiệm, học vấn) để huấn luyện mô hình LTR.

## 🧠 Nội dung chi tiết

### 1. Phân loại và Tối ưu Xếp hạng
- **Không gian nhãn:** tương ứng Reject (Loại), Hold (Cân nhắc), Shortlist (Vào vòng tiếp theo).
  ![Y = {0, 1, 2}](images/label_space.png)
- **Quan hệ xếp hạng:** 
  ![d_i > d_j](images/preference_rel.png)
  - Nếu bằng điểm, áp dụng quy tắc phá hòa (tie-breaking rules).
- **Mục tiêu tối ưu:** Sử dụng **NDCG@k** làm evaluation metric chính.
  ![NDCG](images/ndcg_formula.png)
  ![DCG](images/dcg_formula.png)
  DCG (Discounted Cumulative Gain) phạt nặng sai lệch ở các vị trí top-k vì nhà tuyển dụng chủ yếu chỉ xem ứng viên đầu danh sách.

### 2. Kiến trúc RAG (Retrieval-Augmented Generation)
Kết hợp trí nhớ phi tham số (hệ thống truy xuất) và tham số (LLM) thông qua 3 giai đoạn:
1. **Indexing (Lập chỉ mục):** Phân đoạn (chunking), chuẩn hóa và lưu trữ dạng vector.
2. **Retrieval (Truy xuất):** Rút trích tài liệu liên quan nhất dựa trên đối sánh.
3. **Generation (Sinh nội dung):** LLM sinh phản hồi dựa trên tài liệu đã truy xuất.
   ![g_{rag}](images/rag_generation.png)

*Để giới hạn khả năng sinh thông tin ảo (hallucination), hệ thống sử dụng dữ liệu CV dạng JSON và snippets đối chiếu, ràng buộc chặt chẽ không gian suy luận của LLM.*

### 3. Mô hình Học xếp hạng (LTR) đa tầng
Tối ưu hóa thứ tự tương đối giữa các tài liệu.

**a. LambdaMART:**
- Dựa trên Gradient Boosted Decision Trees.
- Sử dụng $\lambda$-gradients để xấp xỉ hướng tối ưu cho các hàm không khả vi như NDCG.
- Học theo cặp (pairwise comparisons) và kết hợp đa nguồn tín hiệu (từ vựng, ngữ nghĩa, cấu trúc).

**b. Re-ranking với Cross-Encoder:**
- Giải quyết hạn chế của Bi-Encoder (mã hóa độc lập truy vấn và tài liệu).
- Nối (concatenate) truy vấn và tài liệu thành một chuỗi duy nhất, sử dụng self-attention trong Transformer để tìm mối tương quan sâu ở cấp độ token.
- Chi phí tính toán cao, chỉ áp dụng cho một tập ứng viên nhỏ đã được truy xuất sơ bộ (retrieve-then-rerank).

**c. Khai thác đặc trưng cho LTR (Hybrid Retrieval):**
Tích hợp 4 nhóm đặc trưng chính:
- **Nhóm A (Tín hiệu nền tảng):** Điểm Okapi BM25 (chuẩn hóa), Độ tương đồng vector ngữ nghĩa.
  ![x1](images/feature_x1.png)
  ![x2](images/feature_x2.png)
- **Nhóm B (Tương thích kỹ năng):** Mức tương đồng Jaccard kỹ năng cứng, TF-IDF kỹ năng mềm, tỷ lệ bao phủ yêu cầu bắt buộc.
  ![x3](images/feature_x3.png)
- **Nhóm C (Cấu trúc CV):** Cấp bậc, số năm kinh nghiệm, trình độ học vấn.
- **Nhóm D (Từ vựng bổ trợ):** Điểm BM25 trên chức danh (title), TF-IDF của kỹ năng cứng.

### 4. Đánh giá chất lượng (Evaluation Metrics)
- **Xếp hạng (Retrieval):** 
  - **NDCG:** Xem xét vị trí, ưu tiên tài liệu liên quan ở vị trí cao.
  - **MAP:** Độ chính xác trung bình tích lũy.
  - **MRR:** Vị trí của kết quả đúng đầu tiên.
- **Sinh văn bản (RAGAS framework):**
  - **Faithfulness:** Tính trung thực, không bịa đặt ngoài ngữ cảnh.
  - **Answer Relevancy:** Câu trả lời đúng trọng tâm truy vấn.
  - **Context Precision:** Độ chính xác/hữu ích của ngữ cảnh được trích xuất.

### 5. Thiết kế Hệ thống Thực thi (Model Serving)
- **Quản trị lưu trữ 2 tầng (Dual-layer):**
  1. *In-memory/File-based:* Lưu trữ các chỉ mục tĩnh (BM25 Index) trên RAM để giảm trễ I/O.
  2. *Relational Database (PostgreSQL):* Dùng trường `JSONB` để lưu kết quả LLM bán cấu trúc (phi chuẩn hóa), dễ dàng thay đổi prompt mà không cần migrate DB schema.
- **Kiến trúc bất đồng bộ (Asynchronous):** Dùng FastAPI (Async/Await) để gọi API của LLM bên thứ 3 (GPT-4, Claude) mà không gây nghẽn (blocking) luồng hệ thống.

---

## 🤔 Câu hỏi mở / Điểm cần nghiên cứu thêm
*Đây là các hướng bạn cần tự tìm hiểu sâu hơn trong thời gian tới:*
- [ ] Cơ chế tính toán $\lambda$-gradients trong thuật toán LambdaMART cụ thể hoạt động như thế nào?
- [ ] Cách triển khai (code) một Cross-Encoder kết hợp cùng mô hình Bi-Encoder cho pipeline re-ranking.
- [ ] Cách thiết lập thư viện RAGAS để tự động hóa đánh giá Faithfulness và Answer Relevancy cho các phản hồi LLM.
- [ ] Tối ưu hóa In-memory Indexing trong Python như thế nào để phục vụ việc tính toán các hàm Okapi BM25 tốc độ cao?
