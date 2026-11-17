---
title: Storage & Dokumen Pendukung
draft: false
tags:
  - storage
  - dokumen
  - file-metadata
  - presigned-url
---

# Glosarium Domain: Storage & Dokumen Pendukung (Storage)

Modul layanan infrastruktur **Storage** yang mengelola siklus hidup penyimpanan berkas elektronik, tiket URL pra-otorisasi (*presigned URL*), dan metadata dokumen pendukung transaksi keuangan pada object storage MinIO/S3.

---

## 1. Definisi & Peran Bisnis

- **Dokumen Pendukung (Document)**: Berkas digital asli hasil pemindaian (*scan*) dokumen fisik yang menjadi syarat keabsahan tagihan perbendaharaan (misal: kuitansi bermeterai, invoice rekanan, faktur pajak, Berita Acara Serah Terima / BAST pekerjaan, surat tugas dinas).
- **Presigned URL**: Mekanisme pengunggahan dan pengunduhan berkas aman tanpa melewati beban memori aplikasi monolith secara langsung. Klien SPA meminta URL pra-otorisasi berbatas waktu dari backend, kemudian mengunggah berkas biner langsung ke MinIO object storage.
- **File Metadata**: Informasi teknis dan kepatuhan berkas digital, mencakup nama berkas asli, ukuran file dalam bita, MIME type konten, waktu unggah, dan penanda identitas pengunggah.

---

## 2. Invarian & Aturan Bisnis

1. **Keamanan Jalur Penyimpanan**: Path penyimpanan berkas (`objectName`) menggunakan penamaan berbasis UUID atau pola partisi modul terisolasi guna menghindari benturan nama berkas (*file name collision*) dan penimpaan file secara tidak sengaja.
2. **Kekekalan Dokumen SP2D**: Dokumen pendukung yang telah dikaitkan dengan SP2D yang telah cair (`paid = true`) dilarang dihapus dari object storage untuk memenuhi regulasi kepatuhan audit negara (BPK/BPKP).
3. **Masa Berlaku Presigned URL**: Presigned URL unduh/unggah dibatasi masa aktifnya (default 15 s.d. 60 menit) untuk mencegah kebocoran tautan akses publik.

---

## 3. Kamus Data & Kontrak Model Data

### Model Kontrak: `FileMetadataDto`
- **Kelas Model**: `org.ilmi.simkeucore.storage.api.dto.FileMetadataDto`

| Nama Atribut | Tipe Data | Wajib / Nullable | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- |
| `objectName` | `String` | **Wajib** | Kunci/path unik objek berkas pada bucket penyimpanan MinIO/S3 (misal `spp/2026/uuid-kuitansi.pdf`). |
| `originalFileName`| `String` | **Wajib** | Nama asli berkas saat diunggah oleh pengguna (misal `Kuitansi-Bahan-Praktikum.pdf`). |
| `size` | `Long` | **Wajib** | Ukuran berkas dalam satuan bita (*bytes*). |
| `contentType` | `String` | **Wajib** | Tipe media MIME berkas (misal `application/pdf`, `image/png`, `image/jpeg`). |
| `uploadedAt` | `Instant` | **Wajib** | Waktu presisi berkas berhasil disimpan di storage. |
| `uploadedBy` | `String` | Opsional | NIP / username pengunggah berkas. |

---

## 4. Konvensi Penamaan Dokumen

- Gunakan istilah **Dokumen** untuk rekaman bukti sah akuntansi/pembayaran.
- Gunakan istilah **File** untuk unit teknis biner yang disimpan pada storage server.
- Hindari penamaan sembarangan seperti "Attachment" atau "File Info".
