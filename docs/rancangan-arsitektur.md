---
sidebar_position: 2
---

# Rancangan Arsitektur

## **1. Overview**

Website SCIT dibangun menggunakan pendekatan **separated frontend & backend**, sehingga setiap bagian dapat dikembangkan, diperbaiki, atau di-scale secara mandiri. Pembagian ini juga memudahkan generasi developer berikutnya untuk melakukan perawatan sistem.

Arsitektur utama terdiri dari tiga komponen besar:

1. **Frontend** \
   Aplikasi web yang berinteraksi langsung dengan pengguna
2. **Backend API** \
   Server aplikasi yang menangani logika bisnis dan akses database
3. **Database** \
   Tempat penyimpanan data terstruktur seperti project, tim, dan gallery

Berikut adalah gambar diagram sederhana tentang arsitektur yang kami pakai:

```
[ Frontend ]  <-->  [ Backend API ]  <-->  [ Database ]
```

---

## **2. Frontend**

### **2.1 Teknologi**

- **Framework**: Next.js (React)
- **Styling**: Tailwind CSS
- **State Management**: Zustand / Context (tergantung kebutuhan)
- **Fetching API**: Axios / Fetch API
- **Image Optimization**: Built-in Next Image

### **2.2 Alasan Pemilihan**

- **Next.js** mendukung SSR & SSG sehingga performa tinggi dan SEO-friendly
- Mudah di-maintain oleh mahasiswa karena React sudah umum dipelajari
- Komponen dapat dipakai ulang, memudahkan scaling fitur
- Struktur file lebih terarah (app directory) sehingga developer baru cepat adaptasi

### **2.3 Peran Frontend**

Frontend bertanggung jawab untuk:

- Menampilkan data project, tim, dan gallery
- Menyediakan halaman publik (home, project list, detail, gallery, dll)
- Berkomunikasi dengan backend via REST API
- Mengelola halaman admin (CMS) untuk CRUD data
- Menangani login session admin

---

## **3. Backend API**

### **3.1 Teknologi**

- **Node.js + NestJS**
- **ORM**: Prisma
- **Database**: PostgreSQL
- **Auth**: JWT atau Session Token (tergantung implementasi)
- **Storage**: S3-compatible bucket (IDCloudHost, MinIO, dsb)

### **3.2 Alasan Pemilihan**

- **NestJS** memiliki struktur yang sangat rapi, modular, dan scalable
- Cocok untuk tim komunitas karena gampang diwariskan
- Prisma mempermudah query, migration, dan maintenance
- PostgreSQL stabil dan standar industri
- Arsitektur backend dibuat agar developer baru bisa langsung baca alur tanpa tersesat

### **3.3 Peran Backend**

Backend mengatur logika sistem:

- CRUD Project
- CRUD Team & member
- CRUD Gallery
- Autentikasi admin
- Validasi data
- Menyimpan metadata file gambar ke database
- Serve API dengan standar REST yang konsisten

---

## **4. Database**

### **4.1 Teknologi**

- **PostgreSQL** (hosted di server atau layanan managed)

### **4.2 Struktur Data Utama**

Tabel inti dalam sistem:

- `projects`
- `project_images`
- `teams`
- `team_members`
- `gallery`

### **4.3 Alasan Pemilihan**

- Relational database cocok untuk data terstruktur & relasi yang jelas
- PostgreSQL mature, cepat, dan komunitas besar
- Mendukung JSON bila dibutuhkan fleksibilitas tambahan

---

## **5. Cloud Storage**

Untuk penyimpanan gambar:

- Menggunakan layanan **Object Storage** seperti IDCloudHost S3
- Backend hanya menyimpan URL + metadata ke database
- File diunggah via API backend atau direct upload (pilihan opsional)

Keuntungan:

- Skalabilitas tinggi
- Tidak membebani storage server aplikasi
- Bisa dipindah ke layanan lain tanpa mengubah logika aplikasi

---

## **6. Alur Data (Flow)**

### **Contoh alur upload Project Image**

1. Admin mengunggah gambar dari halaman admin
2. Frontend mengirim file → Backend
3. Backend upload file ke S3 Storage
4. Backend menyimpan metadata gambar (url, project_id, caption, dsb) ke database
5. Frontend menampilkan gambar berdasarkan URL yang tersimpan

Flow konsisten untuk gallery dan team images.

---

## **7. Prinsip Desain untuk Pengembang Generasi Selanjutnya**

Untuk mempermudah regenerasi, arsitektur ini mengikuti prinsip:

### **7.1 Kode Mudah Dibaca**

- NestJS dipakai karena struktur module-service-controller sangat jelas
- Frontend menggunakan component-based React/Next agar dapat dibagi-bagi dengan mudah

### **7.2 Dokumentasi Terpusat**

- Semua endpoint API harus terdokumentasi
- Migration Prisma harus dijalankan oleh semua developer baru
- README untuk setup environment harus jelas

### **7.3 Tidak Bergantung pada Layanan Spesifik**

- Storage bisa diganti kapan saja (S3-compatible)
- Database bisa dipindah selama masih PostgreSQL
- Frontend dapat di-deploy ke platform apa pun (Vercel, Docker, dsb)

### **7.4 Mudah Diwariskan**

- Struktur folder rapi dan konsisten
- Nama variabel & file jelas
- Setup dev environment cukup `npm install` lalu `npm run dev`
- Config environment menggunakan `.env.example`

---

## **8. Deployment**

### **Frontend Deployment**

- Bisa di-deploy ke Vercel, Nginx, atau Docker
- Build dengan `next build`

### **Backend Deployment**

- Menggunakan Docker untuk memudahkan maintain
- Atau direct Node.js PM2
- Memperhatikan environment:

  - `DATABASE_URL`
  - `STORAGE_BUCKET`
  - `JWT_SECRET`

### **Database**

- PostgreSQL hosted (IDCloudHost DBaaS atau self-managed)
- Backup berkala mingguan & bulanan

### **Storage**

- Pastikan konfigurasi CORS benar untuk upload

---

## **9. Kesimpulan**

Arsitektur Website SCIT didesain agar:

- **Mudah dipahami oleh anggota baru**
- **Sederhana namun scalable**
- **Menggunakan teknologi modern yang familiar bagi mahasiswa**
- **Dapat diwariskan tanpa perlu pengetahuan mendalam tentang keseluruhan sistem**

Dengan dokumentasi arsitektur ini, diharapkan generasi pengembang berikutnya memiliki fondasi kuat untuk melanjutkan, memperbaiki, atau mengembangkan fitur baru pada website SCIT.
