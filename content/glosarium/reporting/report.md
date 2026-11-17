---
title: Report
draft: false
tags:
  - reporting
  - report
  - thread
  - issue
  - tiket
---

# Glosarium Domain: Report / Pelaporan Kendala (Reporting)

Entitas agregat dalam modul **Reporting** yang mengelola tiket pelaporan isu operasional (*issue tracking*), pelacakan kendala pengguna, utas diskusi (*threads*), dan lampiran dokumen pendukung.

---

## 1. Definisi & Peran Bisnis

- **Report (Tiket Laporan/Kendala)**: Tiket pengaduan masalah teknis atau bisnis yang diajukan oleh pengguna saat menghadapi hambatan operasional (misal kendala pencairan SP2D, kesalahan entri jurnal, atau gangguan unggah dokumen pendukung).
- **Status Tiket (`Status`)**:
  - `OPEN`: Tiket baru dilaporkan dan menunggu respon penugasan.
  - `IN_PROGRESS`: Tiket sedang dalam investigasi atau perbaikan oleh tim pengembang/administrator.
  - `RESOLVED`: Masalah telah diselesaikan dan solusi telah diberikan.
  - `CLOSED`: Tiket telah dikonfirmasi tuntas oleh pelapor.
- **Report Thread (Utas Diskusi)**: Diskusi interaktif berjenjang (*nested comments*) antara pengguna pelapor dan tim teknis untuk tanya jawab kronologi permasalahan.

---

## 2. Invarian & Aturan Bisnis

1. **Struktur Konten TipTap**: Deskripsi laporan disimpan dalam format JSON terstruktur editor TipTap (`Map<String, Object> description`).
2. **Keterikatan Utas**: Setiap komentar dalam `ReportThread` harus terhubung ke satu tiket `Report` induk (`report_id`).

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Report`
- **Tabel Basis Data**: `reports`
- **Kelas Model**: `org.ilmi.simkeucore.reporting.domain.Report`
- **Kelas Induk**: `Auditable<Report>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas laporan (UUID v4). |
| `title` | `title` | `String(100)` | **Wajib** | - | Judul ringkas laporan kendala/isu. |
| `description` | `description` | `JSON (Map)` | Opsional | TipTap JSON | Format teks kaya (*rich-text*) deskripsi kronologi masalah. |
| `priority` | `priority` | `Enum (String)` | Opsional | `Priority` | Tingkat prioritas penanganan (`LOW`, `MEDIUM`, `HIGH`, `CRITICAL`). |
| `status` | `status` | `Enum (String)` | **Wajib** | `Status` | Status tiket penanganan (`OPEN`, `IN_PROGRESS`, `RESOLVED`, `CLOSED`). |
| `type` | `type` | `Enum (String)` | Opsional | `Type` | Klasifikasi isu (`BUG`, `FEATURE_REQUEST`, `SUPPORT`). |
| `assignee` | `assignee` | `String` | Opsional | `User.id` / `User.username` | NIP / username teknisi yang ditugaskan menangani tiket. |
| `attachments` | Tabel: `report_attachments` | `List<ReportAttachment>`| Opsional | `@ElementCollection` | Daftar berkas bukti screenshot atau file error pendukung di Storage. |

---

## 4. Child Entity: Utas Komentar & Diskusi

### Child Entity: `ReportThread`
- **Tabel Basis Data**: `threads`
- **Kelas Model**: `org.ilmi.simkeucore.reporting.domain.ReportThread`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik utas komentar. |
| `content` | `content` | `Text` | **Wajib** | - | Isi pesan komentar atau tanggapan solusi masalah. |
| `report` | `report_id` | `Report` | **Wajib** | `@ManyToOne(Report)` | Referensi ke tiket laporan induk. |
| `parent` | `parent_id` | `ReportThread` | Opsional | `@ManyToOne(ReportThread)` | Referensi ke komentar induk (jika balasan bertingkat). |
| `children` | - | `List<ReportThread>` | Opsional | `@OneToMany` | Daftar balasan langsung di bawah komentar ini. |
| `createdBy` | `created_by` | `String` | Opsional | `User.username` | Pengirim komentar. |
| `createdDate` | `created_date` | `Instant` | Opsional | - | Waktu komentar dikirim. |

---

## 5. Enumerasi Domain Terkait

### `Priority`
- `LOW`: Dampak minor, tidak menghambat operasional rutin.
- `MEDIUM`: Kendala operasional berkala dengan solusi alternatif.
- `HIGH`: Menghambat proses transaksi penting.
- `CRITICAL`: Sistem terhenti (*system-down*) atau pencairan kas terhambat total.

### `Status`
- `OPEN`: Baru diajukan.
- `IN_PROGRESS`: Dalam proses pengerjaan.
- `RESOLVED`: Solusi telah diterapkan.
- `CLOSED`: Ditutup secara permanen.
