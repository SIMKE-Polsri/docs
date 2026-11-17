---
title: UnitAnggaran
draft: false
tags:
  - perencanaan
  - unit-anggaran
  - dipa
  - hierarki-anggaran
---

# Glosarium Domain: UnitAnggaran (Perencanaan)

Entitas agregat dalam modul **Perencanaan** yang menyusun struktur hierarki pohon anggaran DIPA (Daftar Isian Pelaksanaan Anggaran) kampus BLU, mencakup tingkatan Program, Kegiatan, KRO, RO, Komponen, hingga Subkomponen.

---

## 1. Definisi & Peran Bisnis

- **Unit Anggaran (Budget Hierarchy Node)**: Simpul (*node*) dalam hierarki berjenjang klasifikasi anggaran belanja pemerintah. Disusun menggunakan relasi rekursif diri sendiri (*self-referencing parent-child*):
  - **Program**: Kebijakan makro kementerian/lembaga (misal Program Pendidikan Tinggi Vokasi).
  - **Kegiatan**: Aktivitas operasional terukur (misal Pembelajaran dan Kemahasiswaan).
  - **KRO (Klasifikasi Rincian Output)**: Kelompok luaran riil.
  - **RO (Rincian Output)**: Luaran spesifik layanan kampus.
  - **Komponen**: Bagian dari tahapan pencapaian output.
  - **Subkomponen**: Level terendah wadah kegiatan spesifik unit kerja yang menaungi baris-baris mata anggaran belanja (`ItemAnggaran`).

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Kombinasi Simpul**: Kombinasi tahun anggaran, ID induk, dan kode (`tahun_anggaran`, `parent_id`, `kode`) harus unik.
2. **Kalkulasi Total Pagu Hierarkis**:
   $$\text{Pagu Node Induk} = \sum \text{Pagu Children} + \sum \text{Pagu ItemAnggaran}$$
3. **Penyematan Item Anggaran**: `ItemAnggaran` hanya dapat disematkan langsung pada Unit Anggaran tingkat daun (*leaf node* / Subkomponen).

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `UnitAnggaran`
- **Tabel Basis Data**: `unit_anggaran`
- **Kelas Model**: `org.ilmi.simkeucore.perencanaan.domain.UnitAnggaran`
- **Kelas Induk**: `Auditable<UnitAnggaran>`
- **Konstrain Unik**: `tahun_anggaran`, `parent_id`, `kode`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik simpul unit anggaran (UUID v4). |
| `kode` | `kode` | `String` | **Wajib** | - | Kode klasifikasi anggaran (misal `068`, `A`, `001`). |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal berlakunya hierarki DIPA. |
| `uraian` | `uraian` | `String` | **Wajib** | - | Nama atau penjelasan nomenklatur mata anggaran. |
| `volume` | `volume` | `Integer` | Opsional | - | Target volume fisik output kegiatan (jika ada). |
| `satuan` | `satuan` | `String` | Opsional | - | Satuan pengukuran output (misal "Laporan", "Mahasiswa", "Paket"). |
| `parent` | `parent_id` | `UnitAnggaran` | Opsional | `@ManyToOne(UnitAnggaran)` | Referensi ke simpul induk di atasnya (null jika Program level teratas). |
| `children` | - | `Set<UnitAnggaran>` | Opsional | `@OneToMany(mappedBy="parent")` | Daftar simpul turunan hierarki di bawahnya. |
| `itemAnggaran` | - | `Set<ItemAnggaran>` | Opsional | `@OneToMany(mappedBy="unitAnggaran")` | Daftar alokasi mata anggaran belanja pada subkomponen ini. |
