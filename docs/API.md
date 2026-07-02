# API Documentation - AFRILUX CCA Dashboard

## Base URL

```
http://localhost:5000/api
```

## Authentication

All endpoints require JWT token in Authorization header:

```
Authorization: Bearer <token>
```

## Response Format

All responses follow this format:

```json
{
  "data": {},
  "error": null,
  "timestamp": "2026-07-02T08:00:00Z",
  "status": 200
}
```

## Endpoints

### Authentication

#### POST /auth/login
Authenticate user with credentials.

**Request:**
```json
{
  "username": "user@afriluxsmart.cm",
  "password": "password"
}
```

**Response:**
```json
{
  "user": {
    "id": "uuid",
    "username": "user@afriluxsmart.cm",
    "email": "user@afriluxsmart.cm",
    "roles": ["COMM"]
  },
  "token": "jwt_token",
  "expiresIn": "24h"
}
```

#### POST /auth/logout
Logout and invalidate token.

#### GET /auth/me
Get current authenticated user.

---

### Orders

#### GET /api/orders
List all orders with pagination and filters.

**Query Parameters:**
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 30)
- `status` (string): Filter by status code
- `client` (string): Filter by client ID
- `dateFrom` (ISO 8601): Filter by creation date
- `dateTo` (ISO 8601): Filter by creation date
- `search` (string): Search in order number and description
- `archived` (boolean): Include archived orders

**Response:**
```json
{
  "orders": [
    {
      "id": "uuid",
      "orderNumber": "CCA-2026-001",
      "description": "Hardware purchase",
      "amount": 5000000,
      "currency": "XAF",
      "client": "CCA",
      "createdAt": "2026-07-02T00:00:00Z",
      "createdBy": "user_id",
      "status": "PF-01"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 30,
    "total": 50,
    "pages": 2
  }
}
```

#### POST /api/orders
Create new order.

**Request:**
```json
{
  "description": "Hardware purchase",
  "amount": 5000000,
  "currency": "XAF",
  "client": "CCA"
}
```

**Response:** (201 Created)
```json
{
  "id": "uuid",
  "orderNumber": "CCA-2026-001",
  "description": "Hardware purchase",
  "amount": 5000000,
  "createdAt": "2026-07-02T00:00:00Z"
}
```

#### GET /api/orders/:id
Get order details with all related objects.

**Response:**
```json
{
  "order": {
    "id": "uuid",
    "orderNumber": "CCA-2026-001",
    "description": "Hardware purchase",
    "amount": 5000000,
    "objects": {
      "proforma": { "id": "uuid", "status": "PF-04" },
      "purchaseOrder": { "id": "uuid", "status": "BC-02" },
      "delivery": { "id": "uuid", "status": "LV-01" }
    }
  }
}
```

#### PUT /api/orders/:id/status
Update order object status with workflow validation.

**Request:**
```json
{
  "objectType": "PROFORMA",
  "newStatus": "PF-02",
  "reason": "Information collected"
}
```

**Validation Rules:**
- User must have required role for transition
- Previous status must be valid predecessor
- All preconditions must be met (e.g., BC-01 must exist before BC-02)

**Response:** (200 OK)
```json
{
  "orderId": "uuid",
  "objectType": "PROFORMA",
  "newStatus": "PF-02",
  "changedAt": "2026-07-02T08:00:00Z",
  "changedBy": "user_id"
}
```

#### GET /api/orders/:id/audit
Get complete audit trail for order.

**Query Parameters:**
- `page` (integer): Page number
- `limit` (integer): Items per page
- `action` (string): Filter by action type
- `dateFrom` (ISO 8601): Filter by date range
- `dateTo` (ISO 8601): Filter by date range

**Response:**
```json
{
  "orderId": "uuid",
  "logs": [
    {
      "id": "uuid",
      "timestamp": "2026-07-02T08:00:00Z",
      "user": "username",
      "action": "STATUS_CHANGE",
      "objectType": "PROFORMA",
      "oldValue": "PF-01",
      "newValue": "PF-02",
      "reason": "Information collected",
      "ipAddress": "192.168.1.1"
    }
  ],
  "pagination": {
    "total": 15,
    "page": 1,
    "limit": 10
  }
}
```

---

### Dashboards

#### GET /api/dashboards/:role
Get role-specific dashboard data.

**Path Parameters:**
- `role`: One of COMM, FACT, MAG, ACH, COMPT, RECOV, FISC, DG

**Response:**
```json
{
  "role": "COMM",
  "dashboard": {
    "activeOrders": [
      {
        "id": "uuid",
        "orderNumber": "CCA-2026-001",
        "status": "PF-01",
        "daysInStatus": 2
      }
    ],
    "kpis": {
      "activeCount": 12,
      "avgCycleTime": 15,
      "completionRate": 85
    },
    "alerts": [
      {
        "id": "uuid",
        "type": "WARNING",
        "message": "Proforma without action > 7 days",
        "affectedOrders": 2
      }
    ]
  }
}
```

#### GET /api/dashboards/:role/kpis
Get detailed KPIs for role.

**Response:**
```json
{
  "role": "COMM",
  "kpis": {
    "proformasWaiting": 12,
    "proformasInProgress": 5,
    "transformationRate": 85.5,
    "avgProformaTime": 4.2,
    "totalAmount": 50000000
  },
  "period": "2026-07-01T00:00:00Z to 2026-07-02T23:59:59Z",
  "comparison": {
    "previous": {},
    "change": {}
  }
}
```

#### GET /api/dashboards/:role/alerts
Get current alerts for role.

**Query Parameters:**
- `severity`: INFO, WARNING, ERROR, CRITICAL
- `resolved`: boolean

**Response:**
```json
{
  "role": "COMM",
  "alerts": [
    {
      "id": "uuid",
      "type": "PROFORMA_OVERDUE",
      "severity": "WARNING",
      "message": "Proforma CCA-2026-001 without action for 7+ days",
      "orderId": "uuid",
      "createdAt": "2026-07-02T00:00:00Z",
      "resolved": false
    }
  ],
  "summary": {
    "critical": 0,
    "error": 1,
    "warning": 3,
    "info": 5
  }
}
```

---

### Notifications

#### GET /api/notifications
List user notifications.

**Query Parameters:**
- `page` (integer): Page number
- `limit` (integer): Items per page
- `unreadOnly` (boolean): Show unread only
- `type`: Filter by notification type

**Response:**
```json
{
  "notifications": [
    {
      "id": "uuid",
      "type": "STATUS_CHANGE",
      "title": "Proforma Updated",
      "message": "Proforma CCA-2026-001 moved to EN SIGNATURE",
      "read": false,
      "severity": "INFO",
      "actionUrl": "/orders/uuid",
      "createdAt": "2026-07-02T08:00:00Z"
    }
  ],
  "unreadCount": 5,
  "total": 25
}
```

#### PUT /api/notifications/:id/read
Mark notification as read.

**Response:** (200 OK)
```json
{
  "id": "uuid",
  "read": true,
  "readAt": "2026-07-02T08:05:00Z"
}
```

#### DELETE /api/notifications/:id
Delete notification.

**Response:** (204 No Content)

#### POST /api/notifications/mark-all-read
Mark all notifications as read.

**Response:** (200 OK)
```json
{
  "marked": 5,
  "timestamp": "2026-07-02T08:05:00Z"
}
```

---

### Admin

#### GET /api/admin/users
List all users (admin only).

**Query Parameters:**
- `page`, `limit`, `search`, `role`, `active`

**Response:**
```json
{
  "users": [
    {
      "id": "uuid",
      "username": "user@afriluxsmart.cm",
      "email": "user@afriluxsmart.cm",
      "firstName": "John",
      "lastName": "Doe",
      "roles": ["COMM"],
      "isActive": true,
      "lastLogin": "2026-07-02T07:00:00Z",
      "createdAt": "2026-06-01T00:00:00Z"
    }
  ],
  "pagination": { "total": 50, "page": 1, "limit": 30 }
}
```

#### POST /api/admin/users
Create new user (admin only).

**Request:**
```json
{
  "username": "newuser",
  "email": "newuser@afriluxsmart.cm",
  "firstName": "Jane",
  "lastName": "Smith",
  "roles": ["FACT", "COMPT"]
}
```

**Response:** (201 Created)
```json
{
  "id": "uuid",
  "username": "newuser",
  "email": "newuser@afriluxsmart.cm"
}
```

#### GET /api/admin/audit-logs
Get system audit logs (admin only).

**Query Parameters:**
- `page`, `limit`, `dateFrom`, `dateTo`, `action`, `userId`, `orderId`

**Response:**
```json
{
  "logs": [
    {
      "id": "uuid",
      "timestamp": "2026-07-02T08:00:00Z",
      "user": "username",
      "action": "STATUS_CHANGE",
      "objectType": "ORDER",
      "oldValue": {},
      "newValue": {},
      "ipAddress": "192.168.1.1",
      "userAgent": "Mozilla/5.0..."
    }
  ],
  "pagination": { "total": 1000, "page": 1, "limit": 30 }
}
```

#### POST /api/admin/config
Update system configuration (admin only).

**Request:**
```json
{
  "alertThresholds": {
    "proformaOverdueDay": 7,
    "deliveryOverdueDay": 10,
    "paymentOverdueDay": 30
  },
  "notifications": {
    "emailEnabled": true,
    "slackEnabled": false
  }
}
```

**Response:** (200 OK)
```json
{
  "updated": true,
  "timestamp": "2026-07-02T08:05:00Z"
}
```

#### GET /api/admin/config
Get system configuration (admin only).

**Response:**
```json
{
  "alertThresholds": {},
  "notifications": {},
  "security": {},
  "updatedAt": "2026-07-01T00:00:00Z",
  "updatedBy": "admin"
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Invalid request parameters",
  "details": [
    { "field": "amount", "message": "Amount must be positive" }
  ]
}
```

### 401 Unauthorized
```json
{
  "error": "UNAUTHORIZED",
  "message": "Missing or invalid authentication token"
}
```

### 403 Forbidden
```json
{
  "error": "FORBIDDEN",
  "message": "User does not have permission for this action"
}
```

### 404 Not Found
```json
{
  "error": "NOT_FOUND",
  "message": "Resource not found"
}
```

### 409 Conflict
```json
{
  "error": "WORKFLOW_VIOLATION",
  "message": "Cannot transition from PF-01 to PF-04 directly",
  "details": {
    "currentStatus": "PF-01",
    "requestedStatus": "PF-04",
    "validNextStates": ["PF-02"]
  }
}
```

### 500 Internal Server Error
```json
{
  "error": "INTERNAL_ERROR",
  "message": "An unexpected error occurred",
  "requestId": "uuid"
}
```

---

## Rate Limiting

- Rate limit: 100 requests per 15 minutes per IP
- Header: `X-RateLimit-Remaining`

---

## Webhooks

Optional webhook support for real-time event notifications.

**Events:**
- `order.created`
- `order.status_changed`
- `order.completed`
- `notification.triggered`

Configure at `/api/admin/webhooks`
