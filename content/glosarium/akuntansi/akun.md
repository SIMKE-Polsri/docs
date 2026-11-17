---
title: Akun
draft: false
tags:
  - akuntansi
  - coa
  - akun
  - bagan-akun-standar
---

# Glosarium Domain: Akun (Akuntansi)

Entitas agregat dasar dalam modul **Akuntansi** yang menyusun hierarki Bagan Akun Standar (BAS / *Chart of Accounts*) untuk pengelompokan aset, kewajiban, ekuitas, pendapatan, dan beban instansi BLU.

---

## 1. Definisi & Peran Bisnis

- **Akun (Chart of Accounts)**: Wadah pencatatan klasifikasi keuangan bertingkat (hierarkis) yang merepresentasikan kode rekening pembukuan standar pemerintah. Akun disusun mulai dari level tertinggi (*Root Header*, misal `1` = Aset), kelompok (*Group Header*, misal `11` = Kas dan Setara Kas), hingga level posting transaksi operasional (*Detil / Leaf*, misal `11101` = Kas di Bendahara Pengeluaran).
- **Saldo Normal**: Sisi posisi default saldo akun bertambah:
  - `DEBIT`: Bertambah di debit (umum untuk pos Aset dan Beban).
  - `KREDIT`: Bertambah di kredit (umum untuk pos Kewajiban, Ekuitas, dan Pendapatan).
- **Opening History**: Rekam jejak saldo awal (*opening balance*) akun pada setiap tahun anggaran fiskal.

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Kode**: Kode akun (`kode`) harus unik di seluruh sistem.
2. **Aturan Transaksi Akun Detil**: Hanya akun dengan flag `isDetil = true` yang boleh digunakan dalam baris mutasi `Jurnal` atau pemetaan belanja `ItemAnggaran`. Akun header (`isDetil = false`) hanya berfungsi sebagai wadah agregasi laporan.
3. **Pewarisan Saldo Normal**: Akun anak (baik kelompok maupun detil) mewarisi saldo normal (`saldoNormal`) dari akun header induknya.
4. **Status Keaktifan**: Akun yang non-aktif (`active = false`) tidak dapat menerima mutasi pencatatan jurnal baru.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Akun`
- **Tabel Basis Data**: `akun`
- **Kelas Model**: `org.ilmi.simkeucore.akuntansi.v2.domain.Akun`
- **Kelas Induk**: `Auditable<Akun>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik akun buku besar (UUID v4). |
| `kode` | `kode` | `String` | **Wajib** (Unique) | - | Nomor/kode rekening standar (misal `511111` untuk Belanja Gaji Pokok PNS). |
| `uraian` | `uraian` | `String` | **Wajib** | - | Nama atau deskripsi akun rekening. |
| `active` | `active` | `Boolean` | **Wajib** | - | Indikator status keaktifan akun (default `true`). |
| `header` | `header_id` | `Akun` | Opsional | `@ManyToOne(Akun)` | Referensi ke akun induk (header) hierarki atasnya. Bernilai null untuk akun level tertinggi (*root*). |
| `isDetil` | `is_detil` | `Boolean` | **Wajib** | - | Penanda apakah akun merupakan level terbawah (posting akun). Jika `true`, dapat dicatat dalam jurnal. |
| `saldoNormal` | `saldo_normal` | `Enum (String)` | **Wajib** | `Side` | Posisi saldo bertambah normal (`DEBIT` atau `KREDIT`). |
| `openingHistories` | Tabel: `akun_opening_histories` | `List<OpeningHistory>` | Opsional | `@ElementCollection` | Histori saldo awal akun per tahun anggaran. |

---

## 4. Value Object & Koleksi Turunan

### Value Object: `OpeningHistory`
- **Tabel Koleksi**: `akun_opening_histories`
- **Join Column**: `akun_id` $\rightarrow$ `akun.id`
- **Kelas Model**: `org.ilmi.simkeucore.akuntansi.v2.domain.vo.OpeningHistory`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun anggaran berlakunya saldo awal. |
| `nominal` | `nominal` | `BigDecimal(19, 2)` | **Wajib** | - | Besaran nilai saldo awal yang ditetapkan pada awal tahun buku. |
| `side` | `side` | `Enum (String)` | **Wajib** | `Side` | Sisi penempatan saldo awal (`DEBIT` atau `KREDIT`). |

---

## 5. Enumerasi Domain Terkait

### `Side`
- `DEBIT`: Sisi debit pada neraca/buku besar.
- `KREDIT`: Sisi kredit pada neraca/buku besar.
