---
title: Role (Peran)
draft: false
tags:
  - identityaccess
  - role
  - rbac
  - iam
  - permissions
---

# Glosarium Domain: Role / Peran (IdentityAccess)

Entitas dalam modul **IdentityAccess** (v2) yang mengelompokkan sekumpulan izin hak akses (*permissions bundle*) menjadi peran jabatan atau tanggung jawab fungsional dalam pengelolaan keuangan kampus.

---

## 1. Definisi & Peran Bisnis

- **Role (Grup Peran)**: Pengelompokan administratif atas izin-izin operasional (*canonical permissions*) guna mempermudah pemberian hak akses masal kepada staf atau pejabat dengan tanggung jawab serupa (misal peran `BENDAHARA_PENGELUARAN`, `PPK_UNIT`, `OPERATOR_PERENCANAAN`, `VERIFIKATOR_SPP`).
- **Pola Izin Kanonikal**: Izin disimpan dalam format standar hierarkis 3-bagian:
  $$\text{module}::\text{resource}::\text{action}$$
  atau format wildcard penuh (`*`, `akuntansi::*`).

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Nama**: Nama peran (`nama`) harus unik di seluruh sistem (case-sensitive/uppercase disarankan).
2. **Kekekalan Relasi**: Menghapus role akan melepaskan asosiasi pengguna pada tabel `ia_user_roles`, namun tidak menghapus data entitas `User` terkait.
3. **Penyimpanan Izin Eager**: Izin role dimuat bersama entitas role (`FetchType.EAGER`) agar kalkulasi hak otorisasi di `SecurityGuardAspect` berlangsung instan dalam memori $O(1)$.

---

## 3. Kamus Data & Atribut Entitas

### Entitas: `Role`
- **Tabel Basis Data**: `ia_role`
- **Kelas Model**: `org.ilmi.simkeucore.identityaccess.v2.domain.Role`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik data peran (UUID v4). |
| `nama` | `nama` | `String` | **Wajib** (Unique) | - | Nama pengenal peran fungsional (misal `ADMIN`, `BENDAHARA`, `PPK`). |
| `permissions` | Tabel: `ia_role_permissions` | `Set<String>` | Opsional | `@ElementCollection` | Himpunan string izin kanonikal yang dimiliki oleh peran ini. |
| `users` | Tabel: `ia_user_roles` | `Set<User>` | Opsional | `@ManyToMany(mappedBy = "roles")` | Daftar pengguna yang ditugaskan ke peran ini. |

---

## 4. Tabel Koleksi Izin Peran

### Tabel: `ia_role_permissions`
- `role_id` $\rightarrow$ `ia_role.id`
- `permissions` $\rightarrow$ String izin (misal `perealisasian::spp::rekam`, `pebendaharaan::pengeluaran::*`).
