# Real-Time Big Data Recommender System for E-Commerce

Hệ thống gợi ý sản phẩm thời gian thực cho thương mại điện tử: thu nhận luồng sự kiện người dùng (view / cart / purchase), xử lý streaming, huấn luyện mô hình Collaborative Filtering (ALS) và phục vụ gợi ý qua REST API + giao diện web.

## Kiến trúc

```
REES46 events ──► Kafka (producer giả lập luồng) ──► Spark Structured Streaming
                                                        │  ├─ tính đặc trưng / implicit rating
                                                        │  └─ cập nhật popular items, lịch sử user
                                                        ▼
                         Spark MLlib ALS (train định kỳ) ──► Redis (top-N recs, popular, history)
                                                                         │
                                                         FastAPI ◄───────┘
                                                            │
                                                         Web demo
```

## Công nghệ

| Thành phần | Công nghệ |
|---|---|
| Message broker | Apache Kafka |
| Xử lý luồng | Spark Structured Streaming |
| Mô hình gợi ý | Spark MLlib ALS (implicit feedback) |
| Lưu trữ / cache gợi ý | Redis |
| API | FastAPI |
| Triển khai | Docker Compose |
| Dữ liệu | [REES46 eCommerce behavior data](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store) |

## Cấu trúc thư mục

| Thư mục | Nội dung |
|---|---|
| `infra/` | Docker Compose, cấu hình Kafka/Zookeeper, Redis, Spark; script tạo topic |
| `streaming/` | Kafka producer phát lại dữ liệu REES46, job Spark Structured Streaming |
| `model/` | Tiền xử lý, huấn luyện & đánh giá ALS, ghi top-N vào Redis |
| `api/` | FastAPI service đọc gợi ý từ Redis |
| `web/` | Giao diện demo |
| `docs/` | Báo cáo, slide, sơ đồ kiến trúc |
| `data/` | Dữ liệu cục bộ (không commit, đã có trong `.gitignore`) |

## Phân công (nhóm 4 người)

| Vai trò | Phụ trách | Thư mục chính |
|---|---|---|
| **A** | Hạ tầng & Kafka: Docker Compose, Kafka topics, producer phát luồng sự kiện | `infra/`, `streaming/producer` |
| **B** | Spark Structured Streaming: đọc Kafka, làm sạch, tổng hợp, ghi Redis | `streaming/` |
| **C** | Mô hình ALS: tiền xử lý, train, tuning, đánh giá (Precision@K, RMSE), xuất gợi ý | `model/` |
| **D** | API, web demo & báo cáo | `api/`, `web/`, `docs/` |

## Quy ước làm việc

- Mỗi người làm trên nhánh riêng (`feature/<tên-việc>`), mở Pull Request vào `main`.
- Không commit dữ liệu lớn; tải REES46 về `data/`.

## Chạy nhanh (sẽ hoàn thiện)

```bash
docker compose -f infra/docker-compose.yml up -d
```
