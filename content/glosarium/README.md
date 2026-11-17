---
title: Katalog Glosarium & Kamus Data Domain SIMKEU-Core
draft: false
tags:
  - glosarium
  - domain
  - simkeu-core
  - dictionary
---

# Katalog Glosarium & Kamus Data Domain SIMKEU-Core

Katalog lengkap glosarium domain bisnis dan kamus data teknis (*data dictionary*) untuk seluruh entitas agregat pada 15 modul aplikasi sistem informasi manajemen keuangan terintegrasi **simkeu-core** Politeknik Negeri Sriwijaya.

Dokumentasi ini disusun mengacu pada peta arsitektur bounded context [CONTEXT-MAP.md](../../src/main/java/org/ilmi/simkeucore/CONTEXT-MAP.md).

---

## Daftar Modul & Glosarium Domain

### 1. Siklus Keuangan Inti (Core Financial Cycle)

| Modul | Entitas Agregat | Berkas Glosarium | Deskripsi Bisnis Singkat |
| :--- | :--- | :--- | :--- |
| **Periode** | `Periode` | [periode/periode.md](./periode/periode.md) | Tahun fiskal anggaran, siklus hidup tahun buku (`TERBUKA`, `PENYESUAIAN`, `TERTUTUP`). |
| **Perencanaan** | `UnitAnggaran` | [perencanaan/unit-anggaran.md](./perencanaan/unit-anggaran.md) | Hierarki DIPA pohon anggaran (Program, Kegiatan, KRO, RO, Komponen, Subkomponen). |
| | `ItemAnggaran` | [perencanaan/item-anggaran.md](./perencanaan/item-anggaran.md) | Baris mata anggaran belanja, rincian biaya Detil Anggaran, dan pagu alokasi belanja. |
| **Penarikan** | `RpdBulanan` & `RpdHarian` | [penarikan/rpd.md](./penarikan/rpd.md) | Rencana Penarikan Dana bulanan dan harian per akun belanja untuk likuiditas kas. |
| **Perealisasian** | `Spp`, `Spm`, `Sp2d` | [perealisasian/spp.md](./perealisasian/spp.md) | Rantai tagihan belanja: SPP (UP/TUP/GUP/LS), Verifikasi, penerbitan SPM, dan pencairan SP2D. |
| | `Spby` | [perealisasian/spby.md](./perealisasian/spby.md) | Surat Perintah Bayar kas kecil UP bendahara dan revolving pengisian kembali kas. |
| **Perbendaharaan**| `DanaKelolaan` | [pebendaharaan/dana-kelolaan.md](./pebendaharaan/dana-kelolaan.md) | Kas masuk bruto rekening penampung sebelum direkonsiliasi. |
| | `Penerimaan` | [pebendaharaan/penerimaan.md](./pebendaharaan/penerimaan.md) | Realisasi penerimaan kas BLU definitif, pemotongan pajak, dan posting jurnal pendapatan. |
| | `Pengeluaran` | [pebendaharaan/pengeluaran.md](./pebendaharaan/pengeluaran.md) | Realisasi kas keluar berbasis SP2D/SPBy, pemotongan pajak, dan posting kas keluar. |
| **Akuntansi** | `Akun` | [akuntansi/akun.md](./akuntansi/akun.md) | Bagan Akun Standar (BAS / COA), akun header/detil, saldo normal, dan saldo awal. |
| | `Jurnal` | [akuntansi/jurnal.md](./akuntansi/jurnal.md) | Jurnal pembukuan berpasangan (*double-entry lines*), jenis jurnal, dan event posting. |
| | `PengaturanAkun` | [akuntansi/pengaturan-akun.md](./akuntansi/pengaturan-akun.md) | Pemetaan akun sistem terpusat (*system ledger mappings*) untuk integrasi otomatis. |
| **Perpajakan** | `PajakEntity` | [perpajakan/pajak.md](./perpajakan/pajak.md) | Konfigurasi tarif perpajakan PPN (`VAT`) dan PPh pemotongan (`WHT`) beserta akun penampung. |

---

### 2. Master & Entitas Bisnis Pendukung

| Modul | Entitas Agregat | Berkas Glosarium | Deskripsi Bisnis Singkat |
| :--- | :--- | :--- | :--- |
| **Cardlist** | `Card` | [cardlist/card.md](./cardlist/card.md) | Master rekanan penyedia pengadaan (`Supplier`) dan pelanggan/mitra layanan (`Customer`). |
| | `Kontrak` | [cardlist/kontrak.md](./cardlist/kontrak.md) | Surat Perjanjian Kerja (SPK), tahapan termin pencairan, dan addendum kontrak. |
| **Pejabat** | `Pejabat` | [pejabat/pejabat.md](./pejabat/pejabat.md) | Master pejabat pengelola keuangan BLU (KPA/Direktur, PPK, PPSPM, Bendahara Pengeluaran). |
| **Referensi** | `StandarMasukan` | [referensi/standar-masukan.md](./referensi/standar-masukan.md) | Standar Biaya Masukan (SBM), katalog baku uraian anggaran, dan standar satuan biaya. |

---

### 3. Infrastruktur, Keamanan, & Layanan Pendukung

| Modul | Entitas Agregat | Berkas Glosarium | Deskripsi Bisnis Singkat |
| :--- | :--- | :--- | :--- |
| **IdentityAccess**| `User` | [identityaccess/user.md](./identityaccess/user.md) | Akun pengguna sistem, sandi BCrypt, asosiasi unit kerja, dan resolusi hak akses efektif. |
| | `Role` | [identityaccess/role.md](./identityaccess/role.md) | Grup peran fungsional dan pengelompokan izin (*permissions bundle*). |
| | `UnitKerja` | [identityaccess/unit-kerja.md](./identityaccess/unit-kerja.md) | Satuan unit kerja struktural/fungsional kampus untuk pembatasan data operasional. |
| **ESignature** | `ESignature` | [esignature/esignature.md](./esignature/esignature.md) | Tanda tangan elektronik kriptografi HMAC-SHA256, snapshot dokumen, dan audit verifikasi. |
| **Storage** | `FileMetadata` | [storage/storage.md](./storage/storage.md) | Penyimpanan dokumen pendukung fisik/digital di MinIO/S3 melalui presigned URL. |
| **Notification** | `Notification` | [notification/notification.md](./notification/notification.md) | Peringatan sistem asinkron kepada pengguna atas status transaksi penting. |
| **Reporting** | `Report` | [reporting/report.md](./reporting/report.md) | Tiket penanganan kendala pengguna, utas diskusi bertingkat (*threads*), dan attachment. |

---

## Standar Format Dokumentasi

Setiap berkas glosarium domain mematuhi struktur standar 5-bagian:
1. **Definisi & Peran Bisnis**: Penjelasan konseptual dan fungsinya dalam alur keuangan kampus Polsri.
2. **Invarian & Aturan Bisnis**: Validasi integritas data, aturan matematis, dan prasyarat alur status.
3. **Kamus Data & Atribut Entitas**: Tabel terperinci atribut model, nama kolom basis data, tipe data, konstrain, dan relasi.
4. **Child Entities & Value Objects**: Tabel komponen turunan, baris rincian (*line items*), atau komponen embedded.
5. **Enumerasi Domain Terkait**: Definisi nilai-nilai konstan pada enum domain.
