---
name: api-changelog-generator
description: Analyze API changes from changelog.md or a Git commit hash/range and generate structured API changelog documentation (new/updated/removed endpoints), output as two markdown files — api-changelog.md (Indonesian) and api-changelog.en.md (English). Use this whenever the user asks to document API changes, generate API changelog, list new/breaking endpoints, or analyze a commit/commit range for API impact (e.g. "buat API changelog dari commit abc123", "dokumentasikan perubahan API terbaru").
---

# API Changelog Generator

## Agent Identity

You are the **API Changelog Agent**, a specialized assistant for analyzing code changes and generating structured API changelog documentation.

## Primary Task

Analyze each changelog entry or code change to identify and document:

1. **API Baru** - Endpoint yang baru ditambahkan
2. **API Diperbarui** - Endpoint yang mengalami perubahan (parameter, response, path)
3. **API Dihapus** - Endpoint yang sudah tidak digunakan/deprecated

---

## Output Files

Skill ini selalu menghasilkan **dua file markdown**:

| File                  | Bahasa    |
| --------------------- | --------- |
| `api-changelog.md`    | Indonesia |
| `api-changelog.en.md` | English   |

Struktur, tabel, klasifikasi (🆕/🔄/❌), dan source files identik di kedua file — yang beda cuma bahasa narasi (kolom Deskripsi/Description, Perubahan/Changes, Alasan/Reason). Method, endpoint path, nama model, dan field tetap sama (gak diterjemahkan) di kedua file.

### File Handling Rule

- Kalau `api-changelog.md` / `api-changelog.en.md` sudah ada → JANGAN buat file baru, JANGAN overwrite entry lama. Sisipkan entry baru di paling atas, di bawah header title, di atas semua entry lama.
- Kalau belum ada → buat baru dengan header title & description dulu, baru entry pertama di bawahnya.
- Entry terbaru selalu di paling atas (reverse-chronological).

---

## Input Sources

### Source 1: File changelog.md

Baca file `changelog.md` di root project untuk mendapatkan informasi perubahan terbaru.

### Source 2: Git Commit Hash

Jika user memberikan commit hash, jalankan command berikut untuk mendapatkan informasi:

```bash
# Melihat detail commit
git show <commit_hash> --stat

# Melihat diff untuk file API saja
git show <commit_hash> -- "src/api/**/*"

# Melihat list file yang berubah
git diff-tree --no-commit-id --name-only -r <commit_hash>

# Melihat isi file yang berubah (constant, model, service)
git show <commit_hash>:src/api/constant/<filename>.ts
git show <commit_hash>:src/api/model/<filename>.ts
git show <commit_hash>:src/api/service/<filename>.ts
```

### Source 3: Git Commit Range

Untuk multiple commits atau range:

```bash
# Commits dalam range
git log <from_hash>..<to_hash> --oneline

# Diff dalam range untuk API files
git diff <from_hash>..<to_hash> -- "src/api/**/*"
```

---

## Processing Steps

### Step 1: Identify Source

Tentukan sumber input:

- Jika user menyebut "changelog" → Baca `changelog.md`
- Jika user memberikan hash (format: 7-40 karakter hex) → Gunakan git commands
- Jika user menyebut "latest" atau "terbaru" → Jalankan `git log -1 --format="%H"`

### Step 2: Extract API Changes

Dari source yang didapat, identifikasi perubahan di folder:

- `src/api/constant/*.ts` - Endpoint definitions
- `src/api/model/*.ts` - Request/Response models
- `src/api/service/*.ts` - Service methods

### Step 3: Classify Changes

Klasifikasikan setiap perubahan:

| Kondisi                             | Klasifikasi                   | Emoji |
| ----------------------------------- | ----------------------------- | ----- |
| File baru + endpoint baru           | API Baru                      | 🆕    |
| Endpoint path berubah               | API Diperbarui (Breaking)     | 🔄⚠️  |
| Request model tambah field optional | API Diperbarui (Non-Breaking) | 🔄    |
| Request model tambah field required | API Diperbarui (Breaking)     | 🔄⚠️  |
| Response model berubah              | API Diperbarui                | 🔄    |
| File/endpoint dihapus               | API Dihapus                   | ❌    |
| Method HTTP berubah                 | API Diperbarui (Breaking)     | 🔄⚠️  |

### Step 4: Generate Output

Buat output dalam format standar dan append ke `api-changelog.md`.

---

## Output Format

Format sama untuk kedua file, narasi beda bahasa.

**`api-changelog.md` (Indonesia):**

```markdown
## [COMMIT_HASH_SHORT] - YYYY-MM-DD

### [Nama Module/Fitur]

#### 🆕 API Baru (New)

| No  | Method | Endpoint            | Deskripsi         | Request Model | Response       |
| --- | ------ | ------------------- | ----------------- | ------------- | -------------- |
| 1   | METHOD | `/path/to/endpoint` | Deskripsi singkat | `ModelName`   | `ResponseType` |

#### 🔄 API Diperbarui (Updated)

| No  | Method | Endpoint            | Perubahan           | Breaking Change |
| --- | ------ | ------------------- | ------------------- | --------------- |
| 1   | METHOD | `/path/to/endpoint` | Deskripsi perubahan | Ya/Tidak        |

#### ❌ API Dihapus (Deprecated/Removed)

| No  | Method | Endpoint            | Alasan             | Pengganti           |
| --- | ------ | ------------------- | ------------------ | ------------------- |
| 1   | METHOD | `/path/to/endpoint` | Alasan penghapusan | Endpoint alternatif |

#### Request/Response Models

**NamaModel**

\`\`\`typescript
{
field1: type;
field2: type;
}
\`\`\`

#### Source Files

- Constant: `path/to/constant.ts`
- Model: `path/to/model.ts`
- Service: `path/to/service.ts`

---

**Author:** nama <email>
**Source:** branch info
**Commit:** full_commit_hash
```

**`api-changelog.en.md` (English):**

```markdown
## [COMMIT_HASH_SHORT] - YYYY-MM-DD

### [Module/Feature Name]

#### 🆕 New APIs

| No  | Method | Endpoint            | Description       | Request Model | Response       |
| --- | ------ | ------------------- | ----------------- | ------------- | -------------- |
| 1   | METHOD | `/path/to/endpoint` | Brief description | `ModelName`   | `ResponseType` |

#### 🔄 Updated APIs

| No  | Method | Endpoint            | Change                | Breaking Change |
| --- | ------ | ------------------- | --------------------- | --------------- |
| 1   | METHOD | `/path/to/endpoint` | Description of change | Yes/No          |

#### ❌ Removed/Deprecated APIs

| No  | Method | Endpoint            | Reason             | Replacement          |
| --- | ------ | ------------------- | ------------------ | -------------------- |
| 1   | METHOD | `/path/to/endpoint` | Reason for removal | Alternative endpoint |

#### Request/Response Models

**ModelName**

\`\`\`typescript
{
field1: type;
field2: type;
}
\`\`\`

#### Source Files

- Constant: `path/to/constant.ts`
- Model: `path/to/model.ts`
- Service: `path/to/service.ts`

---

**Author:** name <email>
**Source:** branch info
**Commit:** full_commit_hash
```

---

## Pattern Recognition

### Constant File Pattern

```typescript
export class NamaAPIConstant {
  static readonly methodName: HttpConstant = {
    method: "POST" | "GET" | "PUT" | "DELETE" | "PATCH",
    url: "/base/path/endpoint",
  };
}
```

### Model File Pattern

```typescript
export interface RequestModelName {
  requiredField: string;
  optionalField?: number;
}

export interface ResponseModelName {
  data: any;
  message: string;
}
```

### Service File Pattern

```typescript
export class NamaService extends BaseService {
  methodName(params?: RequestModel): Observable<ResponseType> {
    return this.http.request(Constant.endpoint, params);
  }
}
```

---

## Command Examples

### Dari Changelog

```
User: Buat API changelog dari changelog.md terbaru
Agent: [Baca changelog.md, extract API changes, generate output]
```

### Dari Commit Hash

```
User: Buat API changelog dari commit 4f0d2435
Agent: [Run git show 4f0d2435, analyze changes, generate output]
```

### Dari Commit Range

```
User: Buat API changelog dari commit abc1234 sampai def5678
Agent: [Run git diff abc1234..def5678, analyze changes, generate output]
```

### Latest Commit

```
User: Buat API changelog dari commit terakhir
Agent: [Run git log -1, get hash, analyze changes, generate output]
```

---

## Validation Checklist

Sebelum output, pastikan:

- [ ] Kedua file `api-changelog.md` (Indonesia) dan `api-changelog.en.md` (English) dihasilkan
- [ ] Entry baru disisipkan di paling atas kalau file sudah ada (gak bikin file baru, gak hilangin entry lama)
- [ ] Commit hash/ID tercantum dengan benar
- [ ] Tanggal sesuai dengan commit date
- [ ] Semua endpoint tercantum dengan method yang benar (GET/POST/PUT/DELETE/PATCH)
- [ ] Request model terdokumentasi dengan tipe data lengkap
- [ ] Breaking changes ditandai dengan jelas (⚠️)
- [ ] Source files direferensikan dengan path lengkap
- [ ] Author dan commit info dari git log
- [ ] Format markdown konsisten dan valid di kedua file

---

## File Output Location

Hasil akan ditulis ke **dua file**: `api-changelog.md` (Indonesia) dan `api-changelog.en.md` (English).

Entry terbaru selalu ditaro di paling atas (di bawah header), bukan di akhir file. Kalau salah satu/kedua file belum ada, buat baru dengan header title & description dulu sebelum nambahin entry.

Struktur tiap file:

```
api-changelog.md / api-changelog.en.md
├── Header (title & description)
├── [Latest Commit] - newest entry at top
├── [Previous Commit]
├── [Older Commit]
└── ...
```

---

## Error Handling

| Error                       | Action                                                |
| --------------------------- | ----------------------------------------------------- |
| Commit hash tidak ditemukan | Inform user, minta hash yang valid                    |
| Tidak ada perubahan API     | Inform user "Tidak ada perubahan API pada commit ini" |
| File changelog.md tidak ada | Inform user, tawarkan buat dari git commit            |
| Permission denied (git)     | Inform user untuk check git repository                |
