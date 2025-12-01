---
sidebar_position: 1
---

# Global Response Format

Dokumen ini mendefinisikan **standar format response global** yang digunakan secara konsisten di seluruh API. Semua endpoint wajib mengikuti struktur response berikut untuk memastikan konsistensi, kemudahan parsing, dan pengalaman integrasi yang baik bagi client.

## Base URL

```txt
https://api.scituinsk.com
```

---

## Error Handling Standard

Format error respons yang digunakan secara konsisten.

**Response Structure:**

```json
{
  "statusCode": 404,
  "message": "Not Found",
  "error": "Not Found"
}
```

---

## Global Response Format

### **Success Response**

```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {} | []
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
      "perpage": 10,
      "total": 5,
      "totalPages": 42
    }
  }
}
```

---
