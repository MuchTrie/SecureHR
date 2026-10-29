# Git Branching Cheatsheet

Panduan latihan untuk memisahkan setiap fitur ke branch yang berbeda sebelum
digabungkan ke branch utama (`main`).

## 1. Konsep singkat

- `main`: branch utama yang berisi kode stabil.
- `feature/...`: branch untuk mengerjakan satu fitur tertentu.
- `fix/...`: branch untuk memperbaiki bug.
- `docs/...`: branch untuk perubahan dokumentasi.
- Commit: satu unit perubahan yang memiliki pesan jelas.
- Merge atau Pull Request (PR): proses menggabungkan branch fitur ke `main`.

Contoh nama branch:

```text
feature/login
feature/employee-management
feature/attendance
fix/login-validation
docs/git-cheatsheet
```

> Satu branch sebaiknya hanya mengerjakan satu fitur atau satu tujuan yang
> berhubungan erat.

## 2. Persiapan pertama kali

Lihat identitas Git:

```bash
git config --global user.name "Nama Anda"
git config --global user.email "email@example.com"
```

Clone repository jika belum ada di komputer:

```bash
git clone <URL_REPOSITORY>
cd <NAMA_FOLDER_REPOSITORY>
```

Periksa kondisi repository:

```bash
git status
git branch
git remote -v
```

## 3. Alur kerja fitur dari awal sampai selesai

### Langkah A — Mulai dari `main` terbaru

Sebelum membuat branch fitur, pastikan posisi berada di `main` dan ambil
perubahan terbaru dari remote:

```bash
git switch main
git pull origin main
```

Jika Git versi lama belum mendukung `switch`, gunakan:

```bash
git checkout main
```

### Langkah B — Buat branch khusus fitur

```bash
git switch -c feature/nama-fitur
```

Contoh:

```bash
git switch -c feature/employee-management
```

Perintah tersebut membuat branch baru sekaligus berpindah ke branch itu.
Pastikan branch yang aktif:

```bash
git branch --show-current
```

### Langkah C — Kerjakan dan periksa perubahan

Setelah mengubah kode:

```bash
git status
git diff
```

Jalankan test, build, atau lint project sesuai kebutuhan. Jangan langsung
menambahkan semua file jika ada file lokal atau rahasia:

```bash
git add <file-yang-diubah>
git status
```

Jika semua perubahan yang terlihat memang ingin disimpan:

```bash
git add .
```

### Langkah D — Commit perubahan secara bertahap

```bash
git commit -m "feat: menambahkan manajemen karyawan"
```

Contoh prefix pesan commit:

```text
feat: menambahkan halaman absensi
fix: memperbaiki validasi login
docs: memperbarui panduan instalasi
refactor: merapikan service karyawan
test: menambahkan test untuk autentikasi
```

Satu commit sebaiknya fokus pada satu perubahan yang masuk akal. Lihat
riwayat commit:

```bash
git log --oneline --decorate --graph -10
```

### Langkah E — Push branch fitur ke remote

Untuk pertama kali mengirim branch:

```bash
git push -u origin feature/nama-fitur
```

Push berikutnya cukup:

```bash
git push
```

Setelah itu, buat Pull Request dari `feature/nama-fitur` menuju `main`.
Jelaskan perubahan, cara menguji, dan hal yang perlu diperhatikan reviewer.

### Langkah F — Perbarui branch fitur sebelum digabungkan

Jika `main` berubah saat fitur masih dikerjakan:

```bash
git switch main
git pull origin main
git switch feature/nama-fitur
git merge main
```

Jika terjadi conflict:

1. Buka file yang ditandai conflict.
2. Tentukan perubahan yang benar dan hapus penanda
   `<<<<<<<`, `=======`, dan `>>>>>>>`.
3. Periksa hasilnya dan jalankan test.
4. Tandai conflict sebagai selesai:

```bash
git add <file-yang-sudah-diperbaiki>
git commit -m "merge: menyelesaikan conflict dengan main"
git push
```

> Untuk latihan, gunakan `merge` terlebih dahulu karena riwayatnya lebih mudah
> dipahami. Jangan melakukan `rebase` pada branch yang sedang dipakai bersama
> tanpa memahami dampaknya.

### Langkah G — Merge branch fitur ke `main` secara lokal

Jika ingin berlatih melakukan merge langsung dari terminal, pastikan pekerjaan
di branch fitur sudah di-commit dan branch `main` sudah diperbarui:

```bash
# Pastikan tidak ada perubahan yang belum disimpan
git status

# Perbarui referensi dan branch main
git switch main
git pull origin main

# Gabungkan branch fitur ke main
git merge feature/nama-fitur
```

Jika merge berhasil tanpa conflict, Git akan membuat merge commit atau
melakukan fast-forward. Periksa hasilnya:

```bash
git log --oneline --decorate --graph -10
git status
```

Jalankan test, build, atau lint setelah merge. Jika semuanya berhasil, kirim
`main` ke remote:

```bash
git push origin main
```

> Dalam kerja tim, biasanya branch fitur tidak langsung di-merge secara lokal
> lalu di-push. Push branch fitur terlebih dahulu, kemudian buat Pull Request
> agar perubahan dapat direview. Langkah merge lokal di atas cocok untuk
> latihan pribadi atau repository yang tidak mewajibkan Pull Request.

### Langkah H — Merge melalui Pull Request

Alur yang umum digunakan di GitHub:

```bash
# Dari branch fitur
git switch feature/nama-fitur
git push -u origin feature/nama-fitur
```

Kemudian di GitHub:

1. Buka repository.
2. Pilih **Compare & pull request**.
3. Pastikan base branch adalah `main`.
4. Pastikan compare branch adalah `feature/nama-fitur`.
5. Tulis ringkasan perubahan dan cara menguji.
6. Tunggu review dan perbaiki komentar jika ada.
7. Klik **Merge pull request** setelah semua pemeriksaan berhasil.
8. Hapus branch remote jika fitur sudah selesai.

Setelah PR digabungkan, sinkronkan `main` lokal:

```bash
git switch main
git pull origin main
```

### Langkah I — Membatalkan merge yang masih berlangsung

Jika conflict terlalu sulit dan proses merge belum selesai, batalkan merge
tanpa menghapus commit yang sudah ada sebelumnya:

```bash
git merge --abort
```

Setelah itu, periksa kondisi repository:

```bash
git status
```

Jika merge sudah selesai dan sudah menghasilkan commit, gunakan `git revert`
terhadap merge commit. Jangan menghapus riwayat bersama dengan `reset --hard`
atau `push --force`:

```bash
git log --oneline --graph -10
git revert -m 1 <HASH_MERGE_COMMIT>
git push origin main
```

`-m 1` memberi tahu Git bahwa parent utama yang dipertahankan adalah `main`.

### Langkah J — Setelah PR atau merge selesai

Hapus branch lokal dan remote yang sudah selesai digunakan:

```bash
git switch main
git pull origin main
git branch -d feature/nama-fitur
git push origin --delete feature/nama-fitur
```

Jika branch belum pernah digabungkan, Git akan menolak `-d`. Gunakan `-D`
hanya jika benar-benar yakin perubahan branch tersebut tidak diperlukan:

```bash
git branch -D feature/nama-fitur
```

## 4. Contoh beberapa fitur yang dikerjakan terpisah

Misalnya ada tiga pekerjaan:

```text
feature/login
feature/employee-management
feature/attendance
```

Alur masing-masing:

```bash
# Fitur login
git switch main
git pull origin main
git switch -c feature/login
# kerjakan fitur, lalu:
git add .
git commit -m "feat: menambahkan login pengguna"
git push -u origin feature/login
```

```bash
# Fitur manajemen karyawan
git switch main
git pull origin main
git switch -c feature/employee-management
# kerjakan fitur, lalu:
git add .
git commit -m "feat: menambahkan manajemen karyawan"
git push -u origin feature/employee-management
```

```bash
# Fitur absensi
git switch main
git pull origin main
git switch -c feature/attendance
# kerjakan fitur, lalu:
git add .
git commit -m "feat: menambahkan fitur absensi"
git push -u origin feature/attendance
```

Ketiga branch tersebut dapat dibuat dan dikerjakan secara terpisah. Gabungkan
ke `main` satu per satu melalui PR agar setiap perubahan mudah ditinjau.

## 5. Perintah yang sering dipakai

```bash
# Melihat semua branch lokal dan remote
git branch -a

# Berpindah branch
git switch nama-branch

# Membuat branch dari branch saat ini
git switch -c nama-branch

# Melihat perubahan yang belum di-stage
git diff

# Melihat perubahan yang sudah di-stage
git diff --staged

# Melihat file yang sudah di-stage
git diff --name-only --cached

# Membatalkan perubahan lokal yang belum di-commit pada satu file
git restore <file>

# Mengeluarkan file dari staging tanpa menghapus perubahan
git restore --staged <file>

# Melihat commit terakhir
git log --oneline -5

# Mengambil informasi branch remote tanpa mengubah file kerja
git fetch origin

# Melihat branch remote yang sudah tidak ada dan membersihkan referensinya
git fetch --prune
```

## 6. Menyimpan pekerjaan sementara dengan `stash`

Gunakan `stash` jika perlu berpindah branch tetapi pekerjaan saat ini belum
siap di-commit:

```bash
git stash push -m "pekerjaan sementara fitur absensi"
git switch main
# lakukan pekerjaan lain
git switch feature/attendance
git stash pop
```

Lihat atau hapus stash:

```bash
git stash list
git stash drop stash@{0}
```

### Skenario latihan — Pindah branch saat ada perubahan lokal

Situasi ini terjadi ketika sedang mengerjakan fitur di `main`, tetapi ingin
berpindah ke branch lain. Git menolak perpindahan jika perubahan lokal akan
tertindih oleh file pada branch tujuan.

Contoh kondisi awal:

```text
PS D:\PT_Much_Sejahtera\frontend> git switch feature/leave-approval
error: Your local changes to the following files would be overwritten by checkout:
        src/types/index.ts
Please commit your changes or stash them before you switch branches.
Aborting
```

Artinya, file `src/types/index.ts` memiliki perubahan lokal yang belum
di-commit. Jika branch berpindah, perubahan tersebut berisiko tertimpa oleh
versi file dari `feature/leave-approval`.

#### Pilihan 1 — Commit dahulu jika perubahan sudah siap

```bash
git status
git add src/types/index.ts
git commit -m "feat: memperbarui tipe leave"
git switch feature/leave-approval
```

Gunakan pilihan ini jika perubahan memang bagian dari pekerjaan yang ingin
disimpan sebagai riwayat permanen.

#### Pilihan 2 — Gunakan `stash` jika perubahan belum siap di-commit

```bash
git stash push -m "wip: perubahan tipe leave"
git switch feature/leave-approval
```

Perintah `stash` menyimpan sementara perubahan tracked files dan mengembalikan
working tree ke kondisi bersih. Periksa hasilnya:

```bash
git status
git stash list
```

Setelah selesai mengerjakan branch lain, kembalikan pekerjaan tadi:

```bash
git stash pop
git status
```

`git stash pop` mencoba menerapkan kembali perubahan dan menghapus stash jika
berhasil. Jika terjadi conflict, stash biasanya tetap disimpan. Selesaikan
conflict secara manual, lalu:

```bash
git add <file-yang-sudah-diperbaiki>
git commit -m "fix: menyelesaikan conflict saat menerapkan stash"
git stash drop stash@{0}
```

Jangan menjalankan `git stash drop` sebelum memastikan semua perubahan sudah
berhasil dikembalikan.

#### Contoh sesuai workflow frontend

```bash
# Sedang berada di main dan memiliki perubahan lokal
git status

# Simpan sementara perubahan
git stash push -m "wip: leave rules frontend"

# Pindah ke branch latihan
git switch feature/leave-approval

# Kembalikan perubahan lokal ke branch aktif
git stash pop

# Periksa file yang berubah
git status
git diff
```

Perhatikan bahwa `stash` tidak memindahkan commit ke branch lain. `stash pop`
hanya menerapkan isi perubahan lokal ke branch yang sedang aktif. Karena itu,
pastikan branch aktif sudah benar sebelum menjalankannya.

## 7. Membatalkan kesalahan dengan aman

Batalkan commit dengan membuat commit baru (aman untuk branch yang sudah
di-push):

```bash
git revert <HASH_COMMIT>
```

Batalkan perubahan pada file yang belum di-commit:

```bash
git restore <file>
```

> Hindari `git reset --hard` dan `git push --force` saat belajar atau saat
> branch digunakan orang lain. Keduanya dapat menghapus pekerjaan dan
> mengubah riwayat remote.

## 8. Checklist sebelum push atau membuat PR

- [ ] Saya berada di branch fitur yang benar, bukan `main`.
- [ ] Nama branch menjelaskan tujuan perubahan.
- [ ] Tidak ada password, token, atau file rahasia yang ikut di-commit.
- [ ] `git status` sudah diperiksa.
- [ ] Test, build, atau lint yang relevan sudah dijalankan.
- [ ] Commit berisi perubahan yang fokus dan pesannya jelas.
- [ ] Branch fitur sudah diperbarui dari `main` jika diperlukan.
- [ ] Deskripsi PR menjelaskan perubahan dan cara mengujinya.

## 9. Pola singkat yang perlu diingat

```text
main terbaru
  -> buat feature branch
  -> ubah kode
  -> cek diff dan test
  -> commit
  -> push branch
  -> buat Pull Request
  -> review dan perbaiki
  -> merge ke main
  -> hapus branch yang selesai
```
