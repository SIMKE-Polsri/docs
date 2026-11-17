---
title: Pajak
draft: false
tags:
  - perpajakan
  - pajak
  - vat
  - wht
  - ppn
  - pph
---

# Glosarium Domain: Pajak (Perpajakan)

Entitas master konfigurasi dalam modul **Perpajakan** yang mendefinisikan jenis pajak pertambahan nilai (PPN/VAT) dan pajak penghasilan pemotongan/pemungutan (PPh/WHT), besaran tarif persentase, serta pemetaan akun utang/piutang perpajakan untuk transaksi keuangan BLU.

---

## 1. Definisi & Peran Bisnis

- **Pajak (Tax Configuration)**: Parameter resmi tarif perpajakan negara yang diterapkan secara otomatis pada saat pemotongan pembayaran belanja SPP/SPBy (modul Perealisasian) atau pemungutan pendapatan jasa layanan (modul Perbendaharaan).
- **Klasifikasi Tipe Pajak (`PajakType`)**:
  - `VAT` (*Value Added Tax* / PPN): Pajak Pertambahan Nilai atas perolehan barang kena pajak dan jasa kena pajak (tarif standar misal 11% / 12%). Memiliki pemetaan akun `salesAccountId` (PPN Keluaran) dan `purchaseAccountId` (PPN Masukan).
  - `WHT` (*Withholding Tax* / PPh): Pajak Penghasilan pemotongan/pemungutan pasal 21 (gaji/honor), pasal 22 (pengadaan barang pemerintah), pasal 23 (jasa/sewa), dan pasal 4 ayat 2 (final konstruksi). Memiliki pemetaan akun `payableAccountId` (Utang PPh) dan `receivableAccountId` (Piutang PPh).

---

## 2. Invarian & Aturan Bisnis

1. **Rentang Persentase Tarif**: Nilai persentase tarif (`persentase`) dinyatakan dalam desimal proporsional (misal `0.110` untuk 11%, atau `0.020` untuk 2%).
2. **Kesesuaian Akun Buku Besar**: Seluruh referensi akun perpajakan (`salesAccountId`, `purchaseAccountId`, `payableAccountId`, `receivableAccountId`) wajib mengarah ke akun detil aktif pada modul Akuntansi.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `PajakEntity`
- **Tabel Basis Data**: `pajak_entity`
- **Kelas Model**: `org.ilmi.simkeucore.perpajakan.infrastructure.persistence.entity.PajakEntity`
- **Kelas Induk**: `Auditable<PajakEntity>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data konfigurasi pajak (UUID v4). |
| `kode` | `kode` | `String` | Opsional | - | Kode unik pengenal pajak (misal `PPN11`, `PPH21`, `PPH22`, `PPH23`). |
| `uraian` | `uraian` | `String` | Opsional | - | Deskripsi lengkap jenis dan dasar hukum pajak. |
| `persentase` | `persentase` | `BigDecimal(4, 3)` | **Wajib** | - | Tarif persentase perpajakan (misal `0.110` untuk 11%). |
| `type` | `type` | `Enum (Ordinal)` | **Wajib** | `PajakType` | Klasifikasi perpajakan (`VAT` / `WHT`). |
| `salesAccountId` | `sales_account_id` | `UUID` | Opsional | `Akun.id` | Akun penampung PPN Penjualan / Keluaran (khusus VAT). |
| `purchaseAccountId`| `purchase_account_id`| `UUID` | Opsional | `Akun.id` | Akun penampung PPN Pembelian / Masukan (khusus VAT). |
| `payableAccountId` | `payable_account_id` | `UUID` | Opsional | `Akun.id` | Akun Utang Pajak PPh yang dipotong dari rekanan (khusus WHT). |
| `receivableAccountId`| `receivable_account_id`| `UUID` | Opsional | `Akun.id` | Akun Piutang/Uang Muka Pajak (khusus WHT). |

---

## 4. Enumerasi Domain Terkait

### `PajakType`
- `VAT`: Pajak Pertambahan Nilai (*Value Added Tax* / PPN).
- `WHT`: Pajak Penghasilan Pemotongan (*Withholding Tax* / PPh 21, 22, 23, 4(2)).
