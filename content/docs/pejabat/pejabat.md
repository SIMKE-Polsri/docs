---
title: Pejabat
draft: false
tags:
  - pejabat
  - kpa
  - ppk
  - ppspm
  - bendahara
---

# Glosarium Domain: Pejabat (Struktural BLU)

Entitas master data dalam modul **Pejabat** yang menjadi sumber kebenaran (*single source of truth*) data identitas aparatur pejabat pengelola keuangan negara dan pimpinan struktural Badan Layanan Umum Politeknik Negeri Sriwijaya.

---

## 1. Definisi & Peran Bisnis

- **Pejabat Pengelola Keuangan**: Aparatur sipil negara yang ditetapkan melalui Surat Keputusan (SK) Kuasa Pengguna Anggaran (KPA) / Direktur untuk menjalankan kewenangan otorisasi perbendaharaan dan pencairan anggaran.
- **Jabatan Struktural & Pengelola Keuangan**:
  - `DIREKTUR`: Kuasa Pengguna Anggaran (KPA) tertinggi BLU.
  - `WADIR_I`, `WADIR_II`, `WADIR_III`, `WADIR_IV`: Wakil Direktur bidang Akademik, Umum & Keuangan, Kemahasiswaan, dan Kerja Sama.
  - `PPK`: Pejabat Pembuat Komitmen yang berwenang menerbitkan SPP dan menandatangani Kontrak/SPK.
  - `PPSPM`: Pejabat Penandatangan Surat Perintah Membayar yang menguji keabsahan SPP dan menerbitkan SPM.
  - `BENDAHARA_PENGELUARAN`: Pejabat fungsional perbendaharaan pemegang Uang Persediaan (UP) dan pelaksana pengeluaran kas.

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan NIP**: Nomor Induk Pegawai (`nip`) wajib valid dan mewakili satu pejabat nyata.
2. **Keterkaitan Dokumen Resmi**: Setiap dokumen pencairan anggaran (`Spp`, `Spm`, `Sp2d`, `Spby`) wajib mencantumkan referensi nama lengkap dan NIP pejabat penandatangan yang aktif.

---

## 3. Kamus Data & Atribut Entitas

### Entitas: `Pejabat`
- **Tabel Basis Data**: `pejabat`
- **Kelas Model**: `org.ilmi.simkeucore.pejabat.domain.Pejabat`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `spmId` | `UUID` | **Wajib** (PK) | - | Identifikator unik data pejabat (UUID v4). |
| `fullName` | `full_name` | `String` | **Wajib** | - | Nama lengkap pejabat beserta gelar akademik (misal `Dr. Ir. H. Ahmad Fauzi, M.T.`). |
| `nip` | `nip` | `String` | **Wajib** | - | Nomor Induk Pegawai resmi 18 digit (NIP ASN/PNS). |
| `jabatan` | `jabatan` | `Enum (String/Ordinal)` | **Wajib** | `Jabatan` | Posisi jabatan struktural atau fungsi perbendaharaan yang diemban. |

---

## 4. Enumerasi Domain Terkait

### `Jabatan`
- `DIREKTUR`: Kuasa Pengguna Anggaran (KPA).
- `WADIR_I`: Wakil Direktur Bidang Akademik.
- `WADIR_II`: Wakil Direktur Bidang Umum dan Keuangan.
- `WADIR_III`: Wakil Direktur Bidang Kemahasiswaan.
- `WADIR_IV`: Wakil Direktur Bidang Kerjasama.
- `BENDAHARA_PENGELUARAN`: Bendahara Pengeluaran BLU.
- `PPSPM`: Pejabat Penandatangan Surat Perintah Membayar.
- `PPK`: Pejabat Pembuat Komitmen.
