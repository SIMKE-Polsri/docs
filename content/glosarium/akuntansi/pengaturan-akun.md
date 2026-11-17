---
title: PengaturanAkun
draft: false
tags:
  - akuntansi
  - pengaturan-akun
  - mapping
  - system-ledger
---

# Glosarium Domain: PengaturanAkun (Akuntansi)

Entitas singleton/konfigurasi dalam modul **Akuntansi** yang memetakan akun sistem baku (*system ledger mappings*) ke akun buku besar riil untuk kebutuhan posting otomatis lintas modul (Perbendaharaan, Pajak, dan Kas).

---

## 1. Definisi & Peran Bisnis

- **Pengaturan Akun**: Entitas konfigurasi terpusat (tunggal dengan `id = 1`) yang berfungsi menjembatani transaksi otomatis dari modul lain (seperti mutasi Kas Dana Kelolaan, Penerimaan Kas, Pengeluaran SP2D, dan Potongan Pajak) ke akun buku besar spesifik tanpa *hardcoding* ID akun pada kode program.
- **Tautan Akun**: Titik kait (*hook*) simbolik yang mendefinisikan peran akun dalam proses akuntansi BLU.

---

## 2. Invarian & Aturan Bisnis

1. **Singleton Entity**: Hanya ada tepat 1 baris konfigurasi dalam tabel `pengaturan_akun` (selalu memiliki `id = 1`).
2. **Kesesuaian Akun**: Akun yang dipetakan pada setiap `Tautan` wajib merupakan akun detil aktif (`isDetil = true` dan `active = true`).

---

## 3. Kamus Data & Atribut Entitas

### Entitas Konfigurasi: `PengaturanAkun`
- **Tabel Basis Data**: `pengaturan_akun`
- **Kelas Model**: `org.ilmi.simkeucore.akuntansi.v2.domain.PengaturanAkun`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `Long` | **Wajib** (PK) | - | Identifikator konstan (selalu `1L`). |
| `map` | `map` | `Map<Tautan, UUID>` | **Wajib** | JSON Map $\rightarrow$ `Akun.id` | Struktur JSON penyimpan pasangan kunci `Tautan` (nama alias sistem) dan nilai `UUID` (ID akun buku besar tujuan). |

---

## 4. Enumerasi Domain Terkait

### `Tautan`
- `SAL_BLU`: Akun Saldo Anggaran Lebih (SAL) BLU.
- `DK`: Akun Kas Dana Kelolaan BLU.
- `OPS_PNM`: Akun Kas Operasional Penerimaan.
- `OPS_PGL`: Akun Kas Operasional Pengeluaran (Bendahara Pengeluaran).
- `PENERIMAAN_BLU`: Akun Pendapatan Jasa Layanan BLU.
- `PENGELUARAN_TRANSITO_BLU`: Akun Transito Pengeluaran Kas BLU (perantara SP2D ke pencairan riil).
- `HISTORICAL_BALANCING`: Akun penyeimbang historis untuk migrasi data saldo awal.
