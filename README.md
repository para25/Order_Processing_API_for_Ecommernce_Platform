# AeonX Assignment - Order Management System

A robust Node.js/Express backend application for order management with user authentication, background job processing, and Redis caching.

## 🚀 Features

- **User Authentication & Authorization**
  - JWT-based authentication
  - Role-based access control (User & Admin)
  - Secure password hashing with bcrypt

- **Order Management**
  - Create orders with multiple items
  - View order details
  - List orders with pagination and filtering
  - Admin-only order status updates

- **Background Job Processing**
  - Email notifications using BullMQ
  - Invoice generation with Redis queue
  - Async job processing with workers

- **Caching**
  - Redis-based caching for improved performance
  - Automatic cache invalidation

- **Data Validation**
  - Request validation using Joi
  - MongoDB schema validation

## 📋 Prerequisites

- Node.js (v14 or higher)
- MongoDB Atlas account or local MongoDB
- Redis (via Docker or local installation)
- Docker (optional, for Redis)

## 🛠️ Installation

### 1. Clone the Repository
```bash
git clone https://github.com/para25/AeonX_Assignment.git
cd AeonX_Assignment
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Variables

The `.env` file is included in the repository with the following variables:

```env
MONGODB_URI=MOONGODB_URL
JWT_SECRET=AddSecretKey
JWT_EXPIRES_IN=AddExpiryTime
REDIS_URL=redis://localhost:6379
```

### 4. Start Redis (using Docker)
```bash
docker run -d -p 6379:6379 redis
```

Or if you have Redis installed locally:
```bash
redis-server
```

## 🏃‍♂️ Running the Application

### Start the API Server
```bash
npm run dev
```
Server will run on: `http://localhost:5000`

### Start the Background Worker (in a separate terminal)
```bash
npm run worker
```

## 📚 API Documentation

### Base URL
```
http://localhost:5000/api
```

---

## 🔐 Authentication Endpoints

### 1. Register User
**POST** `/api/auth/register`

**Request Body:**
```json
{
  "name": "Prasad Kumar",
  "email": "prasad@example.com",
  "password": "123456",
  "role": "User"
}
```

**Response:**
```json
{
  "data": {
    "user": {
      "id": "6931646ffb948c9a9e813a6d",
      "name": "Prasad Kumar",
      "email": "prasad@example.com",
      "role": "User",
      "createdAt": "2025-12-04T10:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**Note:** To create an Admin, set `"role": "Admin"` in the request body.

---

### 2. Login
**POST** `/api/auth/login`

**Request Body:**
```json
{
  "email": "prasad@example.com",
  "password": "123456"
}
```

**Response:**
```json
{
  "data": {
    "user": {
      "id": "6931646ffb948c9a9e813a6d",
      "name": "Prasad Kumar",
      "email": "prasad@example.com",
      "role": "User"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

## 📦 Order Endpoints

### 1. Create Order
**POST** `/api/orders`

**Headers:**
```
Authorization: Bearer <your-token>
```

**Request Body:**
```json
{
  "items": [
    {
      "itemId": "item001",
      "name": "Gaming Laptop",
      "price": 75000,
      "quantity": 1
    },
    {
      "itemId": "item002",
      "name": "Wireless Mouse",
      "price": 1500,
      "quantity": 2
    }
  ],
  "shippingAddress": {
    "fullName": "Prasad Kumar",
    "phone": "9876543210",
    "addressLine1": "456 Tech Street",
    "addressLine2": "Floor 3",
    "city": "Mumbai",
    "state": "Maharashtra",
    "country": "India",
    "postalCode": "400001"
  }
}
```

**Response:**
```json
{
  "data": {
    "_id": "693186cdc1b048ba4fba66ac",
    "userId": "6931646ffb948c9a9e813a6d",
    "items": [
      {
        "itemId": "item001",
        "name": "Gaming Laptop",
        "price": 75000,
        "quantity": 1,
        "total": 75000
      },
      {
        "itemId": "item002",
        "name": "Wireless Mouse",
        "price": 1500,
        "quantity": 2,
        "total": 3000
      }
    ],
    "totalAmount": 78000,
    "status": "Pending",
    "shippingAddress": { ... },
    "invoiceUrl": null,
    "createdAt": "2025-12-04T11:00:00.000Z",
    "updatedAt": "2025-12-04T11:00:00.000Z"
  }
}
```

**Note:** Background jobs are triggered automatically:
- Email notification sent to user
- Invoice generated and URL updated after ~1.5 seconds

---

### 2. List Orders
**GET** `/api/orders?page=1&limit=10`

**Headers:**
```
Authorization: Bearer <your-token>
```

**Query Parameters:**
- `page` (optional, default: 1)
- `limit` (optional, default: 10)
- `status` (optional): Filter by "Pending", "Shipped", "Delivered", "Cancelled"
- `userId` (Admin only): Filter by specific user
- `from` (optional): Filter from date (ISO format)
- `to` (optional): Filter to date (ISO format)

**Response:**
```json
{
  "data": [
    {
      "_id": "693186cdc1b048ba4fba66ac",
      "userId": "6931646ffb948c9a9e813a6d",
      "items": [...],
      "totalAmount": 78000,
      "status": "Pending",
      "shippingAddress": {...},
      "invoiceUrl": "https://fake-invoices.com/invoice_693186cdc1b048ba4fba66ac.pdf",
      "createdAt": "2025-12-04T11:00:00.000Z",
      "updatedAt": "2025-12-04T11:00:02.500Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "total": 15,
  "totalPages": 2
}
```

**Access Control:**
- **Users**: Can only see their own orders
- **Admins**: Can see all orders

---

### 3. Get Order by ID
**GET** `/api/orders/:id`

**Headers:**
```
Authorization: Bearer <your-token>
```

**Example:**
```
GET /api/orders/693186cdc1b048ba4fba66ac
```

**Response:**
```json
{
  "data": {
    "_id": "693186cdc1b048ba4fba66ac",
    "userId": "6931646ffb948c9a9e813a6d",
    "items": [...],
    "totalAmount": 78000,
    "status": "Pending",
    "shippingAddress": {...},
    "invoiceUrl": "https://fake-invoices.com/invoice_693186cdc1b048ba4fba66ac.pdf",
    "createdAt": "2025-12-04T11:00:00.000Z",
    "updatedAt": "2025-12-04T11:00:02.500Z"
  }
}
```

**Access Control:**
- **Users**: Can only view their own orders
- **Admins**: Can view any order

---

### 4. Update Order Status (Admin Only)
**PATCH** `/api/orders/:id/status`

**Headers:**
```
Authorization: Bearer <admin-token>
```

**Request Body:**
```json
{
  "status": "Shipped"
}
```

**Valid Status Values:**
- `Pending`
- `Shipped`
- `Delivered`
- `Cancelled`

**Response:**
```json
{
  "data": {
    "_id": "693186cdc1b048ba4fba66ac",
    "userId": "6931646ffb948c9a9e813a6d",
    "items": [...],
    "totalAmount": 78000,
    "status": "Shipped",
    "shippingAddress": {...},
    "invoiceUrl": "https://fake-invoices.com/invoice_693186cdc1b048ba4fba66ac.pdf",
    "createdAt": "2025-12-04T11:00:00.000Z",
    "updatedAt": "2025-12-04T11:05:00.000Z"
  }
}
```

**Access Control:**
- **Admin only**: Only admins can update order status

---

## 🏗️ Project Structure

```
AeonX Assignment/
├── src/
│   ├── config/
│   │   └── db.js                 # MongoDB connection
│   ├── controllers/
│   │   ├── auth.controller.js    # Authentication logic
│   │   └── order.controller.js   # Order management logic
│   ├── middleware/
│   │   ├── auth.middleware.js    # JWT verification & authorization
│   │   ├── order.validation.js   # Order validation schemas
│   │   └── validate.middleware.js # Joi validation middleware
│   ├── models/
│   │   ├── user.model.js         # User schema
│   │   └── order.model.js        # Order schema
│   ├── queues/
│   │   ├── bull.config.js        # BullMQ & Redis configuration
│   │   ├── order.queue.js        # Queue job definitions
│   │   └── worker.js             # Background job workers
│   ├── routes/
│   │   ├── index.js              # Main router
│   │   ├── auth.routes.js        # Auth routes
│   │   └── order.routes.js       # Order routes
│   ├── services/                 # Business logic services
│   ├── utils/
│   │   ├── cache.js              # Redis cache utilities
│   │   └── token.util.js         # JWT utilities
│   ├── app.js                    # Express app configuration
│   └── server.js                 # Server entry point
├── .env                          # Environment variables
├── .gitignore                    # Git ignore rules
├── package.json                  # Dependencies
└── README.md                     # This file
```

---

## 🔧 Technologies Used

- **Node.js** - Runtime environment
- **Express.js** - Web framework
- **MongoDB** - Database (Atlas)
- **Mongoose** - ODM for MongoDB
- **Redis** - Caching & job queue
- **BullMQ** - Background job processing
- **JWT** - Authentication
- **Bcrypt** - Password hashing
- **Joi** - Request validation
- **Nodemailer** - Email service (configured)

---

## 🧪 Testing with Postman

### Step 1: Register a User
1. POST `http://localhost:5000/api/auth/register`
2. Save the returned `token`

### Step 2: Register an Admin
1. POST `http://localhost:5000/api/auth/register` with `"role": "Admin"`
2. Save the admin `token`

### Step 3: Create an Order
1. POST `http://localhost:5000/api/orders`
2. Add `Authorization: Bearer <user-token>` header
3. Check worker terminal for background job logs

### Step 4: List Orders
1. GET `http://localhost:5000/api/orders`
2. Add `Authorization: Bearer <user-token>` header

### Step 5: Get Order by ID
1. GET `http://localhost:5000/api/orders/<order-id>`
2. Add `Authorization: Bearer <user-token>` header

### Step 6: Update Order Status (Admin)
1. PATCH `http://localhost:5000/api/orders/<order-id>/status`
2. Add `Authorization: Bearer <admin-token>` header
3. Body: `{"status": "Shipped"}`

---

## 🐛 Common Issues & Solutions

### Issue: "Redis connection refused"
**Solution:** Make sure Redis is running
```bash
docker run -d -p 6379:6379 redis
```

### Issue: "MongoDB connection error"
**Solution:** Check your MongoDB URI in `.env` file

### Issue: "Invoice URL is null"
**Solution:** Make sure the worker is running
```bash
npm run worker
```

### Issue: "Access denied" on order endpoints
**Solution:** Make sure you're using the correct JWT token in the Authorization header

---

## 📝 Assignment Notes

- **Email Sending**: Currently simulated (console logs). Real implementation would use nodemailer with SMTP configuration.
- **Invoice Generation**: Currently generates fake URLs. Real implementation would use PDF libraries like `pdfkit` or `puppeteer` and cloud storage.
- **Background Jobs**: Fully functional with Redis and BullMQ, demonstrating async processing.
- **Caching**: Implemented with Redis for order retrieval with 5-minute TTL.

---

## 👨‍💻 Author

**Prasad Jumare**
- GitHub: [@para25](https://github.com/para25)
- Repository: [AeonX_Assignment](https://github.com/para25/AeonX_Assignment)

---

## 📄 License

This project is created for assignment purposes.

---

## 🙏 Acknowledgments

Built as part of the AeonX technical assignment to demonstrate:
- RESTful API design
- Authentication & Authorization
- Background job processing
- Caching strategies
- Clean code architecture
