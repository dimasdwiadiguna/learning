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



Notes:

1. Mau bikin button yang kayak gini wkwk

   https://dribbble.com/shots/3987277-Perspective-Button-Click

2. Untuk integrasi RoR dengan kekuatan python di ML

   https://www.netguru.co/blog/ruby-on-rails-in-machine-learning-yay-or-nay



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
# atau kadang langsung t.timestamps
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

### Access database via console

Karena namanya yang terikat secara nomenklatur dengan tabel articles, secara otomatis rails akan membangun hubungan diantaranya dan membuat getter setter untuk mereka. Untuk mengeceknya:

```bash
rails console

Article.all
=> #<ActiveRecord::Relation []> 
Article
=> Article(id: integer, title: string, description: text, created_at: datetime, updated_at: datetime)

#Cara update 1
article = Article.new
article.title = "First post"
article.description = "This is my first post. Created in rails console"
article
=> ... #nanti ditunjukkan konten dari article baru tersebut
article.save

#Cara update 2
article = Article.new(tile: "Second!!", description: "This is my second post")
article.save

#Cara update 3
Article.create(title: "Third. Yeah", description: "Because this third post is created using create method, it automatically saves")

Article.all
=> ... #muncul article baru barusan 

exit
```

Untuk melakukan pengeditan di rails console :

```bash
article = Article.find(2)
article.title = "Edited title"
article.save

#Untuk menghapus
article.destroy
```

### Database Validation

Kalau kita ingin memastikan data yang dimasukkan valid, maka dalam article.rb masukkan validasi keberadaan (presence) dan karakter (length) sebagai berikut:

```ruby
class Article < ActiveRecord::Base
   validates :title, presence: true, length: {minimum: 3, maximum: 50}
   validates :description, presence: true, length: {minimum: 10, maximum: 300}
end
```

Restart rails console. Maka ketika article.save dijalankan untuk article yang tidak memenuhi kriteria diatas, secara otomatis transaksi datanya akan dirollback dan output false.

Untuk melihat error log atas transaksi data yang terjadi, perintah ini dapat digunakan. Hal ini sangat berguna karena dapat diberikan kepada user dalam bentuk error report (akan dijelaskan selanjutnya mengenai cara implementasinya)

```bash
article.errors.any?
article.errors.full_messages
```

@article <-- instance variable



## Commonly Used Things : Routes

> Langkah membuat halaman baru : Routes -> Buat controllernya -> Isi controller dengan action -> Buat template di views > (nama controller) > (nama action).erb.html

- Untuk merujuk route yang diinginkan, bisa menggunakan prefix dari route tersebut. Cek di `rake routes` dan lihat prefixnya, lalu tambahkan `_path` untuk mendapatkan alamatnya.

- Untuk membuat hyperlink secara otomatis ke suatu route (misal prefixnya 'pages'), gunakan:

  ```erb
  <%= link_to "Go to pages index", pages_path %>
  ```

- Kalau di URI Pattern menunjukkan ada :id atau yang sejenisnya, maka path tersebut perlu diberikan argumen yang dibutuhkan, misal

  ```erb
  <%= link_to "Edit", edit_pages_path(id) %>
  ```

- Kolom verb menunjukkan method yang digunakan untuk mengakses action yang diinginkan. Semisal di rake routes untuk mengakses pages#destroy dibutuhkan verb DELETE, maka link untuk mengaksesnya adalah:

  ```erb
  <%= link_to "Delete", page_path(id), method: :delete %>
  ```

- 



## Creating UI for Input

Semisal kita ingin di /articles/new

1. Buat jalurnya dulu melalui config > routes.rb

   ```ruby
   resources :article
   ```

   Hal ini membuat model Article (dan database articles) mendapatkan kemampuan CRUD. Buktikan dengan `rake routes` yang menunjukkan articles kini memiliki action index, create, new, edit, show, update, destroy.

2. Buat controller articles_controller.rb berisi isntance variable @article dengan isi Article.new (perintah yang dilakukan di console, namun skrg dilekatkan ke variabel @article)

   ```ruby
   class ArticleController < ApplicationController
      def new
          @article = Article.new
      end
       
      def create
         ... 
         # karena disini tidak akan menampilkan sesuatu (hanya mengolah perintah saja), maka tidak perlu membuat template khusus untuk ini
      end
       
      def show
         @article = Article.find(params[:id])
         # karena disini akan menampilkan sesuatu ke user, jangan lupa buat template show.html.erb
      end
   end
   ```

3. Kalau /articles/new dikunjungi, muncul error missing template. Buatlah folder di views bernama articles, dan buat view untuk action new (new.html.erb). Buatlah kode untuk membuat form menggunakan embedded ruby form_for:

   ```html
   <h1>Create an article</h1>
   <%= form_for @article do |f| %>
   <p>
   	<%= f.label :title %><br/>
       <%= f.text_field :title %>
   </p><p>
   	<%= f.label :description %><br/>
       <%= f.text_area :description %>
   </p><p>
           <%= f.submit %>
           </p>
   
   ```

4. Isi def create untuk menampilkan seadanya seluruh info yang disubmit menggunakan action create

   ```ruby
   render plain: params[:article].inspect
   ```

   Tapi biasanya keperluan ini hanya untuk debugging saja dan menunjukkan bahwa form yang disubmit ke app berbentuk hash (key-value pairs, di python disebut sebagai dict)

5. Untuk mensubmit article baru ke database, gunakan `Article.new(article_params)` yang isi dari article_params tersebut didefinisikan dalam sebuah fungsi private yang melakukan whitelisting variabel title dan description.

   ```ruby
   def create
       @article = Article.new(article_params)
       @article.save
       redirect_to articles_path(@article)
   end
     
   private
   def article_params
     params.require(:article).permit(:title,:description)
   end
   ```
   **Mengapa kita perlu memanggil method require dan permit?** Dalam rails dikenal istilah whitelist, yaitu menunjukkan kepada Rails mengenai variabel apa saja yang diperbolehkan untuk disubmit ke model, karena tidak semua yang di-pass ke dalam app perlu untuk di-pass juga untuk memodifikasi model.

   Berikut adalah sebuah contoh:

   ` params.require(:post).permit(:title,:body)`

> Here you first tell Rails which attributes are allowed for new post objects – title and body in this example – and then you create the new post. This is simple enough and quite readable. You are telling Rails: “data for a post is required and it’s attributes may only include title and body attributes.”

```ruby
# Hash for creating new post
post_attributes = {
    title: "post",
    body: "post body"
}

# Array as an argument passed in permit method
params.permit(
    :title,:body)
```

> Notice the arguments are actually a single array (internally Rails processes the arguments as an array). Each key/value pair on the left maps to an array element on the right. You permit a hash by passing an array.

Bagaimana kalau yang kita submit bukan hanya sesimpel 1 layer (title:"...", body:"...") tapi nested yang didalamnya berisi array (title:"...", body:"...",comments:[{text:"..."},{text:"..."}])

Pat Shaughnessy memberikan penjelasan mengenai kebingungan mengenai whitelisting untuk nested attributes dan strong parameter disini :

http://patshaughnessy.net/2014/6/16/a-rule-of-thumb-for-strong-parameters


6. **Memvalidasi post yang dimasukkan**
   Buatlah sebuah if scenario di def create:

   ```ruby
   def create
       @article = Article.new(article_params)
       if @article.save
           flash[:notice] = "Success"
       	redirect_to articles_path(@article)
       else
           render 'new'
   end
   ```

7. **Menunjukkan respon bahwa input berhasil**

      `flash[:notice] = "Success"`

      > Flash (atau FlashHash) adalah objek Ruby on Rails yang cocok untuk menyimpan error atau notifikasi bagi pengguna karena sifatnya hanya menyimpan secara temporer sampai berhasil di render di template, setelah itu kontennya akan dihapus. (Flash hanya menyimpan objek primitif saja, sehingga kalau butuh link, butuh di-sanitize)

      Di file application.html.erb (master level template), di atas field yield, buatlah sebuah wadah bernama flash tersebut

      ```html
      <body>
      	<% flash.each do |name,msg| %>
      		<ul>
      			<li><%= msg %</li>
      		</ul>
          <% end %>
      	<%= yield %>
      </body>
      ```

      Perhatikan bagian mana dari embedded ruby tersebut yang dirender dan bagian mana yang tidak dirender
8. **Menunjukkan respon bahwa input gagal**

   Karena direncanakan bahwa kegagalan input akan mengembalikan user ke tampilan form (tidak di-redirect kemanapun), maka kita bisa memasukkan wadah untuk menunjukkan error tersebut di halaman new.html.erb saja:

   ```html
   <h1>Create an article</h1>
   <% if @article.errors.any? %>
       <h2>The following errors has occured:</h2>
       <ul>
       	<% @article.errors.full_messages.each do |msg| %>
       	<li><%= msg %></li>
       	<% end %> 
       </ul>
   <% end %>
   ...
   ```

   Bagaimana bila kita ingin menjadikan bagian ini menjadi partial dan bisa dipakai untuk models apapun (tidak hanya @article), maka bagaimana mengganti bagian @article tersebut?

   ...

9. Menunjukkan hasil input sebagai follow-up keberhasilan submit
   Di folder view > articles > show.html.erb

   ```erb
   <h1>Article</h1>
   <p>
     Title: <%= @article.title %>
   </p>
   <p>
     Description: <%= @article.description %>
   </p>
   ```

10. **Mengedit article**

    Dengan mengecek rake routes, ditemukan bahwa path untuk mengedit article adalah /articles/:id/edit. Maka buatlah controllernya di articles_controller

    ```ruby
    def edit
       @article = Article.find(params[:id])
    end
    ```

    Maka jangan lupa pula buat viewnya di bawah views > articles > edit.html.erb. Isinya samakan dengan form new.html.erb

    Ternyata setelah diedit, kurang action update (ini adalah tanda bahwa action edit berkait dengan update, seperti action new berkait dengan create)

    ```ruby
    def update
        @article = Article.find(params[:id])
        if @article.update(article_params)
          flash[:notice] = "Article was successfully updated"
          redirect_to article_path(@article)
        else
          render 'edit'
        end
      end
    ```

11. **Index action**

    Karena route index sudah dibuat dari resources (prefixnya "articles"), maka kita tinggal menyiapkan actionnya (yaitu "index")

    ```ruby
    def index
        @articles = Article.all
    end
    ```

    Disini dibuat instance variable bernama articles (bukan article, biar terasa ada banyak) berisi Article.all

    Siapkan viewnya dengan membuat table berisi link ke halaman edit dan show dari masing-masing article

    ```erb
    <h1>List of all articles</h1>
    <p><%= link_to "Create new article", new_article_path %></p>
    <table>
      <tr>
        <th>Title</th>
        <th>Description</th>
      </tr>
      <% @articles.each do |article| %>
      <tr>
        <td><%= article.title %></td>
        <td><%= article.description %></td>
        <td><%= link_to "Edit", edit_article_path(article) %></td>
        <td><%= link_to "Show", article_path(article) %></td>
      </tr>
      <% end %>
    </table>
    ```

12. **Destroy**

    Buat sebuah def destroy

    ```ruby
    def destroy
        @article = Article.find(params[:id])
        @article.destroy
        redirect_to articles_path
        flash[:notice] = "Article was successfully deleted"
    end
    ```

    Lalu tambahkan kolom baru di halaman index untuk melakukan destroy. 

    ```erb
    <td><%= link_to "Delete", article_path(article), method: :delete, data: {confirm: "Are you sure?"} %></td>
    ```

    Bagian data: confirm: akan memunculkan alert box

13. **Partial**
    Bagian dari erb yang sering digunakan dapat dikumpulkan jadi satu paket layout yang dapat digunakan berulang bernama partial.

    Form di new dan edit sama persis, sehingga dapat dikelompokkan di views > articles > _form.html.erb 

    Lalu di halaman new atau edit, gantilah bagian tersebut dengan perintah:

    ```erb
    <%= render 'form' %>
    ```

    Begitu pula di halaman layouts > application.html.erb, pisahkan bagian flash ke partial sendiri layouts > _messages.html.erb, dan ganti bagian yang dikosongkan dengan

    ```erb
    <%= render 'layouts/messages' %>
    ```

    Kenapa disini digunakan penunjukan folder secara eksplisit? Ternyata kalau nggak pakai folder eksplisit, error yang muncul menunjukkan rails mencari partial messages di folder articles.

14. **Remove redundancy using before_action**

    Kalau dilihat di application_controller, bagian article.find(params[:id]) selalu dipanggil di action edit, show, destroy, dan update. Maka buatlah private method baru bernama set_article

    ```ruby
    def set_article
          @article = Article.find(params[:id])
        end
    ```

    Lalu di bagian awal setelah deklarasi class, masukkan perintah

    ```ruby
    before_action :set_article, only: [:edit, :update, :show, :destroy]
    ```

    Sehingga untuk action tersebut, method set_article akan dijalankan yang memanggil instance variable berisi pencarian id article



## Deploy to Production to Heroku

```bash
git push heroku master

# Kalau cloud9 ga kenal heroku, coba jalankan lagi ini:
nvm i v8

# Migrasi segala tabel yang dibutuhkan model menggunakan setting production
heroku run rake db:migrate
```

Maka app kita sudah di-deploy ke heroku



## Implementasi Bootstrap

Rails memudahkan implementasi bootstrap dengan hanya menginstall gem bootstrap di https://github.com/twbs/bootstrap-sass. Masukkan perintah ini di gemfile tepat diatas perintah sass-rails (ga terlalu penting posisinya, setidaknya memastikan bahwa sass-rails kita terinstall)

```ruby
...
gem 'bootstrap-sass', '~> 3.3.7'
...
```

> Untuk rails 5.1+ masukkan pula gem 'jquery-rails'

Lalu lakukan `bundle install --without production` 

#### Membuat custom CSS.SCSS file

Di folder app > assets > stylesheets, buat file baru bernama `custom.css.scss` dan isi dengan import dari bootstrap-sass diatas:

```css
@import "bootstrap-sprockets";
@import "bootstrap";
```

Di folder javascripts, masukkan perintah yg dicopy dari github boostrap-sass ini di `application.js` tepat dibawah jquery-ujs

```js
...
//= require jquery_ujs
// COPYKAN BARIS DIBAWAH INI DI POSISI DIBAWAH JQUERY UJS
//= require bootstrap-sprockets
...
```

(`//=require jquery` untuk rails 5.1+ dicopy jika tidak ada di file tsb, dan posisinya juga dibawah jquery_ujs)

Restart server dan semestinya tampilan sudah berubah

#### Membuat navbar

Karena navbar selalu muncul di tiap halaman, maka tempatkan script untuk navbar di application.html.erb. Agar lebih rapi, navbar dapat dijadikan sebuah partial (misal namanya layouts > `_navigation.html.erb`). Copykan seluruh kode dari getbootstrap.com ke file ini.

Untuk menampilkannya di application, munculkan perintah render:

```js
<%= render 
```

#### Membuat komponen HTML dari embedded ruby

Semisal kita ingin mengubah

```html
<a class="button-active" id="user" href="www.halamanutama.com">Click Here</a>
```

menjadi bentuk embedded ruby, buatlah menjadi seperti ini:

```erb
<%= link_to "Click Here", root_path, class: "button-active", id: "user" %>
```

#### Memodifikasi CSS

CSS yang digunakan dalam app kita berasal dari bootstrap seperti yang dideklarasikan di custom.css.scss. Bagaimana bila kita ingin mengganti warna background navbar menjadi biru? Letakkan perintah berikut sebelum perintah import:

```css
$navbar-default-bg: #08088A;

@import "bootstrap-sprockets";
...
```

Kalau mau update CSS yang terkait kepada kelas dan id, inputlah CSS tersebut pasca seluruh perintah import di file tsb

```css
@import...

a {
    background-color: red;
}
```

#### Memasukkan image

Image diletakkan di assets > images. Untuk memanggilnya (semisal di CSS), gunakan:

```css
background-image: asset-url('image.png')
```

#### Form styling

Untuk melihat bagaimana form yang dibuat menggunakan form_for dapat di-style menggunakan CSS bootstrap, berikut contohnya:

```erb
...

<div class="row">
  <div class="col-xs-12">
    <%= form_for(@article, :html => {class: "form-horizontal", role: "form"}) do |f|%>
    <div class="form-group">
      <label class="control-label col-sm-2"><%= f.label :title %></label>
      <div class="col-sm-10">
        <%= f.text_field :title, class: "form-control", placeholder: "Title of article", autofocus: true  %>
      </div>
    </div>
    <div class="form-group">
      <label class="control-label col-sm-2"><%= f.label :description %></label>
      <div class="col-sm-10">
        <%= f.text_area :description, class: "form-control", placeholder: "Content of article", rows: 10  %>
      </div>
    </div>
    <div class="form-group">
      <div class="col-sm-offset-2 col-sm-2"><%= f.submit class: "btn btn-success" %> or </div>
      <div class="col-sm-8"><%= link_to "Cancel", articles_path, class: "btn btn-warning" %></div>
    </div>
    <% end %>
    <p>
      <%= link_to "Back to index", articles_path %>
    </p>
  </div>
</div>
```





#### Memahami Bootstrap 12-col grid system

- `col` itu menunjukkan column
- `xs/sm/md/lg` itu menunjukkan breakpoint. Breakpoint itu menunjukkan bagaimana ukuran sebuah komponen pada masing-masing ukuran
- `<angka>` yang menunjukkan berapa column yg diambil. Kalau tidak diisi, maka akan beleber sampe penuh 100%
- Satu baris maksimal berisi 12 kolom. Jika sebaris komponen ternyata jumlahnya lebih dari 12 kolom, maka akan overflow ke baris selanjutnya (sifatnya float)
- Begitu pula kalo kurang, maka kolom sisanya akan kosong (tidak jadi melebar)

