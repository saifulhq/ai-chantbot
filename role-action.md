# Role dan Hak Akses Data Pegawai

Dokumen ini menjelaskan aturan akses pada chatbot HR di `intents/chatbot_hr_v2.ipynb`. Hak akses ditentukan oleh `role_sistem` dan tingkat organisasi `level_jabatan` pada setiap record di `karyawan.json`, bukan oleh klaim pengguna di percakapan.

## Matriks Akses

| Role | Profil sendiri (`profil_saya`) | Daftar pegawai (`data_pegawai`) |
|---|---|---|
| `pegawai` | Profil diri sendiri | Ditolak; tidak dapat melihat daftar atau data pegawai lain |
| `leader` | NIK, nama, jabatan, dan no HP | Bawahan langsung berdasarkan `atasan_id`; hanya NIK, nama, jabatan, dan no HP |
| `manager` | Profil diri sendiri | Data lengkap pegawai yang `divisi`-nya sama dengan manager |
| `hrd` | Profil diri sendiri | Data lengkap pegawai dengan `level_jabatan` di bawah 3; record manager dan direktur tidak disertakan |
| `direktur` | Profil diri sendiri | Data lengkap semua pegawai |

Data lengkap mencakup field bisnis dalam record, seperti NIK, nama, jabatan, divisi, alamat, no HP, atasan, informasi cuti, dan riwayat absensi. Field internal `role_sistem` dan `level_jabatan` tidak dikirim sebagai data pegawai ke model.

Intent pribadi dipisah: `profil_saya` hanya menampilkan profil penanya (NIK, nama, jabatan, divisi, alamat, dan no HP; leader tetap mengikuti batas kolomnya), `sisa_cuti` menampilkan saldo cuti sendiri, dan `absensi_saya` menampilkan absensi sendiri untuk periode yang diminta. Periode absensi default ke bulan berjalan; pertanyaan dapat menyebut bulan kemarin, nama bulan seperti `September 2026`, atau format `YYYY-MM`. Absensi disimpan per pegawai dalam `absensi_bulanan` dengan status `sakit`, `izin`, `cuti`, dan `alpa`. Jika periode tidak tersedia, jumlah status tersebut ditampilkan sebagai 0.

## Rincian Role

### Pegawai (`pegawai`)

Pegawai dapat meminta profilnya sendiri. Data profil lengkap dikembalikan oleh `profil_saya`. Jika pegawai mencoba menggunakan `data_pegawai` untuk melihat orang lain atau daftar pegawai, permintaan ditolak.

### Leader (`leader`)

Leader hanya dapat melihat bawahan langsung yang memiliki `atasan_id` sama dengan ID leader. Untuk bawahan maupun profil leader sendiri, data dibatasi ke empat field berikut:

- `nik`
- `nama`
- `jabatan`
- `no_hp`

Leader tidak mendapat alamat, divisi, atau informasi cuti melalui tool data pegawai.

### Manager (`manager`)

Manager dapat melihat data lengkap semua pegawai dalam divisinya sendiri. Filter memakai kecocokan nilai `divisi` pada record manager dan pegawai. Manager tidak dapat melihat divisi lain melalui `data_pegawai`.

### HRD (`hrd`)

HRD dapat melihat data lengkap semua pegawai di bawah level manager, tanpa pembatasan divisi. Data manager dan direktur (level 3 ke atas) tidak muncul di daftar HRD. Pada sampel saat ini, Dewi Lestari memiliki role HRD tetapi level jabatan manager, sehingga record-nya tidak muncul di daftar; profil dirinya tetap dapat diakses melalui `profil_saya`. Berikan role ini hanya kepada akun yang memang berwenang mengakses data personalia.

### Direktur (`direktur`)

Direktur dapat melihat data lengkap semua pegawai di seluruh divisi.

## Tingkat Organisasi

`level_jabatan` adalah angka yang dipakai untuk membatasi daftar HRD:

| Nilai | Tingkat |
|---:|---|
| 1 | Pegawai |
| 2 | Leader |
| 3 | Manager |
| 4 | Direktur |

Untuk akun HRD, tool `data_pegawai` hanya menyertakan record dengan `level_jabatan < 3`. Nilai level harus diperbarui konsisten ketika jabatan atau struktur organisasi berubah.

## Penetapan Role dan Identitas

Contoh role pada data demo:

| Nama | Role |
|---|---|
| Rina Wijaya | `direktur` |
| Budi Santoso | `manager` |
| Agus Pratama | `leader` |
| Dewi Lestari | `hrd` |
| Sari Dewi, Andi Kurniawan, Eko Prasetyo | `pegawai` |
| Maya Putri | `manager` |

Setiap record memakai `role_sistem`, contohnya:

```json
{
  "id": 4,
  "nama": "Agus Pratama",
  "jabatan": "Team Leader IT",
  "role_sistem": "leader",
  "divisi": "IT",
  "atasan_id": 2
}
```

Di aplikasi nyata, `uid` harus diambil dari sesi login atau identitas terverifikasi, bukan dari prompt atau jawaban LLM. Jangan izinkan pengguna memilih atau mengubah `uid` maupun `role_sistem` melalui chat. LLM hanya mengklasifikasikan intent; fungsi tool tetap menjadi tempat pemeriksaan hak akses.

## Data Demo

NIK `DEMO-*`, alamat contoh, dan nomor `08XX-XXXX-*` adalah placeholder untuk pengujian. Ganti dengan data yang sesuai sebelum uji integrasi, dan jangan commit data pribadi karyawan sungguhan ke repository.
