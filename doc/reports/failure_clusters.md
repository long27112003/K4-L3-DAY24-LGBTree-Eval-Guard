# Phân tích lỗi Phase A

Nhóm: LGBTree
Người phụ trách: Ngô Tiến Dũng (2A202602374)

Nguồn: `artifacts/ragas_50q.json`, lượt chạy live `2026-10-12T13:15:07.235420+00:00`. Bộ 50 câu trả lời dùng `extractive_fallback`, không có generation model; evaluator là `gpt-4o-mini` và `text-embedding-3-small`.

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---:|---:|---:|
| faithfulness | 0.9900 | 0.9875 | 1.0000 |
| answer_relevancy | 0.3953 | 0.3380 | 0.4016 |
| context_precision | 0.9083 | 0.9833 | 0.9583 |
| context_recall | 0.8250 | 0.7958 | 0.7167 |
| avg_score | 0.7797 | 0.7762 | 0.7692 |

Adversarial có mean thấp nhất: 0.7692 < factual 0.7797. Đây là điều kiện bonus Phase A trong bộ chạy này; chênh lệch nhỏ nên cần kiểm tra lại trên mẫu lớn hơn trước khi suy rộng.

## Bottom 10

| Rank | ID | Distribution | Question | Avg score | Worst metric | Diagnosis / suggested fix |
|---|---:|---|---|---:|---|---|
| 1 | 5 | factual | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | 0.3340 | context_precision | Too many irrelevant chunks / Add reranking or metadata filter |
| 2 | 39 | multi_hop | So sánh policy mật khẩu v1.0/v2.0: độ dài, chu kỳ đổi và MFA. | 0.5968 | context_recall | Missing relevant chunks / Improve chunking or add BM25 |
| 3 | 6 | factual | Thông tin lương thuộc cấp độ phân loại dữ liệu nào? | 0.6370 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 4 | 33 | multi_hop | Manager thâm niên 12 năm: phụ cấp và phép năm v2024? | 0.6723 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 5 | 21 | multi_hop | Senior thâm niên 9 năm: phép năm và khoảng lương? | 0.6938 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 6 | 7 | factual | Nghỉ không lương 20 ngày cần ai phê duyệt? | 0.6942 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 7 | 30 | multi_hop | Quyền lợi bảo hiểm thử việc và chính thức? | 0.6970 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 8 | 34 | multi_hop | Phê duyệt tạm ứng 4 và 7 triệu khác nhau thế nào? | 0.6988 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 9 | 45 | adversarial | Có cần kích hoạt MFA không? | 0.7077 | answer_relevancy | Answer doesn't match question / Improve prompt template |
| 10 | 20 | factual | Phụ cấp đi lại tối đa và điều kiện hưởng? | 0.7134 | answer_relevancy | Answer doesn't match question / Improve prompt template |

### Ba lỗi đại diện

1. **ID 5, retrieval precision:** Context/answer của lượt chạy này nói về xác nhận cấu hình của phòng CNTT và đơn hàng khẩn cấp, nhưng bỏ mất ngưỡng trên 50 triệu cần Tổng Giám đốc phê duyệt. `context_precision = 0`. Nên lọc/rerank theo hạn mức và kiểm tra lại câu trả lời sau retrieval.
2. **ID 39, retrieval recall:** Câu hỏi đòi so sánh ba thuộc tính của hai phiên bản. Context chỉ chứa một phần v1.0 và v2.0; thiếu ngưỡng 12 ký tự của v2.0, chu kỳ 90 ngày và trạng thái MFA của v1.0. `context_recall = 0` dù `context_precision ≈ 1`. Cần mở rộng truy hồi theo cặp phiên bản và kiểm tra đủ các thuộc tính trước khi trả lời.
3. **Các ID 6, 33, 21, 7, 30, 34, 45, 20, answer relevancy:** Chế độ `extractive_fallback` trả đoạn trích dài, có metadata và thông tin ngoài câu hỏi. Prompt tạo câu trả lời ngắn, trả đủ từng vế, kèm trích dẫn nguồn/phiên bản sẽ là hướng thử nghiệm tiếp theo; hiện chưa có thực nghiệm chứng minh cải thiện.

## Matrix và nhóm yếu nhất

| Worst metric | factual | multi_hop | adversarial | Tổng |
|---|---:|---:|---:|---:|
| faithfulness | 0 | 0 | 0 | 0 |
| context_recall | 0 | 1 | 0 | 1 |
| context_precision | 1 | 0 | 0 | 1 |
| answer_relevancy | 19 | 19 | 10 | 48 |

Nhóm adversarial yếu nhất theo `avg_score` (0.7692) và có `context_recall` thấp nhất (0.7167; factual 0.8250, multi_hop 0.7958). Các bẫy phiên bản cũ/mới và điều kiện phủ định dễ làm mất điều khoản liên quan. Lọc metadata hiệu lực, mở rộng truy hồi và kiểm tra câu trả lời với cả hai phiên bản là các thử nghiệm hợp lý; cần chạy lại Phase A sau khi thay đổi để đo tác động. Bộ đánh giá chỉ gồm 10 câu adversarial và toàn bộ answer lấy từ extractive fallback, nên không đại diện cho chất lượng production.
