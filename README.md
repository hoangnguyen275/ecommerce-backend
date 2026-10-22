# 🛒 E-Commerce Store — Backend API

A RESTful API for an online store, built with **Node.js**, **Express**, and **Sequelize**. It manages products, the shopping cart, delivery options, payment summaries, and orders. It also serves the compiled React frontend, so the full app runs from a single server.

![Node.js](https://img.shields.io/badge/Node.js-22+-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4-000000?logo=express&logoColor=white)
![Sequelize](https://img.shields.io/badge/Sequelize-6-52B0E7?logo=sequelize&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-default-003B57?logo=sqlite&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-supported-4479A1?logo=mysql&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-supported-4169E1?logo=postgresql&logoColor=white)

> **Frontend:** the companion React app lives in [`ecommerce-project-ts`](../ecommerce-project-ts).

---

## 📑 Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Available Scripts](#-available-scripts)
- [Configuration](#-configuration)
- [Project Structure](#-project-structure)
- [Data Models](#-data-models)
- [API Reference](#-api-reference)
- [Business Rules](#-business-rules)
- [Serving the Frontend](#-serving-the-frontend)
- [Acknowledgements](#-acknowledgements)

---

## ✨ Features

- **REST API** for products, cart items, delivery options, orders, and payment summaries.
- **Product search** by name or keyword (case-insensitive).
- **Optional data expansion** with `?expand=` to include related product details or delivery estimates in one request.
- **Server-side pricing**: shipping, 10% tax, and totals are calculated on the server, never trusted from the client.
- **Zero-setup database**: uses an in-process SQLite engine ([sql.js](https://sql.js.org/)) that is saved to `database.sqlite` after every write. No database server needs to be installed.
- **Production-ready database option**: switch to **MySQL** or **PostgreSQL** (for example, AWS RDS) with environment variables.
- **Auto-seeding**: loads 40+ sample products, delivery options, a sample cart, and sample orders on first run.
- **Reset endpoint** to restore the default data at any time.
- **Static hosting** for product images and the built React frontend, with SPA fallback routing.

---

## 🧰 Tech Stack

| Category       | Technology |
|----------------|------------|
| Runtime        | Node.js 22+ (ES Modules) |
| Web framework  | Express 4 |
| ORM            | Sequelize 6 |
| Databases      | SQLite via `sql.js` (default), MySQL (`mysql2`), PostgreSQL (`pg`) |
| Middleware     | `cors`, `express.json()` |
| Dev tools      | Nodemon, ESLint, `patch-package`, Archiver |

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) **v22 or later**
- npm

### Installation

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>/ecommerce-backend
npm install        # also applies the sql.js patch via patch-package
npm run dev
```

The server starts at **http://localhost:3000**. On first launch the database is created and seeded with default data:

```
Default data added to the database.
Server is running on port 3000
```

Quick check:

```bash
curl http://localhost:3000/api/products
```

---

## 📜 Available Scripts

| Command        | Description |
|----------------|-------------|
| `npm start`    | Starts the server with Node. |
| `npm run dev`  | Starts the server with Nodemon (auto-restarts on file changes). |
| `npm run zip`  | Packages the project into `ecommerce-backend-<n>.zip` (excludes `node_modules` and the database). |

---

## 🔧 Configuration

By default no configuration is needed: the server uses SQLite and port `3000`. To use a hosted relational database, set **all three** of `RDS_HOSTNAME`, `RDS_USERNAME`, and `RDS_PASSWORD`.

| Variable        | Default  | Description |
|-----------------|----------|-------------|
| `PORT`          | `3000`   | Port the HTTP server listens on. |
| `DB_TYPE`       | `mysql`  | `mysql` or `postgres` (used only when the RDS variables are set). |
| `RDS_HOSTNAME`  | —        | Database host. |
| `RDS_PORT`      | `3306` / `5432` | Database port (defaults by `DB_TYPE`). |
| `RDS_DB_NAME`   | —        | Database name. |
| `RDS_USERNAME`  | —        | Database user. |
| `RDS_PASSWORD`  | —        | Database password. |

Example (`.env`-style):

```bash
PORT=3000
DB_TYPE=postgres
RDS_HOSTNAME=my-db.xxxxxx.us-east-1.rds.amazonaws.com
RDS_PORT=5432
RDS_DB_NAME=ecommerce
RDS_USERNAME=admin
RDS_PASSWORD=supersecret
```

> ⚠️ Never commit real credentials. Keep them in environment variables or a secrets manager.

---

## 🗂 Project Structure

```
ecommerce-backend/
├── server.js               # App entry: middleware, routes, static hosting, DB sync & seeding
├── models/
│   ├── index.js            # Sequelize setup (SQLite ↔ MySQL/Postgres) + save-to-file hooks
│   ├── Product.js
│   ├── CartItem.js
│   ├── DeliveryOption.js
│   └── Order.js
├── routes/
│   ├── products.js         # GET /api/products
│   ├── cartItems.js        # CRUD /api/cart-items
│   ├── deliveryOptions.js  # GET /api/delivery-options
│   ├── orders.js           # GET/POST /api/orders
│   ├── paymentSummary.js   # GET /api/payment-summary
│   └── reset.js            # POST /api/reset
├── defaultData/            # Seed data (products, delivery options, cart, orders)
├── images/                 # Product images & icons (served at /images)
├── dist/                   # Built React frontend (generated by the frontend build)
├── patches/                # patch-package fix for sql.js-as-sqlite3
├── documentation.md        # Detailed endpoint documentation
└── zipFiles.js             # Packaging script
```

---

## 🧱 Data Models

**Product**
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Primary key |
| `image` | string | Relative path, e.g. `images/products/...jpg` |
| `name` | string | |
| `rating` | JSON | `{ stars: number, count: number }` |
| `priceCents` | integer | Price in cents |
| `keywords` | string[] | Stored as a comma-separated string |

**CartItem**
| Field | Type | Notes |
|-------|------|-------|
| `productId` | UUID | References `Product.id` |
| `quantity` | integer | |
| `deliveryOptionId` | string | References `DeliveryOption.id` (defaults to `"1"`) |

**DeliveryOption**
| `id` | `deliveryDays` | `priceCents` |
|------|----------------|--------------|
| `"1"` | 7 | 0 (FREE) |
| `"2"` | 3 | 499 |
| `"3"` | 1 | 999 |

**Order**
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Primary key |
| `orderTimeMs` | bigint | Unix timestamp (ms) |
| `totalCostCents` | integer | Includes shipping and tax |
| `products` | JSON | `[{ productId, quantity, estimatedDeliveryTimeMs }]` |

---

## 📡 API Reference

Base URL: `http://localhost:3000/api`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET`    | `/products` | List products. `?search=term` filters by name/keywords. |
| `GET`    | `/delivery-options` | List delivery options. `?expand=estimatedDeliveryTime` adds `estimatedDeliveryTimeMs`. |
| `GET`    | `/cart-items` | List cart items. `?expand=product` includes product details. |
| `POST`   | `/cart-items` | Add a product to the cart (or increase its quantity). |
| `PUT`    | `/cart-items/:productId` | Update `quantity` and/or `deliveryOptionId`. |
| `DELETE` | `/cart-items/:productId` | Remove an item from the cart. |
| `GET`    | `/payment-summary` | Cost breakdown for the current cart. |
| `GET`    | `/orders` | List orders (newest first). `?expand=products` includes product details. |
| `POST`   | `/orders` | Create an order from the cart and empty the cart. |
| `GET`    | `/orders/:orderId` | Get one order. `?expand=products` supported. |
| `POST`   | `/reset` | Reset the database to its default data. |

📘 Full request/response schemas are in **[documentation.md](documentation.md)**.

### Examples

**Add to cart**
```bash
curl -X POST http://localhost:3000/api/cart-items \
  -H "Content-Type: application/json" \
  -d '{"productId": "e43638ce-6aa0-4b85-b27f-e1d07eb678c6", "quantity": 2}'
```
```json
{ "productId": "e43638ce-6aa0-4b85-b27f-e1d07eb678c6", "quantity": 2, "deliveryOptionId": "1" }
```

**Change delivery option**
```bash
curl -X PUT http://localhost:3000/api/cart-items/e43638ce-6aa0-4b85-b27f-e1d07eb678c6 \
  -H "Content-Type: application/json" \
  -d '{"deliveryOptionId": "3"}'
```

**Payment summary**
```bash
curl http://localhost:3000/api/payment-summary
```
```json
{
  "totalItems": 2,
  "productCostCents": 2180,
  "shippingCostCents": 999,
  "totalCostBeforeTaxCents": 3179,
  "taxCents": 318,
  "totalCostCents": 3497
}
```

**Place an order**
```bash
curl -X POST http://localhost:3000/api/orders
```

### Error responses

Errors return JSON in the form `{ "error": "message" }`:

| Status | When |
|--------|------|
| `400` | Product not found, invalid quantity, invalid delivery option, or empty cart when ordering |
| `404` | Cart item or order not found |
| `500` | Unexpected server error |

---

## 📏 Business Rules

- **Quantity** when adding to cart must be a number from **1 to 10**. Adding a product already in the cart increases its quantity.
- **Quantity** when updating must be **≥ 1**.
- New cart items default to **delivery option `"1"`** (free, 7 days).
- **Shipping** is charged **per cart item** based on its delivery option.
- **Tax** is **10%** of (products + shipping), rounded to the nearest cent.
- **Estimated delivery** = order time + `deliveryDays × 24h`.
- Placing an order **empties the cart**.
- All money values are **integers in cents** to avoid floating-point errors.

---

## 🌐 Serving the Frontend

The server also hosts the client app:

- `/images/*` → files in `images/`
- Any non-API route → files in `dist/`, falling back to `dist/index.html` for client-side routing.

To build the frontend into `dist/`:

```bash
cd ../ecommerce-project-ts
npm run build          # outputs to ../ecommerce-backend/dist
cd ../ecommerce-backend
npm start              # full app at http://localhost:3000
```
