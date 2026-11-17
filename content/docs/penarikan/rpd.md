---
title: Rencana Penarikan Dana (RPD)
draft: false
tags:
  - penarikan
  - rpd
  - pagu-bulanan
  - kas-harian
---

# Glosarium Domain: RPD / Rencana Penarikan Dana (Penarikan)

Entitas agregat utama dalam modul **Penarikan** yang mengelola Rencana Penarikan Dana (RPD) bulanan dan harian per akun belanja guna mengendalikan likuiditas kas dan jadwal penyaluran dana BLU.

---

## 1. Definisi & Peran Bisnis

- **Rencana Penarikan Dana (RPD Bulanan)**: Estimasi kebutuhan pencairan anggaran kas bulanan (Januari s.d. Desember) yang disusun oleh masing-masing unit kerja per akun belanja (`Akun.id`). RPD menjadi instrumen perencanaan likuiditas kas kampus agar tidak terjadi ketidaktercukupan dana pada rekening giro operasional saat tagihan masuk.
- **RPD Harian**: Rincian pemecahan jadwal penarikan dana bulanan ke tanggal-tanggal spesifik dalam bulan berjalan untuk memetakan proyeksi arus kas harian (*daily cash flow forecast*).

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Kunci RPD Bulanan**: Kombinasi tahun anggaran, akun belanja, dan bulan (`tahun_anggaran`, `akun_id`, `bulan`) harus unik di seluruh sistem.
2. **Kesesuaian Total Harian vs Bulanan**:
   $$\sum \text{RpdHarian.jumlah} \le \text{RpdBulanan.jumlahPaguBulanan}$$
   Total akumulasi alokasi rencana harian dalam satu bulan tidak boleh melebihi pagu bulanan yang telah disetujui.
3. **Batas Rentang Bulan**: Nilai atribut `bulan` wajib berada di antara 1 (Januari) hingga 12 (Desember).
4. **Validasi Tanggal Harian**: Nilai `tanggal` pada `RpdHarian` wajib berada dalam bulan dan tahun anggaran yang sama dengan `RpdBulanan` induknya.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `RpdBulanan`
- **Tabel Basis Data**: `rpd_bulanan`
- **Kelas Model**: `org.ilmi.simkeucore.penarikan.core.entity.RpdBulanan`
- **Konstrain Unik**: `tahun_anggaran`, `akun_id`, `bulan`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data RPD bulanan (UUID v4). |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran rencana penarikan. |
| `akunId` | `akun_id` | `UUID` | **Wajib** | `Akun.id` | Akun mata anggaran belanja yang akan ditarik. |
| `bulan` | `bulan` | `Integer` | **Wajib** | 1 s.d. 12 | Periode bulan penarikan dana (1 = Jan, 12 = Des). |
| `jumlahPaguBulanan`| `jumlah_pagu_bulanan`| `BigDecimal(19, 2)`| **Wajib** | - | Plafon target maksimal dana yang direncanakan cair pada bulan ini. |
| `rpdHarianList` | Tabel: `rpd_harian` | `List<RpdHarian>` | Opsional | `@OneToMany` | Daftar rincian jadwal penarikan harian pada bulan ini. |
| `createdBy` | `created_by` | `String` | Opsional | `User.username` | NIP / pembuat dokumen RPD. |
| `createdDate` | `created_date` | `Instant` | Opsional | - | Waktu pembuatan RPD. |

---

## 4. Child Entity: Rincian Jadwal Harian

### Child Entity: `RpdHarian`
- **Tabel Basis Data**: `rpd_harian`
- **Kelas Model**: `org.ilmi.simkeucore.penarikan.core.entity.RpdHarian`
- **Konstrain Unik**: `rpd_bulanan_id`, `tanggal`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik rencana penarikan harian. |
| `rpdBulanan` | `rpd_bulanan_id` | `RpdBulanan` | **Wajib** | `@ManyToOne(RpdBulanan)` | Referensi ke induk RPD bulanan. |
| `tanggal` | `tanggal` | `LocalDate` | **Wajib** | - | Tanggal target penarikan dana kas. |
| `jumlah` | `jumlah` | `BigDecimal(19, 2)` | **Wajib** | - | Nominal proyeksi kas keluar pada tanggal tersebut. |
| `keterangan` | `keterangan` | `String` | Opsional | - | Catatan peruntukan penarikan harian (misal "Pencairan Gaji Kontrak Termin 1"). |
