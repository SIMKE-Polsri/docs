---
title: User (Pengguna)
draft: false
tags:
  - identityaccess
  - user
  - iam
  - auth
  - permissions
---

# Glosarium Domain: User / Pengguna (IdentityAccess)

Entitas agregat utama dalam modul **IdentityAccess** (v2) yang mengelola akun autentikasi, kredensial sandi terenkripsi BCrypt, asosiasi unit kerja kampus, serta resolusi izin hak akses murni (*Pure IAM Permissions*).

---

## 1. Definisi & Peran Bisnis

- **User (Pengguna Sistem)**: Akun individu aparatur sipil, dosen, tenaga kependidikan, atau pejabat pengelola keuangan yang memiliki hak masuk ke sistem informasi keuangan *simkeu-core*.
- **Kredensial Sesi & BCrypt**: Sandi disimpan menggunakan algoritma hash kuat BCrypt (`password`). Pengguna masuk menggunakan sesi cookie HttpOnly (`JSESSIONID`) yang dilindungi token CSRF ganda (*double-submit cookie*).
- **Resolusi Izin Efektif**: Pengguna memperoleh hak akses operasional melalui penggabungan (*union*) dua sumber:
  1. Izin langsung perorangan (`permissions` di `ia_user_permissions`).
  2. Izin dari grup peran yang diikutinya (`roles.permissions` di `ia_role_permissions`).
  Formula evaluasi:
  $$\text{EffectivePermissions} = \text{User.permissions} \cup \bigcup_{r \in \text{roles}} r\text{.permissions}$$

---

## 2. Invarian & Aturan Bisnis

1. **Keunikan Identitas**: Nama pengguna (`username`) dan alamat surel (`email`) harus unik di seluruh sistem.
2. **Status Aktif**: Pengguna dengan status `enabled = false` ditolak secara otomatis oleh `SimkeuUserDetailsService` saat mencoba login atau melakukan permintaan API (`DisabledException`).
3. **Penyimpanan Sandi Aman**: Sandi mentah (*plain text*) tidak boleh tersimpan di basis data; wajib di-hash menggunakan `PasswordEncoder (BCrypt)`.
4. **Asosiasi Unit Kerja**: Pengguna dapat diasosiasikan dengan satu atau lebih unit kerja operasional (`UnitKerja`) untuk pembatasan lingkup data (*data tenancy / unit scoping*).

---

## 3. Kamus Data & Atribut Entitas

### Entitas Agregat: `User`
- **Tabel Basis Data**: `ia_users`
- **Kelas Model**: `org.ilmi.simkeucore.identityaccess.v2.domain.User`
- **Kelas Induk**: `Auditable<User>`

| Nama Atribut | Kolom Basis Data | Tipe Data | Wajib / Nullable | Relasi / Referensi | Deskripsi Teknis & Bisnis |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `id` | `id` | `UUID` | **Wajib** (PK) | - | Identifikator unik pengguna (UUID v4). |
| `username` | `username` | `String` | **Wajib** (Unique) | - | Nama akun unik untuk login sistem (biasanya NIP atau identitas kedinasan). |
| `email` | `email` | `String` | **Wajib** (Unique) | - | Alamat surat elektronik kedinasan/pribadi resmi pengguna. |
| `firstName` | `first_name` | `String` | **Wajib** | - | Nama depan pengguna. |
| `lastName` | `last_name` | `String` | **Wajib** | - | Nama belakang pengguna. |
| `password` | `password` | `String` | **Wajib** | - | Hash BCrypt dari kata sandi akun pengguna. |
| `enabled` | `enabled` | `Boolean` | **Wajib** | - | Status keaktifan akun (default `true`). Jika `false`, akun terblokir. |
| `emailVerified` | `email_verified` | `Boolean` | **Wajib** | - | Penanda verifikasi email akun pengguna. |
| `unitKerjas` | Tabel: `unit_kerja` | `List<UnitKerja>` | Opsional | `@ManyToMany` | Daftar unit kerja kampus tempat pengguna ditugaskan. |
| `permissions` | Tabel: `ia_user_permissions` | `Set<String>` | Opsional | `@ElementCollection` | Himpunan string izin khusus langsung yang melekat pada pengguna (`module::resource::action`). |
| `roles` | Tabel: `ia_user_roles` | `Set<Role>` | Opsional | `@ManyToMany(Role)` | Daftar grup peran yang disandang pengguna. |

---

## 4. Tabel Asosiasi & Relasi

### Tabel Asosiasi: `ia_user_roles`
- `user_id` $\rightarrow$ `ia_users.id`
- `role_id` $\rightarrow$ `ia_role.id`

### Tabel Asosiasi: `unit_kerja` (Penugasan Anggota)
- `user_id` $\rightarrow$ `ia_users.id`
- `unit_kerja_id` $\rightarrow$ `ia_unit_kerja.id`

### Tabel Koleksi: `ia_user_permissions`
- `user_id` $\rightarrow$ `ia_users.id`
- `permissions` $\rightarrow$ Format kanonikal izin (misal `akuntansi::jurnal::posting`, `perencanaan::item-anggaran::lihat`).
