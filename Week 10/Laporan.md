# Laporan Praktikum Jaringan Komputer - Modul 10
## Internet Protocol (IP) Analysis

### Identitas Praktikan

| Keterangan | Imformasi |
| :--- | :--- |
| **Nama** | Alif Luthfan Adeefa |
| **NIM** | 103072400163 |
| **Kelas** | IF-04-01 |

---

## 1.1 Tujuan Praktikum

| No. | Tujuan Praktikum | Deskripsi Kompetensi |
| :---: | :--- | :--- |
| **1** | Menganalisis cara kerja protokol IP menggunakan Wireshark | Mampu menangkap paket (*capturing*), menyaring (*filtering*), dan membedah isi data pada lapisan jaringan (*network layer*) secara *real-time*. |
| **2** | Memahami struktur header IPv4 dan field-field penting | Mampu mengidentifikasi fungsi komponen kritis seperti *Version*, *Total Length*, *TTL*, hingga *Source/Destination IP Address*. |
| **3** | Mempelajari fragmentasi IP pada datagram besar | Mampu menganalisis perubahan nilai *Identification*, *Flags*, dan *Fragment Offset* saat paket melebihi kapasitas beban maksimum (MTU). |
| **4** | Mengenal datagram IPv6 | Mampu memahami arsitektur *Fixed Header* 40 bytes serta perbedaan karakteristik pengalamatan dan penanganan paket dibanding IPv4. |

---

## 1.2 Hasil Praktikum

### 1.2.1 Bagian 1: Analisis IPv4 Dasar

**Filter Wireshark yang Digunakan:**
```
ip.src == 192.168.1.8 && ip.dst == 128.119.245.12
```

**Hasil Capture Wireshark:**

![Gambar 1 - Overview Traceroute](assets/gambar1_overview_traceroute.png)

*Gambar 1: Paket ICMP dari traceroute ke gaia.cs.umass.edu (128.119.245.12)*

Dari screenshot di atas, terlihat paket-paket ICMP dengan berbagai TTL yang dikirimkan dari host **192.168.1.8** ke destination **128.119.245.12**. Paket berwarna pink adalah ICMP Echo Request dari client, sedangkan paket berwarna teal/hijau adalah ICMP Time-to-live exceeded dari router intermediate.

| No | Frame | Source | Destination | TTL | Info |
|----|-------|--------|-------------|-----|------|
| 1 | 256 | 192.168.1.8 | 128.119.245.12 | 1 | Echo (ping) request |
| 2 | 257 | 192.168.1.1 | 192.168.1.8 | - | Time-to-live exceeded |
| 3 | 276 | 192.168.1.8 | 128.119.245.12 | 2 | Echo (ping) request |
| 4 | 277 | 10.134.0.1 | 192.168.1.8 | - | Time-to-live exceeded |
| 5 | 291 | 192.168.1.8 | 128.119.245.12 | 3 | Echo (ping) request |
| 6 | 293 | 112.215.248.240 | 192.168.1.8 | - | Time-to-live exceeded |
| 7 | 311 | 192.168.1.8 | 128.119.245.12 | 4 | Echo (ping) request |
| 8 | 312 | 112.215.37.85 | 192.168.1.8 | - | Time-to-live exceeded |

---

### 1.2.2 Detail Header IPv4 dan ICMP — ICMP Destination Unreachable

Pada capture yang dilakukan menggunakan `tracert` Windows, **tidak ditemukan paket ICMP Type 3 (Destination Unreachable)**. Hal ini disebabkan karena `tracert` Windows menggunakan ICMP Echo Request, bukan UDP ke port tinggi seperti `traceroute` Linux/Mac yang biasanya memicu respons Port Unreachable dari router tujuan.

Secara teori, struktur ICMP Destination Unreachable adalah sebagai berikut:

```
Internet Control Message Protocol
    Type: 3 (Destination unreachable)
    Code: 3 (Port unreachable)
    Checksum: [correct]
    [Checksum Status: Good]
    Unused: 00000000
```

**Analisis:**
- **Type 3** = Destination unreachable
- **Code 3** = Port unreachable
- Pesan ini dikirimkan ketika port tujuan tidak tersedia atau tidak dapat dijangkau.
- Type 3 Code 3 khususnya terjadi saat `traceroute` berbasis UDP (Linux/Mac) mengirimkan paket ke port yang tidak digunakan oleh aplikasi apapun di host tujuan.

---

### 1.2.3 Analisis ICMP Echo Request (Ping)

**Detail Frame 256 — ICMP Echo Request:**

![Gambar 3 - Detail ICMP Echo Request](assets/gambar3_icmp_echo_request.png)

*Gambar 3: Detail ICMP Echo Request dengan TTL=1 (Frame 256)*

**Header IPv4 (Frame 256):**

```
Internet Protocol Version 4, Src: 192.168.1.8, Dst: 128.119.245.12
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00 (DSCP: CS0, ECN: Not-ECT)
    Total Length: 92
    Identification: 0x161a (5658)
    Flags: 0x00
        0... .... = Reserved bit: Not set
        .0.. .... = Don't fragment: Not set
        ..0. .... = More fragments: Not set
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 1
    Protocol: ICMP (1)
    Header Checksum: 0x6c53 [validation disabled]
    Source Address: 192.168.1.8
    Destination Address: 128.119.245.12
```

**Header ICMP:**

```
Internet Control Message Protocol
    Type: 8 (Echo (ping) request)
    Code: 0
    Checksum: 0xf79f [correct]
    [Checksum Status: Good]
    Identifier (BE): 1 (0x0001)
    Identifier (LE): 256 (0x0100)
    Sequence Number (BE): 95 (0x005f)
    Sequence Number (LE): 24320 (0x5f00)
    [No response seen]
    Data (64 bytes)
```

---

### 1.2.4 Analisis ICMP Time-to-Live Exceeded

**Detail Frame 257 — ICMP TTL Exceeded:**

![Gambar 4 - Detail ICMP TTL Exceeded](assets/gambar4_icmp_ttl_exceeded.png)

*Gambar 4: Detail ICMP Time-to-live exceeded (Type 11, Code 0)*

**Struktur ICMP TTL-Exceeded:**

```
Internet Control Message Protocol
    Type: Time-to-live exceeded (11)
    [Expert Info (Note/Response): Type indicates an error]
    Code: 0 (Time to live exceeded in transit)
    Checksum: 0x4ff [correct]
    [Checksum Status: Good]
    Unused: 00000000
```

**Analisis:**
- **Type 11** = Time-to-live exceeded
- **Code 0** = TTL expired in transit
- Dikirim oleh router **192.168.1.1** ke **192.168.1.8**
- Router mengurangi TTL dari paket asli menjadi 0, lalu mengirim pesan error ini kembali ke pengirim

**Header IPv4 dari paket TTL-Exceeded (Frame 257):**

| Field | Nilai |
|-------|-------|
| Version | 4 |
| Header Length | 20 bytes |
| Total Length | 120 bytes |
| Identification | 0xee36 (60982) |
| Flags | 0x00 (MF=0, DF=0) |
| Time to Live | 64 |
| Protocol | ICMP (1) |
| Source Address | 192.168.1.1 (Router Hop 1) |
| Destination Address | 192.168.1.8 |

---

### 1.2.5 Analisis TTL (Time to Live)

**Cara Kerja TTL pada Traceroute:**

![Gambar 5 - TTL Bervariasi](assets/gambar5_ttl_bervariasi.png)

*Gambar 5: Paket dengan TTL berbeda (1, 2, 3, 4) menunjukkan mekanisme traceroute*

| TTL | Hop yang Dicapai | Router IP | Response |
|-----|------------------|-----------|----------|
| 1 | Router 1 | 192.168.1.1 | ICMP TTL-exceeded |
| 2 | Router 2 | 10.134.0.1 | ICMP TTL-exceeded |
| 3 | Router 3 | 112.215.248.240 | ICMP TTL-exceeded |
| 4 | Router 4 | 112.215.37.85 | ICMP TTL-exceeded |
| N | Destination | 128.119.245.12 | ICMP Echo Reply |

TTL meningkat secara bertahap (1, 2, 3, 4, ...) sesuai cara kerja traceroute — setiap kali TTL habis di suatu router, router tersebut mengirimkan ICMP TTL-exceeded sehingga client dapat mengidentifikasi IP setiap hop.

---

### 1.2.6 Filter dan Analisis Paket

**Filter yang Digunakan:**

1. Tampilkan semua ICMP:
   ```
   icmp
   ```

2. Tampilkan ICMP ke komputer lokal:
   ```
   icmp && ip.dst == 192.168.1.8
   ```

3. Tampilkan paket ke tujuan:
   ```
   ip.src == 192.168.1.8 && ip.dst == 128.119.245.12
   ```

**Hasil Filter:**

![Gambar 6 - Filter ICMP](assets/gambar6_filter_icmp.png)

*Gambar 6: Hasil filter ICMP packets di Wireshark*

---

### 1.2.7 Bagian 2: Fragmentasi IP

**Catatan Penting:**

Pada capture ini, **tidak terlihat fragmentasi** karena:

1. **Ukuran paket kecil**: Total Length = 92 bytes
2. **MTU jaringan Ethernet** = 1500 bytes
3. **92 < 1500** → **tidak perlu fragmentasi**

**Flags di Header IPv4 (dari capture):**
```
Flags: 0x00
    0... .... = Reserved bit: Not set
    .0.. .... = Don't fragment: Not set
    ..0. .... = More fragments: Not set
    Fragment Offset: 0
```

Nilai **MF (More Fragments) = 0** dan **Fragment Offset = 0** mengonfirmasi bahwa paket tidak terfragmentasi.

**Teori Fragmentasi:**

Jika datagram 3000 bytes dikirim melalui jaringan dengan MTU 1500 bytes:

| Fragment | Total Length | Fragment Offset | Flags (MF) |
|----------|--------------|-----------------|------------|
| 1 | 1500 bytes | 0 | MF=1 (More Fragments) |
| 2 | 1500 bytes | 1480 | MF=1 (More Fragments) |
| 3 | 60 bytes | 2960 | MF=0 (Last fragment) |

**Perhitungan:**
```
Datagram asli  : 3000 bytes
MTU            : 1500 bytes
Header IP      : 20 bytes
Payload maks   : 1500 - 20 = 1480 bytes per fragment

Fragment 1: Offset 0,     Length 1500 (20 header + 1480 data)
Fragment 2: Offset 1480,  Length 1500 (20 header + 1480 data)
Fragment 3: Offset 2960,  Length 60   (20 header + 40 data)
```

---

### 1.2.8 Bagian 3: IPv6 Overview

Pada capture yang dilakukan, **tidak ditemukan datagram IPv6** karena jaringan lokal yang digunakan masih berbasis IPv4 dan `tracert` Windows secara default berjalan pada jalur IPv4.

**Perbandingan IPv4 vs IPv6:**

| Fitur | IPv4 | IPv6 |
|-------|------|------|
| **Panjang Alamat** | 32 bit (4 bytes) | 128 bit (16 bytes) |
| **Header Length** | Variabel (20–60 bytes) | Fixed (40 bytes) |
| **Fragmentasi** | Di router & host | Hanya di host source |
| **Checksum Header** | Ada | Tidak ada |
| **Options** | Ada (variable length) | Extension headers |
| **Contoh Alamat** | 192.168.1.8 | 2001:0db8::1 |

---

## 1.3 Analisis Praktikum

### 1.3.1 Mekanisme Traceroute

**Berdasarkan hasil capture, alur traceroute yang teramati:**

1. **Client** (192.168.1.8) mengirim ICMP Echo Request dengan TTL=1 ke destination 128.119.245.12.
2. **Router 1** (192.168.1.1) mengurangi TTL menjadi 0 → membuang paket → mengirim ICMP Type 11 ke client.
3. **Client** mengirim ulang dengan TTL=2, dijawab oleh **Router 2** (10.134.0.1) dengan TTL-exceeded.
4. **Client** mengirim ulang dengan TTL=3, dijawab oleh **Router 3** (112.215.248.240) dengan TTL-exceeded.
5. **Client** mengirim ulang dengan TTL=4, dijawab oleh **Router 4** (112.215.37.85) dengan TTL-exceeded.
6. Proses berlanjut hingga TTL cukup besar untuk mencapai destination.

**Tabel Hop Traceroute dari Capture:**

| Hop | Router IP | Frame TTL-exceeded | TTL Request |
|-----|-----------|--------------------|-------------|
| 1 | 192.168.1.1 | 257, 264, 266 | 1 |
| 2 | 10.134.0.1 | 277, 279, 281 | 2 |
| 3 | 112.215.248.240 | 293, 295, 297 | 3 |
| 4 | 112.215.37.85 | 312, 314 | 4 |
| Dst | 128.119.245.12 | — | N |

---

### 1.3.2 ICMP Message Types

**Yang terlihat di capture:**

| Type | Code | Message | Keterangan |
|------|------|---------|------------|
| **8** | 0 | Echo (ping) request | Dari client ke server |
| **11** | 0 | Time-to-live exceeded | Dari router saat TTL=0 |
| **3** | 3 | Destination unreachable | Tidak teramati pada capture Windows |

**Penjelasan:**

1. **Type 8 — Echo Request:**
   - Dikirim oleh `tracert` dari host 192.168.1.8
   - Berisi data 64 bytes
   - Identifier: 0x0001
   - Sequence Number meningkat: 95, 96, 97, 98, ...

2. **Type 11 — TTL Exceeded:**
   - Dikirim oleh router intermediate ketika TTL mencapai 0
   - Berisi original datagram dalam payload
   - Digunakan client untuk mengidentifikasi IP setiap hop

3. **Type 3 — Destination Unreachable:**
   - Code 3 = Port unreachable
   - Tidak teramati karena `tracert` Windows menggunakan ICMP, bukan UDP
   - Pada `traceroute` Linux/Mac, Type 3 muncul saat paket UDP mencapai destination di port yang tidak aktif

---

### 1.3.3 Field-Field Penting IPv4

**Yang dianalisis dari capture:**

**1. TTL (Time to Live):**
```
Ukuran : 8 bit (nilai 0–255)
Fungsi : Mencegah paket berputar selamanya di jaringan
Cara   : Setiap router mengurangi TTL minimal 1
         Jika TTL = 0 → paket dibuang + kirim ICMP Type 11
```
Dari capture: TTL bervariasi mulai dari 1, 2, 3, 4, ... sesuai mekanisme traceroute.

**2. Protocol:**
```
ICMP = 1
TCP  = 6
UDP  = 17
```
Dari capture: Protocol = ICMP (1) untuk seluruh paket Echo Request dan TTL-exceeded.

**3. Total Length:**
```
Ukuran : 16 bit (maksimum 65535 bytes)
Isi    : Header IP + Data
```
Dari capture:
- Total Length = **92 bytes** (Echo Request — Frame 256)
- Total Length = **120 bytes** (TTL-exceeded — Frame 257)

**4. Identification:**
```
Fungsi : Penanda unik setiap datagram
         Digunakan untuk reassembly fragment
```
Dari capture:
- Frame 256: Identification = **0x161a (5658)**
- Frame 257: Identification = **0xee36 (60982)**

**5. Flags:**
```
Bit 0 : Reserved (harus 0)
Bit 1 : DF (Don't Fragment)
Bit 2 : MF (More Fragments)
```
Dari capture:
```
Flags: 0x00
    0... .... = Reserved: Not set
    .0.. .... = Don't fragment: Not set
    ..0. .... = More fragments: Not set
```
Paket **tidak terfragmentasi** karena ukurannya (92 bytes) jauh di bawah MTU (1500 bytes).

---

## 1.4 Kesimpulan

### 1. Hasil Analisis Protokol IP & Traceroute

| Poin Analisis | Metodologi / Skenario | Hasil Pengamatan & Mekanisme yang Teramati |
| :--- | :--- | :--- |
| **Analisis Protokol IP** | Eksekusi perintah `tracert` ke host tujuan `gaia.cs.umass.edu` (128.119.245.12) dari host 192.168.1.8. | Aliran data lapisan jaringan (*Network Layer*) berhasil ditangkap dan dianalisis menggunakan Wireshark. |
| **Mekanisme Traceroute** | Pemanfaatan nilai TTL yang bertambah secara bertahap (*increasing TTL*: 1, 2, 3, 4, dst). | Router hop 1 (192.168.1.1), hop 2 (10.134.0.1), hop 3 (112.215.248.240), hop 4 (112.215.37.85) berhasil terpetakan melalui respons ICMP TTL-exceeded. |
| **Efektivitas Wireshark** | Penerapan fitur *Display Filter* dan *Color Coding* untuk membedakan jenis paket. | `icmp`: semua trafik ICMP. `icmp && ip.dst == 192.168.1.8`: fokus paket respon. `ip.src == 192.168.1.8 && ip.dst == 128.119.245.12`: fokus paket request. |

---

### 2. Bedah Detail Header IPv4 & Pesan ICMP

| Komponen Analisis | Nama Field / Tipe Pesan | Nilai / Hasil Capture | Korelasi Teori & Fungsi Jaringan |
| :--- | :--- | :---: | :--- |
| **Header IPv4** | Version | `4` | Menegaskan penggunaan arsitektur protokol IPv4. |
| | Header Length | `20 bytes` | Ukuran standar header minimum tanpa opsi tambahan. |
| | Total Length | `92 bytes` | Akumulasi ukuran header IP ditambah beban data (*payload*). |
| | TTL (Time to Live) | `1, 2, 3, 4...` | Batas lompatan router untuk mencegah paket berputar selamanya. |
| | Protocol | `1` (ICMP) | Menunjukkan bahwa payload di dalamnya berisi protokol ICMP. |
| | Identification | `0x161a (5658)` | Tanda pengenal unik paket untuk keperluan reassembly. |
| | Flags | `0x00` | Menandakan DF=0 dan MF=0; paket tidak terfragmentasi. |
| | Source & Destination | `192.168.1.8` → `128.119.245.12` | Alamat logika pengirim lokal menuju server tujuan. |
| **Pesan ICMP** | Type 8, Code 0 | Echo (ping) Request | Paket permintaan koneksi yang dikirim oleh client. |
| | Type 11, Code 0 | Time-to-live Exceeded | Respons dari router tengah karena nilai TTL paket telah habis. |
| | Type 3, Code 3 | Destination Unreachable | Tidak teramati; hanya muncul pada traceroute UDP (Linux/Mac). |

---

### 3. Analisis Fragmentasi IP dan IPv6 (Kondisi Tidak Teramati)

| Topik | Status | Alasan Tidak Teramati pada Capture | Pemahaman Teori & Solusi Praktikum |
| :--- | :---: | :--- | :--- |
| **Fragmentasi IP** | **Tidak Teramati** | Ukuran paket `tracert` Windows sangat kecil (**92 bytes**), jauh di bawah ambang batas **MTU standar (1500 bytes)**. Nilai `MF=0` dan `Fragment Offset=0` mengonfirmasi tidak ada pecahan. | **Teori:** Datagram besar (misal 3000 bytes) akan dipecah menjadi 3 fragmen dengan ID sama, namun MF dan Fragment Offset berbeda. **Solusi:** Diperlukan sistem operasi Linux/Mac atau file trace khusus pcapng. |
| **Datagram IPv6** | **Tidak Teramati** | Jaringan internet lokal yang digunakan masih berbasis IPv4 dan aplikasi `tracert` Windows secara default berjalan pada jalur IPv4. | **Teori:** IPv6 menggunakan struktur *Fixed Header* 40 bytes, tanpa checksum header, dan fragmentasi hanya di host source. **Solusi:** Analisis harus beralih menggunakan file trace sekunder seperti `ip-wireshark-trace2-1.pcapng`. |