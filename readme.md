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

# Commit 3 Reflection Notes

## Validating Request and Selectively Responding

### Tantangan
- Memahami cara membaca baris pertama HTTP request untuk menentukan path yang diminta
- Membuat logika percabangan yang tepat untuk membedakan request valid dan tidak valid
- Membuat file `404.html` yang terpisah sebagai halaman error
- Memahami mengapa refactoring diperlukan agar kode tidak repetitif

---

### Apa yang Dilakukan
- Mengambil baris pertama HTTP request (`request_line`) sebagai penentu routing
- Menambahkan logika `if-else` untuk membedakan request `GET / HTTP/1.1` dengan request lainnya
- Membuat file `404.html` sebagai halaman yang ditampilkan ketika path tidak ditemukan
- Melakukan refactoring dengan memisahkan penentuan `status_line` dan `filename` dari logika pembentukan response

---

### Apa yang Didapat
- Memahami bahwa routing di level paling dasar hanyalah pengecekan string pada baris pertama HTTP request
- Mengetahui perbedaan HTTP status code `200 OK` (sukses) dan `404 NOT FOUND` (tidak ditemukan)
- Memahami pentingnya refactoring: sebelum refactoring, blok `if` dan `else` masing-masing memiliki kode pembentukan response yang hampir identik — ini melanggar prinsip *DRY (Don't Repeat Yourself)*
- Setelah refactoring, hanya `status_line` dan `filename` yang berbeda antar kondisi, sementara logika membaca file dan membentuk response cukup ditulis sekali
- Memahami bahwa tuple `(status_line, filename)` bisa digunakan sebagai cara elegan untuk mengembalikan dua nilai sekaligus dari sebuah ekspresi `if-else` di Rust

---

### Mengapa Refactoring Diperlukan
Sebelum refactoring, kode terlihat seperti ini:

```rust
if request_line == "GET / HTTP/1.1" {
    // baca hello.html, bentuk response, kirim
} else {
    // baca 404.html, bentuk response, kirim
}
```

Masalahnya, logika membentuk dan mengirim response (`format!`, `write_all`) ditulis dua kali dengan isi yang hampir sama. Jika suatu saat kita ingin mengubah format response, kita harus mengubah di dua tempat sekaligus — rawan bug dan susah di-maintain.

Setelah refactoring:

```rust
let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
    ("HTTP/1.1 200 OK", "hello.html")
} else {
    ("HTTP/1.1 404 NOT FOUND", "404.html")
};
// logika membentuk dan mengirim response cukup ditulis sekali
```

Dengan memisahkan bagian yang *berbeda* (status dan filename) dari bagian yang *sama* (membaca file, format, kirim), kode menjadi lebih bersih, lebih mudah dibaca, dan lebih mudah dikembangkan.

---

### Kesimpulan
Milestone ini mengajarkan konsep dasar routing dan error handling di level HTTP. Server kita kini sudah bisa membedakan request yang valid dan tidak valid, lalu merespons dengan halaman yang sesuai. Proses refactoring yang dilakukan juga memperjelas bahwa kode yang bersih bukan hanya tentang "bisa jalan", tapi juga tentang kemudahan pemeliharaan ke depannya. Inilah fondasi dari bagaimana framework web modern menangani routing dan error page secara otomatis di balik layar.

# Commit 4 Reflection Notes

## Simulation of Slow Request

### Tantangan
- Memahami mengapa single-threaded server menjadi masalah besar ketika ada request yang lambat
- Mensimulasikan kondisi nyata di mana satu request bisa memblokir request lainnya
- Memahami penggunaan `thread::sleep` dan `Duration` untuk membuat delay buatan
- Memahami perbedaan antara `match` dan `if-else` dan kapan sebaiknya menggunakan masing-masing

---

### Apa yang Dilakukan
- Menambahkan `thread` dan `time::Duration` ke dalam `use std::{}` imports
- Mengganti logika `if-else` dengan `match` untuk routing yang lebih bersih dan scalable
- Menambahkan endpoint baru `/sleep` yang mensimulasikan slow request dengan `thread::sleep(Duration::from_secs(10))`
- Menguji server dengan membuka dua tab browser secara bersamaan: satu ke `/sleep` dan satu ke `/`

---

### Apa yang Didapat
- Memahami secara langsung dampak nyata dari single-threaded server: ketika satu tab mengakses `/sleep`, tab lain yang mengakses `/` ikut tertahan dan harus menunggu 10 detik
- Memahami bahwa `thread::sleep` menghentikan eksekusi seluruh thread, bukan hanya satu koneksi — ini yang menjadi inti masalah
- Mengetahui bahwa `match` lebih cocok dari `if-else` untuk routing karena lebih mudah dikembangkan dengan menambah *arm* baru, dan wildcard `_` menangani semua kasus yang tidak terdefinisi
- Mengetahui bahwa server produksi nyata harus mampu menangani banyak request secara bersamaan, yang tidak bisa dicapai dengan pendekatan single-threaded seperti ini

---

### Mengapa Ini Menjadi Masalah
Server kita hanya memiliki satu thread. Artinya, setiap request diproses secara berurutan — satu per satu. Ketika browser pertama mengakses `/sleep`, server masuk ke `thread::sleep` selama 10 detik. Selama waktu itu, server tidak bisa menerima atau memproses request apapun. Browser kedua yang mengakses `/` harus antre dan menunggu hingga request pertama selesai, meskipun request kedua itu sebenarnya sangat ringan.

Di dunia nyata, jika ada satu request berat (misalnya query database yang lama), semua pengguna lain akan ikut merasakan kelambatannya. Inilah alasan mengapa web server modern menggunakan multi-threading atau async I/O.

---

### Kesimpulan
Milestone ini secara efektif memperlihatkan kelemahan fundamental dari single-threaded server melalui simulasi yang sederhana namun nyata. Dengan hanya dua tab browser, kita sudah bisa merasakan betapa tidak responsifnya server ketika ada satu request yang lambat. Ini menjadi motivasi kuat untuk milestone berikutnya, yaitu implementasi thread pool agar server dapat menangani banyak request secara bersamaan tanpa saling memblokir.


# Commit 5 Reflection Notes

## Multithreaded Server Using Threadpool

### Tantangan
- Memahami konsep *thread pool* dan mengapa lebih baik dibanding membuat thread baru untuk setiap request
- Memahami penggunaan `Arc<Mutex<T>>` untuk berbagi data antar thread secara aman
- Memahami cara kerja `mpsc::channel` sebagai jalur komunikasi antara main thread dan worker threads
- Memahami mengapa `Job` didefinisikan sebagai `Box<dyn FnOnce() + Send + 'static>`
- Mengimplementasikan `Drop` trait agar server bisa shutdown dengan bersih tanpa resource leak

---

### Apa yang Dilakukan
- Membuat file `src/lib.rs` yang berisi implementasi `ThreadPool`, `Worker`, dan tipe `Job`
- Mengubah `main.rs` agar menggunakan `ThreadPool::new(4)` sebagai pengganti pemanggilan langsung `handle_connection`
- Mengganti `handle_connection(stream)` dengan `pool.execute(|| { handle_connection(stream); })`
- Mengimplementasikan `Drop` trait pada `ThreadPool` untuk graceful shutdown semua worker

---

### Apa yang Didapat
- Memahami bahwa `ThreadPool` membuat sejumlah thread worker di awal (`Vec<Worker>`), lalu mendistribusikan job ke worker yang tersedia — ini menghindari overhead membuat thread baru setiap ada request
- Memahami peran `mpsc::channel`: `sender` dipegang oleh `ThreadPool` untuk mengirim job, sedangkan `receiver` dibungkus `Arc<Mutex<>>` agar bisa dibagi ke semua worker secara aman
- Memahami `Arc` (*Atomic Reference Counted*): memungkinkan satu data dimiliki oleh banyak thread sekaligus dengan reference counting yang thread-safe
- Memahami `Mutex`: memastikan hanya satu worker yang bisa mengakses `receiver` pada satu waktu, mencegah *race condition*
- Memahami bahwa `Box<dyn FnOnce() + Send + 'static>` adalah tipe yang fleksibel untuk menyimpan closure apapun yang bisa dikirim antar thread dan hanya perlu dipanggil sekali
- Memahami graceful shutdown: `Drop` di-trigger secara otomatis saat `ThreadPool` keluar dari scope — `sender` di-drop terlebih dahulu sehingga semua worker menerima error dari `recv()` dan keluar dari loop dengan bersih

---

### Bagaimana ThreadPool Bekerja
1. `ThreadPool::new(4)` membuat 4 `Worker`, masing-masing langsung menjalankan thread yang menunggu job dari channel
2. Setiap kali ada koneksi masuk, `pool.execute(|| { handle_connection(stream); })` mengirim closure sebagai `Job` melalui `sender`
3. Salah satu worker yang sedang idle akan mengambil job tersebut dari `receiver` (dilindungi `Mutex`), lalu menjalankannya
4. Karena ada 4 worker, server kini bisa menangani hingga 4 request secara bersamaan tanpa saling memblokir
5. Ketika server dimatikan, `Drop` memastikan semua worker selesai mengerjakan job yang sedang berjalan sebelum thread benar-benar dihentikan via `thread.join()`

---

### Kesimpulan
Milestone ini adalah puncak dari perjalanan membangun web server dari nol. Dengan mengimplementasikan `ThreadPool` sendiri, kita tidak hanya menyelesaikan masalah single-threaded blocking dari milestone sebelumnya, tetapi juga memahami secara mendalam bagaimana concurrency yang aman dibangun di Rust menggunakan `Arc`, `Mutex`, dan `mpsc::channel`. Server kita kini bisa melayani banyak request secara paralel tanpa data race — sesuatu yang dijamin oleh Rust di waktu kompilasi, bukan runtime. Ini menunjukkan mengapa Rust dikenal sebagai bahasa yang ideal untuk sistem yang membutuhkan performa tinggi sekaligus keamanan memori.

# Commit 5 Reflection Notes

## Multithreaded Server Using Threadpool

### Tantangan
- Memahami konsep *thread pool* dan mengapa lebih baik dibanding membuat thread baru untuk setiap request
- Memahami penggunaan `Arc<Mutex<T>>` untuk berbagi data antar thread secara aman
- Memahami cara kerja `mpsc::channel` sebagai jalur komunikasi antara main thread dan worker threads
- Memahami mengapa `Job` didefinisikan sebagai `Box<dyn FnOnce() + Send + 'static>`
- Mengimplementasikan `Drop` trait agar server bisa shutdown dengan bersih tanpa resource leak

---

### Apa yang Dilakukan
- Membuat file `src/lib.rs` yang berisi implementasi `ThreadPool`, `Worker`, dan tipe `Job`
- Mengubah `main.rs` agar menggunakan `ThreadPool::new(4)` sebagai pengganti pemanggilan langsung `handle_connection`
- Mengganti `handle_connection(stream)` dengan `pool.execute(|| { handle_connection(stream); })`
- Mengimplementasikan `Drop` trait pada `ThreadPool` untuk graceful shutdown semua worker

---

### Apa yang Didapat
- Memahami bahwa `ThreadPool` membuat sejumlah thread worker di awal (`Vec<Worker>`), lalu mendistribusikan job ke worker yang tersedia — ini menghindari overhead membuat thread baru setiap ada request
- Memahami peran `mpsc::channel`: `sender` dipegang oleh `ThreadPool` untuk mengirim job, sedangkan `receiver` dibungkus `Arc<Mutex<>>` agar bisa dibagi ke semua worker secara aman
- Memahami `Arc` (*Atomic Reference Counted*): memungkinkan satu data dimiliki oleh banyak thread sekaligus dengan reference counting yang thread-safe
- Memahami `Mutex`: memastikan hanya satu worker yang bisa mengakses `receiver` pada satu waktu, mencegah *race condition*
- Memahami bahwa `Box<dyn FnOnce() + Send + 'static>` adalah tipe yang fleksibel untuk menyimpan closure apapun yang bisa dikirim antar thread dan hanya perlu dipanggil sekali
- Memahami graceful shutdown: `Drop` di-trigger secara otomatis saat `ThreadPool` keluar dari scope — `sender` di-drop terlebih dahulu sehingga semua worker menerima error dari `recv()` dan keluar dari loop dengan bersih

---

### Bagaimana ThreadPool Bekerja
1. `ThreadPool::new(4)` membuat 4 `Worker`, masing-masing langsung menjalankan thread yang menunggu job dari channel
2. Setiap kali ada koneksi masuk, `pool.execute(|| { handle_connection(stream); })` mengirim closure sebagai `Job` melalui `sender`
3. Salah satu worker yang sedang idle akan mengambil job tersebut dari `receiver` (dilindungi `Mutex`), lalu menjalankannya
4. Karena ada 4 worker, server kini bisa menangani hingga 4 request secara bersamaan tanpa saling memblokir
5. Ketika server dimatikan, `Drop` memastikan semua worker selesai mengerjakan job yang sedang berjalan sebelum thread benar-benar dihentikan via `thread.join()`

---

### Kesimpulan
Milestone ini adalah puncak dari perjalanan membangun web server dari nol. Dengan mengimplementasikan `ThreadPool` sendiri, kita tidak hanya menyelesaikan masalah single-threaded blocking dari milestone sebelumnya, tetapi juga memahami secara mendalam bagaimana concurrency yang aman dibangun di Rust menggunakan `Arc`, `Mutex`, dan `mpsc::channel`. Server kita kini bisa melayani banyak request secara paralel tanpa data race — sesuatu yang dijamin oleh Rust di waktu kompilasi, bukan runtime. Ini menunjukkan mengapa Rust dikenal sebagai bahasa yang ideal untuk sistem yang membutuhkan performa tinggi sekaligus keamanan memori.



# Commit Bonus Reflection Notes

## Function Improvement: `build` vs `new`

### Tantangan
- Memahami perbedaan filosofi antara fungsi `new` dan `build` dalam Rust
- Mengubah return type dari `ThreadPool` menjadi `Result<ThreadPool, &'static str>`
- Menyesuaikan pemanggilan di `main.rs` agar menggunakan `.unwrap()` setelah `build()`
- Memahami kapan sebaiknya menggunakan `panic` vs mengembalikan `Result`

---

### Apa yang Dilakukan
- Mengganti fungsi `new` dengan fungsi `build` pada `impl ThreadPool` di `lib.rs`
- Mengubah return type menjadi `Result<ThreadPool, &'static str>` agar error bisa ditangani secara eksplisit
- Mengganti `assert!(size > 0)` dengan `if size == 0 { return Err("ThreadPool size must be greater than zero"); }`
- Mengubah nilai return dari `ThreadPool { ... }` menjadi `Ok(ThreadPool { ... })`
- Memperbarui `main.rs` untuk menggunakan `ThreadPool::build(4).unwrap()` sebagai pengganti `ThreadPool::new(4)`

---

### Apa yang Didapat
- Memahami konvensi Rust: fungsi `new` secara idiomatis diasumsikan **selalu berhasil** — jika gagal, ia boleh `panic`. Oleh karena itu, `new` tidak cocok digunakan ketika ada kondisi input yang bisa invalid
- Memahami bahwa fungsi `build` lebih tepat digunakan ketika proses pembuatan objek bisa gagal, karena ia mengembalikan `Result` yang memaksa pemanggil untuk menangani kemungkinan error secara eksplisit
- Memahami perbedaan `assert!` vs `Err`: `assert!` langsung menyebabkan *panic* dan crash program, sedangkan `Err` memberikan kontrol kepada pemanggil untuk memutuskan cara menangani error — jauh lebih fleksibel untuk kode produksi
- Memahami bahwa `&'static str` sebagai tipe error cocok untuk pesan error sederhana yang sudah diketahui di waktu kompilasi

---

### Perbandingan `new` vs `build`

| Aspek | `new` | `build` |
|---|---|---|
| Return type | `ThreadPool` | `Result<ThreadPool, &'static str>` |
| Jika input invalid | `panic!` / `assert!` | Mengembalikan `Err(...)` |
| Error handling | Tidak ada, langsung crash | Diserahkan ke pemanggil |
| Konvensi Rust | Untuk pembuatan yang dijamin berhasil | Untuk pembuatan yang bisa gagal |
| Penggunaan di `main.rs` | `ThreadPool::new(4)` | `ThreadPool::build(4).unwrap()` |

---

### Kesimpulan
Bonus ini mengajarkan prinsip penting dalam desain API Rust: **gunakan `Result` ketika operasi bisa gagal, jangan sembunyikan kemungkinan error di balik `panic`**. Dengan mengubah `new` menjadi `build` yang mengembalikan `Result`, kode kita menjadi lebih robust dan lebih sesuai dengan idiom Rust yang mengutamakan penanganan error secara eksplisit. Di aplikasi produksi nyata, pendekatan ini sangat penting karena `panic` akan menghentikan seluruh program, sementara `Result` memungkinkan program untuk pulih dari kondisi error dengan lebih graceful.

