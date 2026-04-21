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