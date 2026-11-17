---
title: Notification
draft: false
tags:
  - notification
  - pemberitahuan
  - alert
  - event
---

# Glosarium Domain: Notification (Pemberitahuan)

Entitas dalam modul **Notification** yang mengelola pesan pemberitahuan, peringatan operasional, dan riwayat notifikasi in-app kepada pengguna sistem atas peristiwa penting keuangan (*domain events*).

---

## 1. Definisi & Peran Bisnis

- **Notification (Notifikasi)**: Pesan asinkron yang dikirimkan oleh sistem kepada pengguna spesifik (`userId`) saat terjadi perubahan status penting pada transaksi (misal: penolakan verifikasi SPP, penerbitan SP2D, pencairan kas, atau selisih rekonsiliasi kas).
- **Tipe Peringatan**:
  - `SUCCESSFUL`: Berita keberhasilan proses bisnis (misal SP2D berhasil dicairkan).
  - `WARNING`: Peringatan non-fatal yang membutuhkan atensi (misal sisa pagu anggaran menipis atau tenggat waktu termin mendekati batas).
  - `DESTRUCTIVE`: Peringatan kegagalan/penolakan fatal (misal SPP ditolak oleh verifikator atau penandatanganan e-signature gagal).

---

## 2. Invarian & Aturan Bisnis

1. **Target Pengguna Spesifik**: Setiap notifikasi harus terikat pada pengguna penerima (`userId`) yang valid.
2. **Kekekalan Rekam Jejak**: Setelah dicatat, tanggal pembuatan (`createdDate`) bersifat permanen (`updatable = false`) sebagai audit log pemberitahuan.

---

## 3. Kamus Data & Atribut Entitas

### Entitas: `Notification`
- **Tabel Basis Data**: `notification`
- **Kelas Model**: `org.ilmi.simkeucore.notification.core.entity.Notification`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik pemberitahuan (UUID v4). |
| `userId` | `user_id` | `String` | **Wajib** | `User.id` / `User.username` | Identitas pengguna penerima notifikasi. |
| `title` | `title` | `String` | **Wajib** | - | Judul ringkas pesan notifikasi. |
| `message` | `message` | `String(4000)` | **Wajib** | - | Isi lengkap pesan pemberitahuan atau rincian instruksi tindakan. |
| `notificationType` | `notification_type` | `Enum (String)` | **Wajib** | `NotificationType` | Klasifikasi tingkat urgensi pesan (`SUCCESSFUL`, `WARNING`, `DESTRUCTIVE`). |
| `createdDate` | `created_date` | `Instant` | **Wajib** | - | Waktu kapan notifikasi diterbitkan oleh sistem. |

---

## 4. Enumerasi Domain Terkait

### `NotificationType`
- `SUCCESSFUL`: Informasi keberhasilan operasional sistem.
- `WARNING`: Peringatan atau pengingat batas waktu/ambang batas pagu.
- `DESTRUCTIVE`: Pemberitahuan penolakan transaksi, pembatalan dokumen, atau error fatal.
