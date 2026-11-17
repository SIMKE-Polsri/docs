---
title: StandarMasukan
draft: false
tags:
  - referensi
  - standar-masukan
  - sbm
  - satuan
---

# Glosarium Domain: StandarMasukan (Referensi)

Entitas master referensi dalam modul **Referensi** yang mengelola data standar baku masukan anggaran, Standar Biaya Masukan (SBM), standar nomenklatur belanja, dan satuan baku untuk penyusunan DIPA dan RKA-K/L kampus.

---

## 1. Definisi & Peran Bisnis

- **Standar Masukan (Standard Input / SBM)**: Katalog baku referensi masukan yang ditetapkan oleh Kementerian Keuangan atau Peraturan Direktur BLU untuk memastikan keseragaman nomenklatur dan satuan dalam penyusunan perencanaan anggaran.
- **Tipe Standar Masukan (`TipeStandarMasukan`)**:
  - `ITEM_ANGGARAN`: Nomenklatur baku uraian belanja pada level item anggaran (misal: "Honorarium Narasumber/Pakar Luar Kampus").
  - `DETIL_ANGGARAN`: Nomenklatur baku untuk rincian komponen biaya pada detil anggaran (misal: "Konsumsi Rapat Biasa Snack/Makan").
  - `SATUAN`: Standar nama satuan ukuran volume (misal: "Orang/Hari", "Paket", "Kegiatan", "Bulan", "Rim").

---

## 2. Invarian & Aturan Bisnis

1. **Keterikatan Temporal**: Data standar masukan terikat pada tahun anggaran (`tahunAnggaran`) tertentu dan dapat diperbarui setiap tahun fiskal sesuai PMK Standar Biaya Masukan terbaru.
2. **Kesesuaian Penggunaan**: Modul Perencanaan menggunakan master referensi ini sebagai opsi pemilih otomatis (*auto-complete / dropdown reference*) saat unit kerja menyusun usulan anggaran.

---

## 3. Kamus Data & Atribut Entitas

### Entitas: `StandarMasukan`
- **Tabel Basis Data**: `standar_masukan`
- **Kelas Model**: `org.ilmi.simkeucore.referensi.core.entity.StandarMasukan`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data standar masukan (UUID v4). |
| `tipe` | `tipe` | `Enum (Ordinal)` | **Wajib** | `TipeStandarMasukan` | Klasifikasi masukan (`ITEM_ANGGARAN`, `DETIL_ANGGARAN`, `SATUAN`). |
| `masukan` | `masukan` | `String` | **Wajib** | - | Teks nama standar biaya atau satuan baku. |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran berlakunya standar masukan ini. |

---

## 4. Enumerasi Domain Terkait

### `TipeStandarMasukan`
- `ITEM_ANGGARAN`: Referensi teks baku untuk baris Item Anggaran.
- `DETIL_ANGGARAN`: Referensi teks baku untuk rincian Detil Anggaran.
- `SATUAN`: Referensi teks baku untuk satuan ukuran volume.
