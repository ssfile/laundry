# Panduan Menambahkan & Mengelola User Berdasarkan Role via Firebase

Dokumen ini menjelaskan cara menambahkan, mengedit, dan mengatur hak akses pengguna (**User Role**) pada **SSmart Laundry** yang terhubung dengan proyek Firebase `ssmart-laundry`.

---

## 1. Daftar Role & Hak Akses di SSmart Laundry

| Role | Siapa yang Membuat / Mengatur | Hak Akses Utama |
| :--- | :--- | :--- |
| **`SUPER_ADMIN`** | Kredensial awal platform (`admin` / `admin`), dapat diubah di menu **Pengaturan → Akun Super Admin** | Melihat daftar Owner & Tenant, mendaftarkan/mengedit Owner, mengatur masa aktif & paket harga, serta mengatur warna, logo web, favicon, dan footer global. |
| **`OWNER`** | Dibuat & diedit oleh **`SUPER_ADMIN`** (melalui web atau Firestore) | Akses penuh operasional & keuangan cabang miliknya, menambah/mengelola karyawan, mengunggah logo nota, serta membuka menu **Help (WA Support)** dan perpanjangan paket. |
| **`ADMIN`** | Dibuat oleh **`SUPER_ADMIN`** atau **`OWNER`** | Mengelola kasir, order, pelanggan, pembayaran/piutang, pengeluaran, laporan keuangan, layanan/tarif, dan pengaturan profil/nota cabang (tanpa akses manajemen karyawan). |
| **`KASIR`** | Dibuat oleh **`OWNER`** di menu **Karyawan** | Melayani transaksi kasir (`Quick Order`), melihat order, scan barcode/QR invoice, mengelola pelanggan, dan mencatat pelunasan pembayaran. |
| **`OPERATOR`** | Dibuat oleh **`OWNER`** di menu **Karyawan** | Memproses antrean order laundry, melakukan scan nota, dan memperbarui status pengerjaan cucian. |

---

## 2. Cara 1: Menambahkan User Langsung dari Aplikasi (Tersimpan Otomatis ke Cloud Firestore)

Setiap perubahan yang Anda lakukan di aplikasi akan otomatis tersinkronisasi ke Cloud Firestore (`artifacts/ssmart-laundry/public/data/...`):

### A. Menambahkan / Mengedit Akun `SUPER_ADMIN`
1. Login sebagai **Super Admin**.
2. Buka menu **Pengaturan** → tab **Akun Super Admin**.
3. Ubah **Email / Username Login** dan **Password Baru**, lalu klik **Simpan Akun Super Admin**.
4. Data tersimpan otomatis ke dokumen `platform_config/main` di Firestore.

### B. Menambahkan / Mengedit Akun `OWNER` (Beserta Bisnis / Tenant)
1. Login sebagai **Super Admin**.
2. Buka menu **Dashboard Owner** atau **Super Admin SaaS** → klik **+ Daftarkan Owner & Tenant**.
3. Isi:
   - **Nama Owner** & **No. WhatsApp**
   - **Email Login Owner** & **Password Login**
   - **Role Akun** (`OWNER` atau `ADMIN`)
   - **Tanggal Berakhir Masa Aktif** (atau klik tombol `+30 Hr` / `+1 Thn`)
   - **Nama Usaha Laundry**, **Paket Langganan**, dan **Alamat Outlet**
4. Klik **Simpan Data Owner**. Sistem otomatis membuat dokumen di koleksi `owners` dan `businesses` di Firestore.

### C. Menambahkan Akun Karyawan (`KASIR`, `OPERATOR`, `ADMIN`)
1. Login sebagai **Owner** dari outlet terkait.
2. Buka menu **Karyawan** → klik **+ Karyawan**.
3. Isi **Nama Lengkap**, **Email Login**, **Password**, **Nomor HP/WA**, dan pilih **Role / Jabatan** (`KASIR`, `OPERATOR`, atau `ADMIN`).
4. Klik **Simpan**. Data tersimpan ke koleksi `employees` dengan `businessId` sesuai outlet Owner tersebut.

---

## 3. Cara 2: Menambahkan User Manual di Firebase Console (Firestore Database)

Jika Anda ingin menambahkan atau mengedit data user langsung dari **Firebase Console**:

1. Buka [Firebase Console](https://console.firebase.google.com/) → pilih proyek **`ssmart-laundry`**.
2. Masuk ke menu **Build → Firestore Database**.
3. Pastikan **Anonymous Authentication** aktif pada **Build → Authentication → Sign-in method** agar sesi browser dapat terhubung dengan aturan keamanan (`firestore.rules`).
4. Navigasikan ke path koleksi:
   `artifacts / ssmart-laundry / public / data / {nama_koleksi}`

### Contoh Struktur Dokumen per Role

#### 1) Mengedit Kredensial `SUPER_ADMIN`
- **Path:** `artifacts/ssmart-laundry/public/data/platform_config/main`
- **Field di dalam `superAdminAuth`:**
  ```json
  {
    "id": "usr_super_admin",
    "name": "Super Admin",
    "email": "superadmin@ssmartlaundry.id",
    "password": "PasswordKuat123!",
    "role": "SUPER_ADMIN"
  }
  ```

#### 2) Menambahkan Dokumen `OWNER` Baru
- **Path:** `artifacts/ssmart-laundry/public/data/owners/{id_owner}` *(contoh: `own_03`)*
- **Field Dokumen:**
  ```json
  {
    "id": "own_03",
    "businessId": "biz_ssmart_01",
    "name": "Dewi Lestari",
    "email": "dewi@ssmartlaundry.id",
    "password": "password123",
    "phone": "6281200001111",
    "role": "OWNER",
    "status": "ACTIVE",
    "expiresAt": "2026-12-26"
  }
  ```
  *(Pastikan `id` dokumen sama persis dengan nilai field `id` agar lolos validasi `firestore.rules`).*

#### 3) Menambahkan Dokumen Karyawan (`KASIR` / `OPERATOR` / `ADMIN`)
- **Path:** `artifacts/ssmart-laundry/public/data/employees/{id_karyawan}` *(contoh: `emp_04`)*
- **Field Dokumen:**
  ```json
  {
    "id": "emp_04",
    "businessId": "biz_ssmart_01",
    "name": "Rudi Kasir",
    "email": "rudi.kasir@ssmartlaundry.id",
    "password": "password123",
    "phone": "081277778888",
    "role": "KASIR",
    "active": true
  }
  ```

---

## 4. Checklist Keamanan & Operasional
- Selalu gunakan **email unik** untuk tiap akun agar tidak bentrok saat proses login.
- Pastikan field `id` pada dokumen baru sama dengan nama **Document ID** di Firestore sesuai aturan `request.resource.data.id == docId` pada `firestore.rules`.
- Setelah mengubah kredensial atau menambah user di Firestore Console, halaman aplikasi yang sedang terbuka akan memperbarui data secara *real-time* melalui `onSnapshot`.