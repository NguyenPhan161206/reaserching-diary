# Hệ thống Tìm kiếm (Search Systems): Lexical, Dense và Hybrid Search

**Ngày tạo:** 2026-09-04
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

## 🤔 Kiến thức cần đào sâu
- [ ] Cơ sở toán học của thuật toán BM25 và cách tune (điều chỉnh) các tham số k1, b trong Elasticsearch/Opensearch.
- [ ] Các thuật toán tìm kiếm lân cận gần nhất (ANN) trong Vector Database như HNSW, FAISS hoạt động thế nào để giữ độ chính xác nhưng tăng tốc tra cứu lên hàng nghìn lần?
- [ ] Tìm hiểu sâu hơn về kiến trúc và cách triển khai của các Vector DB nổi tiếng hiện nay (như Milvus, Pinecone, Qdrant, ChromaDB).
- [ ] Làm thế nào để setup mô hình Cross-Encoder vào bước Re-ranking thứ 2 nhằm cải thiện độ chính xác cuối cùng của Hybrid Search?

## 📚 Tài liệu nên tham khảo
- Nguồn thu thập chính: [https://share.gemini.google/zEAVGArmeM7e](https://share.gemini.google/zEAVGArmeM7e)
- Nguồn bổ trợ: [https://share.gemini.google/C3HUVEIvYfGX](https://share.gemini.google/C3HUVEIvYfGX)
