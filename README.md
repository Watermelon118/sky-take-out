# Skytower_takeaway (苍穹外卖)

A full-stack food delivery platform consisting of a Spring Boot backend, a Vue 2 + TypeScript admin dashboard, and a WeChat Mini Program customer app.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Database Setup](#database-setup)
- [Backend Setup](#backend-setup)
- [Admin Dashboard Setup](#admin-dashboard-setup)
- [WeChat Mini Program Setup](#wechat-mini-program-setup)
- [Configuration Reference](#configuration-reference)
- [API Documentation](#api-documentation)
- [Deployment](#deployment)

---

## Overview

Skytower Eats is a complete food delivery system designed for restaurant management and customer ordering. It consists of three independent components that work together:

- **Backend API** — handles all business logic, authentication, order processing, and third-party integrations (Aliyun OSS, WeChat Pay)
- **Admin Dashboard** — a web-based management panel for restaurant staff to manage menus, orders, employees, and view business reports
- **Customer Mini Program** — a WeChat Mini Program for customers to browse menus, place orders, and track deliveries in real time

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                            │
│                                                                 │
│   ┌─────────────────────┐      ┌────────────────────────────┐  │
│   │  Admin Dashboard    │      │  WeChat Mini Program       │  │
│   │  Vue 2 + TypeScript │      │  Native WeChat MP API      │  │
│   │  Port: 8888         │      │  WeChat DevTools           │  │
│   └──────────┬──────────┘      └───────────────┬────────────┘  │
└──────────────┼─────────────────────────────────┼───────────────┘
               │ HTTP / REST API                 │ HTTP / REST API
               │ WebSocket                       │ WebSocket
┌──────────────┼─────────────────────────────────┼───────────────┐
│              ▼             Backend              ▼               │
│   ┌──────────────────────────────────────────────────────────┐ │
│   │               Spring Boot 2.7.3 (Port 8080)             │ │
│   │                                                          │ │
│   │   /admin/**  ──►  Admin Controllers  (JWT: token)        │ │
│   │   /user/**   ──►  User Controllers   (JWT: authentication)│ │
│   │   /notify/** ──►  Payment Webhooks                       │ │
│   │   /ws/**     ──►  WebSocket (real-time order updates)    │ │
│   └──────────────────┬───────────────┬───────────────────────┘ │
└─────────────────────┼───────────────┼─────────────────────────┘
                       │               │
           ┌───────────┘               └────────────┐
           ▼                                        ▼
   ┌───────────────┐                      ┌─────────────────┐
   │     MySQL     │                      │      Redis      │
   │  sky_take_out │                      │  localhost:6379 │
   │  Port: 3306   │                      │  Database: 10   │
   └───────────────┘                      └─────────────────┘

External Services:
  ┌──────────────┐   ┌─────────────────┐   ┌──────────────────┐
  │  Aliyun OSS  │   │  WeChat Pay v3  │   │  Baidu Maps API  │
  │ (image upload│   │  (order payment)│   │ (distance calc.) │
  └──────────────┘   └─────────────────┘   └──────────────────┘
```

---

## Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Backend Framework | Spring Boot | 2.7.3 |
| Language (Backend) | Java | 17 |
| ORM | MyBatis | 2.2.0 |
| Connection Pool | Druid | 1.2.1 |
| Database | MySQL | 5.7+ |
| Cache | Redis | 6.x+ |
| Authentication | JWT (JJWT) | 0.9.1 |
| File Storage | Aliyun OSS SDK | 3.10.2 |
| Payment | WeChat Pay API | v3 |
| Real-time | WebSocket (Spring) | — |
| Reporting | Apache POI | 3.16 |
| Pagination | PageHelper | 1.3.0 |
| API Docs | Knife4j (Swagger) | 3.0.2 |
| Frontend Framework | Vue | 2.6.10 |
| Language (Frontend) | TypeScript | 3.6.2 |
| UI Component Library | Element UI | 2.12.0 |
| State Management | Vuex | 3.1.1 |
| HTTP Client | Axios | 0.19.0 |
| Charts | ECharts | 5.3.2 |
| Build Tool | Vue CLI | 3.11.0 |
| Mini Program | Native WeChat MP API | lib 2.24.2 |
| Web Server | Nginx | latest |

---

## Project Structure

```
skytower_takeaway/
├── sky-take-out/                     # Backend (Maven multi-module)
│   ├── sky-common/                   # Shared utilities, JWT, OSS, WeChat Pay
│   ├── sky-pojo/                     # Entities, DTOs, VOs (Swagger annotated)
│   └── sky-server/                   # Main application
│       ├── src/main/java/com/sky/
│       │   ├── controller/
│       │   │   ├── admin/            # Admin API controllers
│       │   │   └── user/             # Customer API controllers
│       │   ├── service/              # Business logic layer
│       │   ├── mapper/               # MyBatis mapper interfaces
│       │   ├── interceptor/          # JWT authentication interceptors
│       │   ├── aspect/               # AOP (auto-fill timestamps & user IDs)
│       │   ├── config/               # Redis, WebMvc, WebSocket, OSS configs
│       │   ├── task/                 # Scheduled background tasks
│       │   └── websocket/            # Real-time WebSocket handler
│       └── src/main/resources/
│           ├── application.yml       # Base configuration
│           ├── application-dev.yml   # Development configuration
│           └── mapper/               # MyBatis XML mapper files
│
├── project-sky-admin-vue-ts/         # Admin Dashboard (Vue 2 + TypeScript)
│   ├── src/
│   │   ├── api/                      # Axios API service layer
│   │   ├── views/                    # Page components
│   │   ├── components/               # Reusable UI components
│   │   ├── store/                    # Vuex state management
│   │   ├── router.ts                 # Route configuration
│   │   └── permission.ts             # Route guard (auth check)
│   ├── .env.development              # Dev environment variables
│   ├── .env.production               # Prod environment variables
│   ├── vue.config.js                 # Dev server & proxy config
│   └── Dockerfile                    # Nginx-based Docker image
│
├── miniProgram/
│   └── mp-weixin/                    # WeChat Mini Program
│       ├── pages/                    # App pages (index, order, pay, etc.)
│       ├── components/               # Reusable components
│       ├── common/                   # Shared utilities
│       ├── app.js                    # App entry point
│       ├── app.json                  # App routing & window config
│       └── project.config.json       # WeChat DevTools project config
│
└── script.sql                        # Full database initialization script
```

---

## Prerequisites

Make sure the following are installed before getting started:

| Tool | Version | Notes |
|---|---|---|
| JDK | 17 | Required for Spring Boot 2.7.3 |
| Maven | 3.6+ | Backend build tool |
| MySQL | 5.7+ | Primary database |
| Redis | 6.x+ | Session cache & shop status |
| Node.js | 14.x or 16.x | Frontend build environment |
| npm | 6.x+ | Comes with Node.js |
| WeChat DevTools | latest | [Download here](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html) |

Optional but recommended:
- **IntelliJ IDEA** — for backend development
- **VS Code** — for frontend development

---

## Database Setup

**1. Create the database:**

```sql
CREATE DATABASE sky_take_out CHARACTER SET utf8mb3;
```

**2. Import the schema and seed data:**

```bash
mysql -u root -p sky_take_out < script.sql
```

This script creates all 10 tables and inserts default data:

| Table | Description |
|---|---|
| `employee` | Restaurant staff accounts |
| `user` | Customer accounts (WeChat login) |
| `category` | Dish and setmeal categories |
| `dish` | Individual menu items |
| `dish_flavor` | Flavor/option variants per dish |
| `setmeal` | Bundled meal packages |
| `setmeal_dish` | Items included in each setmeal |
| `orders` | Customer orders (status 1–7) |
| `order_detail` | Line items within each order |
| `shopping_cart` | Active shopping carts per user |
| `address_book` | Saved delivery addresses |

> **Order status reference:**  
> `1` = Unpaid · `2` = Pending acceptance · `3` = Accepted · `4` = In delivery · `5` = Completed · `6` = Cancelled · `7` = Refunded

---

## Backend Setup

### 1. Configure the application

Edit `sky-take-out/sky-server/src/main/resources/application-dev.yml`:

```yaml
sky:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    host: localhost
    port: 3306
    database: sky_take_out
    username: root           # Change to your MySQL username
    password: 123456         # Change to your MySQL password

  redis:
    host: localhost
    port: 6379
    database: 10             # Redis database index

  alioss:
    endpoint: oss-cn-hangzhou.aliyuncs.com    # Your OSS region endpoint
    access-key-id: YOUR_ACCESS_KEY_ID
    access-key-secret: YOUR_ACCESS_KEY_SECRET
    bucket-name: YOUR_BUCKET_NAME

  wechat:
    appid: YOUR_WECHAT_APPID
    secret: YOUR_WECHAT_APP_SECRET
    mchid: YOUR_MERCHANT_ID
    mchSerialNo: YOUR_MERCHANT_CERT_SERIAL
    privateKeyFilePath: /path/to/apiclient_key.pem
    apiV3Key: YOUR_API_V3_KEY
    weChatPayCertFilePath: /path/to/wechatpay_cert.pem
    notifyUrl: http://YOUR_DOMAIN/notify/paySuccess
    refundNotifyUrl: http://YOUR_DOMAIN/notify/refundSuccess

  baidu:
    ak: YOUR_BAIDU_MAP_AK    # Used for address distance calculation
```

> **Note:** If you do not need WeChat Pay or Aliyun OSS during development, you can stub out those service beans or configure any valid placeholders. The app will still start and core features (menu browsing, cart, admin panel) will work.

### 2. Build and run

```bash
cd sky-take-out

# Install all dependencies and build all modules
mvn clean install -DskipTests

# Start the server
mvn spring-boot:run -pl sky-server
```

Alternatively, open the project in IntelliJ IDEA and run `SkyApplication.java` directly.

**The server starts on port `8080`.**

You can verify it is running by visiting:
- API Docs (Swagger UI): `http://localhost:8080/doc.html`

### 3. Default admin credentials

After importing `script.sql`, the default admin account is:

| Field | Value |
|---|---|
| Username | `admin` |
| Password | `123456` |

> Passwords are stored as MD5 hashes in the database.

---

## Admin Dashboard Setup

### 1. Install dependencies

```bash
cd project-sky-admin-vue-ts
npm install
```

### 2. Configure environment

The development environment file `.env.development` is pre-configured to proxy API calls to the backend:

```env
VUE_APP_BASE_API = '/api'
VUE_APP_URL = 'http://localhost:8080/admin'
VUE_APP_SOCKET_URL = 'ws://localhost:8080/ws/'
```

The dev server (`vue.config.js`) proxies all `/api` requests to `http://localhost:8080/admin`, so no CORS issues in development.

If your backend runs on a different port, update `VUE_APP_URL` in `.env.development` and the proxy target in `vue.config.js` accordingly.

### 3. Start the development server

```bash
npm run serve
```

The admin dashboard is available at `http://localhost:8888`.

Login with the default admin credentials (`admin` / `123456`).

### 4. Build for production

```bash
npm run build
```

The compiled static files are output to the `dist/` folder. Serve them with any static file server (Nginx recommended).

**Available build commands:**

| Command | Output | Notes |
|---|---|---|
| `npm run serve` | Dev server on :8888 | Hot-reload enabled |
| `npm run build` | `dist/` (production) | Uses `.env.production` |
| `npm run build:uat` | `dist/` (UAT) | Uses `.env.production.uat` |
| `npm run lint` | — | ESLint validation |
| `npm run test:unit` | — | Jest unit tests |
| `npm run test:e2e` | — | Cypress end-to-end tests |

---

## WeChat Mini Program Setup

### 1. Install WeChat DevTools

Download and install [WeChat DevTools](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html).

### 2. Import the project

1. Open WeChat DevTools
2. Click **"Import Project"**
3. Set the project directory to `miniProgram/mp-weixin`
4. Enter the AppID: `wx0b11a8df33f11a8b` (or your own registered AppID if you have one)
5. Click **"OK"**

### 3. Configure the backend URL

In the mini program's common utility files (under `mp-weixin/common/`), update the base API URL to point to your running backend. In development you can use an HTTP URL; for production WeChat requires HTTPS with a verified domain.

### 4. Run in simulator

Click **"Compile"** in WeChat DevTools. The app will load in the built-in simulator. You can also scan the preview QR code to test on a real device (requires the AppID to have dev permissions in WeChat's open platform).

### App pages

| Page | Route | Description |
|---|---|---|
| Home | `pages/index/index` | Menu browsing and cart |
| Orders | `pages/order/index` | Current order status |
| Order Details | `pages/details/index` | Single order detail |
| Payment | `pages/pay/index` | WeChat Pay checkout |
| Success | `pages/success/index` | Order placed confirmation |
| Address | `pages/address/address` | Saved addresses list |
| Add/Edit Address | `pages/addOrEditAddress/index` | Address form |
| Remark | `pages/remark/index` | Order notes/special requests |
| Profile | `pages/my/my` | User profile |
| Order History | `pages/historyOrder/index` | Past orders |

---

## Configuration Reference

### Backend JWT settings (`application.yml`)

| Key | Default | Description |
|---|---|---|
| `sky.jwt.admin-secret-key` | `itcast` | JWT signing key for admin tokens |
| `sky.jwt.admin-ttl` | `7200000` | Admin token TTL in ms (2 hours) |
| `sky.jwt.admin-token-name` | `token` | Request header name for admin JWT |
| `sky.jwt.user-secret-key` | `itheima` | JWT signing key for user tokens |
| `sky.jwt.user-ttl` | `7200000` | User token TTL in ms (2 hours) |
| `sky.jwt.user-token-name` | `authentication` | Request header name for user JWT |

### Public (no auth required) endpoints

| Endpoint | Description |
|---|---|
| `POST /admin/employee/login` | Admin login |
| `POST /user/user/login` | Customer WeChat login |
| `GET /user/shop/status` | Get shop open/closed status |

### Redis database usage

The backend uses **Redis database 10** for:
- Shop open/closed status flag
- Spring Cache (`@Cacheable`) for dish and setmeal listings

---

## API Documentation

When the backend is running, interactive API documentation is available at:

```
http://localhost:8080/doc.html
```

This is powered by Knife4j (Swagger UI). All endpoints are grouped by controller and include request/response schemas.

**WebSocket endpoint:**
```
ws://localhost:8080/ws/{sessionId}
```
Used by the admin dashboard to receive real-time order notifications.

---

## Deployment

### Backend (JAR)

```bash
cd sky-take-out
mvn clean package -DskipTests
java -jar sky-server/target/sky-server-1.0-SNAPSHOT.jar
```

For production, set the active profile and override sensitive configs via environment variables or an external config file:

```bash
java -jar sky-server.jar \
  --spring.profiles.active=prod \
  --sky.datasource.password=YOUR_PROD_PASSWORD \
  --sky.redis.host=YOUR_REDIS_HOST
```

### Frontend (Nginx)

1. Build the static files:
   ```bash
   cd project-sky-admin-vue-ts
   npm run build
   ```

2. Copy `dist/` to your Nginx web root:
   ```bash
   cp -r dist/ /usr/share/nginx/html/admin
   ```

3. Configure Nginx to proxy `/api` to the backend:
   ```nginx
   server {
       listen 80;

       location / {
           root /usr/share/nginx/html/admin;
           try_files $uri $uri/ /index.html;
       }

       location /api/ {
           proxy_pass http://localhost:8080/admin/;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

### Frontend (Docker)

A `Dockerfile` is included in `project-sky-admin-vue-ts/`:

```bash
# Build the Vue app first
npm run build

# Build the Docker image
docker build -t skytower-admin .

# Run the container
docker run -d -p 80:80 skytower-admin
```

---

## License

This project is for educational purposes. All third-party service credentials in the config files (Aliyun OSS, WeChat Pay, etc.) are demo values and must be replaced with your own before any real deployment.
