---
name: changelog-document
description: Generate a structured Changelog Document from code changes, Git diffs, commits, or task descriptions, including metadata, summary, before/after comparison, category, and technical details. Use when the user asks to create, document, or explain a changelog document for a change.
---

# Panduan: Auto-Generate Changelog Document dari Analisa Perubahan (Format Markdown)

Panduan ini merangkum konvensi template **Changelog Document** (Samir) DAN cara menganalisa perubahan (code diff, git log, atau deskripsi task) untuk otomatis mengisi template tersebut — tanpa perlu membuat Claude Skill penuh, cukup diikuti sebagai instruksi saat diminta membuat changelog.

---

## Alur Kerja (Wajib Diikuti Berurutan)

1. **Kumpulkan sumber perubahan.** Cari salah satu dari:
   - `git diff` / `git log -p` pada branch atau commit terkait
   - File yang diubah user (upload, atau path yang disebutkan)
   - Deskripsi task/tiket dari user (jika tidak ada akses ke kode)
   Jika tidak ada satupun tersedia, minta user memberikan salah satunya — jangan mengarang isi.

2. **Analisa perubahan** untuk menyimpulkan:
   - **Apa yang berubah** — file, komponen, fungsi, UI, config yang tersentuh
   - **Kondisi sebelum vs sesudah** — perilaku/tampilan/logic lama vs baru (isi `Before`/`After`)
   - **Kategori perubahan** — tentukan otomatis dari sifat perubahan:
     - Penambahan fitur/komponen baru → `New Feature`
     - Perbaikan bug/error → `Bug Fix`
     - Perubahan kecil tanpa fitur baru (styling, performa, cleanup) → `Improvement`
     - Perubahan struktur kode tanpa ubah perilaku → `Refactor`
     - Perbaikan darurat di production → `Hotfix`
     - Update dependency/config/non-fungsional → `Chore`
   - **Dampak teknis** — modul lain yang terpengaruh, breaking change, perlu migrasi, dsb → masuk ke `Details`

3. **Susun draft field-field template** (lihat struktur di bawah) berdasarkan hasil analisa, bukan menunggu user mengisi manual.

4. **Isi metadata yang tidak bisa disimpulkan dari kode** (Project Name, PIC/Role, Date, Task Reference) dari konteks percakapan/profil user; kalau tidak ada, tanyakan singkat satu kali, jangan mengosongkan begitu saja.

5. **Generate file `.md` final** mengikuti template, lalu tampilkan/simpan sebagai output.

---

## Struktur Dokumen

Setiap Changelog Document terdiri dari 3 bagian utama:

1. **Header Info** — tabel metadata perubahan
2. **Summary** — ringkasan singkat perubahan (title, deskripsi, before/after, category)
3. **Details** — penjelasan lengkap/teknis dari perubahan

---

## Template Markdown

```markdown
# Changelog Document

| Field           | Value              |
|-----------------|--------------------|
| Project Name    | {{project_name}}   |
| Change Version  | {{change_version}} |
| Date            | {{date}}           |
| PIC / Role      | {{pic_name}} / {{role}} |
| Task Reference  | {{task_reference}} |

## Summary

**Title**: {{title}}

**Brief Description**: {{brief_description}}

| Before        | After         |
|----------------|---------------|
| {{before}}     | {{after}}     |

**Category**: {{category}}

## Details

{{details}}
```

---

## Keterangan Tiap Field

| Field | Keterangan |
|---|---|
| `Project Name` | Nama project/repo yang berubah (contoh: `domain-m-h5`) |
| `Change Version` | Versi rilis/tag jika ada (boleh kosong) |
| `Date` | Tanggal perubahan, format `YYYY-MM-DD` |
| `PIC / Role` | Nama pembuat perubahan diikuti role, format `Nama / Role` |
| `Task Reference` | Link/ID tiket (Jira, Trello, dll), boleh kosong |
| `Title` | Judul singkat perubahan, 1 baris |
| `Brief Description` | 1–2 kalimat ringkasan apa yang berubah dan kenapa |
| `Before` / `After` | Kondisi sebelum vs sesudah perubahan (boleh berupa poin singkat atau screenshot link) |
| `Category` | Salah satu dari: `New Feature`, `Bug Fix`, `Improvement`, `Refactor`, `Hotfix`, `Chore` |
| `Details` | Penjelasan teknis lengkap: file/komponen yang diubah, alasan, dampak, cara testing, dsb. Bisa pakai sub-heading atau bullet list |

---

## Cara Memicu (Trigger) Auto-Generate

User cukup memberi salah satu dari ini, tanpa perlu mengisi field manual:

```
Buatkan changelog document untuk perubahan di [nama branch/commit/file],
project: domain-m-h5, PIC: Andika Satrio Nugroho / Frontend
```

atau

```
Ini diff-nya: [paste git diff / tempel kode sebelum-sesudah]
Buatkan changelog document-nya.
```

atau (upload file yang diubah langsung)

```
[upload file lama & file baru]
Buatkan changelog document dari perubahan ini.
```

Claude lalu menjalankan **Alur Kerja** di atas: membaca diff/file, menyimpulkan Title, Brief Description, Before/After, Category, dan Details secara otomatis, lalu mengisi template dan mengembalikan file `.md` siap pakai — user hanya perlu konfirmasi/koreksi kecil bila ada yang meleset.

---

## Catatan Kualitas Analisa

- **Before/After** harus konkret (contoh: "Button submit disabled saat form kosong" bukan "ada perbaikan validasi").
- **Details** sebaiknya mencantumkan: file/komponen yang diubah, root cause (jika bug fix), cara testing/verifikasi, dan potensi dampak ke modul lain.
- Jika perubahan mencakup beberapa hal berbeda sekaligus, pertimbangkan membuat beberapa changelog document terpisah per perubahan, bukan digabung dalam satu Title.
