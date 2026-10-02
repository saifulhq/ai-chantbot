# Catatan Intent Chatbot HR

Catatan untuk melanjutkan pekerjaan di `intents/chatbot_hr_v2.ipynb`.

## Lokasi Penting

- Cell 3 mendefinisikan daftar intent dan `PROMPT_ROUTER`.
- Cell 4 menjalankan tools, pemeriksaan akses, ekstraksi parameter, dan pagination.
- Cell 5 berisi simulasi beberapa role.
- Cell 6 menguji klasifikasi intent menggunakan LLM.
- Cell 7 adalah uji manual; saat ini UID 5 dan query pencarian Sari halaman 2.
- Data demo berada di `karyawan.json`.

## Intent Saat Ini

| Intent | Kegunaan |
|---|---|
| `profil_saya` | Profil penanya sendiri saja |
| `sisa_cuti` | Saldo cuti penanya |
| `absensi_saya` | Absensi penanya; default bulan berjalan, mendukung bulan relatif, nama bulan, dan `YYYY-MM` |
| `struktur_saya` | Atasan, rekan, atau bawahan |
| `data_pegawai` | Cari atau daftar pegawai sesuai hak akses |
| `berita_kantor` | Berita dan pengumuman kantor |
| `rujuk_hr` | Gaji atau bagian restricted yang tidak boleh diberikan |
| `lainnya` | Sapaan dan obrolan umum |

`intent_dari_ai(q)` tetap mengembalikan daftar nama intent. `proses(q, uid)` membuat respons intent berparameter untuk tool pegawai dan menampilkannya saat debug, contohnya:

```python
{
    "intents": ["data_pegawai"],
    "parameters": {
        "data_pegawai": {
            "nama": ["Sari Dewi"],
            "page": 2,
            "page_size": 10,
            "semua": False
        }
    }
}
```

## Hak Akses Pegawai

Pemeriksaan akses dilakukan oleh `data_pegawai(uid)` sebelum hasil difilter atau dipaginasi:

- `hrd`: record dengan `level_jabatan < 3`.
- `direktur`: semua record.
- `manager`: record satu divisi.
- `leader`: bawahan langsung, dengan field terbatas.
- `pegawai`: permintaan daftar pegawai ditolak.

Filter nama dan pagination hanya diterapkan pada daftar yang sudah lolos pemeriksaan role.

## Pagination Pencarian

`parameter_data_pegawai(q)` mengambil nama yang disebut, halaman, dan ukuran halaman. Default-nya halaman 1 dengan 10 hasil; `page_size` dibatasi maksimum 100. `saring_data_pegawai(data, parameter)` menghasilkan:

- `nama`, `page`, `page_size`, `total`
- `has_next`, `next_page`
- `has_previous`, `previous_page`
- `results`

Untuk API nyata, ganti pemotongan list lokal dengan query server-side. Terapkan scope akses sebelum filter/pagination dan jangan mengirim seluruh daftar ke LLM. Jika nama ambigu, minta nama lengkap atau identitas pembeda sebelum mengirim detail.

## Kondisi dan Hasil Terakhir

- UID 5 (Dewi Lestari) ber-role `hrd`; daftar HRD berisi empat pegawai level di bawah manager.
- Pencarian Sari pada data demo menemukan satu record. Query page 2 menghasilkan `total: 1`, `results: []`, `has_previous: true`, dan `next_page: null`. Ini hasil yang benar untuk sampel saat ini.
- Uji sintetis dengan 25 record bernama Sari menghasilkan 10 record per halaman dan `next_page` berpindah dari 2 ke 3.
- Query Cell 7 saat ini: `cari informasi tentang Sari page 2 dan berikan ringkasannya`.
- Sebelumnya Cell 6 lulus 26/26 untuk klasifikasi intent. Jalankan ulang setelah ada perubahan pada router.

## Lanjutan yang Disarankan

1. Tambahkan uji regresi pagination ke notebook: halaman pertama, halaman kedua, halaman terakhir, nama tidak ditemukan, serta nama ambigu.
2. Pastikan frasa `semua data pegawai` tidak tertukar dengan `semua informasi tentang Sari`.
3. Saat mengganti JSON dengan API, gunakan cursor dari API jika tersedia; jangan mengandalkan offset untuk dataset besar yang berubah saat paginasi.
4. Pertahankan pengecekan role di server/tool. Jangan mengandalkan classifier LLM sebagai otorisasi.
