---
title: Pengeluaran
draft: false
tags:
  - pebendaharaan
  - pengeluaran
  - kas-keluar
  - belanja
---

# Glosarium Domain: Pengeluaran (Perbendaharaan)

Entitas agregat dalam modul **Perbendaharaan** yang mencatat eksekusi pencairan kas keluar nyata dari rekening kampus BLU, baik berdasarkan perintah Surat Perintah Pencairan Dana (SP2D) maupun Surat Perintah Bayar (SPBy).

---

## 1. Definisi & Peran Bisnis

- **Pengeluaran Kas**: Bukti eksekusi nyata perpindahan uang dari rekening kas bendahara pengeluaran/rekening giro BLU kepada pihak ketiga (rekanan/pegawai). Pengeluaran kas mengurangi saldo likuiditas bank dan memicu pencatatan pembukuan kas pada modul Akuntansi.
- **Sumber Pengeluaran (`PengeluaranSource`)**:
  - `SP2D`: Pencairan dana berbasis Surat Perintah Pencairan Dana (pembayaran LS kontrak atau pengisian UP/TUP/GUP).
  - `SPBY`: Pencairan tunai/langsung berbasis Surat Perintah Bayar uang persediaan.
- **Pencairan Parsial (`isPartial`)**: Fitur pencairan dana secara bertahap jika transfer bank dilakukan dalam beberapa kali pengiriman atau kliring bertahap.

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Nomor Pengeluaran**: Nomor transaksi kas keluar (`nomor`) harus unik.
2. **Keseimbangan Jurnal**: Baris pembukuan `entries` wajib seimbang antara total debit dan total kredit.
3. **Penyelarasan Nilai Bersih Kas Keluar**:
   $$\text{Kas Keluar Bersih} = \sum \text{Bruto Belanja} - \sum \text{TaxLine.nilai} - \sum \text{Potongan Lain}$$
4. **Penerbitan Event Posting**: Pemanggilan method `posting()` menerbitkan `PengeluaranDipostingEvent` untuk sinkronisasi otomatis ke buku besar akuntansi.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Pengeluaran`
- **Tabel Basis Data**: `pengeluaran`
- **Kelas Model**: `org.ilmi.simkeucore.pebendaharaan.domain.Pengeluaran`
- **Kelas Induk**: `Auditable<Pengeluaran>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas pengeluaran (UUID v4). |
| `nomor` | `nomor` | `String` | **Wajib** (Unique) | - | Nomor dokumen kas keluar resmi bendahara pengeluaran. |
| `uraian` | `uraian` | `String` | **Wajib** | - | Penjelasan detail peruntukan pengeluaran kas. |
| `isPartial` | `is_partial` | `Boolean` | Opsional | - | Menandakan apakah transaksi ini merupakan pencairan bertahap (default `false`). |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran aktif pengeluaran. |
| `source` | `source_*` | `PengeluaranSource`| **Wajib** | `@Embedded` | Sumber pemicu kas keluar (`SP2D` / `SPBY` beserta nomor referensi). |
| `entries` | Tabel: `pengeluaran_entries` | `List<EntryLine>` | **Wajib** | `@ElementCollection` | Baris pembukuan debit kredit pengeluaran kas. |
| `taxes` | Tabel: `pengeluaran_taxes` | `List<TaxLine>` | Opsional | `@ElementCollection` | Daftar rincian potongan pajak penghasilan/PPN yang disetor ke kas negara. |

---

## 4. Value Object & Komponen Embedded

### Value Object: `PengeluaranSource`
- `type`: Jenis dokumen sumber (`SP2D` atau `SPBY`).
- `referenceId`: UUID dokumen sumber terkait (`Sp2d.id` atau `Spby.id`).
- `referenceNo`: Nomor dokumen sumber (misal `SP2D-2026-0089`).

### Value Object: `EntryLine`
- `akunId`: ID akun buku besar (`Akun.id`).
- `debit`: Besaran sisi debit.
- `credit`: Besaran sisi kredit.

### Value Object: `TaxLine`
- `pajakId`: ID jenis pajak (`PajakEntity.id`).
- `nilai`: Besaran pemotongan pajak.
