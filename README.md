# Reflection

Reflection notes untuk Tutorial 6.

### 1. single threaded web server
Di metode `handle_connection`, kita membuat variable baru bernama `buf_reader`; sebuah instance `BufReader` yang membungkus referensi mutable kepada variable `stream`. BufReader ini menambahkan buffering dengan mengatur calls kepada `std::io::Read`. Setelahnya, kita juga membuat variable bernama `http_request` bertipe Vector yang mengumpulkan baris hasil request browser ke server kita.

`BufReader` mengimplementasikan `std::io::BufRead` mengembalikan `Result<String, std::io::Error>` dengan memotong stream data setiap kali ia melihat byte newline. Untuk mendapatkan setiap string, kita map dan unwrap setiap `Result`.

Browser akan menandakan titik akhir dari HTTP request dengan mengirimkan dua karakter newline berturut-turut, jadi untuk mendapatkan satu request dari stream, kita mengambil terus baris hingga mendapatkan baris yang kosong (empty string). Setelah mengumpulkan baris-barisnya ke dalam vektor http_request, http_request dapat dicetak dengan format `"Request: {:#?}"` untuk memperlihatkan semua instruksi yang dikirimkan web browser ke server kita.

<br>

### 2. returning html
![Commit 2 screen capture](/assets/images/commit2.jpg)

Variable `status_line` yang ditambahkan mendefinisikan response sukses yang ingin dikirim. Metode `as_bytes` dipanggil untuk mengubah data response menjadi bytes yang dikirim langsung ke koneksi. Unwrap digunakan di metode ini karena operasi `write_all` bisa saja gagal.

`fs` ditambahkan di bagian use dari file untuk mengimport standard library untuk filesystem. fs ini digunakan saat mendefinisi variable `contents` untuk membaca file `hello.html` sebagai string. File contents kemudian dihitung panjangnya dan disimpan dalam variable `length`.

Terakhir, digunakan metode `format!` untuk menambahkan status_line, length dari content, dan content tersebut sendiri sebagai success response. Content-length yang sesuai dengan ukuran hello.html perlu ditambahkan di header untuk memastikan response HTTP yang dikirim valid.

<br>

### 3. validating request and selectively responding
![Commit 3 screen capture](/assets/images/commit3.jpg)
Response yang diberikan dibedakan menjadi dua, yang pertama adalah menampilkan hello.html saat request_line yang didapat berupa "GET / HTTP/1.1", dan menampilkan 404.html saat bukan. Request "GET / HTTP/1.1" hanya didapatkan saat url yang diakses adalah http://127.0.0.1:7878, jadi jika berupa url lain seperti http://127.0.0.1:7878/bad, ia akan masuk ke `else` block dan menampilkan response 404.

Refactoring yang dilakukan diperlukan karena code yang sebelumnya mengulangi banyak lines yang secara fungsional sama. If-else block yang panjang membuat metode `handle_connection` menjadi lebih panjang, susah dibaca, dan menyusahkan programmer bila ingin membuat kasus handling tambahan lagi. Dengan hasil refactor yang sekarang,

```
let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
```

Fungsi dari variable status_line dan filename menjadi lebih jelas, dan lebih mudah lagi untuk menambahkan handling routing lain (dengan menambahkan else-if block tambahan). Metode handle_connection juga menjadi lebih singkat dan mudah dimengerti.
