# Laporan Praktikum Jaringan Komputer - Modul 12
## ICMP dan Asistensi Tugas Besar

### Identitas Praktikan

| Keterangan | Imformasi |
| :--- | :--- |
| **Nama** | Alif Luthfan Adeefa |
| **NIM** | 103072400163 |
| **Kelas** | IF-04-01 |
---

## 1. Tujuan Praktikum

| No | Tujuan Praktikum | Langkah / Target Capaian | Output / Bukti Pengerjaan |
| :---: | :--- | :--- | :--- |
| **1** | **Investigasi Protokol ICMP** | • Menangkap lalu lintas data menggunakan Wireshark.<br>• Melakukan *ping* lewat Terminal/CMD.<br>• Menyaring paket dengan filter `icmp`. | • Tangkapan layar (*screenshot*) paket ICMP.<br>• Analisis struktur *header* (*Type*, *Code*, *Checksum*). |
| **2** | **Membuat ICMP Pinger** | • Menulis skrip pinger berbasis Python.<br>• Menggunakan *raw socket* (`socket.SOCK_RAW`).<br>• Menangani respon sukses dan *Request Timed Out* (RTO). | • File kode program Python (`.py`).<br>• Hasil eksekusi program di Terminal/CMD (wajib akses *root*/*admin*). |
| **3** | **Asistensi Tugas Besar** | • Melaporkan progress pengerjaan kelompok.<br>• Menjelaskan pembagian tugas anggota.<br>• Mendiskusikan kendala teknis dengan aslab/dosen. | • Dokumen progress/lembar kendali Tugas Besar.<br>• Nilai atau bukti persetujuan (*approval*) asistensi. |

---

## 2. Langkah Kerja

Berikut adalah langkah-langkah yang dilakukan selama praktikum Modul 12:

### 2.1 ICMP dan Ping

1. Membuka aplikasi **Windows Command Prompt** sebagai Administrator.
2. Menjalankan **Wireshark** dan memulai packet capture pada interface yang aktif (Wi-Fi).
3. Menjalankan perintah ping ke host di benua lain:
   ```cmd
   ping -n 10 www.ust.hk
   ```
4. Menunggu hingga 10 paket ping selesai dikirim dan diterima.
5. Menghentikan capture pada Wireshark.
6. Memfilter paket dengan mengetikkan `icmp` pada filter bar Wireshark.
7. Menganalisis struktur paket ICMP Echo Request dan Echo Reply.

### 2.2 ICMP dan Traceroute

1. Membuka **Command Prompt** dan menjalankan Wireshark.
2. Memulai packet capture pada interface yang aktif.
3. Menjalankan perintah traceroute ke host tujuan:
   ```cmd
   tracert www.mit.edu
   ```
4. Menunggu hingga proses traceroute selesai.
5. Menghentikan capture dan memfilter paket dengan `icmp`.
6. Menganalisis paket ICMP Time Exceeded dan Echo Reply yang dihasilkan.

---

## 3. Hasil dan Pembahasan

### 3.1 Output Command Prompt - Ping

Berikut adalah hasil eksekusi perintah `ping -n 10 www.ust.hk`:

![Command Prompt Ping](assets/cmd_ping.png)
*Gambar 1: Output Command Prompt setelah menjalankan perintah `ping -n 10 www.ust.hk`.*

Dari gambar di atas, terlihat bahwa:

| Parameter Uji | Hasil Pengamatan | Analisis / Kesimpulan |
| :--- | :---: | :--- |
| **Paket Terkirim (*Request*)** | 10 Paket | Seluruh permintaan ICMP Echo Request berhasil dipropagasikan oleh sistem. |
| **Paket Diterima (*Reply*)** | 10 Paket | Server tujuan merespon balik seluruh paket tanpa kendala. |
| **Paket Hilang (*Packet Loss*)** | **0% loss** | Koneksi jaringan sangat stabil, tidak ada degradasi jalur komunikasi internasional. |
| **Rata-rata RTT** | **88 ms** | Performa transmisi baik untuk kategori interkoneksi antar-negara (Indonesia → Hong Kong). |
| **Minimum RTT** | **87 ms** | Waktu tempuh tercepat paket bolak-balik ke server Hong Kong. |
| **Maximum RTT** | **91 ms** | Waktu tempuh terlama paket bolak-balik ke server Hong Kong. |
| **TTL (*Time to Live*)** | **47** | Estimasi melewati **81 router/hops** di sepanjang jalur global (Asumsi nilai awal TTL = 128, karena 128 - 47 = 81). |

### 3.2 Analisis Paket ICMP Ping di Wireshark

Setelah memfilter dengan `icmp`, Wireshark menampilkan 20 paket: 10 Echo Request dan 10 Echo Reply.

![Wireshark ICMP Ping](assets/wireshark_ping.png)
*Gambar 2: Daftar paket ICMP hasil capture ping di Wireshark beserta detail Echo Request.*

#### Detail Paket Echo Request (Tipe 8, Kode 0)

| Field | Nilai | Keterangan |
|-------|-------|-----------|
| **Type** | **8** | Echo Request |
| **Code** | **0** | — |
| **Checksum** | **0x4d5a** | Status: Good/Correct |
| **Identifier (BE)** | **1 (0x0001)** | Big Endian |
| **Identifier (LE)** | **256 (0x0100)** | Little Endian |
| **Sequence Number (BE)** | **1 (0x0001)** | Urutan paket ke-1 |
| **Sequence Number (LE)** | **256 (0x0100)** | Little Endian |
| **Data Length** | **32 bytes** | Payload: "abcdefghijklmnop..." |

#### Detail Paket Echo Reply (Tipe 0, Kode 0)

![ICMP Echo Reply Detail](assets/wireshark_ping_reply.png)
*Gambar 3: Struktur paket ICMP Echo Reply yang diperluas di Wireshark.*

| Field | Nilai | Keterangan |
|-------|-------|-----------|
| **Type** | **0** | Echo Reply |
| **Code** | **0** | — |
| **TTL** | **47** | Sisa TTL setelah melewati 81 hop dari Hong Kong |
| **Identifier (BE)** | **1 (0x0001)** | Big Endian |
| **Identifier (LE)** | **256 (0x0100)** | Little Endian |
| **Source** | **143.89.209.9** | Host tujuan (www.ust.hk, Hong Kong) |
| **Destination** | **192.168.1.8** | Local machine |

Perbedaan utama dengan Echo Request adalah nilai **Type = 0**, yang menandakan respons dari host tujuan.

**Analisis Paket Ping di Wireshark:**
- Terlihat 20 paket ICMP (frame 863–1186)
- Pattern: Request–Reply berpasangan ✅
- Sequence numbers: 1, 2, 3, ..., 10
- Response times konsisten: 87–91 ms
- Tidak ada packet loss
- Source Request: **192.168.1.8** (local machine)
- Destination: **143.89.209.9** (www.ust.hk — Hong Kong)

---

### 3.3 Output Command Prompt - Traceroute

Berikut adalah hasil eksekusi perintah `tracert www.mit.edu`:

![Command Prompt Traceroute](assets/cmd_tracert.png)
*Gambar 4: Output Command Prompt setelah menjalankan perintah `tracert www.mit.edu`.*

Dari gambar di atas:

| Parameter Pelacakan | Hasil Pengamatan | Penjelasan Mekanis & Keamanan Jaringan |
| :--- | :--- | :--- |
| **Total Lompatan (*Hops*)** | **7 hops** | Paket data melewati 6 perangkat perantara sebelum tiba di host tujuan akhir. |
| **Paket Probe per Hop** | **3 Paket** | Setiap simpul router diuji sebanyak 3 kali menggunakan peningkatan nilai TTL (1, 2, 3, dst.). |
| **Respon Mayoritas Hop** | **ICMP Time Exceeded**<br>(Type 11, Code 0) | Router perantara membuang paket karena TTL mencapai nilai 0, lalu mengirimkan pesan galat ini ke pengirim. |
| **Hop 6** | `* * *` (Request Timed Out) | Router tersebut memblokir/memfilter paket ICMP — umum terjadi di jaringan transit internasional. |
| **Hop Akhir (Hop 7)** | `104.68.37.236`<br>(Akamai CDN — mit.edu) | Server target berhasil dicapai dan merespon dengan **ICMP Echo Reply** (Type 0, Code 0). |

**Network Path Analysis:**
```
Hop 1:  192.168.1.1          → Local Gateway (Router rumah)
Hop 2:  10.134.0.1           → ISP Internal Network
Hop 3:  112.215.248.240      → ISP Network (XL Axiata)
Hop 4:  112.215.197.10       → XL Core Network
Hop 5:  112.215.81.42        → XL Exit Gateway
Hop 6:  * * *                → Request Timed Out (ICMP diblokir firewall)
Hop 7:  104.68.37.236        → Destination (Akamai CDN — www.mit.edu)
```

---

### 3.4 Analisis Paket ICMP Traceroute di Wireshark

![Wireshark ICMP Traceroute](assets/wireshark_traceroute.png)
*Gambar 5: Paket ICMP Time Exceeded hasil capture traceroute di Wireshark.*

#### Detail Paket ICMP Time Exceeded (Tipe 11, Kode 0)

![ICMP Time Exceeded Detail](assets/wireshark_time_exceeded.png)
*Gambar 6: Struktur paket ICMP Time Exceeded yang diperluas di Wireshark.*

| Field | Nilai | Keterangan |
|-------|-------|-----------|
| **Type** | **11** | Time Exceeded |
| **Code** | **0** | Time to live exceeded in transit |
| **Checksum** | **0x4ff** | Status: Good |
| **Unused** | **0x00000000** | Tidak digunakan (4 bytes) |

**Struktur Original IP Header (salinan paket penyebab error):**

| Field | Nilai | Keterangan |
|-------|-------|-----------|
| **Original Src** | **192.168.1.8** | Local machine (pengirim) |
| **Original Dst** | **104.68.37.236** | Target (www.mit.edu via Akamai) |
| **Original TTL** | **1** | Inilah penyebab TTL exceeded |
| **Original Protocol** | ICMP (1) | |
| **Original Seq (BE)** | **11 (0x000b)** | |

**Analisis Paket Traceroute di Wireshark:**
- Multiple hops dengan TTL berbeda: 1, 2, 3, dst.
- Router merespons dengan **Type 11 Code 0** (Time Exceeded)
- Hop 6 tidak merespons ("no response found!")
- Hop yang berhasil merespons: **192.168.1.1**, **10.134.0.1**, **112.215.248.240**, **112.215.197.10**
- Final destination: **104.68.37.236** (www.mit.edu via Akamai CDN)

---

## 4. Pembahasan

### 4.1 Perbandingan Fungsional Mekanisme ICMP (Ping vs Traceroute)

| Karakteristik | ICMP Ping (Konektivitas *End-to-End*) | ICMP Traceroute (Pemetaan Jalur / *Hops*) |
| :--- | :--- | :--- |
| **Tipe ICMP Utama** | • `Type 8` (Echo Request)<br>• `Type 0` (Echo Reply) | • `Type 8` (Echo Request) dengan TTL inkremental<br>• `Type 11` (Time Exceeded) dari router perantara |
| **Perlakuan TTL** | Konstan/Default (Windows: 128) | Naik bertahap secara berkala (1, 2, 3, dst.) |
| **Tujuan Utama** | Mengukur *Round-Trip Time* (RTT) & keandalan koneksi. | Mengidentifikasi identitas IP dan performa di tiap *hop* jalur data. |
| **Hasil Studi Kasus** | Sukses mencapai Hong Kong dengan RTT **87–91 ms** dan **0% packet loss**. | Sukses memetakan **7 hops** menuju server MIT (via Akamai CDN). |

### 4.2 Analisis Kuantitatif Performa, Nilai TTL, dan Packet Loss

| Parameter Analisis | Hasil Evaluasi Ping (`www.ust.hk`) | Hasil Evaluasi Traceroute (`www.mit.edu`) |
| :--- | :--- | :--- |
| **Analisis Performa** | **Sangat Baik & Stabil**<br>• Rata-rata RTT rendah (88 ms)<br>• Nilai *Jitter* (variasi delay) sangat minim (hanya 4 ms). | **Sangat Baik untuk Jarak Jauh**<br>• Hanya 7 hop untuk mencapai server internasional.<br>• mit.edu menggunakan Akamai CDN sehingga RTT lebih rendah dari biasanya. |
| **Analisis Nilai TTL** | **Sisa TTL = 47**<br>• Perhitungan: $128 - 47 = 81$<br>• Menandakan paket melewati sekitar **81 router** dari Indonesia ke Hong Kong. | **TTL Berinkremen**<br>• Dikirim berurutan dari TTL=1.<br>• Setiap router mengurangi nilai TTL sebesar 1 hingga menjadi 0 di perangkat perantara, lalu mengirim Time Exceeded. |
| **Packet Loss** | **0%** — Tidak ada paket yang hilang. | **Hop 6** tidak merespons (firewall), namun hop 7 (destination) berhasil dicapai. |

---

## 5. Kesimpulan

Dari praktikum ini dapat disimpulkan:

1. **Protokol ICMP** berperan penting dalam diagnostik jaringan, baik untuk uji konektivitas (*ping*) maupun pemetaan jalur (*traceroute*).
2. **ICMP Echo Request (Type 8)** dan **Echo Reply (Type 0)** digunakan oleh perintah `ping` untuk mengukur RTT dan mendeteksi packet loss.
3. **ICMP Time Exceeded (Type 11, Code 0)** dihasilkan oleh router perantara saat TTL paket mencapai nilai 0, yang dimanfaatkan oleh `traceroute` untuk memetakan setiap hop dalam jalur data.
4. Hasil ping ke `www.ust.hk` menunjukkan koneksi yang **sangat stabil** dengan rata-rata RTT 88 ms dan 0% packet loss, melewati estimasi 81 hop.
5. Hasil traceroute ke `www.mit.edu` berhasil memetakan **7 hop** dengan 1 hop yang memblokir ICMP (Request Timed Out), yang merupakan hal umum dalam jaringan internasional.

---

## Lampiran — Checklist Screenshot yang Harus Disisipkan

> Letakkan semua file gambar di folder `assets/` yang berada satu direktori dengan file `.md` ini.

| No | Nama File | Isi Gambar | Digunakan di Bagian |
|----|-----------|-----------|-------------------|
| 1 | `assets/cmd_ping.png` | Output CMD `ping -n 10 www.ust.hk` | 3.1 (Gambar 1) |
| 2 | `assets/wireshark_ping.png` | List paket ICMP ping di Wireshark + detail Echo Request | 3.2 (Gambar 2) |
| 3 | `assets/wireshark_ping_reply.png` | Detail paket Echo Reply di Wireshark | 3.2 (Gambar 3) |
| 4 | `assets/cmd_tracert.png` | Output CMD `tracert www.mit.edu` | 3.3 (Gambar 4) |
| 5 | `assets/wireshark_traceroute.png` | List paket ICMP traceroute di Wireshark | 3.4 (Gambar 5) |
| 6 | `assets/wireshark_time_exceeded.png` | Detail paket Time Exceeded di Wireshark | 3.4 (Gambar 6) |