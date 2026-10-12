# Đánh giá LLM Judge Phase B

Nhóm: LGBTree

Người phụ trách: Hoàng Anh Minh - MSSV: 2A202602566

Nguồn: `artifacts/judge_results.json`; lượt chạy live: 2026-10-12T13:15:39.039739+00:00.
Models generation/judge: generation: [] (extractive fallback từ Day 18 RAG); judge: gpt-4o-mini.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ `artifacts/answers_50q.json:answer`; B từ `fixtures/human_labels_10q.json:model_answer`; reference từ `fixtures/test_set_50q.json:ground_truth`.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| Question ID | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 (đã map về A/B) | Final Winner | Position Consistent | Scores Pass 1 | Scores Pass 2 | Lý do chính |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Trích từ nghi_phep_dac_biet.md (3 ngày phép) | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | A | A | A | True | A: 1.00, B: 0.80 | A: 1.00, B: 0.70 | Judge ưu tiên câu có ngữ cảnh và nguồn; B cũng trả lời đúng, gọn hơn. |
| 5 | Trích từ mua_sam.md (CNTT xác nhận cấu hình, đơn khẩn cấp) | Cần Giám đốc phòng ban phê duyệt. | B | A | tie | False | A: 0.40, B: 0.60 | A: 0.70, B: 0.30 | A thiếu thẩm quyền CEO trong đoạn trích; B nêu sai người phê duyệt. Cần review thủ công. |
| 12 | Trích từ thuong_tet.md (≥6 tháng, tối thiểu 1 tháng lương) | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | A | A | True | A: 1.00, B: 0.50 | A: 1.00, B: 0.60 | A nêu rõ điều kiện từ 6 tháng, B chỉ nêu mức thưởng. |
| 21 | Trích từ nghi_phep_nam_v2024.md (18 ngày phép, không có khoảng lương) | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | A | B | tie | False | A: 1.00, B: 0.80 | A: 0.60, B: 1.00 | B trả đủ hai vế; judge đổi kết luận sau đảo vị trí. Cần review thủ công. |
| 41 | Trích từ nghi_phep_nam_v2024.md (15 ngày phép) | Nhân viên được nghỉ 12 ngày phép năm. | A | A | A | True | A: 1.00, B: 0.00 | A: 1.00, B: 0.20 | A theo v2024; B dùng số ngày cũ. |

Position bias count/rate: 2/5 cặp = 0.40 (40%). Verbosity win rate: 3/3 cặp có winner quyết định = 1.00 (100%); hai cặp tie không thuộc mẫu số.

### Phân tích hiện tượng bias và kết quả so sánh

1. **Tính nhất quán vị trí:** ID 5 và 21 đổi winner sau khi đảo vị trí, nên `final_winner=tie`. Swap-and-Average giúp phát hiện bất ổn, chưa chứng minh judge hết position bias. ID 5 còn có cả hai candidate chưa trả đúng CEO; ID 21 candidate B đầy đủ hơn vì có cả lương. Các trường hợp này cần người rà soát.
2. **Thiên lệch độ dài:** Ba winner quyết định đều là candidate A dài hơn (ID 1, 12, 41), nên chỉ số đo được là 100%. A ở ID 12 có điều kiện ≥6 tháng, ID 41 dùng phiên bản hiện hành; vì vậy chỉ số này không tự chứng minh judge thiên vị câu dài. Mẫu ba cặp quá nhỏ để loại trừ bias.
3. **Chất lượng lập luận:** Một số reasoning của judge mâu thuẫn với reference hoặc mapping sau đảo vị trí, nhất là ID 5 và 21. Phải đọc câu trả lời gốc và reference khi đánh giá; không xem điểm pairwise là ground truth.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong `fixtures/human_labels_10q.json`; không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference. Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| Question ID | Model Answer gốc | Ground Truth tham chiếu | Nhãn Judge (0/1) | Nhãn Human (0/1) | Agree? | Lý do đánh giá của Judge |
|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn, không trừ vào phép năm. | 1 | 1 | True | Judge chấp nhận đúng số ngày; reasoning trong JSON còn quy thuộc cho model_answer ý “không trừ phép năm” vốn chỉ có ở reference. |
| 5 | Cần Giám đốc phòng ban phê duyệt. | Đơn hàng trên 50.000.000 VNĐ cần Tổng Giám đốc (CEO) phê duyệt. | 0 | 0 | True | Sai thẩm quyền phê duyệt: trên 50 triệu phải do CEO phê duyệt. |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | Nhân viên chính thức có thời gian làm việc từ 6 tháng trở lên được thưởng Tết tối thiểu 1 tháng lương. | 1 | 1 | True | Đúng mức thưởng Tết tối thiểu 1 tháng lương cho nhân viên từ 6 tháng trở lên. |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | Theo chính sách v2024: 15 ngày cơ bản + 3 ngày thâm niên = 18 ngày phép. Lương Senior từ 20-35 triệu VNĐ/tháng. | 1 | 1 | True | Tính đúng 18 ngày phép và nêu chuẩn khung lương Senior 20-35 triệu. |
| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | Nghỉ việc dưới 12 tháng hoàn trả 100% chi phí tài trợ đào tạo (25 triệu VNĐ). | 1 | 1 | True | Đúng tỷ lệ hoàn trả 100% (25 triệu) do làm việc dưới 12 tháng sau khóa học. |
| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | Tạm ứng 8 triệu cần Trưởng phòng và Kế toán trưởng duyệt; phạt pro-rata quá hạn 15 ngày = 80.000 VNĐ. | 0 | 0 | True | Thiếu phê duyệt của Kế toán trưởng và chưa tính đúng mức phạt pro-rata quá hạn. |
| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | Phép: 19 ngày (15 cơ bản + 4 thâm niên); phụ cấp trách nhiệm Manager: 1.500.000 VNĐ/tháng. | 1 | 1 | True | Đúng số ngày phép (19 ngày) và phụ cấp chức vụ Manager 1.500.000 VNĐ. |
| 41 | Nhân viên được nghỉ 12 ngày phép năm. | Theo v2024 hiện hành: 15 ngày cơ bản. Chính sách v2023 cũ (đã thay thế) là 12 ngày. | 0 | 0 | True | Dùng số ngày của phiên bản 2023 cũ đã bị thay thế bởi phiên bản 2024. |
| 46 | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | Thử việc không được nghỉ phép năm hưởng lương; nếu nghỉ việc riêng phải xin nghỉ không lương. | 1 | 1 | True | Judge chấp nhận đúng quy định chính; reasoning trong JSON còn nhắc phê duyệt trưởng phòng dù model_answer/reference không nêu. |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | Nghiêm cấm dùng VPN cá nhân (NordVPN); bắt buộc dùng VPN WireGuard nội bộ công ty. | 0 | 0 | True | Sai quy định an toàn thông tin: VPN cá nhân bị cấm, bắt buộc dùng VPN công ty. |

Cohen’s κ: 1.00.

Observed/chance agreement và diễn giải:
- **Tỷ lệ đồng thuận quan sát ($p_o$)**: 10/10 trường hợp trùng khớp hoàn toàn giữa LLM Judge và Human Label, do đó $p_o = 1.00$.
- **Tỷ lệ đồng thuận ngẫu nhiên ($p_e$)**:
  - Phân bố nhãn của Judge: 6 nhãn 1 (chấp nhận), 4 nhãn 0 (từ chối).
  - Phân bố nhãn của Human: 6 nhãn 1 (chấp nhận), 4 nhãn 0 (từ chối).
  - Công thức tính: $p_e = \frac{6 \times 6 + 4 \times 4}{10 \times 10} = \frac{36 + 16}{100} = 0.52$.
- **Hệ số Cohen's $\kappa$**:
  $$\kappa = \frac{p_o - p_e}{1 - p_e} = \frac{1.00 - 0.52}{1.00 - 0.52} = 1.00$$
- **Nhận xét**: Theo thang đo Landis-Koch, $\kappa = 1.00$ thuộc mức **Almost Perfect Agreement** (Đồng thuận hoàn hảo). LLM Judge phân biệt rất tốt giữa câu trả lời đúng, đầy đủ với các câu trả lời sai thẩm quyền, thiếu điều kiện phạt hoặc sử dụng tài liệu hết hạn.

## Giới hạn và cách dùng judge

1. **Hạn chế về quy mô mẫu và mô hình**:
   - Thử nghiệm hiệu chuẩn được thực hiện trên tập mẫu nhỏ (5 cặp pairwise và 10 mẫu nhãn nhị phân), do đó hệ số $\kappa = 1.00$ là bằng chứng tốt trên tập test kiểm chuẩn nhưng chưa bao quát được toàn bộ phân bố câu hỏi phong phú trong thực tế production.
   - LLM Judge sử dụng model `gpt-4o-mini` với prompt đánh giá 3 tiêu chí cấu trúc JSON. Hai cặp đảo vị trí không nhất quán và một số lập luận chưa đáng tin; không suy rộng κ của calibration sang độ chính xác pairwise.

2. **Cách kiểm chứng và đối chiếu**:
   - Khi triển khai production, không bao giờ xem LLM Judge là sự thật tuyệt đối (Ground Truth). Judge chỉ đóng vai trò là tầng **tự động hóa sàng lọc (automated screening gate)** trong CI/CD.
   - Các trường hợp judge cho điểm thấp hoặc không nhất quán sau 2 lượt swap-and-average (`final_winner == "tie"`, điểm số chênh lệch < 0.1) cần được tự động định tuyến về hàng đợi kiểm duyệt của con người (Human-in-the-Loop review).
   - Định kỳ lấy mẫu ngẫu nhiên từ 5% đến 10% kết quả chấm của LLM Judge để chuyên gia nhân sự/chính sách gán nhãn mù (blind labeling) nhằm phát hiện sớm hiện tượng suy giảm chất lượng (judge drift).
