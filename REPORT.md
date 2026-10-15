# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | `Hoàng Trung Khải` |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `202602947` |
| GitHub | `khaihoang004` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `4516 / 4887` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `3` |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1.263 | 20.9% | Chặn |
| IDM | 1.508 | 0.1% | Baseline |
| EgoMLP | 0.768 | 19.9–29.3% | Chặn |
| PercMLP | 0.619 | 0.7–1.4% | Chặn |
| PercMLP+ (sau khi sửa) | 0.819 | 0.2–0.3% | RELEASE |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

Audit có 9.993 quỹ đạo ban đầu; loại 383 bản sao còn 9.610. Tương quan `Acc_LV` với đạo hàm tốc độ chỉ 0,045, còn `Acc_FAV` là 0,929, nên dùng gia tốc suy ra từ `Speed_LV` đã làm mượt. Chia theo quỹ đạo để khung gần nhau của cùng chuyến không rò rỉ giữa train và test.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Refactor float32 khác bit ở 600/600 đoạn nhưng không đoạn nào vượt dung sai; sai khác lớn nhất là 1,58e-5 so với ngưỡng 1e-3. AEB đổi 14/600 đoạn khi TTC dưới 1,5 giây: ở segment 65, khung 75 TTC=1,48 s làm lệnh đổi −4,38→−6,0 m/s²; khung 76–77 TTC còn 1,46–1,44 s. Phát nhanh nhất có 0 trễ/rớt; thời gian thật có 581 trễ, 578 rớt/59.541 khung và 369 đoạn đổi.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Lưới dùng cruise 10 m/s, 60 khoảng cách × 45 tốc độ tiến gần = 2.700 ô; khoảng cách 2,76–59,31 m và closing speed 0–10 m/s. Ô bất khả thi tăng 0,96%→5,15% (+4,19 điểm) khi trễ tăng 0→0,6 s; va chạm IDM khớp đúng biên này. Người lái phanh mạnh nhất 3,62 m/s², nhưng 5,22% ô cần hơn mức này; 614 ca log không phủ vùng khó.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Hạng vòng hở / kín |
|---|---:|---:|---:|
| const-vel | 1,263 | 20,9 | 3 / 3 |
| IDM | 1,508 | 0,1 | 4 / 1 |
| EgoMLP | 0,768 | 25,0 | 2 / 4 |
| PercMLP | 0,619 | 1,0 | 1 / 2 |

Các cặp đổi thứ tự gồm IDM–const-vel, IDM–EgoMLP, IDM–PercMLP và const-vel–EgoMLP. IDM có L2 cao vì không bắt chước từng thao tác người lái nhưng vẫn giữ quy tắc khoảng cách an toàn. EgoMLP thiếu gap và tốc độ xe dẫn nên L2 0,768 m vẫn đi kèm 25,0% va chạm.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

EgoMLP dao động từ 19,94% đến 29,28% giữa 3 seed, chênh 9,33 điểm phần trăm. Seed xấu nhất có Wilson 95% [27,64%; 30,96%], còn seed tốt nhất có [18,53%; 21,44%]; hai khoảng không chồng lấp. Chỉ chạy một seed có thể chọn nhầm kết quả tốt hoặc xấu và che độ bất ổn của mô hình.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

T hiệu chỉnh tốt nhất là 1,4 s (W1 = 0,35), tốt hơn xe không phản ứng (W1 = 0,89). Ở trễ 0,6 s, tỷ lệ có va chạm trên lưới là:

| Bộ điều khiển | Xe sau phát lại | Xe sau phản ứng |
|---|---:|---:|
| const-vel | 73,4% | 73,4% |
| IDM | 70,1% | 5,1% |
| EgoMLP | 70,5% | 70,5% |
| PercMLP | 79,2% | 16,5% |

Với IDM, 64,9% là va chạm phía sau khi xe sau phát lại, giảm về 0% khi xe sau phản ứng. Xe phát lại không biết xe thử đã phanh nên tạo “va chạm giả”; vì vậy thứ hạng IDM đổi mạnh khi mô hình hóa phản ứng.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP rớt R2 (cận trên 1,84% > 1,07%), R3 (8,4% > 5,5%), R4 (11,37% > 2%) và R6 (16,5% > 7,1%); R5 đạt 0,7–1,4%. R4/R6 cho thấy thiếu kịch bản cut-in trong log; R2/R3 là lỗi tích lũy khi tự lái. Thêm 20.000 cut-in (196.930 cửa sổ) và 2 vòng DAgger gán nhãn IDM: PercMLP+ còn 0,2–0,3% va chạm log, 0,48% lưới và qua R1–R7. Nhưng L2 xấu từ 0,619 lên 0,819; IDM vẫn 0,1% va chạm log và 0% lưới, nên không thể nói PercMLP+ tốt hơn. Chỉ nên xem đây là không-kém-hơn theo ngưỡng; cần thêm thử nghiệm OOD ở 20 m/s và trên phần cứng/đường kín trước khi thay baseline.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Kết quả chỉ dùng mô phỏng dọc 1-D, không có cảm biến hay tương tác ngang nên không đại diện đầy đủ tình huống đường thật. Xe dẫn phát lại không phản ứng với xe thử, còn xe sau phản ứng dùng IDM giả định; cả hai có thể làm sai tỷ lệ va chạm. Dữ liệu cũng chỉ có 7–11 giây và lưới thử bị giới hạn; cần thêm kịch bản thực, mô phỏng được kiểm chứng và thử nghiệm phần cứng/đường kín trước khi quyết định phát hành.
