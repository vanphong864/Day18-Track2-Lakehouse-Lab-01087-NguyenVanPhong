# Lakehouse Architecture Reflection

Trong 5 Anti-Patterns của Lakehouse, hệ thống dữ liệu của team dễ mắc phải nhất là **"Small-Files Problem do Streaming/Micro-batch Ingestion mà bỏ quên Compaction (OPTIMIZE)"**.

### 1. Nguyên nhân & Rủi ro:
* Pipeline thu thập clickstream và LLM telemetry liên tục ghi dữ liệu với tần suất cao (1–5 phút/batch).
* Việc này sinh ra hàng chục nghìn file Parquet nhỏ (vài chục KB). Khi truy vấn, query engine chịu chi phí metadata lookup và HTTP GET overhead lớn trên Cloud Storage (S3), khiến tốc độ truy vấn suy giảm nghiêm trọng và chi phí I/O tăng vọt.

### 2. Giải pháp khắc phục:
* **Tự động Compaction:** Định kỳ lập lịch chạy `OPTIMIZE` gộp các file nhỏ thành kích thước tối ưu (128–512 MB).
* **Kết hợp Z-ORDER:** Gom cụm dữ liệu theo các cột lọc phổ biến (`user_id`, `timestamp`) giúp tăng tỷ lệ file pruning ($\ge 10\times$).
* **Bảo trì vòng đời:** Chạy định kỳ `VACUUM` và quét dọn file orphan để giải phóng dung lượng lưu trữ rác.
