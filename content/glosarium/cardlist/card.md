---
title: Card (Rekanan)
draft: false
tags:
  - cardlist
  - card
  - supplier
  - customer
  - rekanan
---

# Glosarium Domain: Card / Rekanan (Cardlist)

Entitas agregat dasar dalam modul **Cardlist** yang mengelola data master pihak ketiga yang bertransaksi dengan instansi BLU, mencakup Rekanan/Penyedia Barang dan Jasa (*Supplier*) serta Pelanggan/Mitra Layanan (*Customer*).

---

## 1. Definisi & Peran Bisnis

- **Card (Kartu Rekanan/Kontak)**: Representasi identitas hukum atau perorangan rekanan BLU Politeknik Negeri Sriwijaya. Menggunakan pola pewarisan tabel tunggal (*Single-Table Inheritance*) dengan pembeda kolom diskriminator `card_type`:
  - `SUPPLIER`: Penyedia barang/jasa atau kontraktor pelaksana pekerjaan pengadaan/belanja modal/operasional kampus.
  - `CUSTOMER`: Mitra kerja sama, mahasiswa, penyewa fasilitas gedung/lahan, atau pihak luar pembayar tarif layanan BLU.
- **Designation**: Bentuk badan hukum rekanan:
  - `COMPANY`: Badan usaha formal (PT, CV, Firma, Koperasi, Yayasan) yang memiliki NPWP badan.
  - `INDIVIDUAL`: Perorangan / konsultan / narasumber ahli.

---

## 2. Invarian & Aturan Bisnis

1. **Integritas Transaksi**: Card yang telah digunakan dalam kontrak aktif (`Kontrak.cardId`) atau Surat Permintaan Pembayaran (`Spp.penerimaId`) tidak boleh dihapus secara permanen.
2. **Kesesuaian Tipe**: Transaksi belanja pengadaan/SPP LS pihak ketiga wajib menggunakan rekanan berjenis `SUPPLIER`. Transaksi penerimaan pendapatan jasa sewa/layanan BLU menggunakan `CUSTOMER`.
3. **NPWP & Rekening Bank**: Setiap rekanan badan wajib mencantumkan Nomor Pokok Wajib Pajak (NPWP) dan rekening bank resmi penerima transfer dana.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Card`
- **Tabel Basis Data**: `card`
- **Strategi Pewarisan**: `InheritanceType.SINGLE_TABLE` (Discriminator: `card_type`)
- **Kelas Model**: `org.ilmi.simkeucore.cardlist.core.model.aggregate.Card`
- **Subkelas**: `Supplier` (`card_type = 'SUPPLIER'`), `Customer` (`card_type = 'CUSTOMER'`)

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data rekanan (UUID v4). |
| `cardType` | `card_type` | `String` | **Wajib** | Diskriminator | Tipe entitas: `SUPPLIER` atau `CUSTOMER`. |
| `designation` | `designation` | `Enum (Integer)` | **Wajib** | `Designation` | Bentuk subjek hukum (`COMPANY` / `INDIVIDUAL`). |
| `nama` | `nama` | `String` | **Wajib** | - | Nama resmi perusahaan (PT/CV) atau nama lengkap individu. |
| `alamat` | Kolom Embedded | `Alamat` | Opsional | `@Embedded` | Komponen alamat domisili hukum/operasional. |
| `contact` | Kolom Embedded | `Contact` | Opsional | `@Embedded` | Komponen kontak narahubung (telepon, surel, kontak person). |

### Atribut Khusus Subkelas: `Customer`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `detilPenjualan` | `detil_penjualan_*` | `SellingDetails` | Opsional | `@Embedded` | Informasi pemetaan akun pendapatan layanan. |
| `detilPiutang` | `detil_piutang_*` | `DetilPiutang` | Opsional | `@Embedded` | Batas kredit dan termin piutang pelanggan. |

---

## 4. Value Object & Komponen Embedded

### Value Object: `Alamat`
- `jalan`: Nama jalan, nomor kantor, RT/RW.
- `kota`: Kabupaten / Kota domisili.
- `provinsi`: Provinsi domisili rekanan.
- `kodePos`: Kode pos pos wilayah.

### Value Object: `Contact`
- `email`: Alamat surel korespondensi resmi.
- `telepon`: Nomor telepon kantor / seluler aktif.
- `contactPerson`: Nama petugas narahubung/penanggung jawab rekanan.

### Value Object: `Rekening` (Perbankan Rekanan)
- `namaBank`: Nama institusi perbankan (misal: Bank Mandiri, BNI, BRI, BSI).
- `nomorRekening`: Nomor rekening bank penerima pembayaran transfer.
- `atasNama`: Nama pemilik rekening yang tertera di buku tabungan/rekening koran.

---

## 5. Enumerasi Domain Terkait

### `Designation`
- `COMPANY`: Entitas bisnis berbadan hukum (PT, CV, Koperasi, BUMN, dll).
- `INDIVIDUAL`: Perorangan / tenaga ahli / narasumber.
