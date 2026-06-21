# Laporan Praktikum Jaringan Komputer - Modul 5
## User Datagram Protocol (UDP)

### Identitas Praktikan

| Keterangan | Imformasi |
| :--- | :--- |
| **Nama** | Alif Luthfan Adeefa |
| **NIM** | 103072400163 |
| **Kelas** | IF-04-01 |

---

## 1. Tujuan Praktikum

| No | Tujuan | Penjelasan Sederhana |
|----|--------|---------------------|
| 1 | Investigasi cara kerja UDP | Mengerti bagaimana UDP kirim data tanpa "jabatan tangan" dulu |
| 2 | Identifikasi struktur header UDP | Tahu isi 4 field di header UDP dan fungsinya |
| 3 | Analisis port source-destination | Paham bagaimana port saling "balas" saat komunikasi |
| 4 | Hitung kapasitas payload UDP | Bisa hitung berapa maksimal data yang bisa dikirim UDP |

---

## 2. Dasar Teori (Versi Simpel)

### 2.1 Apa Itu UDP?

| Pertanyaan | Jawaban |
|------------|---------|
| **Kepanjangan** | User Datagram Protocol |
| **Lapisan OSI** | Transport Layer (Layer 4) |
| **Sifat Utama** | Connectionless (tanpa koneksi tetap), unreliable (tidak jamin sampai), fast (cepat) |
| **Analogi Sederhana** | Seperti kirim surat pos: kirim langsung, tidak tunggu konfirmasi |
| **Beda dengan TCP** | TCP = kirim paket terdaftar (ada konfirmasi), UDP = kirim surat biasa (langsung kirim) |

### 2.2 Kapan Pakai UDP?

| Aplikasi | Alasan Pakai UDP |
|----------|-----------------|
| Online Gaming | Butuh cepat, kalau telat sedikit tidak masalah |
| Video Streaming | Lebih baik gambar skip sedikit daripada buffering lama |
| VoIP / Zoom | Suara real-time lebih penting daripada sempurna |
| DNS Query | Query kecil, cepat, tidak perlu koneksi lama |
| Broadcast/Multicast | Kirim ke banyak perangkat sekaligus |

---

## 3. Langkah Kerja Praktikum

### 3.1 Ringkasan Prosedur

| Tahap | Aktivitas | Perintah / Filter | Tujuan |
|-------|-----------|------------------|--------|
| **Persiapan** | Buka Wireshark, pilih interface Wi-Fi | - | Siap capture traffic |
| **Flush DNS** | Hapus cache DNS lama | `ipconfig /flushdns` | Pastikan query DNS benar-benar terjadi |
| **Cek IP Lokal** | Lihat alamat IPv4 sendiri | `ipconfig` | Dipakai untuk filter Wireshark |
| **Generate Traffic** | Jalankan nslookup ke DNS server eksplisit | `nslookup -type=A youtube.com 8.8.8.8` | Memicu traffic UDP DNS ke 8.8.8.8 lewat IPv4 |
| **Stop Capture** | Klik tombol Stop di Wireshark | - | Selesai merekam paket |
| **Filter Paket** | Terapkan filter UDP + DNS | `udp && ip.addr == 192.168.1.5 && dns.qry.name == "youtube.com"` | Tampilkan hanya paket yang relevan |
| **Analisis** | Pilih paket → lihat detail UDP | - | Identifikasi field header dan hitung payload |

**Catatan teknis:** percobaan pertama dengan `nslookup youtube.com` (tanpa server eksplisit) tidak menghasilkan paket yang tertangkap di filter IPv4, karena query DNS ternyata dikirim lewat IPv6 link-local (`fe80::1`) ke router. Solusinya adalah memaksa resolusi lewat IPv4 dengan `-type=A` dan menentukan server DNS publik (`8.8.8.8`) secara eksplisit.

---

## 4. Hasil dan Pembahasan

### 4.1 Hasil Capture Paket UDP

> **Gambar 1**: DNS Query (UDP)
> ![DNS Query](assets/req.png)

> **Gambar 2**: DNS Response (UDP)
> ![DNS Response](assets/res1.png)

> **Gambar 2**: DNS Response (UDP)
> ![DNS Response](assets/res2.png)


**Ringkasan Paket yang Tertangkap:**

| Frame | Tipe | Source | Destination | Length (Frame) | Keterangan |
|-------|------|--------|-------------|-----------------|------------|
| 3277 | DNS Query | `192.168.1.5:65271` | `8.8.8.8:53` | 71 byte | Client tanya DNS |
| 3281 | DNS Response | `8.8.8.8:53` | `192.168.1.5:65271` | 135 byte | Server jawab query |

**Penjelasan Simpel:**
- Client pakai port acak `65271` untuk kirim pertanyaan
- Server DNS publik Google selalu pakai port `53` (standar internasional)
- Response "membalik" port: yang tadi tujuan jadi sumber, dan sebaliknya

---

### 4.2 Analisis Detail Header UDP

#### 4.2.1 Perbandingan Query vs Response

| Field Header | Pada Query (Frame 3277) | Pada Response (Frame 3281) |
|-------------|------------------------|---------------------------|
| **Source Port** | 65271 (ephemeral client) | 53 (DNS server) |
| **Destination Port** | 53 (DNS server) | 65271 (ephemeral client) |
| **Length** | 37 byte | 101 byte |
| **Checksum** | `0x6b1b` | `0x06c4` |

#### 4.2.2 Perhitungan Payload UDP

| Paket | Length (UDP) | Header UDP | Payload (Data Asli) |
|-------|-------------|------------|-------------------|
| Query | 37 byte | 8 byte | **29 byte** |
| Response | 101 byte | 8 byte | **93 byte** |

**Apa isi payload-nya?**
- Payload query: Pertanyaan DNS "Berapa IP youtube.com?" (tipe A / IPv4)
- Payload response: Jawaban DNS berupa 4 alamat IPv4 untuk youtube.com

---

### 4.3 Perhitungan Teknis UDP

#### 4.3.1 Kapasitas Maksimum UDP

| Parameter | Rumus / Penjelasan | Hasil |
|-----------|-------------------|-------|
| Maksimum Length field | 16-bit unsigned integer → 2¹⁶ - 1 | **65.535 byte** |
| Maksimum Payload | 65.535 - 8 (header UDP) | **65.527 byte** |
| Rentang Port Number | 0 sampai 2¹⁶ - 1 | **0 - 65.535** |
| Protocol Number di IP Header | UDP = 17 (desimal) / 0x11 (hex) | **17** |

#### 4.3.2 Batasan Praktis (Agar Tidak Fragmentasi)

- MTU Ethernet Standar = 1500 byte
- IP Header (tanpa opsi) = 20 byte
- UDP Header = 8 byte
- Maksimum Payload Aman = 1500 - 20 - 8 = 1472 byte

| Skenario | Maksimum Payload | Keterangan |
|----------|-----------------|------------|
| Teoritis (tanpa batas jaringan) | 65.527 byte | Bisa, tapi berisiko fragmentasi |
| Praktis (Ethernet standar) | **1472 byte** | Aman, tidak fragmentasi |
| Praktis (dengan IP options) | < 1472 byte | Kurangi lagi kalau IP header lebih besar |

---

### 4.4 Pola Komunikasi Request-Response UDP

#### 4.4.1 Mapping Port & IP

```
REQUEST:  192.168.1.5:65271 → 8.8.8.8:53
RESPONSE: 8.8.8.8:53        → 192.168.1.5:65271
```

#### 4.4.2 Poin Kunci Komunikasi UDP

| Konsep | Penjelasan | Contoh pada Praktikum |
|--------|-----------|---------------------|
| **Port Reversal** | Port source-destination dibalik saat response | Query: src=65271→dst=53 ; Response: src=53→dst=65271 |
| **Ephemeral Port** | Port sementara client (range dinamis) | Client pakai `65271` (masuk range 49152-65535) |
| **Well-Known Port** | Port standar layanan (0-1023) | DNS server selalu di port `53` |
| **Transaction ID** | ID unik untuk cocokkan query-response | DNS pakai ID `0x0004` sama di query & response |
| **Stateless** | Server tidak simpan "sesi" antar paket | Setiap query DNS independen, tidak ingat query sebelumnya |

#### 4.4.3 Hasil Query DNS dalam Praktikum

| Field | Nilai | Keterangan |
|-------|-------|------------|
| Query Type | A (1) - Host Address | Minta alamat IPv4 secara langsung |
| Domain | `youtube.com` | Domain yang ditanyakan |
| Jumlah Answer | 4 | `142.250.4.93`, `142.250.4.190`, `142.250.4.136`, `142.250.4.91` |
| Transaction ID | `0x0004` | Sama di query & response → untuk matching |

---

## 5. Ringkasan Hasil Praktikum

### 5.1 Tabel Ringkasan Parameter UDP

| Parameter | Nilai / Hasil | Keterangan |
|-----------|--------------|------------|
| Jumlah field header UDP | 4 field | Source Port, Dest Port, Length, Checksum |
| Ukuran total header UDP | 8 byte | Fixed, tidak berubah-ubah |
| Payload query DNS | 29 byte | 37 - 8 |
| Payload response DNS | 93 byte | 101 - 8 |
| Maksimum payload teoritis | 65.527 byte | 2¹⁶ - 1 - 8 |
| Maksimum payload praktis (Ethernet) | ~1472 byte | Agar tidak fragmentasi IP |
| Rentang port UDP | 0 - 65.535 | 16-bit field |
| Protocol number UDP di IP header | 17 (0x11) | Identifier di field "Protocol" IP |
| Pola port request-response | Dibalik (source ↔ destination) | Client port ↔ Server port |
| Protokol transport DNS | UDP | Query tipe A, ukuran kecil, sesuai standar |

### 5.2 Perbandingan UDP vs TCP (Singkat)

| Aspek | UDP | TCP |
|-------|-----|-----|
| Koneksi | Connectionless | Connection-oriented (3-way handshake) |
| Keandalan | Tidak jamin sampai/urut | Ada ACK, retransmission, sequencing |
| Overhead Header | 8 byte (kecil) | 20+ byte (lebih besar) |
| Kecepatan | Lebih cepat | Lebih lambat karena kontrol |
| Flow Control | Tidak ada | Ada (windowing) |
| Cocok Untuk | Streaming, gaming, DNS | Web, email, file transfer |

---

## 6. Kesimpulan

Praktikum ini menunjukkan bahwa UDP bekerja sebagai protokol transport yang ringan dan cepat, terbukti dari header-nya yang hanya berukuran 8 byte tetap (Source Port, Destination Port, Length, Checksum). Analisis terhadap query dan response DNS ke domain `youtube.com` melalui server `8.8.8.8` memperlihatkan pola request-response yang khas: port source dan destination saling bertukar posisi antara query dan response, sementara Transaction ID tetap sama untuk mencocokkan kedua paket tersebut. Perhitungan payload juga mengonfirmasi bahwa data aktual (29 byte pada query, 93 byte pada response) selalu sama dengan Length total dikurangi 8 byte header UDP.