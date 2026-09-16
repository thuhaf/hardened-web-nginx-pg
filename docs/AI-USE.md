# Khai báo sử dụng công cụ AI

| Công cụ và phiên bản | Mục đích | Phần giữ nguyên và phần đã sửa | Cách kiểm chứng |
|---|---|---|---|
| ChatGPT (GPT-5.6 Sol) | Hỗ trợ diễn đạt phần phân tích khi lớp phòng thủ bên ngoài bị vượt qua trong `docs/threat-model.json` | Giữ nguyên dữ liệu mối đe dọa, thứ tự ưu tiên và kiến trúc do nhóm xây dựng; sử dụng nội dung AI hỗ trợ để diễn đạt trường `dieu_con_lai_neu_lop_ngoai_vo` | Đối chiếu với `docs/threat-model.json`, kiến trúc nginx → web app → PostgreSQL và yêu cầu hai lớp phòng thủ độc lập của Chủ đề A |
| ChatGPT (GPT-5.6 Sol) | Hỗ trợ tổng hợp và trình bày mô hình đe dọa 1 trang trong `docs/threat-model-draft.md` | Giữ nguyên dữ liệu T1, T2, T3, điểm tác động, khả năng xảy ra và thứ tự ưu tiên từ `docs/threat-model.json`; AI hỗ trợ cấu trúc và diễn đạt bản Markdown | Kiểm tra lại từng thông tin với `docs/threat-model.json` và xem trước file Markdown trên GitHub trước khi commit |
