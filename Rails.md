



### Membuat proyek baru

Buat proyek Rails baru (sekaligus menjalankan `bundle install`)

```bash
rails new hello_app
```

Untuk menyesuaikan gem yang akan digunakan dalam proyek, modifikasi file gemfile

> Simbol `>=` menunjukkan versi yang digunakan adalah versi yang lebih baru dari yang ditentukan. Sementara simbol `~>` menunjukkan update hanya dilakukan jika digit terakhir versi mengalami perubahan

Lalu jalankan perintah

```bash
bundle update
bundle install
```

Buka terminal kedua, masuk ke direktori yang sama lalu jalankan

```bash
rails server
```

> Kenapa dijalankan di terminal kedua? Agar kita masih bisa utak-atik skrip tanpa start ulang server



### Local Git

Setting global git untuk nama dan email

```bash
git --global config user.name "Dimas Dwi A"
git --global config user.email dimasdwiadiguna@gmail.com
```

Lakukan inisialisasi dan masukkan seluruh file ke dalam pantauan git (kecuali yang diabaikan yang terdata di gitignore)

```bash
git init
git add -A
```

Lihat status staging area

```bash
git status
```

Eksekusi dengan

```bash
git commit -m "Initialize repository"
```

> - Best practice comment adalah menggunakan present tense imperative
> - Untuk undo pekerjaan kita di working area ke commit terakhir, gunakan perintah `git checkout -f` (flag -f menunjukkan force overwriting)

Untuk mulai memecah cabang dari git kita, gunakan `git checkout -b <nama>`.  Flag -b bermakna untuk membuat cabang baru.

Untuk melihat cabang yang tersedia

```bash
git branch
```

dan current branch ditandai dengan tanda bintang (*)

Langkah untuk menggabung branch baru ke branch master adalah:

```bash
git checkout master --> untuk masuk ke branch master
git merge <name> --> untuk menggabungkan branch ke current branch
git branch -d <name> --> untuk menghapus branch yang baru digabung
```

> Terkadang branch tidak perlu didelete agar bisa direview kembali



### Deploy to Heroku

Heroku menggunakan PostgreSQL sehingga kita perlu mengedit Gemfile untuk memasukkan gem pg saat tahap produksi

```ruby
group :production do
	gem 'pg', '0.20.0'
end
```

Lalu jalankan `bundle install --without production` . Walaupun jadinya seakan tidak menginstall apa-apa, tapi ini diperlukan untuk mengupdate Gemfile.lock dengan gem pg. Buat commit ke branch yang akan di-push (semisal branch master) menggunakan `git commit -am "deploy to heroku"`

Install Heroku toolbelt, cek apakah sudah terinstall dengan benar dengan menjalankan `heroku --version`.

```bash
cd dir\tempat\proyek\disimpan
heroku login
heroku keys:add --> terakhir kali saya gagal disini

heroku create --> buat apps baru
git push heroku master --> push branch master ke heroku
heroku rename nama_yang_mau_dikasih
```

Kalau error, cek log heroku `heroku logs --tail` dan lihat error yang muncul. Jika masalahnya ada di sqlite3, pastikan database adapter untuk production stage di database.yml menggunakan postgresql

```yaml
default: &default
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000

development:
  <<: *default
  adapter: sqlite3
  ...

test:
  <<: *default
  adapter: sqlite3
  ...

production:
  <<: *default
  adapter: postgresql <-- IMPORTANT
  ...
```



### Scaffold

Salah satu cara untuk membuat model adalah dengan scaffolding. Rails otomatis akan menyiapkan sistem untuk melihat, menambah, dan memanipulasi:

```bash
rails generate scaffold User name:string email:string
```

yang secara otomatis akan menghasilkan skema bernama users (lihat di output log, Rails secara otomatis menamakannya demikian). Migrasikan dengan `rails db:migrate` 

> Perintah `rake` digunakan untuk versi ruby < 5

Rails otomatis membuat halaman /users untuk melihat seluruh user, lengkap dengan akses id nya (/users/1 untuk id 1, dan users/1/edit untuk mengeditnya). Untuk menambah user, akseslah /users/new



1. The browser issues a request for the /users URL.
2. Rails routes /users to the `index` action in the Users controller.
3. The `index` action asks the User model to retrieve all users (`User.all`).
4. The User model pulls all the users from the database.
5. The User model returns the list of users to the controller.
6. The controller captures the users in the `@users` variable, which is passed to the `index` view.
7. The view uses embedded Ruby to render the page as HTML.
8. The controller passes the HTML back to the browser.[3](https://www.railstutorial.org/book/toy_app#cha-2_footnote-3)