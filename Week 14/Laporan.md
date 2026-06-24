# Laporan Praktikum Jaringan Komputer - Modul 14
## Analisis Protokol IEEE 802.11 (WiFi)

### Identitas Praktikan

| Keterangan | Imformasi |
| :--- | :--- |
| **Nama** | Alif Luthfan Adeefa |
| **NIM** | 103072400163 |
| **Kelas** | IF-04-01 |

---

# 1. Tujuan Praktikum

Berdasarkan Modul Praktikum Jaringan Komputer Semester Genap 2025/2026, tujuan praktikum ini adalah:

| No | Tujuan Praktikum |
| :---: | :--- |
| **1** | Mahasiswa dapat menginvestigasi cara kerja protokol WiFi 802.11 menggunakan Wireshark. |
| **2** | Mahasiswa mampu menganalisis struktur frame 802.11 (Beacon, Data, Management). |
| **3** | Mahasiswa memahami mekanisme asosiasi, disosiasi, dan transfer data pada jaringan nirkabel. |
| **4** | Mahasiswa dapat membedakan karakteristik frame 802.11 dengan frame Ethernet kabel. |

---

# 2. Langkah Kerja

## 2.1 Membuka File Capture

1. Mengunduh file `wireshark-traces.zip`.
2. Mengekstrak file `Wireshark_802_11.pcap`.
3. Membuka file tersebut menggunakan Wireshark.
4. Mengamati seluruh paket yang terdapat pada file capture (total 2364 paket).

## 2.2 Analisis Beacon Frame

1. Menggunakan filter:

```text
wlan.fc.type_subtype == 0x08
```

2. Memilih salah satu paket Beacon.
3. Mengamati informasi berikut:

* Timestamp
* Beacon Interval
* Capability Information
* SSID
* Supported Rates
* Channel

## 2.3 Analisis Transfer Data

1. Menggunakan filter:

```text
http
```

2. Menemukan paket HTTP yang dikirim melalui jaringan WiFi.
3. Menganalisis struktur frame IEEE 802.11 Data yang membawa payload HTTP.

## 2.4 Analisis Deauthentication dan Association

1. Menggunakan filter:

```text
wlan.fc.type_subtype == 0x0c
```

untuk mengidentifikasi frame **Deauthentication**.

2. Menggunakan filter:

```text
wlan.fc.type_subtype == 0x00 || wlan.fc.type_subtype == 0x01
```

untuk mengidentifikasi frame **Association Request** dan **Association Response**.

3. Mengamati urutan proses pemutusan dan penyambungan kembali koneksi antara klien dan Access Point.

---

# 3. Hasil dan Pembahasan

## 3.1 Tampilan Awal File Capture

![Wireshark 802.11 Overview](assets/wireshark_80211_overview.png)

**Gambar 1.** Tampilan file capture IEEE 802.11 pada Wireshark.

File capture memperlihatkan berbagai jenis frame IEEE 802.11 yang terdiri dari frame Management, Control, dan Data. Paket-paket tersebut digunakan untuk mendukung komunikasi antara klien dan Access Point pada jaringan WiFi. Total terdapat 2364 paket dalam file capture ini.

---

## 3.2 Analisis Beacon Frame

![Beacon Frame Detail](assets/beacon_frame.png)

**Gambar 2.** Detail Beacon Frame dengan SSID "30 Munroe St".

Beacon Frame merupakan frame Management yang dikirim secara periodik oleh Access Point untuk mengumumkan keberadaan jaringan nirkabel kepada perangkat di sekitarnya.

Berdasarkan hasil pengamatan pada capture, informasi yang ditemukan pada Beacon Frame adalah sebagai berikut:

| Field                  | Nilai yang Ditemukan                    |
| ---------------------- | ---------------------------------------- |
| Source MAC Address     | 00:16:b6:f7:1d:51 (CiscoLinksys_f7:1d:51) |
| Timestamp               | 174319001986                            |
| Beacon Interval        | 0,102400 detik (≈ 100 ms)                |
| Capability Information | 0x0601 (ESS, QoS Implemented, Short Slot Time In use) |
| SSID                   | "30 Munroe St"                           |
| Supported Rates        | 1, 2, 5.5, 11 Mbit/sec                   |
| Extended Supported Rates | 6, 9, 12, 18, 24, 36, 48, 54 Mbit/sec   |
| DS Parameter Set (Channel) | 6                                     |

Berdasarkan hasil pengamatan, Access Point "30 Munroe St" mengirimkan Beacon secara berkala setiap kurang lebih 100 ms untuk menginformasikan SSID, channel operasi (channel 6), dan kemampuan jaringan kepada perangkat klien di sekitarnya.

---

## 3.3 Analisis Transfer Data HTTP

![HTTP over 802.11 Data Frame](assets/http_80211_data.png)

**Gambar 3.** Frame Data IEEE 802.11 yang membawa paket HTTP request.

Saat klien mengakses halaman web, paket HTTP dikirim melalui frame Data IEEE 802.11. Pada capture ditemukan permintaan HTTP berikut:

* **Waktu 24.828253 detik**: Klien (Intel_d1:b6:4f) mengirim `GET /wireshark-labs/alice.txt HTTP/1.1` ke server dengan IP **128.119.245.12**.
* **Waktu 25.126724 detik**: Server membalas dengan `HTTP/1.1 200 OK`.

### Struktur Alamat pada Frame Data

| Field     | Nilai yang Ditemukan                              |
| --------- | --------------------------------------------------- |
| Receiver Address (RA)    | CiscoLinksys_f7:1d:51 (00:16:b6:f7:1d:51) — Access Point |
| Transmitter Address (TA) | Intel_d1:b6:4f (00:13:02:d1:b6:4f) — Klien (host)        |
| BSS Id                   | CiscoLinksys_f7:1d:51 (00:16:b6:f7:1d:51)                |
| STA Address              | Intel_d1:b6:4f (00:13:02:d1:b6:4f)                       |

Berbeda dengan Ethernet yang hanya memiliki alamat sumber dan tujuan, frame Data IEEE 802.11 di atas memiliki empat alamat MAC sekaligus (Receiver, Transmitter, BSS Id, dan STA Address) karena harus merepresentasikan baik perangkat nirkabel maupun Access Point yang menjadi perantara.

### Struktur Payload

```text
IEEE 802.11 QoS Data Frame
├── Radiotap Header
├── MAC Header
├── Logical-Link Control (LLC)
├── IP Header
├── TCP Header
└── HTTP Payload (GET /wireshark-labs/alice.txt)
```

Hasil pengamatan menunjukkan bahwa frame IEEE 802.11 tidak langsung membawa paket IP, melainkan melalui lapisan **Logical-Link Control (LLC)** terlebih dahulu sebelum diteruskan ke protokol IP, TCP, dan akhirnya payload HTTP.

---

## 3.4 Analisis Deauthentication

![Deauthentication Frame](assets/deauthentication.png)

**Gambar 4.** Frame Deauthentication dari klien ke Access Point "30 Munroe St".

Pada waktu **49.609617 detik**, klien (Intel_d1:b6:4f) mengirim frame **Deauthentication** kepada Access Point "30 Munroe St" (CiscoLinksys_f7:1d:51) dengan **Reason Code: Unspecified reason (0x0001)**.

Frame ini termasuk ke dalam kategori **Management Frame** dan berfungsi untuk mengakhiri hubungan otentikasi antara klien dan Access Point, sebagai langkah awal sebelum klien mencoba berpindah ke jaringan lain.

Setelah deauthentication tersebut, ditemukan pula rangkaian frame Deauthentication tambahan sekitar waktu **63 detik**, kali ini dikirim oleh Access Point "linksys_SES_24086" (CiscoLinksys_f5:ba) kepada klien, yang menunjukkan bahwa upaya klien untuk bergabung ke AP tersebut ditolak.

---

## 3.5 Analisis Association

![Association Request/Response](assets/association.png)

**Gambar 5.** Urutan proses Association Request dan Association Response.

Setelah proses Deauthentication dari AP "30 Munroe St", klien mencoba melakukan asosiasi ke Access Point lain, yaitu **"linksys_SES_24086"**. Berdasarkan hasil pengamatan:

* Klien mengirim **Association Request** ke "linksys_SES_24086" secara berulang kali sejak waktu **49.651078 detik** hingga **62.176945 detik** (lebih dari 10 kali percobaan).
* Seluruh percobaan tersebut **tidak mendapatkan Association Response** dari AP "linksys_SES_24086", kemungkinan karena AP tersebut memerlukan autentikasi dengan kunci (key) tertentu sehingga menolak permintaan asosiasi terbuka dari klien.

Setelah seluruh upaya tersebut gagal, klien akhirnya mencoba kembali ke Access Point semula:

* Pada waktu **63.169910 detik**, klien mengirim **Association Request** ke Access Point **"30 Munroe St"**.
* Pada waktu **63.192101 detik**, Access Point "30 Munroe St" membalas dengan **Association Response** yang menunjukkan asosiasi berhasil dilakukan.

Proses Association ini memungkinkan perangkat klien memperoleh kembali izin untuk bertukar data melalui jaringan WiFi "30 Munroe St" setelah sebelumnya gagal berpindah ke jaringan lain.

---

# 4. Kesimpulan

Berdasarkan hasil analisis capture IEEE 802.11 di atas, dapat disimpulkan bahwa:

1. Access Point secara berkala mengirimkan **Beacon Frame** untuk mengumumkan keberadaan jaringan (SSID, channel, kemampuan jaringan) kepada perangkat di sekitarnya.
2. Transfer data HTTP pada jaringan nirkabel tetap melalui struktur frame 802.11 yang membawa hingga empat alamat MAC, berbeda dengan Ethernet kabel yang hanya memiliki dua alamat.
3. Proses **Deauthentication** dapat terjadi baik atas inisiatif klien (memutuskan diri dari AP) maupun atas penolakan dari AP terhadap klien.
4. Proses **Association** tidak selalu berhasil pada percobaan pertama; klien dapat melakukan beberapa kali percobaan ke AP yang berbeda sebelum akhirnya berhasil terhubung kembali ke AP semula.