---
title: ESignature
draft: false
tags:
  - esignature
  - tanda-tangan-elektronik
  - hmac
  - security
---

# Glosarium Domain: ESignature (Tanda Tangan Elektronik)

Entitas agregat dalam modul **ESignature** yang mengelola penerbitan tanda tangan digital berbasis kriptografi HMAC-SHA256, snapshot metadata dokumen resmi (SPM, SP2D), serta pencatatan audit verifikasi forensik.

---

## 1. Definisi & Peran Bisnis

- **ESignature (Tanda Tangan Elektronik)**: Segel digital kriptografis yang dihasilkan untuk memastikan integritas (*data integrity*) dan keabsahan (*non-repudiation*) dokumen keuangan negara/BLU (seperti Surat Perintah Membayar / SPM dan Surat Perintah Pencairan Dana / SP2D). Tanda tangan elektronik mengunci snapshot data dokumen pada saat penandatanganan oleh pejabat berwenang (PPK, PPSPM, atau Bendahara).
- **Metadata Snapshot & Hash**: Seluruh atribut krusial dokumen (nominal, nama penerima, nomor dokumen, tanggal) diserialisasi menjadi JSON dan di-hash (`metadata_hash`). Jika dokumen dimanipulasi setelah penandatanganan, nilai verifikasi tanda tangan akan otomatis gagal (*invalid*).
- **ESignature Verification**: Riwayat log audit atas setiap upaya verifikasi dokumen, mencatat siapa yang memverifikasi, waktu pengecekan, serta hasil validasi keaslian berkas.

---

## 2. Invarian & Aturan Bisnis

1. **Kekekalan Tanda Tangan (*Immutability*)**: Rekord `ESignature` yang telah diterbitkan tidak dapat diubah nilainya. Setiap perubahan data dokumen sumber memerlukan pembatalan atau penerbitan ulang tanda tangan.
2. **Kesesuaian Algoritma**: Standar algoritma yang digunakan adalah `HMAC-SHA256` dengan kunci rahasia (*secret key*) yang dikelola di tingkat server atau HSM (*Hardware Security Module*).
3. **Penyimpanan Snapshot Lengkap**: Metadata snapshot wajib disimpan utuh dalam format teks/JSON (`metadataSnapshot`) sebagai bukti forensik jika terjadi sengketa hukum atau audit eksternal (BPK/Itjen).

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `ESignature`
- **Tabel Basis Data**: `esignature`
- **Kelas Model**: `org.ilmi.simkeucore.esignature.core.entity.ESignature`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik tanda tangan elektronik (UUID v4). |
| `signature` | `signature` | `String(64)` | **Wajib** | - | String heksadesimal 64 karakter hasil kalkulasi HMAC-SHA256. |
| `algorithm` | `algorithm` | `Enum (String)` | **Wajib** | `ESignatureAlgorithm` | Algoritma kriptografi yang digunakan (misal `HMAC_SHA256`). |
| `documentReference` | `document_reference` | `String(255)` | **Wajib** (Indexed) | - | Nomor dokumen sumber yang ditandatangani (misal `SPM-2026-0012`). |
| `contextType` | `context_type` | `String(50)` | **Wajib** (Indexed) | - | Tipe modul pemilik dokumen (misal `PEREALISASIAN`, `PEBENDAHARAAN`). |
| `metadataHash` | `metadata_hash` | `String(64)` | **Wajib** (Indexed) | - | Hash SHA-256 dari metadata payload untuk pembandingan cepat. |
| `metadataSnapshot` | `metadata_snapshot` | `LongText` | **Wajib** | - | Snapshot JSON lengkap dari isi dokumen saat ditandatangani. |
| `createdBy` | `created_by` | `String` | Opsional | `User.username` | NIP / username pejabat penandatangan. |
| `createdDate` | `created_date` | `Instant` | Opsional (Indexed) | - | Waktu presisi tanda tangan dibubuhkan. |

---

## 4. Child Entity: Riwayat Verifikasi

### Child Entity: `ESignatureVerification`
- **Tabel Basis Data**: `esignature_verification`
- **Kelas Model**: `org.ilmi.simkeucore.esignature.core.entity.ESignatureVerification`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik audit verifikasi. |
| `signature` | `signature_id` | `ESignature` | **Wajib** | `@ManyToOne(ESignature)` | Referensi ke entitas tanda tangan yang diuji keabsahannya. |
| `isValid` | `is_valid` | `Boolean` | **Wajib** | - | Hasil verifikasi: `true` jika tanda tangan cocok dan dokumen asli; `false` jika data telah termanipulasi. |
| `verifiedAt` | `verified_at` | `Instant` | **Wajib** (Indexed) | - | Waktu kapan verifikasi dilakukan. |
| `verifiedBy` | `verified_by` | `String` | Opsional (Indexed) | `User.username` | Username/sistem yang meminta pengujian keabsahan dokumen. |
| `verificationMethod` | `verification_method` | `String` | Opsional | - | Metode verifikasi yang dijalankan (misal `MANUAL_CHECK`, `API_AUTO`). |
| `notes` | `notes` | `String` | Opsional | - | Catatan hasil verifikasi atau keterangan ketidaksesuaian (*mismatch*). |

---

## 5. Enumerasi Domain Terkait

### `ESignatureAlgorithm`
- `HMAC_SHA256`: Hash-based Message Authentication Code menggunakan fungsi hash SHA-256.
