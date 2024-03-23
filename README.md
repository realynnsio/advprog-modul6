# Reflection

Reflection notes untuk Tutorial 6.

### 1. handle_connection method
Di metode `handle_connection`, kita membuat variable baru bernama `buf_reader`; sebuah instance `BufReader` yang membungkus referensi mutable kepada variable `stream`. BufReader ini menambahkan buffering dengan mengatur calls kepada `std::io::Read`. Setelahnya, kita juga membuat variable bernama `http_request` bertipe Vector yang mengumpulkan baris hasil request browser ke server kita.

`BufReader` mengimplementasikan `std::io::BufRead` mengembalikan `Result<String, std::io::Error>` dengan memotong stream data setiap kali ia melihat byte newline. Untuk mendapatkan setiap string, kita map dan unwrap setiap `Result`.

Browser akan menandakan titik akhir dari HTTP request dengan mengirimkan dua karakter newline berturut-turut, jadi untuk mendapatkan satu request dari stream, kita mengambil terus baris hingga mendapatkan baris yang kosong (empty string). Setelah mengumpulkan baris-barisnya ke dalam vektor http_request, http_request dapat dicetak dengan format `"Request: {:#?}"` untuk memperlihatkan semua instruksi yang dikirimkan web browser ke server kita.
