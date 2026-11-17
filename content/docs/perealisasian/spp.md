---
title: SPP, SPM, & SP2D
draft: false
tags:
  - perealisasian
  - spp
  - spm
  - sp2d
  - belanja
---

# Glosarium Domain: SPP, SPM, & SP2D (Perealisasian)

Rantai agregat utama dalam modul **Perealisasian** yang mengelola siklus hidup permintaan pembayaran belanja negara/BLU mulai dari Surat Permintaan Pembayaran (SPP), Pengujian Verifikasi Dokumen, Surat Perintah Membayar (SPM), hingga Surat Perintah Pencairan Dana (SP2D).

---

## 1. Definisi & Peran Bisnis

- **Surat Permintaan Pembayaran (SPP)**: Dokumen tagihan resmi yang diterbitkan oleh Pejabat Pembuat Komitmen (PPK) kepada Pejabat Penandatangan SPM (PPSPM) untuk meminta pencairan dana atas beban anggaran BLU.
- **Tipe SPP (`SppType`)**:
  - `UP` (Uang Persediaan): Uang muka kerja awal tahun anggaran untuk membiayai operasional sehari-hari bendahara pengeluaran.
  - `TUP` (Tambahan Uang Persediaan): Tambahan uang muka saat kebutuhan mendesak melampaui sisa UP.
  - `GUP` (Ganti Uang Persediaan): Penggantian dana UP yang telah terpakai melalui revolving SPBy.
  - `LS` (Langsung): Pembayaran belanja langsung ke rekening pihak ketiga (honorarium, sewa non-kontrak).
  - `LS_KONTRAKTUAL`: Pembayaran langsung berdasarkan termin kontrak pengadaan/SPK pada pihak ketiga.
- **Verifikasi SPP**: Proses pengujian administratif, substantif, dan kelayakan anggaran oleh staf verifikator sebelum SPM disetujui.
- **Surat Perintah Membayar (SPM)**: Perintah pencairan dana yang ditandatangani oleh PPSPM kepada Bendahara Pengeluaran / Kuasa BUN.
- **Surat Perintah Pencairan Dana (SP2D)**: Perintah resmi pencairan dan pemindahbukuan kas dari rekening bank kampus kepada penerima hak.

---

## 2. Invarian & Aturan Bisnis

1. **Alur Transisi Status Linier**:
   $$\text{DRAFT} \longrightarrow \text{VERIFIED} \longrightarrow \text{SPM\_ISSUED} \longrightarrow \text{SP2D\_ISSUED} \longrightarrow \text{PAID}$$
2. **Keterikatan Anggaran & Pagu**: Total nilai realisasi anggaran belanja (`realisasi`) tidak boleh melebihi sisa pagu anggaran pada `ItemAnggaran`.
3. **Klausul Khusus LS-Kontraktual**: SPP tipe `LS_KONTRAKTUAL` wajib mereferensikan kontrak dan termin yang valid (`kontrakId` dan `terminId`).
4. **Penerbitan Event Kas**: Saat SP2D dicairkan (`paid = true`), sistem menerbitkan `Sp2dDiterbitkanEvent` dan `SppPaidEvent` untuk eksekusi kas keluar di modul Perbendaharaan.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat Root: `Spp`
- **Tabel Basis Data**: `spp`
- **Kelas Model**: `org.ilmi.simkeucore.perealisasian.v2.domain.Spp`
- **Kelas Induk**: `Auditable<Spp>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik berkas SPP (UUID v4). |
| `nomor` | `nomor` | `String` | **Wajib** (Unique) | - | Nomor dokumen resmi SPP (misal `SPP-001/LS/JTI/2026`). |
| `uraian` | `uraian` | `String` | **Wajib** | - | Uraian maksud dan keperluan pembayaran. |
| `type` | `type` | `Enum (String)` | **Wajib** | `SppType` | Jenis SPP (`UP`, `TUP`, `GUP`, `LS`, `LS_KONTRAKTUAL`). |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran pelaksanaan. |
| `status` | `status` | `Enum (String)` | **Wajib** | `SppStatus` | Status verifikasi dan proses (`DRAFT`, `VERIFIED`, `REJECTED`, `SPM_ISSUED`, `SP2D_ISSUED`). |
| `paid` | `paid` | `Boolean` | **Wajib** | - | Penanda apakah dana SPP sudah cair ke rekening penerima (`true`/`false`). |
| `spm` | `spm_id` | `Spm` | Opsional | `@OneToOne(Spm)` | Entitas dokumen SPM yang diterbitkan atas SPP ini. |
| `sp2d` | `sp2d_id` | `Sp2d` | Opsional | `@OneToOne(Sp2d)` | Entitas dokumen SP2D yang diterbitkan atas SPM ini. |
| `verifikasi` | Tabel: `verifikasi_spp`| `List<VerifikasiSpp>` | Opsional | `@OneToMany` | Daftar riwayat proses verifikasi kelayakan dokumen SPP. |
| `realisasi` | Tabel: `spp_realisasi` | `List<RealisasiAnggaran>` | Opsional | `@ElementCollection` | Rincian pembebanan anggaran belanja pada Item Anggaran (untuk LS). |
| `revolves` | Tabel: `spp_revolves` | `List<SpbyRevolve>` | Opsional | `@ElementCollection` | Daftar SPBy yang diganti (khusus tipe GUP). |
| `pajak` | Tabel: `spp_pajak` | `List<PotonganPajak>` | Opsional | `@ElementCollection` | Daftar rincian pemotongan PPN / PPh transaksi. |
| `potongan` | Tabel: `spp_potongan` | `List<Potongan>` | Opsional | `@ElementCollection` | Daftar potongan selain pajak (misal denda keterlambatan). |
| `dokumen` | Tabel: `spp_dokumen_pendukung` | `List<DokumenPendukung>` | Opsional | `@ElementCollection` | Daftar berkas bukti fisik tagihan di modul Storage (kuitansi, invoice, BAST). |
| `kontrak` | `lsk_*` | `KontrakDetail` | Opsional | `@Embedded` | Detail ID kontrak dan termin (khusus tipe `LS_KONTRAKTUAL`). |

---

## 4. Child Entities Terkait

### Child Entity: `Spm` (Surat Perintah Membayar)
- **Tabel Basis Data**: `spm`
- **Kelas Model**: `org.ilmi.simkeucore.perealisasian.v2.domain.entity.Spm`
- **Kolom Kunci**: `id` (UUID), `nomor` (String), `tanggal` (LocalDate), `tertanda` (NIP/Nama PPSPM), `eSignatureId` (UUID ref ke modul ESignature).

### Child Entity: `Sp2d` (Surat Perintah Pencairan Dana)
- **Tabel Basis Data**: `sp2d`
- **Kelas Model**: `org.ilmi.simkeucore.perealisasian.v2.domain.entity.Sp2d`
- **Kolom Kunci**: `id` (UUID), `nomor` (String), `tanggal` (LocalDate), `nomorTransaksiBank` (String), `tertanda` (Kuasa BUD), `paidAt` (Instant).

### Child Entity: `VerifikasiSpp`
- **Tabel Basis Data**: `verifikasi_spp`
- **Kelas Model**: `org.ilmi.simkeucore.perealisasian.v2.domain.entity.VerifikasiSpp`
- **Kolom Kunci**: `id` (UUID), `status` (SETUJU / TOLAK), `catatan` (String), `verifikator` (Nama/NIP), `poinPenilaianList` (Tabel: `poin_penilaian`).

---

## 5. Enumerasi Domain Terkait

### `SppType`
- `UP`: Uang Persediaan awal tahun.
- `TUP`: Tambahan Uang Persediaan operasional.
- `GUP`: Ganti Uang Persediaan atas realisasi SPBy.
- `LS`: Pembayaran Langsung operasional / honorarium.
- `LS_KONTRAKTUAL`: Pembayaran Langsung pengadaan pihak ketiga via termin kontrak.

### `SppStatus`
- `DRAFT`: SPP baru disusun PPK.
- `SUBMITTED`: Diajukan ke verifikator.
- `VERIFIED`: Lolos verifikasi kepatuhan.
- `REJECTED`: Ditolak verifikator dengan catatan perbaikan.
- `SPM_ISSUED`: Telah diterbitkan SPM oleh PPSPM.
- `SP2D_ISSUED`: Telah diterbitkan SP2D oleh Kuasa BUD.
- `PAID`: Dana telah ditransfer/dicairkan.
