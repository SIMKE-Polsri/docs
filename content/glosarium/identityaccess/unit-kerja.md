---
title: UnitKerja
draft: false
tags:
  - identityaccess
  - unit-kerja
  - organisasi
  - struktur
---

# Glosarium Domain: UnitKerja (IdentityAccess)

Entitas dalam modul **IdentityAccess** (v2) yang merepresentasikan struktur organisasi kampus Politeknik Negeri Sriwijaya (Jurusan, Program Studi, Biro, Bagian, Unit Pelaksana Teknis / UPT, dan Lembaga) untuk alokasi wewenang dan pembatasan data operasional.

---

## 1. Definisi & Peran Bisnis

- **Unit Kerja**: Satuan organisasi kerja struktural atau fungsional di lingkungan kampus yang bertanggung jawab atas pelaksanaan program kerja, pengajuan perencanaan anggaran (`UnitAnggaran`), pengikatan kontrak pengadaan, dan permohonan pembayaran belanja (`Spp`).
- **Keanggotaan Pengguna**: Pengguna (`User`) dapat dimasukkan sebagai anggota (`members`) dari satu atau lebih unit kerja untuk membatasi hak akses data transaksi keuangan hanya pada lingkup unit kerjanya (*unit data-scoping*).
- **Status Tutup (*Closed*)**: Unit kerja yang telah dilikuidasi atau digabungkan (*merger*) dapat ditandai dengan `closed = true` sehingga tidak dapat menerima alokasi anggaran atau transaksi belanja baru.

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Kode Unit**: Kode unit (`kode`) harus unik di seluruh sistem (misal `JTI` untuk Jurusan Teknologi Informasi, `BAUK` untuk Biro Administrasi Umum dan Keuangan).
2. **Keterikatan Pengguna**: Pelepasan penugasan anggota dari unit kerja (`unassign`) tidak menghapus akun pengguna, hanya memutuskan asosiasi pada tabel `unit_kerja`.

---

## 3. Kamus Data & Atribut Entitas

### Entitas: `UnitKerja`
- **Tabel Basis Data**: `ia_unit_kerja`
- **Kelas Model**: `org.ilmi.simkeucore.identityaccess.v2.domain.UnitKerja`
- **Kelas Induk**: `Auditable<UnitKerja>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data unit kerja (UUID v4). |
| `kode` | `kode` | `String` | **Wajib** (Unique) | - | Kode unik satuan kerja (misal `JTI`, `TE`, `TM`, `BAUK`). |
| `nama` | `nama` | `String` | **Wajib** | - | Nama lengkap resmi unit organisasi (misal "Jurusan Teknik Informatika"). |
| `closed` | `closed` | `Boolean` | **Wajib** | - | Indikator apakah unit kerja masih aktif beroperasi (`false`) atau ditutup (`true`). |
| `members` | Tabel: `unit_kerja` | `List<User>` | Opsional | `@ManyToMany(mappedBy = "unitKerjas")` | Daftar pengguna/staf yang menjadi anggota unit kerja ini. |
