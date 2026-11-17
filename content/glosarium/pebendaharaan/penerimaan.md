---
title: Penerimaan
draft: false
tags:
  - pebendaharaan
  - penerimaan
  - kas-masuk
  - pendapatan
---

# Glosarium Domain: Penerimaan (Perbendaharaan)

Entitas agregat dalam modul **Perbendaharaan** yang mencatat realisasi penerimaan pendapatan definitif kas BLU (Tarif Layanan Pendidikan/UKT, Kerja Sama, Jasa Giro, Sewa Fasilitas) lengkap dengan rincian pemotongan pajak dan pembukuan jurnal otomatis.

---

## 1. Definisi & Peran Bisnis

- **Penerimaan Kas**: Bukti sah pembukuan kas masuk definitif yang diterbitkan oleh Bendahara Penerimaan. Penerimaan kas mengikat pos akun pendapatan operasional BLU, menghitung kewajiban potongan pajak yang disetor, serta menggenerate baris debit kas dan kredit pendapatan (`EntryLine`).
- **Posting Penerimaan**: Eksekusi metode `posting()` yang menerbitkan event domain `PenerimaanDipostingEvent` ke modul Akuntansi untuk membentuk jurnal pembukuan umum secara otomatis.

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Nomor Bukti**: Nomor bukti penerimaan (`nomor`) harus unik dalam satu tahun anggaran.
2. **Keseimbangan Baris Pembukuan**: Total baris debit dan kredit pada `entries` wajib seimbang (*zero-sum balance*).
3. **Penyelarasan Nilai Transaksi & Pajak**:
   $$\text{Total Kas Diterima} = \sum \text{TransactionLine.nilai} - \sum \text{TaxLine.nilai}$$
4. **Kekekalan Pasca Posting**: Penerimaan yang telah diposting tidak dapat dimutasi kembali nilainya.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Penerimaan`
- **Tabel Basis Data**: `penerimaan`
- **Kelas Model**: `org.ilmi.simkeucore.pebendaharaan.domain.Penerimaan`
- **Kelas Induk**: `Auditable<Penerimaan>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas penerimaan (UUID v4). |
| `nomor` | `nomor` | `String` | **Wajib** (Unique) | - | Nomor bukti resmi penerimaan kas (misal `PNM-2026-0042`). |
| `uraian` | `uraian` | `String` | **Wajib** | - | Penjelasan sumber atau tujuan penerimaan kas BLU. |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran aktif pencatatan. |
| `entries` | Tabel: `penerimaan_entries` | `List<EntryLine>` | **Wajib** | `@ElementCollection` | Baris pembukuan akuntansi debit/kredit yang dihasilkan. |
| `taxes` | Tabel: `penerimaan_taxes` | `List<TaxLine>` | Opsional | `@ElementCollection` | Daftar rincian pungutan/potongan pajak terkait. |
| `transactions` | Tabel: `penerimaan_transactions` | `List<TransactionLine>` | **Wajib** | `@ElementCollection` | Rincian nominal pendapatan per akun BAS pendapatan. |

---

## 4. Value Object & Koleksi Turunan

### Value Object: `TransactionLine`
- **Tabel Koleksi**: `penerimaan_transactions`
- `akunId`: ID akun BAS pendapatan (`Akun.id`).
- `nilai`: Nominal pendapatan riil yang dialokasikan ke akun tersebut.

### Value Object: `TaxLine`
- **Tabel Koleksi**: `penerimaan_taxes`
- `pajakId`: Referensi ke entitas master `PajakEntity.id`.
- `nilai`: Besaran nominal pungutan pajak yang dipotong/disetorkan.

### Value Object: `EntryLine`
- **Tabel Koleksi**: `penerimaan_entries`
- `akunId`: Akun buku besar yang didebit/dikredit.
- `debit`: Nominal sisi debit.
- `credit`: Nominal sisi kredit.
