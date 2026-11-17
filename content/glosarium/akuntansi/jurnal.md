---
title: Jurnal
draft: false
tags:
  - akuntansi
  - jurnal
  - double-entry
  - posting
---

# Glosarium Domain: Jurnal (Akuntansi)

Entitas agregat utama dalam modul **Akuntansi** yang mencatat mutasi transaksi finansial berpasangan (*double-entry bookkeeping*) untuk pembukuan dan pelaporan keuangan Badan Layanan Umum (BLU).

---

## 1. Definisi & Peran Bisnis

- **Jurnal**: Catatan historis terurut kronologis atas setiap transaksi finansial yang memengaruhi saldo akun pada Bagan Akun Standar (BAS). Setiap jurnal terdiri atas sekurang-kurangnya dua baris pembukuan (*lines*) yang harus memenuhi prinsip keseimbangan debit dan kredit (*zero-sum balance*).
- **Jenis Jurnal**:
  - `UMUM`: Transaksi operasional rutin (penerimaan kas, realisasi pengeluaran, penyesuaian belanja).
  - `PENYESUAIAN`: Jurnal koreksi akhir periode, penyusutan aset, atau akrual pendapatan/beban.
  - `PENUTUP`: Jurnal penutup saldo nominal pada akhir tahun anggaran fiskal.
  - `PEMBALIK`: Jurnal pembalik pada awal periode baru untuk pos akrual tertentu.
- **Posting**: Tindakan finalisasi pencatatan jurnal yang menetapkan tanggal efektif pembukuan (`postedAt`) dan menerbitkan domain event `JurnalPostedEvent` ke modul lain (misal untuk pembaruan buku besar). Jurnal yang sudah diposting bersifat *immutable*.

---

## 2. Invarian & Aturan Bisnis

1. **Prinsip Keseimbangan (*Double-Entry Balance*)**:
   $$\sum \text{Debit} = \sum \text{Kredit}$$
   Jika total debit tidak sama persis dengan total kredit, entitas akan menolak pencatatan dan melemparkan `AkuntansiException(ExceptionCode.NOT_BALANCED)`.
2. **Nilai Non-Negatif**: Nilai debit dan kredit pada setiap baris jurnal tidak boleh bernilai negatif ($\ge 0.00$).
3. **Validitas Akun**: Setiap baris jurnal wajib mereferensikan akun buku besar yang valid (`akunId`) dan berstatus aktif serta merupakan akun detil (`isDetil = true`).
4. **Partisi Temporal**: Setiap jurnal wajib terikat pada satu tahun anggaran fiskal (`tahunAnggaran`).
5. **Kekekalan Pasca Posting**: Setelah method `posting()` dieksekusi, status `postedAt` terisi dan baris transaksi tidak dapat dimutasi lagi.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Jurnal`
- **Tabel Basis Data**: `jurnal`
- **Kelas Model**: `org.ilmi.simkeucore.akuntansi.v2.domain.Jurnal`
- **Kelas Induk**: `Auditable<Jurnal>` (menyediakan metadata `createdBy`, `createdDate`, `lastModifiedBy`, `lastModifiedDate`)

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas jurnal (UUID v4 otomatis). |
| `memo` | `memo` | `String` | Opsional | - | Deskripsi atau uraian transaksi jurnal (keterangan alasan pencatatan). |
| `referenceNo` | `reference_no` | `String` | Opsional | - | Nomor dokumen sumber transaksi (misal nomor SP2D, nomor bukti kuitansi kas, atau nomor SPM). |
| `postedAt` | `posted_at` | `LocalDate` | Opsional | - | Tanggal efektif pembukuan/posting jurnal ke buku besar. Bernilai null jika masih berstatus draft. |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran aktif di mana transaksi jurnal ini dibukukan. |
| `jenis` | `jenis` | `Enum (String)` | **Wajib** | `JenisJurnal` | Klasifikasi jenis jurnal (`UMUM`, `PENYESUAIAN`, `PENUTUP`, `PEMBALIK`). |
| `lines` | Tabel: `jurnal_lines` | `List<Line>` | **Wajib** | `@ElementCollection` | Daftar rincian baris debit/kredit yang membentuk satu kesatuan jurnal. |

---

## 4. Value Object & Koleksi Turunan

### Value Object: `Line` (Baris Jurnal)
- **Tabel Koleksi**: `jurnal_lines`
- **Join Column**: `jurnal_id` $\rightarrow$ `jurnal.id`
- **Kelas Model**: `org.ilmi.simkeucore.akuntansi.v2.domain.vo.Line`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `akunId` | `akun_id` | `UUID` | **Wajib** | `Akun.id` | Referensi ke akun spesifik pada Bagan Akun Standar (BAS). |
| `debit` | `debit` | `BigDecimal(19, 2)` | **Wajib** | - | Nilai mutasi pada sisi debit (default `0.00`, minimal `0.00`). |
| `kredit` | `kredit` | `BigDecimal(19, 2)` | **Wajib** | - | Nilai mutasi pada sisi kredit (default `0.00`, minimal `0.00`). |

---

## 5. Enumerasi Domain Terkait

### `JenisJurnal`
- `UMUM`: Transaksi operasional reguler kas dan non-kas.
- `PENYESUAIAN`: Koreksi pembukuan, pengakuan beban penyusutan, atau penyesuaian akhir bulan.
- `PENUTUP`: Menutup akun-akun nominal pendapatan dan beban pada akhir tahun buku.
- `PEMBALIK`: Membalik jurnal penyesuaian tertentu di awal tahun anggaran berikutnya.
