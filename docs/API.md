# API Documentation

## Base URL
```
http://localhost:8000/api/v1
```

## Authentication

All protected endpoints require a JWT token in the Authorization header:
```
Authorization: Bearer <your_token_here>
```

## Response Format

All API responses follow this structure:

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Error description",
    "details": { ... }
  }
}
```

## Authentication Endpoints

### Register User
**POST** `/auth/register`

Create a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "first_name": "John",
  "last_name": "Doe",
  "phone": "+1234567890"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user_id": "uuid-here",
    "email": "user@example.com",
    "token": "jwt-token-here"
  }
}
```

### Login
**POST** `/auth/login`

Authenticate and receive access token.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "jwt-token-here",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "first_name": "John",
      "role": "customer"
    }
  }
}
```

## Product Endpoints

### List Products
**GET** `/products`

Get paginated list of products with filters.

**Query Parameters:**
- `page` (int): Page number (default: 1)
- `limit` (int): Items per page (default: 20, max: 100)
- `category` (string): Filter by category
- `search` (string): Search in title and description
- `min_price` (float): Minimum price filter
- `max_price` (float): Maximum price filter
- `sort_by` (string): Sort field (price, name, created_at)
- `order` (string): Sort order (asc, desc)

**Response:**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "uuid",
        "name": "Product Name",
        "description": "Product description",
        "price": 29.99,
        "sale_price": 24.99,
        "category": "Electronics",
        "images": ["url1", "url2"],
        "stock": 50,
        "rating": 4.5,
        "reviews_count": 120
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8
    }
  }
}
```

### Get Product Details
**GET** `/products/{product_id}`

Get detailed information about a specific product.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Product Name",
    "description": "Detailed description",
    "price": 29.99,
    "sale_price": 24.99,
    "category": {
      "id": "uuid",
      "name": "Electronics",
      "slug": "electronics"
    },
    "images": [
      {
        "url": "https://cdn.example.com/image.jpg",
        "alt": "Product image",
        "is_primary": true
      }
    ],
    "specifications": {
      "brand": "BrandName",
      "model": "XYZ-123",
      "color": "Black"
    },
    "stock": 50,
    "rating": 4.5,
    "reviews_count": 120,
    "created_at": "2025-01-01T00:00:00Z"
  }
}
```

## Cart Endpoints

### Get Cart
**GET** `/cart`
🔒 Requires authentication

Get current user's cart.

**Response:**
```json
{
  "success": true,
  "data": {
    "cart_id": "uuid",
    "items": [
      {
        "product_id": "uuid",
        "product_name": "Product Name",
        "price": 29.99,
        "quantity": 2,
        "subtotal": 59.98,
        "image": "url"
      }
    ],
    "subtotal": 59.98,
    "tax": 5.40,
    "shipping": 10.00,
    "total": 75.38
  }
}
```

### Add to Cart
**POST** `/cart/items`
🔒 Requires authentication

**Request Body:**
```json
{
  "product_id": "uuid",
  "quantity": 1
}
```

### Update Cart Item
**PUT** `/cart/items/{item_id}`
🔒 Requires authentication

**Request Body:**
```json
{
  "quantity": 3
}
```

### Remove from Cart
**DELETE** `/cart/items/{item_id}`
🔒 Requires authentication

## Order Endpoints

### Create Order
**POST** `/orders`
🔒 Requires authentication

Create a new order from cart items.

**Request Body:**
```json
{
  "shipping_address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zip_code": "10001",
    "country": "USA"
  },
  "billing_address": { ... },
  "payment_method": "razorpay"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "order_number": "ORD-20250101-0001",
    "total": 75.38,
    "status": "pending_payment",
    "payment_url": "https://razorpay.com/checkout/..."
  }
}
```

### Get Order History
**GET** `/orders`
🔒 Requires authentication

Get user's order history.

**Query Parameters:**
- `page` (int): Page number
- `limit` (int): Items per page
- `status` (string): Filter by status

**Response:**
```json
{
  "success": true,
  "data": {
    "orders": [
      {
        "order_id": "uuid",
        "order_number": "ORD-20250101-0001",
        "total": 75.38,
        "status": "delivered",
        "created_at": "2025-01-01T00:00:00Z",
        "items_count": 3
      }
    ],
    "pagination": { ... }
  }
}
```

### Get Order Details
**GET** `/orders/{order_id}`
🔒 Requires authentication

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "order_number": "ORD-20250101-0001",
    "status": "shipped",
    "items": [ ... ],
    "shipping_address": { ... },
    "tracking_number": "TRK123456",
    "total": 75.38,
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-02T00:00:00Z"
  }
}
```

## Payment Endpoints

### Create Razorpay Order
**POST** `/payments/razorpay/create`
🔒 Requires authentication

Initiate Razorpay payment.

**Request Body:**
```json
{
  "order_id": "uuid",
  "amount": 75.38
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "razorpay_order_id": "order_xyz",
    "amount": 7538,
    "currency": "INR",
    "key_id": "rzp_test_xxx"
  }
}
```

### Verify Payment
**POST** `/payments/razorpay/verify`
🔒 Requires authentication

**Request Body:**
```json
{
  "razorpay_order_id": "order_xyz",
  "razorpay_payment_id": "pay_xyz",
  "razorpay_signature": "signature"
}
```

## Admin Endpoints

### Get Analytics Dashboard
**GET** `/admin/analytics`
🔒 Requires admin authentication

Get comprehensive analytics data.

**Query Parameters:**
- `period` (string): today, week, month, year
- `start_date` (date): Custom start date
- `end_date` (date): Custom end date

**Response:**
```json
{
  "success": true,
  "data": {
    "revenue": {
      "total": 125000.50,
      "period_comparison": "+15%"
    },
    "orders": {
      "total": 542,
      "pending": 23,
      "shipped": 145,
      "delivered": 370,
      "cancelled": 4
    },
    "products": {
      "total": 1250,
      "out_of_stock": 15,
      "low_stock": 45
    },
    "customers": {
      "total": 3450,
      "new_this_period": 120
    },
    "top_products": [ ... ],
    "revenue_chart": [ ... ]
  }
}
```

### Manage Products (Admin)
**POST** `/admin/products`
🔒 Requires admin authentication

Create a new product.

**Request Body:**
```json
{
  "name": "New Product",
  "description": "Description",
  "price": 29.99,
  "category_id": "uuid",
  "stock": 100,
  "images": ["base64_or_url"]
}
```

**PUT** `/admin/products/{product_id}`
🔒 Requires admin authentication

Update product details.

**DELETE** `/admin/products/{product_id}`
🔒 Requires admin authentication

Delete a product.

## Error Codes

| Code | Description |
|------|-------------|
| `AUTH_INVALID` | Invalid authentication credentials |
| `AUTH_EXPIRED` | Token has expired |
| `AUTH_REQUIRED` | Authentication required |
| `NOT_FOUND` | Resource not found |
| `INVALID_INPUT` | Invalid request data |
| `OUT_OF_STOCK` | Product out of stock |
| `PAYMENT_FAILED` | Payment processing failed |
| `PERMISSION_DENIED` | Insufficient permissions |
| `RATE_LIMIT` | Rate limit exceeded |
| `SERVER_ERROR` | Internal server error |

## Rate Limiting

- **Guest users**: 60 requests per minute
- **Authenticated users**: 100 requests per minute
- **Admin users**: 200 requests per minute

Rate limit info is included in response headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1609459200
```

## Pagination

All list endpoints support pagination with these parameters:
- `page`: Page number (starting from 1)
- `limit`: Items per page (default: 20, max: 100)

Pagination info is included in responses:
```json
{
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "pages": 8,
    "has_next": true,
    "has_prev": false
  }
}
```

## Image Upload

For image uploads, use multipart/form-data:

**POST** `/admin/products/images`
🔒 Requires admin authentication

**Form Data:**
- `image`: File (max 5MB, supported: jpg, png, webp)
- `product_id`: UUID

**Response:**
```json
{
  "success": true,
  "data": {
    "url": "https://cdn.example.com/products/image.jpg",
    "thumbnail": "https://cdn.example.com/products/thumb_image.jpg",
    "size": 245678
  }
}
```
