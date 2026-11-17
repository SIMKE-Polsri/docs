---
title: SPBy (Surat Perintah Bayar)
draft: false
tags:
  - perealisasian
  - spby
  - uang-persediaan
  - kas-kecil
---

# Glosarium Domain: SPBy / Surat Perintah Bayar (Perealisasian)

Entitas agregat dalam modul **Perealisasian** yang mengelola penerbitan Surat Perintah Bayar (SPBy) untuk pembayaran langsung belanja operasional bernilai kecil menggunakan kas Uang Persediaan (UP) Bendahara Pengeluaran.

---

## 1. Definisi & Peran Bisnis

- **Surat Perintah Bayar (SPBy)**: Perintah bayar internal yang diterbitkan oleh PPK kepada Bendahara Pengeluaran untuk mencairkan kas uang persediaan (UP) secara tunai atau transfer langsung kepada pihak penerima (kegiatan rapat, perjalanan dinas, pembelian ATK mendesak).
- **Mekanisme Revolving (GUP)**: SPBy yang telah dibayarkan oleh bendahara (`status = PAID`) dikumpulkan sebagai bukti riil belanja untuk kemudian diganti melalui penerbitan `SPP GUP` (Ganti Uang Persediaan) guna mengisi kembali kas UP.

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Nomor SPBy**: Nomor dokumen SPBy (`nomor`) harus unik dalam satu tahun anggaran.
2. **Kesesuaian Sumber Dana UP**: SPBy hanya boleh membebani mata anggaran operasional yang dialokasikan untuk skema Uang Persediaan.
3. **Penerbitan Event Pembayaran**: Ketika kas telah diserahkan dan status berubah menjadi `PAID`, sistem menerbitkan event `SpbyPaidEvent`.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Spby`
- **Tabel Basis Data**: `spby`
- **Kelas Model**: `org.ilmi.simkeucore.perealisasian.v2.domain.Spby`
- **Kelas Induk**: `Auditable<Spby>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data SPBy (UUID v4). |
| `nomor` | `nomor` | `String` | **Wajib** (Unique) | - | Nomor dokumen resmi SPBy (misal `SPBY-015/BAUK/2026`). |
| `uraian` | `uraian` | `String` | **Wajib** | - | Penjelasan maksud pembayaran kas UP. |
| `status` | `status` | `Enum (String)` | **Wajib** | `SpbyStatus` | Status pemrosesan SPBy (`DRAFT`, `APPROVED`, `PAID`, `REJECTED`, `REVOLVED`). |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran aktif. |
| `realisasi` | Tabel: `spby_realisasi` | `List<RealisasiAnggaran>`| **Wajib** | `@ElementCollection` | Alokasi pembebanan anggaran belanja pada Item Anggaran. |
| `potongan` | Tabel: `spby_potongan` | `List<Potongan>` | Opsional | `@ElementCollection` | Daftar potongan non-pajak. |
| `pajak` | Tabel: `spby_pajak` | `List<PotonganPajak>`| Opsional | `@ElementCollection` | Daftar potongan pajak penghasilan/PPN yang dipungut. |

---

## 4. Enumerasi Domain Terkait

### `SpbyStatus`
- `DRAFT`: SPBy baru dibuat dan diverifikasi internal unit.
- `APPROVED`: Disetujui PPK untuk dibayarkan oleh Bendahara.
- `PAID`: Uang telah diserahkan kepada pihak penerima oleh Bendahara Pengeluaran.
- `REJECTED`: Ditolak karena ketidaklengkapan kuitansi/bukti fisik.
- `REVOLVED`: Telah dipertanggungjawabkan dan diajukan dalam SPP GUP.
