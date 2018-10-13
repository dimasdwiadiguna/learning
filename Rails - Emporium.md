# Crumbs on Emporium Tutorial





```ruby
rails new emporium
```

### Setting MySQL

- Using MySQL client, create 2 database : emporium_development and emporium_test

  ```mysql
  create database emporium_development;
  create database emporium_test;
  ```

  Lalu lihat hasilnya menggunakan perintah `show databases;`

  > Selalu pisahkan database untuk testing dengan development, terlebih lagi buat produksi, karena ketika testing bisa jadi akan wipe out semua data yang kita entry

- Buat sebuah akun dengan seluruh privilege untuk mengelola kedua database ini

  ```mysql
  grant all on emporium_development.* to \
    'emporium'@'localhost' identified by 'hacked';
  grant all on emporium_test.* to \
    'emporium'@'localhost' identified by 'hacked';
  ```

  lalu lihat user yang dibuat menggunakan perintah `select host,user from mysql.user;`

  > Jangan berikan user seperti ini ke database production karena biasanya user di database poduction hanya membutuhkan privilege select, insert, update, delete

- Edit file database.yml (file untuk mengurusi urusan database, menggunakan bahasa YAML atau YAML aint markup language) untuk mengganti setting database sebagai berikut:

  ```yaml
  default: &default
    adapter: mysql2
  ...
  
  development:
    <<: *default
    database: emporium_development
    username: emporium
    password: hacked
  ...
  
  test:
    <<: *default
    database: emporium_test
    username: emporium
    password: hacked
  ...
  ```

- Pastikan Gemfile sudah memasukkan `mysql2` sebagai salah satu dependency. Gem tsb adalah adapter untuk mengoneksikan app dengan database

  ```ruby
  # Use mysql
  gem 'mysql2'
  ```

- Jalankan server, buka browser dan ketikkan `localhost:3000` dan lihat hingga tampilan sukses muncul

  ```bash
  rails server
  ```



### Skema MVC

Ruby on Rails is built around the Model-View-Controller (MVC) pattern. MVC is a design pattern used for separating an application’s data model, user interface, and control logic into three separate layers with minimal dependencies on each other:

- The **controller** is the component that receives the request from the browser and performs the user-specified action.
- The **model** is the data layer that is used, usually from a controller, to read, insert, update, and delete data stored, for example, in a relational database.
- The **view** is the representation of the page that the users see in their browser; usually, the model is shown.

![](D:\Documents\TutAll\img\mvc_ror.PNG)

### Membuat controller

Kita akan membuat halaman 'About' untuk perusahaan. Setidaknya dibutuhkan 2 hal untuk menyajikan sebuah tampilan : controller dan views.

```bash
rails generate controller about index
```

Rails akan membuat controller bernama 'about' dengan action '#index'

> Untuk menghapus controller, gunakan `rails d controller <nama>`

Untuk menampilkan halaman ini di subhalaman 'about', bukalah folder config dan edit routes.rb dengan perintah berikut:

```ruby
get 'about' => 'about#index'
```

Sehingga rails akan mengarahkan url 'localhost:3000/about' kepada controller 'about' dengan action '#index'. Secara otomatis rails akan menampilkan halaman di folder 'views/about/index.html.erb'.

> file erb adalah embedded ruby, yaitu html yang diperkaya untuk bisa memproses baris perintah ruby seara langsung

Editlah file index tersebut menjadi seperti ini:

```html
<p>Online bookstore located in downtown Manhattan, New York</p>
<h2>Mailing Address</h2>
<address>
Emporium<br/>
P.O. Box 0000<br/>
Grand Cayman, Cayman Islands<br/>
</address>
```



### Layout

Desain yang senantiasa konsisten untuk seluruh halaman dapat disimpan sebagai layout, yang kontennya kemudian berubah secara dinamis. File ini ada di folder 'views/layout/application.html.erb'

```erb
<!DOCTYPE html>
<html>
  <head>
    <title><%= @page_title || 'Emporium'%></title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <h1><%= @page_title if @page_title %></h1>
    <%= yield %>
  </body>
</html>

```

Tag <%= yield %> menunjukkan lokasi dimana konten dari view akan dimasukkan ke dalam layout yang dibuat.

> Tag <%= ... %> digunakan untuk mengevaluasi perintah ruby dan menampilkan hasilnya. Tag <% ... > digunakan untuk mengevaluasi perintah ruby tanpa menampilkan hasil. Sementara tag <%# ... > adalah comment

Baris ke-4 menunjukkan bahwa title dari halaman adalah @page_title ATAU jika tidak ada judul halamannya maka ditampilkan teks 'Emporium'

Seluruh kebutuhan stylesheet dan javascript disimpan pada folder app/assets

- Jika ingin ada layout yang spesifik hanya pada kondisi tertentu:

  ```ruby
  class EternalLifeController < ApplicationController
      # Option 1
      layout 'default'
      # Option 2
      layout :determine_layout
  
      def index
          # Uses app/views/layouts/default.rhtml
      end
  
      def popup
          # Uses app/views/layouts/popup.rhtml
          # Option 3
          render :layout => 'popup'
      end
  
      def determine_layout
          if params[:id].nil?
          	return "fancy_layout"
          else
          	return "default"
      	end
  	end
  end
  ```

Untuk memberikan argumen pada layout, berikan perintah ini pada controller:

```ruby
class AboutController < ApplicationController
    def index
    	@page_title = 'About Emporium'
    end
end
```



### Test-Driven Development

1. Write a test that specifies a bit of functionality.
2. Ensure the test fails. (You haven’t built the functionality yet!)
3. Write only the code necessary to make the test pass.
4. Refactor the code, ensuring that it has the simplest design possible for the functionality
    built to date.

Rails menggunakan `Test::Unit` bawaan Ruby yang dibangun menggunakan perintah assert yang membandingkan hasil output. Ruby on Rails menggunakan 3 skema testing: unit test, functional test, dan integration test.



### Skema

Rails menggunakan sistem migrations yang memantau setiap perubahan dari skema database

```bash
rails generate model Author
```

> Angka acak pada nama awal file yang dibuat (seperti contoh 20180914165017_create_authors.rb) menunjukkan versi migrasi

Editlah file migration tersebut untuk memberitahu hal apa yang akan dimodifikasi dari skema kita

```ruby
class CreateAuthors < ActiveRecord::Migration[5.1]
  def change
    create_table :authors do |t|
      t.string :first_name
      t.string :last_name
      t.timestamps
    end
  end
end
```

> - Keuntungan modifikasi database melalui migration dibanding langsung di editor databasenya adalah sifat migration yang database-agnostic, sehingga ketika engine dari database kita akan berubah maka migration akan diterjemahkan menjadi perintah SQL yang sesuai
> - Terdapat dua gaya penulisan migration, change dan self.up-self.down. Keduanya sama secara goals namun memiliki kekuatannya sendiri. 
>   - def self.up memberitahukan hal apa saja yang perlu diubah ketika rake db:migrate dieksekusi (migrasi naik versi), sementara self.down memberitahukan perubahan ketika turun versi
>   - def change cukup memberikan hal apa yang diubah ketika naik versi, dan ketika diturunkan versi maka Rails cukup melakukan sebaliknya
> - Untuk memasukkan kolom baru pada def change, cukup jalankan perintah `t.<tipe> :<nama_kolom>`

Eksekusi migrasi tersebut menggunakan perintah:
```bash
rake db:migrate
```
Rails secara otomatis akan menghasilkan tabel authors (hasil dari create_table :authors), schema.rb yang menyimpan skema termutakhir (jangan edit manual!), dan schema_migrations yang menyimpan data version dari setiap perubahan yang terjadi pada database tersebut.
>Secara default, migration akan diterapkan ke database development. Jika ingin menerapkan ke database production, gunakan perintah `rake db:migrate RAILS_ENV=production`



### Unit Testing Pertama!

File untuk melakukan unit test atas model authors yang sudah dibuat bisa dibuka di folder test/models/authors_test.rb. Terapkan perintah ini:
```ruby
class AuthorTest < Test::Unit::TestCase
    fixtures :authors
    def test_name
        author = Author.create(:first_name => 'Joel',
        :last_name => 'Spolsky')
        assert_equal 'Joel Spolsky', author.name
    end
end
```
Jalankan perintah pengujian dengan
```bash
rake
```
Muncul error dikarenakan `name` belum dikenali. Maka untuk mengatasinya, bukalah folder app/models/author.rb dan berikan method name yang mengembalikan string penggabungan first_name dan last_name:
```ruby
class Author < ActiveRecord::Base
    def name
        "#{first_name} #{last_name}"
    end
end
```
