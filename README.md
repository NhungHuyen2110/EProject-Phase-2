# 🧩 EPROJECT PHASE 2 – MICROSERVICES SYSTEM WITH DOCKER

## 🏫 Trường Đại Học Công Nghiệp Thành phố Hồ Chí Minh  
**Môn học:** Lập trình hướng dịch vụ
**Sinh viên thực hiện:** Trần Thị Nhung Huyền
**MSSV:** [22658921]  
**Giảng viên hướng dẫn:** [Huỳnh Nam]  
**Ngày hoàn thành:** [01/11/2025]

---

## 🌐 1. GIỚI THIỆU DỰ ÁN

Dự án **EProject Phase 2** được phát triển với **kiến trúc microservices**, triển khai bằng **Docker Compose**, giúp tách biệt các chức năng chính của hệ thống thành từng dịch vụ nhỏ độc lập, dễ mở rộng và quản lý.

### 🎯 Mục tiêu:
- Xây dựng hệ thống thương mại điện tử gồm **4 microservices**:
  1. **Auth Service** – Quản lý người dùng (đăng ký, đăng nhập, xác thực JWT)
  2. **Product Service** – Quản lý sản phẩm
  3. **Order Service** – Quản lý đơn hàng và xử lý giao tiếp với RabbitMQ
  4. **API Gateway** – Kết nối giữa client và các microservices

### 🧠 Công nghệ sử dụng:
| Thành phần | Công nghệ |
|-------------|------------|
| Ngôn ngữ | Node.js (Express.js) |
| Cơ sở dữ liệu | MongoDB |
| Message Queue | RabbitMQ |
| Containerization | Docker & Docker Compose |
| Kiểm thử API | Postman |
| Authentication | JSON Web Token (JWT) |

## ⚙️ 2. KIẾN TRÚC HỆ THỐNG
Hệ thống được thiết kế theo mô hình **Microservices Architecture**:

      ┌─────────────┐
      │  API Gateway│
      └──────┬──────┘
             │
┌───────────────┼────────────────┬────────────────┐
│ │ │ │
▼ ▼ ▼ ▼
Auth Service Product Service Order Service RabbitMQ
(Port 3000) (Port 3001) (Port 3002) (Port 5672)

## 🧱 3. CẤU TRÚC THƯ MỤC DỰ ÁN
**EProject-Phase-2/**
│
├── auth/ # Dịch vụ xác thực người dùng
├── product/ # Dịch vụ quản lý sản phẩm
├── order/ # Dịch vụ quản lý đơn hàng
├── api-gateway/ # Gateway trung gian
├── docker-compose.yml # File cấu hình Docker Compose
└── README.md # Tài liệu hướng dẫn

---

## 🧩 4. CÁC BIẾN MÔI TRƯỜNG (.env)

📁 auth/.env
PORT=3000
MONGO_URI=mongodb://mongo:27017/auth_service
JWT_SECRET=supersecretkey

📁 product/.env
PORT=3001
MONGO_URI=mongodb://mongo:27017/product_service

📁 order/.env
PORT=3002
MONGO_URI=mongodb://mongo:27017/order_service
RABBITMQ_URL=amqp://rabbitmq

📁 api-gateway/.env
PORT=3003
AUTH_SERVICE=http://auth:3000
PRODUCT_SERVICE=http://product:3001
ORDER_SERVICE=http://order:3002

Hệ thống được thiết kế theo mô hình **Microservices Architecture**:

