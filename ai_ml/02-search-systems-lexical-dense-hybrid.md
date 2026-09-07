# Hệ thống Tìm kiếm (Search Systems): Lexical, Dense và Hybrid Search

**Ngày tạo:** 2026-09-04
**Ngày bổ sung kiến thức:** 2026-09-07
**Tags:** `#AI`, `#NLP`, `#SearchEngine`, `#VectorDatabase`, `#InformationRetrieval`
**Tài liệu tham khảo/Nguồn:** [Gemini Share](https://share.gemini.google/zEAVGArmeM7e) & [https://share.gemini.google/C3HUVEIvYfGX]

---

## 📌 Tóm tắt ý chính (TL;DR)
- **Lexical Retrieval (BM25):** Phương pháp tìm kiếm truyền thống so khớp mặt chữ (Keyword-based). Cực nhanh, giải thích được, nhưng kém trong việc hiểu đồng nghĩa ngữ cảnh.
- **Dense Retrieval (Vector Search):** Sử dụng các mô hình AI Embeddings để hiểu ngữ nghĩa (Semantic-based). Giải quyết triệt để lỗi chính tả và từ đồng nghĩa.
- **Hybrid Search:** Sự kết hợp sức mạnh của cả Lexical và Dense thông qua thuật toán RRF, giúp hệ thống không bị lọt từ khóa kỹ thuật (keyword) mà vẫn thấu hiểu ý định (intent) của người dùng.

## 🧠 Nội dung chi tiết

### 1. Lexical Retrieval & Thuật toán BM25
**Lexical Retrieval (hay Sparse Retrieval)** là cơ chế hoạt động của các Search Engine đời cũ, hoạt động dựa trên việc quét chính xác các từ khóa (keywords).
- **Điểm mạnh:** Latency cực thấp, tốn ít tài nguyên compute, khả năng giải thích (Explainability) rõ ràng. Rất cần thiết để tra cứu các mã lỗi, ID, tên riêng chuyên ngành.
- **Điểm yếu:** Mắc hội chứng "bất đồng ngôn ngữ" (Vocabulary Mismatch) - Không thể match nếu query dùng "xe hơi" còn document dùng "ô tô". Khó bắt lỗi chính tả.

**Thuật toán BM25 (Best Matching 25):** 
Thuật toán "quốc dân" (core của Elasticsearch/Lucene) để ranking cho Lexical Retrieval. Chấm điểm dựa trên 3 yếu tố cốt lõi:
1. **Term Frequency (TF):** Tần suất từ khóa xuất hiện. Dùng hàm log để bão hòa, chống tình trạng spam keyword lặp lại hàng trăm lần.
2. **Inverse Document Frequency (IDF):** Độ hiếm của từ. Những từ quá phổ biến (stop-words) bị hạ trọng số, những từ hiếm/đặc thù được boost điểm cao.
3. **Document Length (Độ dài tài liệu):** Phạt các tài liệu quá dài để duy trì tính công bằng về mật độ từ khóa với các tài liệu ngắn.

### 2. Embeddings và Dense Retrieval
**Embeddings là gì?**
Máy tính không hiểu text. Embedding là quá trình dùng các mô hình AI (như OpenAI Text-Embedding, BERT) để nén/dịch một đoạn văn bản thành một **Vector** (mảng số thực đa chiều).
- Điểm đột phá là các khái niệm có ý nghĩa tương đồng sẽ nằm gần nhau trong không gian vector (Toán học hóa ngữ nghĩa).
- Gọi là **Dense** (Dày đặc) vì vector thường ngắn (384-1536 chiều) nhưng chiều nào cũng mang giá trị số thực ý nghĩa. Khác với Lexical sinh ra **Sparse** Vector khổng lồ (bằng số lượng từ vựng) nhưng toàn số 0.

**Quy trình vận hành Dense Retrieval (Vector/Semantic Search):**
1. **Indexing:** Push toàn bộ raw document qua mô hình Embedding -> Lưu vào **Vector Database**.
2. **Querying & Similarity Search:** User gõ query -> Chạy qua cùng mô hình Embedding đó để ra Vector -> Tính toán khoảng cách (ví dụ bằng *Cosine Similarity*) để kéo về k kết quả gần nhất, dù không hề trùng một mặt chữ nào.

### 3. Hybrid Search (Sự kết hợp hoàn hảo)
Các hệ thống tìm kiếm hiện đại (như Google) đều dùng Hybrid Search.
- **Tại sao cần Hybrid?** Giả sử query là: `"Hướng dẫn fix lỗi OOM-KILLER"`. 
  - Nếu chỉ dùng Vector: Nó có thể trả về các bài "cách sửa lỗi hết RAM" (đúng nghĩa nhưng mất tiêu từ khóa kỹ thuật OOM-KILLER quan trọng). 
  - Nếu chỉ dùng BM25: Nó có thể trả về một đoạn log spam lặp lại quá nhiều chữ OOM-KILLER mà không phải bài "hướng dẫn".

**Cách xử lý:** 
Chạy song song 2 luồng Lexical và Dense. Nhánh Lexical làm "người gác cổng" neo keyword cứng, nhánh Dense làm người hiểu ý định bao quát.

**Hợp nhất và Re-ranking (RRF):**
Hệ thống không thể cộng trực tiếp 2 loại điểm (vì công thức tính BM25 và Cosine khác hẳn nhau). Thay vào đó, áp dụng thuật toán **RRF (Reciprocal Rank Fusion)**:
- Bỏ qua điểm số (score) tuyệt đối.
- Chỉ quan tâm đến **Thứ hạng (Rank)** của document trên từng nhánh. 
- Một document đạt top trên cả 2 bảng xếp hạng sẽ được thưởng điểm khổng lồ, vươn lên giành vị trí hiển thị tối ưu nhất.

## 🚀 Đào sâu kiến thức bổ sung (Cập nhật 2026-09-07)

### 1. Cơ sở toán học của BM25 và cách tune tham số (k1, b)
Công thức gốc của BM25 để tính điểm cho tài liệu $D$ với truy vấn $Q$:
![BM25 Formula](/home/binhnguyen/reaserching-diary/ai_ml/images/bm25_formula.png)
Trong Elasticsearch/OpenSearch, bạn có thể tune 2 tham số chính:
- **$k_1$ (Term frequency saturation):** Thường mặc định = 1.2. Nó kiểm soát mức độ bão hòa điểm số khi một từ khóa xuất hiện nhiều lần.
  - Tăng $k_1$ (VD: 2.0): Điểm sẽ tiếp tục tăng mạnh nếu từ lặp lại nhiều lần (phù hợp với văn bản dài như bài báo).
  - Giảm $k_1$ (VD: 0.1): Chỉ quan tâm từ đó *có* xuất hiện hay không, lặp lại 100 lần cũng không tăng điểm đáng kể.
- **$b$ (Length normalization):** Thường mặc định = 0.75. Kiểm soát mức độ phạt tài liệu dài.
  - Tăng $b$ về 1: Phạt nặng tài liệu dài. Phù hợp nếu tài liệu của bạn thường rất ngắn (như title, tweets).
  - Giảm $b$ về 0: Không phạt độ dài tài liệu. Phù hợp với tài liệu có độ dài rất chênh lệch nhưng độ dài không phản ánh mức độ nhiễu.

### 2. Thuật toán ANN (Approximate Nearest Neighbor) trong Vector DB
Để tra cứu hàng tỷ vector mà không cần duyệt toàn bộ (KNN), Vector DB sử dụng ANN chấp nhận đánh đổi một chút độ chính xác (Recall) để đổi lấy tốc độ cực nhanh (Latency).
- **HNSW (Hierarchical Navigable Small World):** Cấu trúc dạng đồ thị nhiều tầng (giống Skip List). Các đỉnh trên cùng thưa thớt, giúp thuật toán nhảy những bước rất dài qua không gian vector. Khi đến gần mục tiêu, nó hạ dần xuống các tầng thấp dày đặc hơn để tìm kết quả chính xác. HNSW cho tốc độ và Recall tuyệt vời, đang là tiêu chuẩn công nghiệp.
- **FAISS (Facebook AI Similarity Search):** Nổi tiếng với kỹ thuật **IVF-PQ**. Chia không gian thành các cụm (Voronoi cells - Inverted File Index) để chỉ tìm trong cụm gần nhất. Đồng thời dùng Product Quantization (PQ) nén vector từ dạng float32 sang mã byte gọn nhẹ, tiết kiệm cực nhiều RAM.

### 3. Kiến trúc các Vector DB nổi tiếng
- **Pinecone:** Giải pháp SaaS fully-managed, cloud-native. Cực kỳ dễ dùng, không cần lo về hạ tầng, nhưng mã nguồn đóng (closed-source) và tốn phí theo usage.
- **Milvus:** Hệ sinh thái phân tán khổng lồ. Tách biệt hoàn toàn tầng lưu trữ (Storage) và tính toán (Compute/Query). Rất lý tưởng cho doanh nghiệp xử lý tập dữ liệu ở scale siêu lớn (hàng tỷ vectors) đòi hỏi cluster phức tạp.
- **Qdrant:** Viết bằng Rust. Nổi bật với tính năng "Payload-based Filtering", kết hợp cực tốt giữa việc lọc siêu dữ liệu truyền thống (SQL-like) và tìm kiếm vector mà không làm chậm tốc độ.
- **ChromaDB:** Viết bằng Python/TypeScript. Rất nhẹ, thường chạy dưới dạng thư viện nhúng (In-memory/Local SQLite) trong quá trình phát triển (dev/PoC) các ứng dụng LLM/LangChain, dĩ nhiên vẫn có chế độ Client-Server để scale nhẹ.

### 4. Setup Cross-Encoder cho bước Re-ranking trong Hybrid Search
Bi-Encoder (dùng để tạo vector cho Dense Search) rất nhanh vì vector được tính sẵn, nhưng mô hình bị "mù" sự tương tác chéo giữa từng từ của truy vấn và tài liệu.
Để cải thiện đột phá độ chính xác, kiến trúc phổ biến nhất là **Retrieve-then-Rerank**:
- **Giai đoạn 1 (Retrieve):** Dùng Hybrid Search (Bi-Encoder + BM25) với tốc độ siêu tốc kéo về top 100 tài liệu tiềm năng nhất.
- **Giai đoạn 2 (Re-rank):** Nạp câu query và 100 tài liệu này qua mô hình **Cross-Encoder**. Khác với Bi-encoder, Cross-Encoder nhận chuỗi đầu vào theo dạng `[CLS] Câu Query [SEP] Câu Tài Liệu [SEP]` và đẩy qua các tầng Transformer.
- **Cơ chế:** Nhờ cơ chế Self-Attention, mọi token trong query sẽ được tương tác chéo trực tiếp với mọi token trong document. Tuy tốn chi phí tính toán (rất chậm), nhưng do chỉ áp dụng cho 100 kết quả nên vẫn đáp ứng được thời gian thực, mang lại độ chính xác tổng thể (MAP, NDCG) tăng vọt.

## 📚 Tài liệu nên tham khảo
- Nguồn thu thập chính: [https://share.gemini.google/zEAVGArmeM7e](https://share.gemini.google/zEAVGArmeM7e)
- Nguồn bổ trợ: [https://share.gemini.google/C3HUVEIvYfGX](https://share.gemini.google/C3HUVEIvYfGX)
