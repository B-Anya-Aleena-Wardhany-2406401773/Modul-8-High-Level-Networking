# Reflection

Anya Aleena Wardhany

2406401773

## 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?
- **Unary:** Klien mengirim satu request ke server dan menunggu satu response balasan. Ini cocok untuk interaksi sederhana seperti mengambil satu data dari database, proses autentikasi, atau melakukan satu kalkulasi.  
- **Server Streaming:** Klien mengirim satu request, tetapi server merespons dengan mengirimkan banyak data dalam bentuk aliran (stream) yang berkesinambungan. Skenario yang pas adalah saat server harus mendorong banyak data secara real-time seperti update harga saham, berita, peringatan cuaca, atau mengirim file berukuran besar.  
- **Bi-directional Streaming:** Klien dan server dapat saling mengirim banyak pesan dalam aliran data secara terus-menerus dan mandiri. Skenario ini sangat ideal untuk komunikasi interaktif real-time, seperti aplikasi chat atau analitik real-time dua arah.  

## 2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?
Berdasarkan materi yang ada, gRPC secara arsitektur sangat menguntungkan dari sisi keamanan karena sudah memiliki dukungan bawaan (built-in support) untuk autentikasi. Fitur bawaan ini dapat dimanfaatkan untuk meningkatkan keamanan serta skalabilitas pada sistem terdistribusi secara lebih mudah.

## 3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?
Tantangan utamanya adalah mengelola alur komunikasi asinkronus yang berumur panjang (long-lived communication). Di Rust, kita harus menggunakan task asinkronus terpisah (seperti `tokio::spawn`) agar proses penerimaan pesan tidak memblokir tugas utama. Selain itu, kita harus cermat menentukan ukuran buffer channel (seperti `mpsc::channel(10)`) agar performanya optimal. Kita juga perlu menggunakan penanganan error yang anggun (seperti unwrap_or_else) agar jika terjadi masalah pada salah satu penerimaan pesan, layanan utama tidak ikut crash.

## 4. What are the advantages and disadvantages of using the `tokio_stream::wrappers::ReceiverStream` for streaming responses in Rust gRPC services?
Keuntungan utamanya adalah utilitas ini sangat mempermudah konversi antara receiver dari tokio menjadi bentuk stream yang dapat diproses dan dikirimkan oleh framework tonic. Hasil stream ini kemudian bisa dengan mudah dibungkus ke dalam objek Response gRPC untuk dikembalikan ke klien secara asinkronus. Kelemahannya tidak secara eksplisit dibahas di dalam materi, namun secara konsep, ini menambahkan lapisan pembungkus (wrapper) untuk menerjemahkan mekanisme channel Rust/Tokio ke antarmuka streaming gRPC.

## 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?
Struktur kode gRPC di Rust bisa dibuat modular dengan mendeklarasikan modul khusus (misalnya `mod services`) lalu menjadikannya publik (`pub mod`) agar bisa diakses dari tempat lain. Kita juga bisa memisahkan file klien (`grpc_client.rs`) dan server (`grpc_server.rs`). Penggunaan makro `tonic::include_proto!` memungkinkan kita memisahkan definisi kontrak API (di file `.proto`) dengan logika kodenya. Selain itu, menggunakan struct khusus (seperti `MyPaymentService`) untuk setiap layanan juga membuat kode lebih rapi dan mudah untuk dikembangkan di kemudian hari.

## 6. In the **MyPaymentService** implementation, what additional steps might be necessary to handle more complex payment processing logic?
Pada contoh tutorial, `MyPaymentService` hanya dibuat untuk langsung mengembalikan respons sukses demi tujuan demonstrasi. Di dunia nyata, langkah ekstra yang dibutuhkan adalah mengganti bagian respons instan tersebut dengan logika pemrosesan request yang sesungguhnya (process the request and return a response), misalnya menyambungkan ke layanan pembayaran eksternal atau database, serta memetakan error ke Status gRPC yang sesuai jika pembayaran gagal.

## 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?
Adopsi gRPC sangat meningkatkan konektivitas pada arsitektur sistem yang kompleks, seperti microservices. Dampak terbesarnya pada interoperabilitas adalah dukungan multibahasa; gRPC memiliki pustaka untuk berbagai bahasa pemrograman dan mendukung auto-generation kode klien berkat Protocol Buffers. Karena gRPC menyembunyikan kompleksitas komunikasi seolah-olah hanya memanggil fungsi lokal (function call), tim pengembang bisa bebas menggunakan teknologi atau bahasa apa pun di masing-masing layanan dan tetap bisa saling berkomunikasi dengan lancar.

## 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?
- **Keuntungan**: Berbeda dengan HTTP/1.1 yang memproses request secara berurutan, HTTP/2 memiliki fitur multiplexing yang memungkinkan pengiriman banyak request dan response secara bersamaan dalam satu koneksi TCP. HTTP/2 juga menggunakan format biner yang lebih efisien dari teks, memiliki kompresi header (HPack) untuk mengecilkan ukuran data, dan fitur server push.  
- **Kelemahan**: Meskipun gRPC sangat kuat karena HTTP/2, kelemahan utamanya adalah kurangnya dukungan penuh dari browser saat ini, sehingga penggunaannya sering kali harus dibatasi untuk komunikasi sistem ke sistem (internal) saja.

## 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?
REST API berjalan pada siklus request/response secara unary, di mana komunikasi diinisiasi murni oleh klien satu per satu, sehingga kurang responsif untuk pembaruan real-time. Sebaliknya, bidirectional streaming gRPC memungkinkan kedua belah pihak (klien dan server) mengirimkan aliran data secara konstan tanpa perlu membuka koneksi atau membuat request baru berulang kali. Ini membuatnya jauh lebih responsif dan interaktif untuk kebutuhan masa kini.

## 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?
Pendekatan skema menggunakan Protobuf (di gRPC) secara otomatis memvalidasi setiap pesan sesuai kontrak API yang disepakati. Karena formatnya biner, ini mengonsumsi lebih sedikit sumber daya CPU dan ukurannya lebih kecil sehingga transfer data lebih cepat. Di sisi lain, JSON (di REST) menggunakan format plain-text yang memang lebih mudah dibaca oleh manusia, tetapi memiliki kelemahan yaitu membutuhkan langkah ekstra untuk memvalidasi setiap data yang masuk.