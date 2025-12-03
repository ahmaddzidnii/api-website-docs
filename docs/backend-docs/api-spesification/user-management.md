---
sidebar_position: 3
---

# User Management

## 1. Overview

Modul Manajemen User menyediakan API untuk mengelola user dalam sistem, termasuk pembuatan, pembaruan, penghapusan, dan pengambilan data user. Modul ini mendukung fitur autentikasi dan otorisasi berbasis peran (RBAC) untuk memastikan keamanan dan kontrol akses yang tepat.

**Contoh isi:**

- Endpoint untuk manajemen user
- Fitur CRUD untuk user
- Format request dan response menggunakan standar global response dan format JSON.

---

## 2. Authentication

Modul ini memerlukan autentikasi menggunakan token JWT yang diperoleh dari modul Authentication. Setiap permintaan ke endpoint manajemen user harus menyertakan token ini di header Authorization.

```http
Authorization: Bearer <accessToken>
```

:::tip

jika anda menggunakan metode authentikasi dengan`cookie` token akan dikirim browser secara otomatis.

:::

---

## 3. API Endpoints

:::warning
**Catatan Penting:**

- Hanya user dengan peran `SUPER_ADMIN` yang dapat mengakses endpoint ini.
- Pastikan untuk menyertakan token autentikasi yang valid pada setiap permintaan.
- Semua respons mengikuti format respons global yang telah ditentukan.
  :::

:::info
Untuk mengenai **role** silahkan lihat dokumentasi [RBAC](./authentication#manajemen-sesi--hak-akses-rbac).
:::

---

### **POST /users**

**Description:** Membuat user baru.

**Authentication:** Required (roles: `ADMIN`, `SUPER_ADMIN`)

**Request Body:**

```json
{
  "name": "string",
  "username": "string",
  "email": "user@example.com",
  "password": "string",
  "role": "USER" // optional: USER | ADMIN | SUPER_ADMIN (subject to RBAC)
}
```

:::info

- `username` harus unik di seluruh sistem.
- `email` harus valid dan unik.
- `password` harus memenuhi kebijakan keamanan (misalnya minimal 8 karakter, kombinasi huruf besar, huruf kecil, angka, dan simbol).
  :::

**Response (201 Created):**

```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "email": "user@example.com",
    "role": "USER",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

---

### **GET /users**

**Description:** Mengambil daftar user (paginated).

**Authentication:** Required (roles: `ADMIN`, `SUPER_ADMIN`)

**Query Params:**

- `page` (optional)
- `pageSize` (optional)
- `q` (optional) — search by name/email
- `role` (optional) — filter by role

**Response (200):** _(paginated)_

```json
{
  "status": "success",
  "data": [
    {
      "id": "string",
      "name": "string",
      "email": "string",
      "role": "string",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 10,
    "totalPages": 3,
    "totalItems": 25
  }
}
```

---

### PATCH `/users/{id}` (untuk partial update)

**Description:** Memperbarui data user.

**Authentication:** Required (roles: `ADMIN`, `SUPER_ADMIN`, or the user themself for profile updates)

**Request Body (contoh):**

```json
{
  "name": "string",
  "email": "user@example.com",
  "password": "string", // optional untuk ganti password
  "role": "ADMIN" // only allowed for ADMIN/SUPER_ADMIN depending on RBAC
}
```

**Response (200):**

```json
{
  "status": "success",
  "data": {
    "id": "string",
    "name": "string",
    "email": "user@example.com",
    "role": "ADMIN",
    "updatedAt": "2024-01-02T00:00:00Z"
  }
}
```

---

### DELETE `/users/{id}`

**Description:** Menghapus user.

**Authentication:** Required (roles: `SUPER_ADMIN`).

**Response (200):**

```json
{
  "status": "success",
  "data": null
}
```
