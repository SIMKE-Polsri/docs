---
title: Periode
draft: false
tags:
  - periode
  - tahun-anggaran
  - fiscal-year
  - temporal
---

# Glosarium Domain: Periode (Tahun Fiskal Anggaran)

Entitas fondasi temporal dalam modul **Periode** yang menetapkan batas siklus hidup tahun fiskal/anggaran (*fiscal year boundary*) dan mengendalikan izin transaksi mutasi bagi seluruh modul keuangan di *simkeu-core*.

---

## 1. Definisi & Peran Bisnis

- **Periode (Tahun Anggaran)**: Wadah temporal kalender keuangan negara (1 Januari s.d. 31 Desember) yang menjadi partisi utama atas seluruh perencanaan anggaran, alokasi penarikan dana, komitmen belanja SPP/SPM/SP2D, serta pembukuan jurnal akuntansi.
- **Siklus Hidup Status Periode (`PeriodeStatus`)**:
  - `MENGINISIALISASI`: Tahap bootstrap awal tahun anggaran, migrasi saldo akun, dan persiapan sistem.
  - `PERENCANAAN`: Tahap penyusunan usulan DIPA dan pagu unit kerja (modul Perencanaan aktif).
  - `TERBUKA`: Tahap pelaksanaan anggaran riil operasional penuh (seluruh transaksi komitmen, penerimaan, dan pengeluaran kas aktif).
  - `PENYESUAIAN`: Tahap tutup buku sementara untuk penyusunan laporan keuangan dan penyesuaian akhir tahun (*adjusting entries*).
  - `TERTUTUP`: Tahap final pasca audit di mana seluruh transaksi baru pada tahun anggaran tersebut diblokir secara permanen (*read-only*).

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Tahun Anggaran**: Nilai `tahunAnggaran` (misal 2026) harus unik di seluruh sistem.
2. **Kekekalan Status Tertutup**: Periode yang berstatus `TERTUTUP` tidak boleh menerima transaksi mutasi jurnal, SPP, penerimaan, atau pengeluaran baru. Seluruh modul memeriksa keaktifan status periode sebelum memproses permintaan.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Periode`
- **Tabel Basis Data**: `periode`
- **Kelas Model**: `org.ilmi.simkeucore.periode.domain.Periode`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data periode (UUID v4). |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** (Unique) | Partisi Lintas Modul | Tahun fiskal anggaran (misal `2025`, `2026`, `2027`). |
| `periodeStatus` | `periode_status` | `Enum (Ordinal)` | **Wajib** | `PeriodeStatus` | Status aktif siklus hidup tahun anggaran. |
| `createdBy` | `created_by` | `String` | Opsional | `User.username` | NIP / username pembuat periode. |
| `createdDate` | `created_date` | `Instant` | Opsional | - | Waktu inisialisasi periode. |

---

## 4. Enumerasi Domain Terkait

### `PeriodeStatus`
- `MENGINISIALISASI`: Tahap persiapan awal dan migrasi akun.
- `PERENCANAAN`: Penyusunan pagu dan RKA-K/L unit kerja.
- `TERBUKA`: Pelaksanaan anggaran dan operasional kas aktif.
- `PENYESUAIAN`: Rekonsiliasi, audit internal, dan jurnal penyesuaian akhir tahun.
- `TERTUTUP`: Periode buku resmi ditutup permanen (*read-only*).
