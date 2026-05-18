# Laporan Praktikum Jaringan Komputer - Modul 1

### Identitas Praktikan

| Item | Keterangan |
| :--- | :--- |
| **Nama** | Alif Luthfan Adeefa |
| **NIM** | 103072400163 |
| **Kelas** | IF-04-01 |

---

### 1. Tujuan Pembalajaran dan praktikum
Berdasarkan modul praktikum Jaringan Komputer Semester Genap 2025/2026, setelah menyelesaikan modul ini mahasiswa diharapkan mampu:

Menginvestigasi cara kerja protokol HTTP menggunakan Wireshark.
Mahasiswa memahami Conditional GET, dokumen panjang, embedded objects, dan autentikasi HTTP.

### 2. Dasar Teori

| Aspek | Deskripsi Singkat |
| :--- | :--- |
| **Basic GET/Response** | Interaksi dasar: klien meminta dokumen, server merespons dengan status code (misal: 200 OK). |
| **Conditional GET** | Mekanisme caching dengan header `If-Modified-Since`; server merespons `304 Not Modified` jika tidak ada perubahan. |
| **HTTP & TCP** | Dokumen besar dipecah menjadi beberapa segmen TCP (`TCP segment of a reassembled PDU`). |
| **Embedded Objects** | Halaman HTML dengan gambar/objek lain memicu *multiple* HTTP GET requests. |
| **HTTP Authentication** | Kredensial dikirim via header `Authorization: Basic` (Base64 encoded). |

---

### 3. Langkah Kerja & Prosedur

### 3.1 Ringkasan Prosedur per Skenario
Praktikum dilakukan dengan mengakses beberapa URL target sesuai skenario berikut:

| Skenario | URL Target | Langkah Kunci | Output yang Diharapkan |
| :--- | :--- | :--- | :--- |
| **Basic GET** | `.../file1.html` | Clear cache → Capture → Akses URL → Stop. | Paket GET + Response 200 OK. |
| **Conditional GET** | `.../file2.html` | Akses 2x (refresh) → Analisis header kedua. | Header `If-Modified-Since` + Status 304. |
| **Long Document** | `.../file3.html` | Akses dokumen besar (~4500 byte). | `[TCP segment of a reassembled PDU]`. |
| **Embedded Objects** | `.../file4.html` | Akses halaman dengan 2 gambar. | Multiple GET requests (HTML + gambar). |
| **Authentication** | `.../file5.html` | Login dengan kredensial yang ditentukan. | Header `Authorization: Basic`. |

### 3.2 Kredensial Autentikasi
Untuk skenario autentikasi, digunakan parameter login sebagai berikut:

* **Username**: `wireshark-students`
* **Password**: `network`
* **Encoding**: Base64
* **Header Format**: `Authorization: Basic <encoded_string>`

---

### 4. Hasil dan Pembahasan

### 4.1 Basic HTTP GET/Response
![Wireshark](./Assets/BasicGET.jpeg)

| Percobaan | Header Khusus | Status Code | Keterangan |
| :--- | :--- | :--- | :--- |
| **Akses Pertama** | - | `200 OK` | Server mengirimkan konten secara penuh. |
| **Akses Kedua (Refresh)** | `If-Modified-Since` | `304 Not Modified` | Konten tidak berubah, browser menggunakan cache lokal. |



### 5. Kesimpulan