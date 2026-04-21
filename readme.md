# Commit 1 Reflection Notes

## Handle-Connection, Check Response

### Tantangan
- Memahami konsep *ownership* dan *borrowing* di Rust, terutama saat passing `TcpStream` ke fungsi `handle_connection`
- Membiasakan diri dengan tipe `Result<T>` dan penggunaan `.unwrap()` yang tersebar di mana-mana
- Memahami mengapa `BufReader` diperlukan dan tidak bisa langsung membaca dari `TcpStream`
- Mengerti kapan iterator chain seperti `.take_while(|line| !line.is_empty())` harus berhenti

---

### Apa yang Dilakukan
- Membuat project Rust baru dengan `cargo new hello`
- Menghubungkan repository lokal ke remote GitLab/GitHub
- Menulis TCP listener yang mendengarkan koneksi di `127.0.0.1:7878`
- Menambahkan fungsi `handle_connection` untuk membaca dan mencetak HTTP request dari browser
- Menggunakan `BufReader` dan iterator chaining untuk mem-parsing header HTTP request

---

### Apa yang Didapat
- Memahami cara kerja `TcpListener::bind()` dan `listener.incoming()` di Rust
- Mengetahui struktur HTTP request: *request line*, *headers*, dan pemisah baris kosong
- Memahami fungsi `BufReader` sebagai lapisan buffering untuk efisiensi pembacaan I/O
- Mengetahui bahwa HTTP pada dasarnya hanya teks terstruktur yang dikirim lewat TCP
- Memahami alasan browser mengirim beberapa koneksi sekaligus saat tidak mendapat respons

---

### Kesimpulan
Milestone pertama ini memberikan gambaran nyata tentang bagaimana sebuah web server bekerja di level paling dasar. Dengan Rust, kita bisa membangun server dari nol tanpa framework apapun, sekaligus belajar konsep jaringan TCP dan protokol HTTP secara langsung. Server yang dibuat masih sangat sederhana karena belum mengirimkan respons apapun ke browser, namun fondasi ini menjadi titik awal yang penting untuk memahami cara kerja web server sesungguhnya di balik framework seperti Django.


# Commit 2 Reflection Notes

## Returning HTML

### Tantangan
- Memahami format HTTP response yang valid agar browser bisa me-render HTML dengan benar
- Mengetahui bahwa `hello.html` harus berada di direktori yang sama saat `cargo run` dijalankan
- Memahami pentingnya header `Content-Length` dan mengapa browser membutuhkannya
- Memastikan response ditulis ke stream dengan format `\r\n` yang tepat sesuai standar HTTP

---

### Apa yang Dilakukan
- Menambahkan `fs` dari standard library untuk membaca file HTML dari disk
- Membuat file `hello.html` dengan konten sederhana sebagai halaman yang akan ditampilkan
- Memodifikasi `handle_connection` agar membentuk HTTP response lengkap dan mengirimkannya ke browser
- Menggunakan `stream.write_all()` untuk menulis response sebagai bytes ke koneksi TCP

---

### Apa yang Didapat
- Memahami struktur HTTP response: *status line*, *headers*, baris kosong, lalu *body*
- Mengetahui fungsi `Content-Length` sebagai header yang memberitahu browser seberapa banyak byte yang harus dibaca sebagai body
- Memahami bahwa `fs::read_to_string()` membaca seluruh isi file sekaligus ke dalam `String`
- Mengetahui cara menggunakan `format!()` untuk menyusun response string secara dinamis
- Memahami bahwa HTTP response pada dasarnya hanya bytes yang dikirim melalui TCP stream

---

### Kesimpulan
Pada milestone ini, server kita akhirnya bisa mengembalikan sesuatu yang nyata ke browser. Kuncinya adalah memahami bahwa HTTP response memiliki struktur yang ketat: status line, diikuti headers, diikuti satu baris kosong (`\r\n\r\n`), barulah body HTML. Tanpa `Content-Length` yang benar, browser tidak akan tahu kapan harus berhenti membaca data. Ini memperlihatkan bahwa komunikasi web sebenarnya hanyalah pertukaran teks terstruktur di atas TCP, dan framework seperti Django hanya menyembunyikan kompleksitas ini di balik abstraksi yang lebih tinggi.


<img width="1600" height="269" alt="WhatsApp Image 2026-04-21 at 6 23 54 PM" src="https://github.com/user-attachments/assets/bf960d2e-f18f-493b-b16f-903b01c93ee9" />
<img width="1600" height="371" alt="WhatsApp Image 2026-04-21 at 6 24 04 PM" src="https://github.com/user-attachments/assets/35234def-8781-46f6-8f9b-409000d0debf" />

