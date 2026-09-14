# hardened-web-nginx-pg
Dịch vụ web nội bộ nhiều lớp được làm cứng: nginx (TLS + mTLS cho /admin), ứng dụng web, PostgreSQL với Row Level Security. Hai lớp phòng thủ độc lập chống truy cập vượt quyền giữa phòng ban, kèm bằng chứng testssl, pgaudit và ca kiểm âm.
