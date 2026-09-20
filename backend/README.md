# Arsitektur Backend & Layanan API (SALDO)

Dokumen ini menyajikan ringkasan arsitektur teknis, desain sistem, dan alur pemrosesan data backend yang menopang aplikasi **SALDO**.

---

## Ringkasan Arsitektur

Backend SALDO dibangun menggunakan pendekatan RESTful API berbasis micro-services modular yang di-deploy pada lingkungan serverless. Sistem menangani pemrosesan bahasa alami (Natural Language Processing) untuk transaksi keuangan, manajemen buku besar (ledger) multi-akun, kalkulasi analitik arus kas, serta autentikasi pengguna yang aman.

```text
                  +----------------------------------+
                  |         Frontend Client          |
                  |     (React 19, Tailwind CSS)     |
                  +-----------------+----------------+
                                    |
                                    v
                     [HTTPS / JWT Bearer Token]
                                    |
                                    v
                  +----------------------------------+
                  |       API Gateway & Security     |
                  |     (CORS, Rate Limiter, Auth)   |
                  +-----------------+----------------+
                                    |
         +--------------------------+--------------------------+
         |                          |                          |
         v                          v                          v
+------------------+      +-------------------+      +------------------+
|  Natural Language|      |   Ledger & Sync   |      |   Cashflow &     |
|   Parser Engine  |      |   Manager Engine  |      | Analytics Engine |
+--------+---------+      +---------+---------+      +--------+---------+
         |                          |                         |
         v                          v                         v
+------------------+      +-------------------+               |
| Intent Guard &   |      |  Atomic Mutation  |               |
| Prompt Defense   |      |  Balance Updater  |               |
+--------+---------+      +---------+---------+               |
         |                          |                         |
         +--------------------------+-------------------------+
                                    |
                                    v
                  +----------------------------------+
                  |         Database Cluster         |
                  |       (MongoDB & Mongoose)       |
                  +----------------------------------+
```

---

## Tech Stack Backend

- **Runtime & Framework**: Node.js, Express 5
- **Database & ODM**: MongoDB Atlas, Mongoose 9
- **Natural Language Parsing**: AI Model Integration (OpenRouter API - GPT-4o-mini) dengan *structured JSON output schema*
- **Keamanan & Guardrails**: Heuristic Intent Guard, Regex Defense System, Rate Limiting
- **Autentikasi**: Stateless JSON Web Tokens (JWT), Bcryptjs Password Hashing, verifikasi OTP via Email (Nodemailer SMTP)
- **Deployment Platform**: Vercel Serverless Functions

---

## Komponen Sistem Utama

### 1. Natural Language Parser & Intent Guard
Pipeline khusus yang memproses kalimat santai pengguna (baik melalui input teks langsung maupun transkripsi suara).
- **Penyaringan Intent**: Menolak obrolan umum, pertanyaan non-finansial, dan upaya manipulasi prompt (*prompt injection*).
- **Normalisasi Slang Finansial**: Menerjemahkan istilah angka lokal Indonesia seperti *25k*, *50rb*, *1.5jt*, serta tanggal relatif seperti *kemarin*, *tadi siang*, *3 hari lalu*.
- **Pemetaan Entitas Terstruktur**: Menghasilkan objek transaksi tervalidasi yang memuat tipe (*expense*, *income*, *transfer*), nominal, kategori, akun sumber, dan akun tujuan.

### 2. Multi-Account Ledger & Balance Reconciliation
Sistem pembukuan yang memastikan integritas saldo pengguna di berbagai akun:
- Mutasi saldo berjalan secara atomik saat transaksi dicatat, diperbarui, atau dihapus.
- Dukungan transfer antar-rekening pribadi tanpa mendistorsi laporan arus kas bersih.
- Sinkronisasi real-time antara saldo total dan rincian per dompet (Tunai, Bank, E-Wallet).

### 3. Financial Analytics & Payday Forecasting Engine
Modul kalkulasi matematis di sisi server untuk menghasilkan *insight* finansial:
- **Spending Velocity**: Menghitung rata-rata laju pengeluaran harian dan laju belanja terkini.
- **Payday Runway**: Memproyeksikan estimasi sisa uang saat mencapai tanggal gajian berikutnya berdasarkan kebiasaan transaksi riil.
- **Health Ratio**: Menilai rasio tabungan, kebutuhan operasional, dan pengeluaran diskresioner.

---

## Proprietary Notice (Pemberitahuan Kode Sumber Privat)

> [!NOTE]
> Seluruh kode implementasi logika bisnis backend, prompt system AI khusus, aturan heuristik intent guard, serta kredensial lingkungan produksi pada direktori ini dilindungi hak cipta dan proprietary intellectual property pengembang.
>
> Folder ini disediakan pada repositori showcase publik untuk memberikan gambaran arsitektural komprehensif mengenai cara kerja sistem di balik layar tanpa mengorbankan keamanan data produksi dan rahasia dagang.
