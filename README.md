# Reflection

Reflection notes untuk Tutorial 6.

### 1. handle_connection method
Di metode `handle_connection`, kita membuat variable baru bernama `buf_reader`; sebuah instance `BufReader` yang membungkus referensi mutable kepada variable `stream`. BufReader ini menambahkan buffering dengan mengatur calls kepada `std::io::Read`. Setelahnya, kita juga membuat variable bernama `http_request` bertipe Vector yang mengumpulkan baris hasil request browser ke server kita.

`BufReader` mengimplementasikan `std::io::BufRead` mengembalikan `Result<String, std::io::Error>` dengan memotong stream data setiap kali ia melihat byte newline. Untuk mendapatkan setiap string, kita map dan unwrap setiap `Result`.

Browser akan menandakan titik akhir dari HTTP request dengan mengirimkan dua karakter newline berturut-turut, jadi untuk mendapatkan satu request dari stream, kita mengambil terus baris hingga mendapatkan baris yang kosong (empty string). Setelah mengumpulkan baris-barisnya ke dalam vektor http_request, http_request dapat dicetak dengan format `"Request: {:#?}"` untuk memperlihatkan semua instruksi yang dikirimkan web browser ke server kita.

<br>

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length:
        {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}

### 2. returning html

![Commit 2 screen capture](/assets/images/commit2.jpg)

Variable `status_line` yang ditambahkan mendefinisikan response sukses yang ingin dikirim. Metode `as_bytes` dipanggil untuk mengubah data response menjadi bytes yang dikirim langsung ke koneksi. Unwrap digunakan di metode ini karena operasi `write_all` bisa saja gagal.

`fs` ditambahkan di bagian use dari file untuk mengimport standard library untuk filesystem. fs ini digunakan saat mendefinisi variable `contents` untuk membaca file `hello.html` sebagai string. File contents kemudian dihitung panjangnya dan disimpan dalam variable `length`.

Terakhir, digunakan metode `format!` untuk menambahkan status_line, length dari content, dan content tersebut sendiri sebagai success response. Content-length yang sesuai dengan ukuran hello.html perlu ditambahkan di header untuk memastikan response HTTP yang dikirim valid.
