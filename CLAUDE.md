# CLAUDE.md: Instructions for Agents in this Repository

> **STATUS 12 November 2026.** Pengerjaan WebApp dibuka kembali oleh owner
> dan dikerjakan bersama Claude Code. Alur kerjanya ada di `docs/WORKFLOW.md`.
> Lapisan web sudah direstrukturisasi menjadi MVC (lihat
> `docs/ARCHITECTURE.md` bagian 4).
>
> Dokumen ini tetap berlaku sebagai rujukan keputusan. Isinya jangan dihapus.

## 1. Roles
| Agent | Tool | Role | Authority |
|------|------|-------|------------|
| Codex CLI | OpenAI Codex CLI | Author and auditor in one session | Writes and modifies code according to SPEC, ARCHITECTURE, DESIGN, then verifies its own work against the rules in section 4 |
| Antigravity CLI | Antigravity IDE 1.107.0, chat mode --mode agent | Interface author | Writes and modifies app/views/templates/index.html and docs/DESIGN.md according to DESIGN.md and after owner approval |

Codex CLI works alone here. It writes the code and then verifies that same
code, so it is not an independent auditor. Section 4 exists because of that,
and every rule in it is mandatory.

The industrial brutalist interface redesign was carried out by Antigravity CLI
version 1.107.0.

The Codex CLI version number is not recorded because `codex --version` was
unavailable on the machine where the later reviews ran.

## 2. Sources of truth
Priority order when a conflict arises:
1. SPEC.md
2. ARCHITECTURE.md
3. DESIGN.md
4. PRD.md and CONTEXT.md (draft status until validated)
5. AGENTS.md
6. Existing code

An agent must not change a decision recorded in SPEC.md without explicit approval from the project owner.

## 3. Rules for the author
1. Do one task per change. Never combine a refactor with a new feature.
2. Follow the interface in DESIGN.md. If the interface must change, write up the proposal and wait for approval.
3. Never change the lookup table [L] or the class weighting rules without approval.
4. Never use test data for any decision during training.
5. Never store credentials, tokens, or absolute paths in code.
6. Every finished task must come with tests according to DESIGN.md Section 5.
7. When done, write the change summary and the file list for the project owner.
8. The summary must be complete enough that the owner can judge the task without guessing its scope.
9. Never declare a task finished before the verification in section 4 has actually been run.
10. Route every question to the document that owns the answer before acting on it.

    | Question about | Authoritative document |
    |----------------|-------------------------|
    | Interface, colour, contrast, typography, layout, components, UI behaviour | `docs/DESIGN.md` |
    | Model architecture, inference path, data contract | `docs/ARCHITECTURE.md` |
    | Agent rules, finding format, severity vocabulary | `AGENTS.md` section 4 |
    | Data, dataset, provenance | `README.md` and `data/` |

11. Before reporting a defect, check whether a written decision already exists in one of the documents
    above, and whether a test already locks it in. If either one exists, it is not a new defect.

## 4. Rules for the auditor (the same session)
1. Verification happens before a task is considered finished. It is not optional.
2. Check conformance with SPEC.md and ARCHITECTURE.md.
3. Check quality using the report grading rubric: conceptual understanding, novelty and relevance, technical depth, and systematicity.
4. Recompute the numbers from their source. Never copy a number from the output that produced it.
5. Confirm a number through a second path before calling it correct. A green test only counts once you
   have seen it fail on the code that was supposed to fail it.
6. Label every claim that comes from your own report and was not verified through a second path as
   "not independently verified". Do not state it as settled fact.
7. Check specifically:
   - data leakage between train, validation, and test;
   - use of class weights computed from data other than train data;
   - the lookup table being applied only at the training stage;
   - macro F1-score being computed and reported.
8. Never delete or weaken a test to make a task pass. Never invent a cause when the cause is unknown.
9. Finding format: `[severity] file:line - description - proposed fix`.
   Severity: `critical`, `major`, `minor`.
10. Report defects found in your own work. Staying silent repeats the mistake with nobody to catch it.
11. Critical findings block task completion until fixed. Minor findings do not block, but must be recorded.
12. There are only two verdicts: PASS or FAIL. PASS is given when there are no critical or major findings.
13. After all findings are closed, compile one final report in the format of section 8.

## 5. Workflow
```
Task from the owner
      |
      v
Codex CLI: implementation + tests
      |
      v
Codex CLI: verification per section 4, recompute the numbers
      |
      v
Codex CLI: findings list, propose fixes, request approval from the owner
      |
      v
Owner: approve or reject each finding
      |
      v
Codex CLI: apply the approved fixes
      |
      v
Codex CLI: re-verify if there are critical or major findings
      |
      v
Codex CLI: final report with verdict
      |
      v
Task is done when the verdict is PASS
```

If the verdict is FAIL, the flow returns to the verification stage. There is no limit on the number of rounds, but
each round must produce a real change or an explanation of why that finding was closed.

## 5.1 Pull request workflow

Numbered as a subsection of section 5 on purpose. Inserting it as its own
numbered top-level section would have forced the sections after it to be
renumbered, and the existing section numbers are referenced elsewhere.

`main` is protected. A direct push to `main` is rejected by the server and is
forbidden by policy as well. Every change reaches `main` through a pull request
that the project owner merges.

### The current flow
```
Work on a branch, never on main
      |
      v
Commit locally with a message that says what changed
      |
      v
Push the branch
      |
      v
Open a pull request targeting main
      |
      v
Wait for the three CI status checks
      |
      v
Owner reviews and merges
```

### Forbidden
- Pushing to `main`. The server rejects it, and policy forbids it too.
- Merging your own pull request. The owner merges.
- Force pushing to any branch.
- Deleting a branch that is not fully merged.

### Required gates
A pull request cannot be merged until all three status checks are green:

- `CI / Tes pada Python 3.11`
- `CI / Tes pada Python 3.12`
- `CI / Tes pada Python 3.13`

Plus at least one approving review.

### One finding, one branch, one pull request
Do not bundle unrelated changes. Two problems means two branches and two pull
requests.

### When the pull request is created
After the verdict is PASS, not before. The pull request is created after the
verification, so the review on GitHub checks something already verified rather
than searching for problems from the start.

### Secrets
A token never appears in a file, a commit, or a pull request comment. To call the
GitHub API, use the `gh` CLI, which is already logged in. When CI needs a
credential, use GitHub secrets; never write it into a workflow.

## 6. General limits
- No agent uploads DIBaS data to an external service.
- No agent runs full training without confirmation, because of the consumer GPU load.
- Every change is recorded in git history with a commit message that references the task.
- Raw DIBaS data is not committed. Only `data/index.csv` and
  `data/raw/zips_manifest.csv` are committed as provenance evidence.

## 7. Commit Message Format
`[type] short summary` with types: feat, fix, test, docs, refactor.

## 8. Final Report Format
The final report contains five sections in a fixed order:
1. **Result summary** - what was verified and how it was verified.
2. **Findings** - remaining findings in `[severity] file:line - description - proposal` format.
   If there are none, write "No findings".
3. **Status of each previous finding** - for each finding from the previous round, state its
   status: closed, rejected with a reason, or still open.
4. **Specification conformance** - assessment against SPEC.md and the rules in this AGENTS.md.
5. **Verdict** - PASS or FAIL, with a brief reason.

---

## Catatan Prioritas

Bagian 9 sampai 24 di bawah diadaptasi dari AGENTS.md proyek BacteriaCV dan
berlaku menyeluruh di repo ini. Bagian 1 sampai 8 tetap berlaku dan tidak
diganti.

Kalau dua aturan bertabrakan, yang lebih ketat yang menang. Kalau yang
bertabrakan bukan soal ketat-longgar melainkan dua keputusan berbeda,
berhenti dan tanya owner sebelum mengeksekusi.

## 9. Proyek dan Lingkup

BacteriaCV adalah aplikasi web untuk mengklasifikasi bentuk sel dan status
Gram bakteri dari citra mikroskop. Model berjalan di server, bukan di
peramban. Repo ini dinyatakan selesai oleh owner pada 12 Oktober 2026.

Lingkup yang masih berlaku:

- perbaikan cacat yang terbukti, disertai bukti reproduksi
- pekerjaan yang diminta eksplisit oleh owner
- halaman dokumentasi dan Changelog yang direncanakan owner

Di luar itu jangan menambah fitur, jangan mengubah antarmuka, dan jangan
mengubah angka model. Kalau ragu apakah sesuatu masih dalam lingkup, tanya
dulu.

Aplikasi Android ada di repo terpisah, `BacteriaCVMobile`. Jangan
mengerjakannya dari repo ini.

## 10. Gaya Bahasa

- Jawab hanya dalam Bahasa Indonesia untuk semua keluaran, termasuk
  komentar kode dan dokumentasi yang ditulis agent.
- Sebut diri sendiri "Saya", bukan "Aku" atau "Gue".
- Sebut pengguna "Anda", bukan "Kamu".
- Nada formal tetapi agak santai, tidak kaku.

## 11. Kebijakan Jaringan dan Server

- Server hanya melayani aplikasi web ini. Tidak ada panggilan ke layanan
  pihak ketiga, tidak ada telemetri, tidak ada analytics, tidak ada cloud.
- Data DIBaS tidak pernah diunggah ke layanan eksternal mana pun.
- Jangan menambah dependency jaringan baru tanpa persetujuan owner.
- Endpoint publik tidak boleh dipakai untuk mengumpulkan data pengguna.
  Tidak ada log yang menyimpan citra unggahan.

## 12. Aturan Arsitektur

- Jalur inferensi dijelaskan di `docs/ARCHITECTURE.md`. Jangan mengubahnya
  tanpa keputusan arsitektur baru yang dicatat di dokumen itu.
- Lookup table [L] hanya boleh dipakai di tahap training, tidak pernah di
  jalur inferensi.
- Bobot kelas hanya boleh dihitung dari data train.
- Format `checkpoints/heads.pt` tidak boleh berubah tanpa persetujuan. SHA-256
  checkpoint ini dipakai sebagai bukti provenance.
- Kontrak data endpoint inferensi tidak boleh berubah tanpa persetujuan,
  karena aplikasi mobile dan halaman web memakai model yang sama.
- Jangan menghapus atau menyederhanakan tahap prapemrosesan yang sudah ada
  hanya karena "terlihat redundan". Setiap tahap punya pasangan di sisi
  mobile.
- Jangan mengubah data checkpoint, data kuota, atau data riwayat tanpa
  persetujuan eksplisit owner.

## 13. Verifikasi dan Bukti

Aturan ini lahir dari kejadian berulang di proyek ini: klaim status yang
tidak diperiksa ulang ternyata salah, lalu dipakai untuk menulis instruksi
dan tes yang salah sasaran.

- Setiap klaim "sudah" atau "belum" wajib dicek ke kode, ke perangkat, atau
  ke endpoint, bukan dipercaya dari dokumentasi, plan lama, atau laporan
  sesi sebelumnya. Dokumen memberi arah, kode memberi kenyataan.
- Setiap angka yang dicantumkan dalam laporan atau instruksi harus berasal
  dari perintah yang benar-benar dijalankan di sesi ini. Kalau belum dicek,
  tulis eksplisit "belum dicek". Jangan menebak atau mewarisi angka dari
  sesi lain.
- Definisi yang dipakai untuk menguji kebenaran sebuah modul harus
  independen dari definisi yang dipakai modul itu sendiri. Mengimpor ambang
  atau konstanta dari modul yang sedang diuji hanya menguji konsistensi
  internal, bukan kebenaran.
- Ambil contoh literal untuk nilai batas saat menulis tes, jangan
  menurunkan nilainya dari rumus ambang itu sendiri.
- Lakukan warm-up sebelum mengukur performa. Biaya pertama kali akan salah
  terbaca sebagai bagian dari kerja yang diukur.
- Satu atau dua titik data tidak cukup untuk membangun kesimpulan performa.
  Butuh kontrol dengan variabel yang sama.
- Verifikasi dari isi, bukan dari metadata atau tanda tangan. Timestamp file
  bukan bukti isinya benar.
- Compile tanpa error bukan bukti kode benar atau lengkap.

## 14. Ambiguitas dan Klarifikasi

- Kalau sebuah instruksi menyebut ikon, aset visual, atau elemen UI tanpa
  menyebut sumbernya, tanya dulu sebelum mengubah apa pun.
- Kalau sebuah instruksi bisa ditafsirkan lebih dari satu cara, baik dari
  segi lingkup, lokasi berkas, maupun pendekatan, ajukan pertanyaan sebelum
  mengeksekusi.
- Jangan mengusulkan alternatif "lebih baik", saran desain, atau perbaikan
  tambahan di luar yang diminta, kecuali owner meminta rekomendasi.
- Kalau ragu antara bertanya atau melanjutkan, bertanya.
- Pertanyaan produk adalah keputusan owner, bukan keputusan teknis. Jangan
  memutuskan sendiri.

## 15. Dokumentasi

- Dokumen sumber ada di `docs/`: `SPEC.md`, `ARCHITECTURE.md`, `DESIGN.md`,
  `PRD.md`, `CONTEXT.md`, dan `WORKFLOW.md`.
- Perbarui dokumen yang relevan ketika fitur, keputusan arsitektur, atau
  status kerja berubah.
- `docs/WORKFLOW.md` mengatur alur kerja, bukan sumber kebenaran produk.
  Yang berwenang tetap `SPEC.md`, `ARCHITECTURE.md`, `DESIGN.md`, dan
  `PRD.md`.
- Saat melaporkan status, pisahkan klaim yang sudah diverifikasi ulang di
  sesi ini dari klaim yang diwarisi dari laporan sebelumnya.
- Jangan menyebut konstanta, nomor baris, atau nama simbol dari kode yang
  belum dibaca di sesi berjalan. Tulis "belum dicek" kalau memang belum.

## 16. Pengembangan

- Gunakan konvensi yang sudah ada dan pertahankan perubahan working tree
  yang tidak terkait.
- Periksa `git status` sebelum menyunting dan jangan menimpa perubahan
  owner.
- Jaga encoding tetap valid. Jangan menghasilkan karakter pengganti atau
  mojibake.
- Jangan memakai karakter em-dash (Unicode U+2014) di berkas mana pun,
  termasuk kode, teks antarmuka, dokumentasi, dan komentar.
- Sebelum commit, periksa seluruh berkas yang berubah dari encoding rusak
  dan karakter em-dash.
- Jangan commit `.env`, kredensial, log, hasil build, artefak perangkat,
  atau artefak alat lokal.
- Dua sesi atau dua instance kerja tidak boleh menyentuh working tree
  bersamaan, terutama saat menjalankan suite tes yang memodifikasi berkas
  repo lalu memulihkannya.
- Nama variabel bukan bukti perilaku kode. Baca isi fungsinya untuk
  memastikan apa yang sebenarnya dilakukan.

## 17. Toolchain dan Efek Samping

- Sebelum menjalankan perintah yang bisa install, generate, atau update
  dependency, nyatakan berkas tracked mana yang mungkin berubah.
- Setelah menjalankan perintah seperti itu, jalankan `git status` dan
  `git diff` untuk memastikan tidak ada berkas tracked yang berubah di luar
  dugaan.
- Kalau perintah itu mengubah berkas tracked di luar lingkup task, kembalikan
  berkas tersebut dan hapus artefak untracked yang dihasilkannya.
- Saat efek samping dikembalikan, laporkan berkas mana yang tersentuh dan
  bagaimana dipulihkan.

## 18. Penyerahan

- Jelaskan rencana lingkup sebelum perubahan besar ketika persyaratan masih
  ambigu.
- Laporkan berkas yang berubah, hasil validasi, dan risiko yang tersisa.
- Setiap klaim "selesai" atau "berhasil" dalam laporan ke owner harus
  disertai cara verifikasinya: perintah yang dijalankan, hash, atau hasil
  tes. Bukan sekadar pernyataan status.
- Jangan membuat release atau mengubah versi aplikasi tanpa permintaan
  eksplisit.

## 19. Perencanaan dan Pelacakan

- Ikuti `docs/WORKFLOW.md` untuk urutan rencana, persetujuan, branch, dan
  PR. Dokumen itu bukan pengganti spesifikasi.
- Setelah rencana kerja disetujui, catat perubahan keputusan di dokumen
  sumber yang sesuai.
- Jangan mengasumsikan adanya `TODO.md`, `CHANGELOG.md`, `worklog.md`, atau
  `DEVIATIONS.md`. Kalau perlu, tanyakan dulu.
- Pertahankan keputusan teknis dan status yang sudah ada. Jangan menimpa
  bagian dokumentasi yang tidak terkait.

## GitHub Issue dan Pull Request

- Sebelum implementasi, periksa dan laporkan status Git serta perubahan lokal di root dan worktree yang relevan. Jangan menimpa, membuang, atau memasukkan perubahan yang tidak terkait.
- Sebelum implementasi, tampilkan rencana yang menyebut lingkup, pendekatan, berkas yang diperkirakan berubah, validasi, dan risiko. Tunggu persetujuan eksplisit user sebelum membuat perubahan implementasi.
- Setelah rencana disetujui, buat worktree dan branch khusus dari `origin/main` terbaru. Buat satu GitHub Issue khusus untuk pekerjaan tersebut sebelum mulai implementasi.
- Periksa Issue dan PR terbuka yang relevan sebelum membuat item baru. Gunakan judul yang menjelaskan lingkup BacteriaCV dan hubungkan Issue dengan PR hanya jika keduanya untuk pekerjaan yang sama.
- Jangan membuat Issue, Pull Request, komentar, atau rilis GitHub di luar alur yang sudah disetujui user.
- Jangan menggabungkan Pull Request. Penggabungan selalu dilakukan user.

## Branch and Commit Rules

- Buat branch implementasi hanya di worktree khusus yang dibuat dari `origin/main` terbaru setelah rencana disetujui. Gunakan nama deskriptif seperti `fix/docx-resume-segment-rotation` atau `feat/markdown-preview`.
- Periksa `git status` sebelum membuat worktree/branch, staging, commit, push, atau menghapus worktree. Pertahankan perubahan lokal dan berkas tak terlacak yang bukan bagian tugas.
- Setiap commit hanya boleh berisi perubahan pada satu berkas. Jika satu tugas mengubah beberapa berkas, buat commit terpisah untuk setiap berkas.
- Gunakan pesan Conventional Commit dengan awalan tipe, misalnya `fix:`, `feat:`, `docs:`, atau `chore:`, dan jelaskan perubahan secara spesifik.
- Jangan masukkan secrets, `.env`, log, artefak build, berkas perangkat, atau perubahan format yang tidak terkait ke commit.
- Sebelum push, verifikasi `git diff --check` dan jalankan validasi relevan sesuai rencana yang disetujui. Laporkan validasi yang tidak dijalankan.
- Jangan gunakan perintah destruktif seperti `git reset --hard`, `git checkout --`, atau reset database tanpa persetujuan eksplisit.

## Pull Request Rules

- Buat satu PR dari branch implementasi ke `main` untuk Issue dan worktree yang sama.
- Gunakan judul berawalan tipe yang jelas. Body PR harus berupa Markdown multiline nyata dengan bagian:
  - `Ringkasan`
  - `Perubahan`
  - `Validasi`
  - `Dampak Data` atau `Deployment Checklist` jika relevan
- Tautkan Issue terkait dengan `Closes #<number>` atau `Fixes #<number>` jika PR menyelesaikan Issue itu.
- Nyatakan apakah migrasi database, perubahan variabel lingkungan, perubahan penyimpanan, atau tindakan deployment manual diperlukan.
- Buat PR sebagai draft sampai laporan review konsolidasi tersedia dan pemeriksaan relevan ditinjau. Setelah branch didorong, laporan tersedia, dan pemeriksaan ditinjau, PR boleh ditandai siap.
- Laporkan URL Issue dan PR, nama branch, hash commit, serta hasil validasi kepada user.
- Agent tidak boleh menggabungkan PR, sekalipun semua pemeriksaan lulus. Hanya user yang menggabungkan PR.

## Alur Pull Request

- Setelah rencana disetujui, buat worktree khusus dari `origin/main` terbaru, lalu buat Issue khusus sebelum implementasi.
- Implementasikan hanya lingkup yang disetujui. Commit perubahan satu berkas per commit, jalankan validasi relevan, lalu push branch.
- Buat PR draft ke `main` dengan body berbahasa Indonesia dan tautan ke Issue yang sama.
- Setelah PR dibuat, tinjau seluruh perubahan dan sampaikan satu laporan Code Review konsolidasi berbahasa Indonesia kepada user sesuai `docs/WORKFLOW.md`. Laporan memuat temuan, status perbaikan, validasi, dan risiko/batas verifikasi.
- Tinjau status CI/checks dan laporan review sebelum menandai PR siap. Jika tidak ada checks yang dikonfigurasi, laporkan secara eksplisit; jangan menyebutnya lulus.
- Jangan menyatakan PR siap sebelum branch didorong, laporan tersedia, dan hasil pemeriksaan yang relevan ditinjau.

## Worktree Completion Rules

- Setiap `.worktrees/<name>` adalah checkout branch tersendiri. Jangan merge atau cherry-pick branch worktree langsung ke branch root.
- Satu worktree menangani satu Issue dan satu PR tersendiri. Jangan gabungkan beberapa worktree atau Issue ke satu PR.
- Jangan menggabungkan PR worktree. Tunggu user menggabungkannya melalui GitHub.
- Setelah user menyatakan PR sudah di-merge, verifikasi status merged melalui GitHub dan fetch `origin/main`. Pastikan working tree worktree bersih sebelum menghapusnya dengan `git worktree remove`.
- Jangan menghapus folder worktree secara manual atau menghapus worktree saat PR masih terbuka/belum terverifikasi merged. Jangan hapus branch remote kecuali user meminta atau aturan repo mengharuskannya.
- Jika diminta menerapkan perubahan worktree ke root, artikan sebagai membuat/memverifikasi PR sesuai alur ini, bukan menggabungkan berkas langsung ke branch root.

## Release Rules

- Hitung hanya PR yang benar-benar berstatus merged sejak GitHub Release sebelumnya. PR terbuka, ditutup tanpa merge, atau hanya digabung secara lokal tidak dihitung.
- Jika belum ada GitHub Release sebelumnya, hitung PR merged sejak awal riwayat repository.
- Saat hitungan mencapai lima PR merged, periksa changelog yang tersedia, seluruh perubahan sejak release sebelumnya, status validasi, serta tag dan GitHub Release terakhir sebelum menyiapkan release.
- Jika changelog belum ada atau tidak mencakup perubahan yang akan dirilis, jangan mengarang riwayat. Siapkan ringkasan dari PR merged dan minta keputusan user tentang pembaruan changelog sebelum publikasi.
- Gunakan konvensi versi/tag repo yang sudah ada. Buat annotated Git tag dan GitHub Release yang merangkum PR merged tersebut setelah validasi relevan diperiksa. Jangan mengubah versi aplikasi secara otomatis.
- Pembuatan release setelah ambang lima PR ini merupakan persetujuan tetap user. Jangan membuat release sebelum ambang terpenuhi atau saat changelog/validasi belum siap.
- Jangan menggabungkan PR untuk mencapai ambang release. Penggabungan tetap dilakukan user. Setelah release, mulai hitung ulang dari PR merged berikutnya.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

- Skill **graphify** tersedia di `.agents/skills/graphify/SKILL.md` untuk pertanyaan tentang knowledge graph. Ikuti instruksi skill tersebut saat graphify relevan.

Rules:

- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

## Skill Routing (WAJIB)

Folder `.agents/skills/` berisi banyak skill. JANGAN membaca semua skill.
Ikuti aturan ini untuk menghemat token:

1. Sebelum mengerjakan tugas, identifikasi jenis tugas dan cocokkan dengan tabel di bawah.
2. Baca HANYA `SKILL.md` dari skill yang cocok (maksimal 3 skill per tugas).
3. Folder `references/` dan `examples/` dibaca hanya jika `SKILL.md` menyuruhnya dan tugas benar-benar membutuhkannya.
4. Jika tidak ada skill yang cocok, kerjakan tanpa skill. Jangan menebak atau membaca skill lain "untuk jaga-jaga".
5. Jangan membaca ulang skill yang sudah dibaca dalam sesi yang sama.
6. Abaikan `differential-review/` kecuali diminta review kode/PR secara eksplisit.
7. Skill berat (`taste-skills`, `llm-council`) hanya dipakai jika user memintanya secara eksplisit.

### Tabel Routing

| Jenis tugas                                     | Skill yang dibaca                                                          |
| ----------------------------------------------- | -------------------------------------------------------------------------- |
| Membuat/menambah unit test Dart                 | `dart-add-unit-test`                                                       |
| Mock untuk test Dart                            | `dart-generate-test-mocks`                                                 |
| Coverage test                                   | `dart-collect-coverage`                                                    |
| Migrasi ke package `checks`                     | `dart-migrate-to-checks-package`                                           |
| Error runtime Dart/Flutter                      | `dart-fix-runtime-errors`                                                  |
| Static analysis / lint                          | `dart-run-static-analysis`                                                 |
| Konflik dependency / pubspec                    | `dart-resolve-package-conflicts`                                           |
| Dokumentasi kode Dart                           | `dart-write-documentation`, `dart-use-doc-examples`                        |
| Pattern matching / primary constructor          | `dart-use-pattern-matching`, `dart-use-primary-constructors`               |
| Manipulasi path file                            | `dart-use-path-package`                                                    |
| FFI / native binding                            | `dart-setup-ffi-assets`, `dart-use-ffigen`                                 |
| CLI app Dart                                    | `dart-build-cli-app`                                                       |
| Widget test Flutter                             | `flutter-add-widget-test`                                                  |
| Integration test Flutter                        | `flutter-add-integration-test`                                             |
| Widget preview                                  | `flutter-add-widget-preview`                                               |
| Arsitektur app Flutter                          | `flutter-apply-architecture-best-practices`                                |
| Layout responsif                                | `flutter-build-responsive-layout`                                          |
| Layout overflow / bug layout                    | `flutter-fix-layout-issues`                                                |
| JSON serialization                              | `flutter-implement-json-serialization`                                     |
| Routing / navigasi                              | `flutter-setup-declarative-routing`                                        |
| Lokalisasi / i18n                               | `flutter-setup-localization`                                               |
| HTTP / API call                                 | `flutter-use-http-package`                                                 |
| Desain UI / UX app Flutter                      | `ui-ux-pro-max` (stack Flutter), `design-system`, `ui-styling` (pilih 1-2) |
| Desain halaman web (HTML/CSS)                   | `frontend-design`; `taste-skills` hanya jika diminta eksplisit             |
| UI dari gambar/screenshot                       | `image-to-code`                                                            |
| Review kode / perubahan (diff)                  | `differential-review`                                                      |
| Issue / project GitHub                          | `github-project-management`                                                |
| Pertanyaan arsitektur / hubungan antar file     | `graphify` (query graph dulu, jangan baca banyak file)                     |
| Keputusan strategis                             | `llm-council` hanya jika user menulis "council this" / "debate this"       |
| Menulis teks/dokumentasi agar tidak terkesan AI | `stop-slop`                                                                |

