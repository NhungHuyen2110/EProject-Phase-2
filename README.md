# 🧠 EProject Phase

> **Sinh viên:** Trần Thị Nhung Huyền  
> **Môn học:** Lập Trình Hướng Dịch Vụ  
> **Giảng viên hướng dẫn:** Huỳnh Nam 
> **Dự án:** Hệ thống thương mại điện tử sử dụng kiến trúc Microservices (Auth, Product, Order, API Gateway, MongoDB, RabbitMQ)

---

## 🚀 1. Hệ thống giải quyết vấn đề gì

Dự án này xây dựng **một hệ thống thương mại điện tử mini**, áp dụng **kiến trúc microservices**, nhằm giải quyết các nghiệp vụ cơ bản:
- Đăng ký / đăng nhập người dùng (Authentication & Authorization)
- Quản lý danh sách sản phẩm (Product Management)
- Quản lý đơn hàng (Order Management)
- Định tuyến request tập trung qua API Gateway
- Giao tiếp giữa các dịch vụ qua **RabbitMQ**

🎯 **Mục tiêu:**  
- Giúp tách biệt trách nhiệm giữa các module.  
- Hỗ trợ mở rộng dễ dàng (scale từng service).  
- Tăng độ tin cậy, giảm phụ thuộc giữa các thành phần.

---

## 🧩 2. Cấu trúc hệ thống — Gồm bao nhiêu dịch vụ?

Hệ thống gồm **4 microservices chính** + **2 thành phần hạ tầng**:

| Loại | Tên dịch vụ | Cổng | Chức năng |
|------|--------------|------|------------|
| 🧱 Service | **Auth Service** | 3000 | Đăng ký, đăng nhập, cấp JWT |
| 🧱 Service | **Product Service** | 3001 | CRUD sản phẩm, quản lý tồn kho |
| 🧱 Service | **Order Service** | 3002 | Tạo và xử lý đơn hàng |
| 🧱 Service | **API Gateway** | 3003 | Định tuyến request từ client đến các service |
| ⚙️ Infra | **MongoDB** | 27017 | Lưu trữ dữ liệu |
| ⚙️ Infra | **RabbitMQ** | 5672 | Giao tiếp bất đồng bộ (message broker) |

---

## 🧠 3. Ý nghĩa từng dịch vụ

### 🛡️ **Auth Service**
- Quản lý người dùng, đăng ký / đăng nhập / xác thực JWT.
- Trả token cho client, giúp API Gateway xác minh người dùng.

### 🏷️ **Product Service**
- CRUD sản phẩm: thêm, sửa, xóa, xem chi tiết.
- Cập nhật tồn kho khi có đơn hàng mới.
- Cung cấp endpoint `/products/:id` để lấy chi tiết sản phẩm.

### 📦 **Order Service**
- Tạo và xử lý đơn hàng.
- Gửi sự kiện `order.created` lên RabbitMQ để **product** cập nhật tồn kho.
- Quản lý trạng thái đơn hàng.

### 🌐 **API Gateway**
- Là cổng duy nhất cho client.
- Định tuyến yêu cầu đến từng microservice.
- Kiểm tra JWT (xác thực người dùng).
- Có thể mở rộng thêm các chức năng như rate limit, logging, caching.

---

## 🧩 4. Các mẫu thiết kế được sử dụng (Design Patterns)

| Mẫu thiết kế | Giải thích | Vị trí trong code |
|---------------|------------|-------------------|
| **Layered Architecture** | Chia tầng rõ ràng: Controller → Service → Repository | `src/controllers`, `src/services`, `src/repositories` |
| **Repository Pattern** | Đóng gói truy vấn DB, dễ test và bảo trì | `userRepository.js`, `productRepository.js` |
| **Proxy Pattern (API Gateway)** | Định tuyến và điều phối request | `api-gateway/index.js` |
| **Pub/Sub (Event-driven)** | Giao tiếp bất đồng bộ giữa các service | `order/src/publisher.js`, `product/src/consumer.js` |
| **Singleton Pattern** | Dùng một instance kết nối DB duy nhất | `db.js` trong từng service |
| *(Khuyến nghị)* **Circuit Breaker / SAGA** | Bảo vệ khi có lỗi giữa các service, rollback transaction | Có thể bổ sung sau |

---

## 🔁 5. Cách các dịch vụ giao tiếp với nhau

Hệ thống sử dụng **2 hình thức giao tiếp chính:**

### ⚡ A. Giao tiếp **đồng bộ (HTTP/REST)**
- Client gửi request → API Gateway → Service đích.  
- Ví dụ:  
  `POST http://localhost:3003/api/auth/login` → forward đến `auth:3000/login`  
  `GET http://localhost:3003/api/products` → forward đến `product:3001/products`  
- Dữ liệu trả về trực tiếp qua gateway về client.  
- Phù hợp cho các yêu cầu cần phản hồi ngay.

### 📨 B. Giao tiếp **bất đồng bộ (RabbitMQ)**
- **Order Service** publish event (ví dụ `order.created`).  
- **Product Service** subscribe event để cập nhật tồn kho.
- Giúp tách biệt logic và tăng khả năng chịu tải.

### 🧱 C. **Database per Service**
- Mỗi service dùng MongoDB riêng (hoặc schema riêng).  
- Không service nào truy cập trực tiếp DB của service khác → tránh phụ thuộc.

### 🌉 D. **Docker Network**
- Các container giao tiếp bằng tên service (`auth`, `product`, `order`, `api-gateway`).  
- Ví dụ: gateway gọi `http://auth:3000` qua mạng Docker.

---

## 🧾 6. Minh chứng hệ thống chạy qua Docker + Postman

### ⚙️ A. Kiểm tra container đang chạy:
```bash
docker ps
```
→ Hiển thị các container:
```
api-gateway
auth
product
order
mongo
rabbitmq
```

### 🧪 B. Kiểm tra qua Postman:
1. **Đăng ký người dùng:**
   ```
   POST http://localhost:3003/api/auth/register
   Body: { "username": "admin", "email": "a@gmail.com", "password": "123456" }
   ```
2. **Đăng nhập:**
   ```
   POST http://localhost:3003/api/auth/login
   → nhận token JWT
   ```
3. **Thêm sản phẩm:**
   ```
   POST http://localhost:3003/api/products
   Headers: Authorization: Bearer <token>
   ```
4. **Tạo đơn hàng:**
   ```
   POST http://localhost:3003/api/orders
   ```
5. **Quan sát logs:**
   ```bash
   docker logs -f api-gateway
   docker logs -f product
   docker logs -f order
   ```
→ Khi gọi POSTMAN, logs trong container thay đổi → chứng minh request vào đúng Docker service.

---

## 🧮 7. Trả lời mẫu cho phần vấn đáp

| Câu hỏi | Câu trả lời ngắn gọn |
|----------|-----------------------|
| **1. Hệ thống giải quyết vấn đề gì?** | Quản lý người dùng, sản phẩm và đơn hàng cho cửa hàng trực tuyến; tách nhỏ bằng microservices để dễ mở rộng và deploy. |
| **2. Hệ thống có bao nhiêu dịch vụ?** | 4 service chính (auth, product, order, gateway) + 2 hạ tầng (MongoDB, RabbitMQ). |
| **3. Ý nghĩa từng dịch vụ?** | Auth: đăng nhập/đăng ký; Product: CRUD sản phẩm; Order: tạo đơn; Gateway: cổng giao tiếp duy nhất. |
| **4. Các mẫu thiết kế đã dùng?** | Layered, Repository, Proxy, Pub/Sub, Singleton. |
| **5. Các dịch vụ giao tiếp thế nào?** | HTTP REST (đồng bộ) qua API Gateway + RabbitMQ (bất đồng bộ giữa Order ↔ Product). |

---

## 📁 8. Cấu trúc thư mục (rút gọn)
```
EProject-Phase-2/
│
├── api-gateway/
├── auth/
├── product/
├── order/
├── docker-compose.yml
└── README.md
```

---

## 🧰 9. Công nghệ sử dụng

| Thành phần | Công nghệ |
|-------------|------------|
| Backend | Node.js + Express.js |
| Database | MongoDB |
| Message Queue | RabbitMQ |
| Container | Docker, Docker Compose |
| Authentication | JWT |
| Test tool | Postman |
| Pattern chính | Microservices Architecture |

---

## ✅ 10. Kết luận

Dự án áp dụng **kiến trúc microservices** để chia nhỏ hệ thống thương mại điện tử thành các dịch vụ độc lập:
- **Dễ mở rộng**, **dễ bảo trì**, **triển khai linh hoạt** bằng Docker.  
- Sử dụng **RabbitMQ** để xử lý bất đồng bộ giữa các service.  
- Đảm bảo **bảo mật** qua JWT, **ổn định** và **mở rộng tốt**.
