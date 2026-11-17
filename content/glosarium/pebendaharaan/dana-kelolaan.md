---
title: DanaKelolaan
draft: false
tags:
  - pebendaharaan
  - dana-kelolaan
  - rekonsiliasi
  - kas
---

# Glosarium Domain: DanaKelolaan (Perbendaharaan)

Entitas agregat dalam modul **Perbendaharaan** yang mengelola pencatatan kas masuk bruto pada rekening penampung / rekening operasional sebelum direkonsiliasi menjadi pos Penerimaan BLU definitif.

---

## 1. Definisi & Peran Bisnis

- **Dana Kelolaan (Unreconciled Treasury Funds)**: Aliran dana kas masuk yang diterima pada rekening giro/bank kampus (misal transfer mahasiswa via Virtual Account perbankan, bunga bank, atau titipan pihak ketiga) yang telah tercatat di mutasi rekening koran namun belum diidentifikasi pos rekening pendapatan atau perpajakannya.
- **Rekonsiliasi Kas**: Proses bisnis bendahara penerimaan untuk memvalidasi dan memecah (*split/map*) Dana Kelolaan ke dalam entitas `Penerimaan` definitif dengan rincian pos akun pendapatan (`TransactionLine`) dan kewajiban perpajakan (`TaxLine`).

---

## 2. Invarian & Aturan Bisnis

1. **Rekonsiliasi Tuntas**: Setelah proses rekonsiliasi berhasil dijalankan (`rekonsiliasi(...)`), entitas menerbitkan satu atau lebih objek `Penerimaan` yang nilainya mencerminkan total dana kelolaan.
2. **Nilai Positif**: Nilai dana kelolaan (`nilai`) harus lebih besar dari nol ($> 0.00$).
3. **Validitas Sumber Kas**: Sumber kas (`source`) wajib mencantumkan tipe rekening (`BANK` atau `GIRO`) beserta nomor rekening bank penampung.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `DanaKelolaan`
- **Tabel Basis Data**: `dana_kelolaan`
- **Kelas Model**: `org.ilmi.simkeucore.pebendaharaan.domain.DanaKelolaan`
- **Kelas Induk**: `Auditable<DanaKelolaan>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas dana kelolaan (UUID v4). |
| `nilai` | `nilai` | `BigDecimal(19, 2)` | **Wajib** | - | Nominal bruto mutasi kas masuk yang tercatat pada rekening koran. |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran aktif pencatatan dana. |
| `source` | `source_*` | `DanaKelolaanSource` | **Wajib** | `@Embedded` | Informasi perbankan sumber penerimaan dana penampung. |

---

## 4. Value Object & Komponen Embedded

### Value Object: `DanaKelolaanSource`
- `type`: Tipe rekening bank (`BANK` atau `GIRO`).
- `nomorRekening`: Nomor rekening bank penampung resmi BLU.
- `namaBank`: Nama bank mitra operasional penerima (misal Bank Mandiri, BNI, BRI, BSI).
