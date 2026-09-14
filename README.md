# 🔒 Hardened Web Nginx PG


![Docker](https://img.shields.io/badge/Docker-Compose-2496ED.svg)
![Status](https://img.shields.io/badge/status-in%20progress-yellow.svg)
![License](https://img.shields.io/badge/license-academic-lightgrey.svg)


Đồ án cuối kỳ môn An toàn hệ thống máy tính - Chủ đề A: dựng và làm cứng một
dịch vụ web nội bộ nhiều lớp (nginx → app → PostgreSQL) cho doanh nghiệp vừa
và nhỏ, kèm bằng chứng kiểm chứng được bằng CI.
## 1. Tổng quan

Doanh nghiệp vừa và nhỏ vận hành một hệ thống nội bộ, không có đội an toàn thông tin
riêng, ngân sách công cụ dưới 30 triệu đồng/năm. Đồ án này dựng và bàn giao một cấu
hình an toàn cho hệ thống đó - kèm bằng chứng kiểm chứng được và phân tích chi phí —
thay vì chỉ mô tả lý thuyết.


Hệ thống gồm ba tầng, mỗi tầng được làm cứng khỏi cấu hình mặc định:


```
                      ┌─────────────────────────┐
   Internet/LAN  ───▶ │   nginx (gateway)       │
                      │   TLS 1.2/1.3 toàn bộ   │
                      │   mTLS bắt buộc /admin  │
                      └───────────┬─────────────┘
                                  │  (mạng nội bộ, chỉ nginx→app)
                      ┌───────────▼─────────────┐
                      │   web app                │
                      │   không giữ quyền cao    │
                      └───────────┬─────────────┘
                                  │  kết nối bắt buộc SSL
                      ┌───────────▼─────────────┐
                      │   PostgreSQL             │
                      │   Row Level Security     │
                      │   pgaudit bật ghi log     │
                      └──────────────────────────┘
```

## 📁 2. Cấu trúc thư mục

```
compose/                       # docker compose, mọi ảnh ghim theo digest
config/                        # cấu hình dịch vụ - KHÔNG chứa bí mật
tests/                         # pytest, mỗi khẳng định an toàn = 1 ca kiểm (kèm ca âm)
evidence/                      # đầu ra công cụ thật, kèm MANIFEST.txt có sha256
docs/report.pdf                # báo cáo kỹ thuật (≤ 20 trang)
docs/slides.pdf                # slides bảo vệ (≤ 14 trang)
docs/logbook.md                # nhật ký điều phối: ngày, người, việc
docs/threat-model.json         # mô hình đe dọa có cấu trúc
docs/AI-USE.md                 # khai báo sử dụng công cụ AI
.github/workflows/verify.yml   # do giảng viên phát, KHÔNG được sửa
sbom.cdx.json + cosign.sig      # danh mục thành phần phần mềm + chữ ký
Makefile                        # preflight, up, attack, defend, verify, export-evidence, down
```
## ⚙️ 3. Cài đặt

Yêu cầu: Docker + Docker Compose, không cần cài thêm gì trên máy host.


```bash
git clone https://github.com/thuhaf/hardened-web-nginx-pg.git
cd hardened-web-nginx-pg
```

## 🔧 4. Trước khi chạy — bắt buộc

**1. Ghim ảnh theo digest.** Mở `compose/docker-compose.yml`, thay `<digest-thật>`:


```bash
docker pull nginx:latest
docker inspect --format='{{index .RepoDigests 0}}' nginx:latest
```


**2. Sinh chứng chỉ TLS/mTLS.** Làm theo hướng dẫn trong `compose/nginx/certs/README.md`
- tạo CA nội bộ, chứng chỉ server và chứng chỉ client dùng để test `/admin`.


**3. Không đẩy** file `.key` thật lên Git — đã bị chặn sẵn trong `.gitignore`, kiểm tra lại
trước khi commit.

## 🚀 5. Chạy hệ thống

```bash
make preflight     # ghi kiến trúc CPU + phiên bản Docker vào evidence/
make up             # dựng nginx + app + PostgreSQL
```


Để nguyên terminal chạy, kiểm tra bằng:


```bash
curl -sk https://localhost/
```


Dừng bằng:


```bash
make down
```

## 🧪 6. Kiểm thử

```bash
make verify
```


Chạy 4 nhóm ca kiểm: TLS/mTLS, Row Level Security, ép SSL ở `pg_hba.conf`. Mỗi ca **có
ca âm** - ví dụ phòng ban A đọc dữ liệu phòng ban B phải trả về 0 dòng.

## 🔐 7. Bằng chứng bắt buộc

Sau khi verify xanh, sinh lại MANIFEST:


```bash
make export-evidence
```

| Bằng chứng | File |
|---|---|
| Đầu ra testssl | `evidence/testssl-output.json` |
| Ca kiểm âm RLS | `evidence/rls-negative-test.log` |
| Log pgaudit | `evidence/pgaudit.log` |
| Đối chiếu trước/sau | `evidence/before-after-hardening.md` |

## 🧵 8. Ba mối đe dọa phải xử lý

1. Truy cập vượt quyền giữa các phòng ban → **Row Level Security**
2. Lộ dữ liệu khi truyền trong mạng nội bộ → **TLS + mTLS**
3. DB mặc định cho phép kết nối không mã hóa → **ép SSL trong `pg_hba.conf`**
