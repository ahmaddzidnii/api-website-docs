---
sidebar_position: 3
---

# User Management

## 1. Overview

Berisi ringkasan singkat mengenai tujuan API, sistem yang terlibat, serta gambaran besar fungsionalitas.

**Contoh isi:**

- API ini digunakan untuk …
- Sistem yang berkomunikasi: …
- Format respons standar: JSON

---

## 2. Authentication

Jelaskan metode autentikasi yang digunakan.

**Termasuk:**

- Jenis auth (JWT / OAuth2 / API Key / Cookie-based)
- Mekanisme token (expiration, refresh)
- Header atau cookie yang harus dikirim

**Contoh:**

- Access Token dikirim via cookie `accessToken`
- Refresh Token dikirim via cookie `refreshToken`
- Expiration: Access Token 15 menit, Refresh Token 7 hari

---

## 3. Base URL

```txt
https://api.example.com/v1
```

---

## 4. Error Handling Standard

Format error respons yang digunakan secara konsisten.

**Response Structure:**

```json
{
  "status": "error",
  "message": "Deskripsi error",
  "code": "ERROR_CODE"
}
```

**Contoh Error Code:**

- `AUTH_INVALID_TOKEN`
- `VALIDATION_ERROR`
- `RESOURCE_NOT_FOUND`
- `SERVER_ERROR`

---

## 5. Global Response Format

### **Success Response**

```json
{
  "status": "success",
  "data": {}
}
```

### **Paginated Response**

```json
{
  "status": "success",
  "data": {
    "items": [],
    "pagination": {
      "page": 1,
      "pageSize": 10,
      "totalPages": 5,
      "totalItems": 42
    }
  }
}
```

---

# 6. API Endpoints

## 6.1. Authentication Module

### **POST /auth/login**

**Description:** Login user dan menghasilkan Access & Refresh Token.

**Request Body:**

```json
{
  "email": "string",
  "password": "string"
}
```

**Response:**

- Cookie:

  - `accessToken`
  - `refreshToken`

- Body:

```json
{
  "status": "success",
  "data": {
    "userId": "string",
    "email": "string"
  }
}
```

---

### **POST /auth/refresh**

**Description:** Menghasilkan Access Token baru menggunakan Refresh Token.

**Response:**

- Cookie baru: `accessToken`

```json
{
  "status": "success",
  "data": {
    "accessTokenExpiresIn": 900
  }
}
```

---

### **POST /auth/logout**

**Description:** Menghapus sesi user.

**Response:**

```json
{
  "status": "success",
  "message": "Logged out"
}
```

---

## 6.2. User Module

### **GET /users**

**Description:** List semua user (admin only).

**Query Params:**

- `page` (optional)
- `pageSize` (optional)

**Response:** _(paginated)_

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "string",
        "name": "string",
        "email": "string",
        "role": "string"
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 10,
      "totalPages": 3,
      "totalItems": 25
    }
  }
}
```

---

## 6.3. Resource Module (Contoh Generic)

### **POST /resources**

**Description:** Membuat resource baru.

**Request Body Example:**

```json
{
  "name": "string",
  "value": "string"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "string",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

---

# 7. Data Models

## **User**

| Field | Type   | Description       |
| ----- | ------ | ----------------- |
| id    | string | Unique ID         |
| name  | string | Full name         |
| email | string | User email        |
| role  | string | Role-based access |

---

# 8. Role & Permission Matrix

| Role  | Permission                                         |
| ----- | -------------------------------------------------- |
| Admin | Manage users, manage all resources                 |
| User  | Manage all resource modules except user management |

---

# 9. Rate Limiting

- **100 requests/min per IP**
- Error response jika limit terlampaui:

```json
{
  "status": "error",
  "message": "Rate limit exceeded",
  "code": "RATE_LIMIT"
}
```

---

# 10. Versioning

- Semua endpoint berada di bawah prefix `/v1`
- Perubahan breaking akan dirilis pada `/v2`

---

# Jika mau, saya bisa:

✅ Buatkan versi yang lebih detail
✅ Buatkan versi khusus Postman Collection
✅ Buatkan versi OpenAPI 3.0 (YAML/JSON)
Cukup bilang _“buatkan versi openapi”_ atau _“buatkan dokumentasi lebih lengkap”_
