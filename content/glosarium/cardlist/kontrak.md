---
title: Kontrak
draft: false
tags:
  - cardlist
  - kontrak
  - spk
  - termin
  - addendum
---

# Glosarium Domain: Kontrak (Cardlist)

Entitas agregat utama dalam modul **Cardlist** yang mengelola komitmen pengadaan barang/jasa, surat perjanjian kerja (SPK), tahapan termin pencairan dana, serta rekam jejak addendum kontrak BLU.

---

## 1. Definisi & Peran Bisnis

- **Kontrak (Surat Perjanjian Kerja / SPK)**: Dokumen hukum pengikatan komitmen antara Pejabat Pembuat Komitmen (PPK) kampus dengan Rekanan/Penyedia (`Supplier`) atas pekerjaan konstruksi, pengadaan barang, atau jasa lainnya yang membebani pagu anggaran tertentu.
- **Termin Kontrak**: Tahapan pencairan pembayaran berkala yang disepakati berdasarkan pencapaian tonggak pekerjaan (*milestone*), misal Uang Muka, Progress 50%, BAST 100%, atau Retensi Pemeliharaan.
- **Addendum**: Perubahan resmi terhadap klausul kontrak asli (perpanjangan waktu pelaksanaan, perubahan volume/spesifikasi, atau penyesuaian nilai kontrak).

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Nomor Kontrak**: Nomor kontrak (`nomor`) harus unik di seluruh sistem.
2. **Keseimbangan Nilai Termin**:
   $$\sum \text{nilaiPembayaran (Termin)} \le \text{nilaiKontrak}$$
   Total nilai seluruh termin yang didaftarkan tidak boleh melampaui plafon total nilai kontrak.
3. **Kalkulasi Nilai Outstanding**:
   $$\text{nilaiOutstanding} = \text{nilaiKontrak} - \sum \text{Termin Terbayar}$$
4. **Relasi ke SPP LS-Kontraktual**: Termin yang berstatus siap cair menjadi basis pembuatan Surat Permintaan Pembayaran Langsung (`SPP LS-Kontraktual`) di modul Perealisasian.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `Kontrak`
- **Tabel Basis Data**: `kontrak`
- **Kelas Model**: `org.ilmi.simkeucore.cardlist.core.model.aggregate.Kontrak`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data kontrak (UUID v4). |
| `nomor` | `nomor_kontrak` | `String(50)` | **Wajib** (Unique) | - | Nomor resmi Surat Perjanjian Kerja / Kontrak Pengadaan. |
| `tanggalKontrak` | `tanggal_kontrak` | `LocalDate` | **Wajib** | - | Tanggal resmi penandatanganan dokumen kontrak. |
| `judulPekerjaan` | `judul_pekerjaan` | `String` | **Wajib** | - | Deskripsi nama paket pengadaan atau pekerjaan proyek. |
| `unitKerja` | `unit_kerja` | `String(100)` | **Wajib** | `UnitKerja` | Unit kerja pemohon/pelaksana pengadaan di lingkungan kampus. |
| `status` | `status_kontrak` | `Enum (Ordinal)` | **Wajib** | `StatusKontrak` | Status siklus kontrak (`DRAFT`, `ACTIVE`, `COMPLETED`, `CANCELED`, `ADDENDUM`). |
| `cardId` | `card_id` | `UUID` | **Wajib** | `Card.id` (Supplier) | Referensi ke rekanan penyedia pelaksana pekerjaan. |
| `nilaiKontrak` | `nilai_total_kontrak` | `BigDecimal(19, 2)` | **Wajib** | - | Nilai total pagu komitmen kontrak. |
| `nilaiOutstanding`| `nilai_outstanding` | `BigDecimal(19, 2)` | **Wajib** | - | Sisa komitmen kontrak yang belum direalisasikan/dibayarkan. |
| `terms` | Tabel: `termin_kontrak` | `List<Termin>` | Opsional | `@OneToMany` | Daftar rincian tahapan termin pembayaran kontrak. |
| `addendums` | Tabel: `addendum` | `List<Addendum>` | Opsional | `@OneToMany` | Daftar histori addendum/perubahan kontrak. |

---

## 4. Child Entities & Rincian Termin

### Child Entity: `Termin`
- **Tabel Basis Data**: `termin_kontrak`
- **Kelas Model**: `org.ilmi.simkeucore.cardlist.core.model.entity.Termin`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik termin pembayaran. |
| `kontrak` | `kontrak_id` | `Kontrak` | **Wajib** | `@ManyToOne(Kontrak)` | Referensi ke kontrak induk pemilik termin. |
| `nomorTermin` | `nomor_termin` | `Integer` | **Wajib** | - | Nomor urut tahapan termin (1, 2, 3, dst.). |
| `nilaiPembayaran`| `nilai_pembayaran`| `BigDecimal(19, 2)` | **Wajib** | - | Nominal dana yang dicairkan pada tahapan termin ini. |
| `syaratPembayaran`| `syarat_pembayaran`| `String(255)` | **Wajib** | - | Klausul syarat pencairan (misal "Kemajuan Fisik 50%", "BAST 100%"). |
| `targetTanggal` | `target_tanggal` | `LocalDate` | Opsional | - | Estimasi jadwal target pencairan pembayaran termin. |
| `status` | `status_termin` | `String` | **Wajib** | - | Status termin (`PENDING`, `READY_FOR_PAYMENT`, `PAID`). |

### Child Entity: `Addendum`
- **Tabel Basis Data**: `addendum`
- **Kelas Model**: `org.ilmi.simkeucore.cardlist.core.model.entity.Addendum`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas addendum. |
| `kontrak` | `kontrak_id` | `Kontrak` | **Wajib** | `@ManyToOne(Kontrak)` | Referensi ke kontrak induk yang diubah. |
| `nomorAddendum` | `nomor_addendum` | `String` | **Wajib** | - | Nomor surat resmi addendum kontrak. |
| `tanggalAddendum`| `tanggal_addendum`| `LocalDate` | **Wajib** | - | Tanggal berlakunya addendum. |
| `perubahanNilai` | `perubahan_nilai`| `BigDecimal(19, 2)` | Opsional | - | Selisih penambahan atau pengurangan nilai kontrak. |
| `keterangan` | `keterangan` | `String` | Opsional | - | Rincian pokok klausul yang mengalami perubahan. |

---

## 5. Enumerasi Domain Terkait

### `StatusKontrak`
- `DRAFT`: Kontrak baru disusun dan belum disetujui PPK.
- `ACTIVE`: Kontrak telah ditandatangani dan berlaku aktif.
- `COMPLETED`: Seluruh pekerjaan selesai dan pembayaran termin lunas.
- `CANCELED`: Kontrak dibatalkan karena wanprestasi atau force majeure.
- `ADDENDUM`: Kontrak sedang dalam masa penyesuaian melalui addendum.
