---
title: ItemAnggaran
draft: false
tags:
  - perencanaan
  - item-anggaran
  - pagu
  - detil-anggaran
---

# Glosarium Domain: ItemAnggaran (Perencanaan)

Entitas agregat operasional dalam modul **Perencanaan** yang menyimpan detail alokasi pagu mata anggaran belanja spesifik, rincian biaya (*cost breakdown* / Detil Anggaran), dan status persetujuan usulan DIPA BLU.

---

## 1. Definisi & Peran Bisnis

- **Item Anggaran (Line Item Belanja)**: Baris alokasi mata anggaran belanja definitif yang mengikat sebuah akun BAS belanja (`akunId`, misal `521211` Belanja Bahan) pada suatu Subkomponen (`UnitAnggaran`) untuk satu unit kerja kampus (`unitKerjaId`).
- **Pagu Alokasi**: Total dana belanja yang disetujui:
  $$\text{Pagu} = \text{volume} \times \text{hargaSatuan}$$
- **Detil Anggaran**: Rincian sub-biaya atau rincian komponen penyusun satu item anggaran (misal pembelian kertas, konsumsi rapat, honor penguji) beserta atribut dinamis (*custom fields*).
- **Status Siklus Perencanaan**:
  - `USULAN`: Diajukan oleh unit kerja pelaksana (Fakultas/Jurusan/Biro) dalam tahap perencanaan awal (RKA-K/L).
  - `RESMI`: Telah disetujui oleh Bagian Perencanaan dan disahkan ke dalam DIPA/RKAT definitif kampus.

---

## 2. Invarian & Aturan Bisnis

1. **Akun Belanja Valid**: Atribut `akunId` wajib mereferensikan akun detil aktif pada modul Akuntansi (`isDetil = true`).
2. **Kesesuaian Detil vs Pagu Total**:
   Jika terdapat `detilAnggaran`, total akumulasi nilai detil anggaran belanja harus konsisten dengan nilai pagu `ItemAnggaran`.
3. **Pengendalian Realisasi**: Modul Perealisasian (`Spp` dan `Spby`) memeriksa `ItemAnggaran` ini saat memvalidasi ketersediaan sisa pagu anggaran sebelum menyetujui komitmen tagihan kas.

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `ItemAnggaran`
- **Tabel Basis Data**: `item_anggaran`
- **Kelas Model**: `org.ilmi.simkeucore.perencanaan.domain.ItemAnggaran`
- **Kelas Induk**: `Auditable<ItemAnggaran>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik baris mata anggaran belanja (UUID v4). |
| `kode` | `kode` | `String` | **Wajib** | - | Kode unik baris belanja. |
| `akunId` | `akun_id` | `UUID` | **Wajib** | `Akun.id` (BAS) | Referensi ke akun rekening belanja pada modul Akuntansi. |
| `unitAnggaran` | `unit_anggaran_id` | `UnitAnggaran` | **Wajib** | `@ManyToOne(UnitAnggaran)` | Referensi ke subkomponen induk tempat belanja dialokasikan. |
| `tahunAnggaran` | `tahun_anggaran` | `Integer` | **Wajib** | `Periode.tahunAnggaran` | Tahun fiskal anggaran aktif. |
| `unitKerjaId` | `unit_kerja_id` | `String` | **Wajib** | `UnitKerja.kode` | Kode unit kerja pemilik/pengelola alokasi anggaran. |
| `uraian` | `uraian` | `String` | **Wajib** | - | Deskripsi peruntukan belanja anggaran. |
| `volume` | `volume` | `BigDecimal(19, 4)` | **Wajib** | - | Jumlah kuantitas belanja (misal 100.0000). |
| `satuan` | `satuan` | `String` | **Wajib** | - | Satuan kuantitas (misal "Orang/Bulan", "Rim", "Paket"). |
| `hargaSatuan` | `harga_satuan` | `BigDecimal(19, 2)` | Opsional | - | Harga satuan per unit output (mengacu pada Standar Biaya Masukan / SBM). |
| `status` | `jenis_item_anggaran` | `Enum (Ordinal)` | **Wajib** | `Status` | Status persetujuan perencanaan (`USULAN` / `RESMI`). |
| `detilAnggaran` | Tabel: `detil_anggaran` | `Set<DetilAnggaran>` | Opsional | `@OneToMany` | Daftar rincian komponen sub-biaya penyusun item anggaran. |
| `customFields` | Tabel: `item_anggaran_custom_fields` | `List<CustomField>` | Opsional | `@ElementCollection` | Definisi atribut tambahan dinamis untuk pengelompokan biaya. |

---

## 4. Child Entity: Rincian Detil Anggaran

### Child Entity: `DetilAnggaran`
- **Tabel Basis Data**: `detil_anggaran`
- **Kelas Model**: `org.ilmi.simkeucore.perencanaan.domain.entity.DetilAnggaran`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik detil anggaran. |
| `itemAnggaran` | `item_anggaran_id` | `ItemAnggaran` | **Wajib** | `@ManyToOne(ItemAnggaran)` | Referensi ke item anggaran induk. |
| `unitKerjaId` | `unit_kerja_id` | `String` | **Wajib** | `UnitKerja.kode` | Unit kerja pelaksana spesifik. |
| `uraian` | `uraian` | `String` | **Wajib** | - | Uraian spesifik sub-belanja. |
| `volume` | `volume` | `BigDecimal(19, 4)` | **Wajib** | - | Volume kuantitas rincian. |
| `satuan` | `satuan` | `String` | **Wajib** | - | Satuan ukuran rincian. |
| `hargaSatuan` | `harga_satuan` | `BigDecimal(19, 2)` | Opsional | - | Tarif satuan rincian sub-biaya. |

---

## 5. Enumerasi Domain Terkait

### `Status` (Jenis Item Anggaran)
- `USULAN`: Usulan awal unit kerja sebelum pagu alokasi disetujui.
- `RESMI`: Pagu anggaran definitif DIPA yang telah disahkan dan siap direalisasikan.
