---
sidebar_position: 2
---

# Authentication

## 1. Overview

Dokumen ini memberikan gambaran tingkat tinggi mengenai **Authentication Service API**. Modul ini bertanggung jawab penuh atas manajemen identitas pengguna, keamanan sesi, dan validasi akses dalam ekosistem aplikasi.

### Tujuan API

API ini dirancang untuk menangani seluruh siklus hidup keamanan pengguna (User Security Lifecycle). Tujuan utamanya adalah memastikan bahwa hanya pengguna yang terverifikasi yang dapat mengakses sumber daya sistem, sambil menjaga integritas data pengguna.

**Fungsi Utama:**

- Mengelola mekanisme _login_ yang aman menggunakan standar industri.
- Menangani manajemen sesi (pembaruan token dan logout).

## 2. Authentication

Bagian ini menjelaskan standar keamanan yang diterapkan untuk memverifikasi identitas pengguna, mengelola sesi, dan mengatur hak akses dalam aplikasi.

### Jenis Autentikasi

Sistem menggunakan **Hybrid JWT Authentication** yang mendukung dua metode autentikasi secara bersamaan untuk memberikan fleksibilitas maksimal kepada berbagai jenis klien.

- **Core Technology:** JSON Web Token (JWT).
- **Dual Authentication Methods:**

  1. **Cookie-based Authentication** (Recommended untuk Web Browser)

     - Token disimpan dan dikelola sepenuhnya melalui **HTTP Cookies** yang dikirimkan oleh server.
     - Cookie diset dengan atribut keamanan (`HttpOnly`, `Secure`, `SameSite`) untuk meminimalisir risiko serangan XSS (_Cross-Site Scripting_) dan CSRF.
     - Browser secara otomatis menyertakan cookie pada setiap request.

  2. **Bearer Token Authentication** (Untuk Mobile Apps, SPA, & API Clients)
     - Token dikirimkan melalui **Authorization Header** dengan format: `Authorization: Bearer <token>`.
     - Memberikan fleksibilitas penuh untuk aplikasi mobile, single-page applications, atau integrasi API pihak ketiga.
     - Client bertanggung jawab menyimpan token dengan aman (misalnya di Secure Storage untuk mobile).

### Manajemen Sesi & Hak Akses (RBAC)

Sistem dirancang untuk mendukung fleksibilitas akses pengguna dan pembatasan fitur berbasis peran.

- **Multi-Session Support:**
  - Satu akun user diizinkan memiliki **banyak sesi aktif** secara bersamaan.
- **Role-Based Access Control:**
  - **Role `USER`:** Memiliki otoritas penuh untuk mengelola seluruh _resource_ aplikasi (CRUD data operasional), **KECUALI** modul "Manajemen User" (menambah/menghapus admin atau user lain).
  - **Role `ADMIN`:** Memiliki otoritas penuh untuk mengelola seluruh _resource_ aplikasi.

### Mekanisme Token

Sistem menggunakan strategi _Dual-Token Architecture_ untuk menyeimbangkan keamanan dan kenyamanan pengguna.

| Tipe Token        | Variabel Cookie | Fungsi Utama                                                          | Karakteristik Masa Berlaku (Expiration)                                            |
| :---------------- | :-------------- | :-------------------------------------------------------------------- | :--------------------------------------------------------------------------------- |
| **Access Token**  | `accessToken`   | Kunci utama untuk mengakses _endpoint_ API yang dilindungi.           | **Singkat** (Short-lived). Dirancang untuk kedaluwarsa dengan cepat demi keamanan. |
| **Refresh Token** | `refreshToken`  | Kunci untuk meminta _Access Token_ baru ketika yang lama kedaluwarsa. | **Panjang** (Long-lived). Lebih lama dari Access Token (misal: 7 hari).            |

**Alur Pembaruan (Refresh Flow):**

1.  Saat `accessToken` kedaluwarsa, aplikasi (client) tidak perlu meminta user login ulang.
2.  Client akan memanggil endpoint refresh menggunakan cookie `refreshToken` yang masih valid.
3.  Server memvalidasi `refreshToken`. Jika valid, server mengirimkan cookie `accessToken` baru.

### Transmisi Data (Header & Cookies)

Sistem mendukung **dua cara pengiriman token** untuk memberikan fleksibilitas kepada berbagai jenis klien:

#### **Opsi 1: Cookie-based (Otomatis untuk Browser)**

Klien tidak perlu menambahkan header `Authorization` secara manual. Browser akan secara otomatis menyertakan cookie pada setiap _request_ ke domain API.

**Ketentuan Pengiriman Cookie oleh Server:**
Server akan mengirimkan `HTTP Only Cookie` pada respons HTTP saat:

1. **Login Berhasil:** Mengirimkan `accessToken` dan `refreshToken` sebagai cookie.
2. **Refresh Token Berhasil:** Mengirimkan `accessToken` baru (dan opsional `refreshToken` baru/rotasi) sebagai cookie.
3. **Logout:** Mengirimkan instruksi untuk menghapus/mengosongkan kedua cookie dan melakukan invalidasi `refresh token`.

#### **Opsi 2: Bearer Token (Manual untuk Mobile/API Clients)**

Klien harus menyertakan token pada setiap request yang memerlukan autentikasi dengan menambahkan header:

```http
Authorization: Bearer <accessToken>
```

**Ketentuan untuk Client:**

1. **Login Berhasil:** Client menerima `accessToken` dan `refreshToken` dalam response body dan harus menyimpannya dengan aman.
2. **Setiap Request:** Client wajib menyertakan header `Authorization` dengan `accessToken`.
3. **Refresh Token:** Client mengirimkan `refreshToken` di request body saat meminta token baru.
4. **Logout:** Client menghapus token dari storage dan mengirimkan request logout dengan `accessToken` di header.

**Prioritas Server:**
Jika request menyertakan **kedua metode** (cookie dan Bearer token), server akan **memprioritaskan Bearer Token** di header `Authorization`.

---

# 3. API Endpoints

### **POST /auth/signin**

**Description:** Login user dan menghasilkan Access & Refresh Token.

**Request Body:**

```json
{
  "username": "username",
  "password": "password"
}
```

**Response (Cookie-based):**

- **Set-Cookie Headers:**

  - `accessToken` (HttpOnly, Secure, SameSite)
  - `refreshToken` (HttpOnly, Secure, SameSite)

- **Response Body:**

```json
{
  "statusCode": 201,
  "message": "User loggedin successfully",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjEsInVzZXJuYWddd1lIjoiYWRtaW4iLCJyb2xlIjoiQURNSU4iLCJpYXQiOjE3NjQ1ODI4MDYsImV4cCI6MTc2NDU4MzcwNn0.Ko3Mz9pFWIlawgLpa47mHWdtZfGpVO_q6HvwW65C25k",
    "refreshToken": "eyJhbGciOiJIUzI1ddNiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjEsInVzZXJuYW1lIjoiYWRtaW4iLCJpYXQiOjE3NjQ1ODI4MDYsImV4cCI6MTc2NTdE4NzYwNn0.q7eC7ezxImN3Z4zdM2VwIt9bzGy7LefGPGVPDMoewkAE"
  }
}
```

---

### **POST /auth/refresh**

**Description:** Menghasilkan Access Token baru menggunakan Refresh Token.

**Request (Cookie-based):**

Tidak perlu request body. Server akan membaca `refreshToken` dari cookie secara otomatis.

**Request (Bearer Token mode):**

Header:

```http
Authorization: Bearer <refreshToken>
```

**Response (Cookie-based):**

```json
{
  "statusCode": 200,
  "message": "Tokens refreshed successfully",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjEsInVzZXJuYW1lIjoiYWRtaW4iLCJyb2xlIjoiQURNSU4iLCJpYXQgiOjE3NjQ1ODI5MzjgsImV4cCI6MTc2NDU4MzgzOH0.qObHvt9qkTNrymYtG92hkOQNZFL_gARKgrSZPmwHYcM",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsIdnR5cCI6IkpXVCJ9.eyJzdWIiOjEsInVzZXJuYW1lIjoiYWRtaW4iLCJpYXQiOjE3NjQ1ODI5fMzgsImV4cCI6MTc2NTE4NzczOH0.sFHWVhF3p7kNZMfzUuTXfSu4gTF2vJcAWRlMFyzpoQs"
  }
}
```

---

### **POST /auth/signout**

**Description:** Menghapus sesi user dan invalidasi refresh token.

**Request (Cookie-based):**

Tidak perlu request header. Server akan membaca token dari cookie dan menghapusnya.

**Request (Bearer Token mode):**

Header:

```http
Authorization: Bearer <refreshToken>
```

**Response:**

```json
{
  "statusCode": 200,
  "message": "Logged out successfully",
  "data": null
}
```

**Note:**

- Untuk cookie-based, server akan mengirimkan instruksi `Set-Cookie` untuk menghapus cookie.
- Untuk bearer token, client bertanggung jawab menghapus token dari storage setelah menerima response sukses.

### **GET /auth/session**

**Description:** Mendapatkan informasi sesi user yang sedang aktif (cek status login).

**Request (Cookie-based):**

Tidak perlu request header. Server akan membaca token dari cookie.

**Request (Bearer Token mode):**

Header:

```http
Authorization: Bearer <accessToken>
```

**Response:**

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "userId": 1,
    "name": "Administrator",
    "username": "admin",
    "role": "ADMIN"
  }
}
```

**Note:**

- Untuk cookie-based, server akan membaca token dari cookie.
- Untuk bearer token, client mengirimkan access token di header Authorization.
