# Ruby on Rails in Cloud9

Cloud9 adalah layanan dari Amazon Web Service



Di pane environment, klik gear lalu ceklis Show root folder dan unceklis Show environment folder agar seluruh folder dari root dapat dilihat dari pane tersebut



## Langkah Persiapan

1. Cloud9 menggunakan RVM (Ruby Version Manager) untuk mengatur versi ruby yang akan digunakan. Untuk melihat versi ruby yang digunakan saat ini dan mana yang menjadi default, ketikkan perintah `rvm list`
2. Jika versi ruby yang ingin digunakan berbeda, gunakan perintah berikut `rvm install ruby-2.3.4` (misal untuk menginstall versi tersebut)
   :warning: Umumnya untuk menginstall versi ruby menggunakan rvm tidak bisa dilakukan tanpa privilege superuser, sehingga lakukanlah dalam lingkungan superuser dengan terlebih dahulu mengetikkan `superuser su` hingga tanda $ berubah menjadi tanda #
3. Masih dalam privilege superuser, setelah versi ruby yang diinginkan selesai diinstall, set pula versi tersebut sebagai versi default yang digunakan menggunakan perintah `rvm --default use 2.3.4` sehingga setiap kali kita akan memulai IDE ini, versi 2.3.4 lah yang digunakan
4. Keluarlah dari privilege superuser menggunakan perintah `exit`
5. Periksalah apakah rails sudah terinstall dengan mengetikkan `rails -v`. Jika belum, lakukan instalasi dengan `gem install rails -v 2.4.5 --no-ri --no-docs `.
6. Lalu lakukan gem install bundler --no-ri --no-docs
7. Lalu install gem install rake
8. Lalu buatlah aplikasi baru menggunakan `rails _2.4.5_ new toy_app`
9. Jalankan server dengan mengetikkan `rails s -b $IP -p $PORT`



```matlab
function [a,k] = metodeNewton(J0,F0,x0,tol,iterMaks)
    format shortEng;
    disp('%%% Script Metode Newton oleh Dimas Dwi Adiguna (20916005) %%%');
    %disp('%% Kalkulasi untuk tebakan awal : ');disp(transpose(x0));
    a = transpose(x0);
    for k = 1:iterMaks
        %disp(['%%%% Iter:',num2str(k)])
        syms x1 x2;
        J = double(subs(J0,[x1,x2],a));
        F = double(subs(F0,[x1,x2],a));
        F = double(F.*(-1));
        y = linsolve(J,F);
        if isinf(y)
            disp('Sistem linear tidak tersolusikan')
            break
        end
        a = a + transpose(y);
        if norm(y) <= tol
            disp(['Converges in step number:',num2str(k)]);disp(double(a));
            break
        end
    end
end
```



## Koneksi dengan Github

Untuk mengoneksikan proyek kita dengan repo di Github

```bash
git config --global --replace-all user.name "Dimas Dwi Adiguna"
git config --global --replace-all user.email "dimasdwiadiguna@gmail.com"
git config --list

git add origin git@github.com:dimasdwiadiguna/alpha-blog.git

git remote -v
```

SSH dapat digunakan untuk koneksi tanpa menggunakan username dan password sehingga lebih aman. Membuat SSH dapat dilakukan dengan:

```bash
ssh-keygen -t rsa -b 4096 -C "dimasdwiadiguna@gmail.com"

cat ~/.ssh/id_rsa.pub
```



```bash
git push -u origin master
```



## Koneksi dengan Heroku

Di Cloud9, heroku dapat diinstall menggunakan npm

```bash
nvm i v8
npm install heroku
```

untuk menjalankan heroku butuh update ke rails 4.2.8 minimalnya. editlah gemfile sehingga rails yg digunakan adalah versi 4.2.8 dan pindahkan gem 'sqlite3' yang biasa digunakan di proses development ke group development, lalu tambahkan gem 'pg','~0.15' di group production serta gem 'rails_12factor' (khusus untuk rails dibawah versi 5)

```ruby
group :production do
	gem 'pg', '~> 0.15'
	gem 'rails_12factor'
end
```

Follow up perubahan gemfile tanpa mengeksekusi grup production dengan `bundle install --without production` lalu jalankan ini untuk push ke heroku

```bash
heroku login

#untuk mensubmit public key kita ke heroku
heroku keys:add

heroku create
heroku create nama_baru

git push heroku master
```

Semestinya semua bisa dipush dengan baik

(kalau muncul application error) Modifikasi versi pg pada gemfile sesuai spesifikasi yang ditetapkan oleh heroku (klo gasalah untuk rails 4.2.8 digunakan pg '~> 0.15'). Karena sudah modifikasi gemfile, jangan lupa `bundle install` untuk update gemfile.lock. Push ke heroku kembali

(kalau masih masalah) Jalankan dbmigrate

```bash
heroku run rake db:migrate
```



## Database!

Dalam rails, model adalah tempat penyimpanan data-data yang terdiri atas tabel-tabel. Model memiliki nomenklatur camel case dan singular (Article). Sementara database memiliki nomenklatur lower case plural (articles). Nama file model tersebut lower case singular (article.rb) dan nama controllernya lower case plural (articles_controller.rb)

```bash
rails generate migration create_articles
```

Secara otomatis rails akan mendeteksi keyword create sebagai migration untuk membuat sebuah database. Sehingga ketika kita membuka folder db > migrate akan muncul file migration dengan template def create articles do. Isilah dengan perintah berikut untuk menginisiasi kolom:

```ruby
t.string   "title"
t.text     "description"
t.datetime "created_at"
```

Lalu save dan eksekusi perintah migrasi dengan `rake db:migrate`

Kalau ingin membuat perubahan, generate migration lagi dengan nama yang deskriptif, lalu di dalam def change do, masukkan:

```ruby
add_column :articles, :updated_at, :datetime
```

Buatlah sebuah file di app > model dengan nama `article.rb`

```ruby
class Article < ActiveRecord::Base
   # Kalo rails 5, class ini diturunkan dari ApplicationRecord 
end
```

Karena namanya yang terikat secara nomenklatur dengan tabel articles, secara otomatis rails akan membangun hubungan diantaranya dan membuat getter setter untuk mereka. Untuk mengeceknya:

```bash
rails console

Article.all
=> #<ActiveRecord::Relation []> 
Article
=> Article(id: integer, title: string, description: text, created_at: datetime, updated_at: datetime)
```

