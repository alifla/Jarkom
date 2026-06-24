# Laporan Praktikum Jaringan Komputer - Modul 7
## Socket Programming: UDP dan TCP

### Identitas Praktikan
| Keterangan | Imformasi |
| :--- | :--- |
| **Nama** | Alif Luthfan Adeefa |
| **NIM** | 103072400163 |
| **Kelas** | IF-04-01 |

---

## 1. Tujuan Praktikum

| No | Tujuan | Penjelasan |
|----|--------|-----------|
| 1 | Membuat aplikasi client-server UDP | Memahami implementasi socket UDP untuk komunikasi tanpa koneksi |
| 2 | Membuat aplikasi client-server TCP | Memahami implementasi socket TCP dengan mekanisme koneksi |
| 3 | Memahami perbedaan UDP dan TCP | Mengetahui karakteristik dan use case masing-masing protokol |
| 4 | Menganalisis pertukaran data | Mampu melacak alur komunikasi antara client dan server |

---

## 2. Dasar Teori

### 2.1 Konsep Socket Programming

| Istilah | Definisi |
|---------|----------|
| **Socket** | Endpoint untuk komunikasi jaringan antara dua program |
| **Client** | Aplikasi yang memulai permintaan/koneksi ke server |
| **Server** | Aplikasi yang menunggu dan melayani permintaan client |
| **Binding** | Proses mengaitkan socket dengan alamat IP dan port tertentu |
| **Listen** | Server dalam mode siap menerima koneksi masuk |
| **Accept** | Server menerima koneksi dari client dan membuat socket khusus |
| **Connect** | Client memulai proses koneksi ke server |

### 2.2 Perbandingan UDP dan TCP

| Karakteristik | UDP | TCP |
|--------------|-----|-----|
| **Jenis Protokol** | Connectionless | Connection-oriented |
| **Handshake** | Tidak ada | 3-way handshake (SYN, SYN-ACK, ACK) |
| **Keandalan** | Tidak dijamin | Dijamin (ACK, retransmission) |
| **Urutan Data** | Tidak dijamin | Dijamin berurutan |
| **Overhead Header** | 8 byte | 20+ byte |
| **Kecepatan** | Lebih cepat | Ada delay handshake |
| **Flow Control** | Tidak ada | Ada (windowing mechanism) |
| **Use Case** | DNS, streaming, gaming | Web, email, file transfer |

---

## 3. Praktikum UDP Socket

### 3.1 Kode Program UDP Server

**File:** `udpserver.py`

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))

print("server siap bro")

while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

**Penjelasan:**
- Server membuat socket UDP dengan `SOCK_DGRAM`
- Bind ke port 12000
- Menampilkan pesan `"server siap bro"` saat siap menerima
- Looping terus menerus untuk menerima pesan dari client
- Mengubah pesan menjadi uppercase dengan `.upper()` lalu mengirim balik ke client

---

### 3.2 Kode Program UDP Client

**File:** `udpclient.py`

```python
from socket import *

serverName = 'localhost'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('input sesuatu: ')
clientSocket.sendto(message.encode(), (serverName, serverPort))

modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())

clientSocket.close()
```

**Penjelasan:**
- Client membuat socket UDP dengan `SOCK_DGRAM` (tidak perlu bind port)
- Meminta input dari user dengan prompt `'input sesuatu: '`
- Langsung kirim pesan ke server menggunakan `sendto()` beserta alamat tujuan
- Menerima response dari server dengan `recvfrom()` lalu menampilkannya
- Tidak memerlukan `connect()` karena UDP bersifat connectionless

---

### 3.3 Hasil Eksekusi UDP

**Langkah Testing:**
1. Buka terminal 1 → jalankan server dengan `python udpserver.py`
2. Buka terminal 2 → jalankan client dengan `python udpclient.py`
3. Input pesan dan lihat hasilnya

**Terminal 1 - UDP Server:**
![UDP Server](Assets/Program/UDPServer.png)

Server berjalan dan menampilkan `"server siap bro"` lalu menunggu pesan dari client.

**Terminal 2 - UDP Client:**
![UDP Client](Assets/Program/UDPClient.png)

Client mengirim pesan dan menerima response uppercase dari server.

**Hasil:**
- Input: `yo` → Output: `YO`
- Input: `kimai` → Output: `KIMAI`
- Pesan berhasil dikonversi ke uppercase oleh server

---

## 4. Praktikum TCP Socket

### 4.1 Kode Program TCP Server

**File:** `tcpserver.py`

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)

print('server tcp siap bro')

while True:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```

**Penjelasan:**
- Server membuat socket TCP dengan `SOCK_STREAM`
- `listen(1)` → server siap menerima koneksi dengan max 1 antrian
- Menampilkan `'server tcp siap bro'` saat siap
- `accept()` → menunggu dan menerima koneksi dari client, menghasilkan `connectionSocket` baru
- Data diterima, diubah ke uppercase, lalu dikirim balik
- `connectionSocket.close()` menutup koneksi per client (serverSocket tetap terbuka untuk client berikutnya)

---

### 4.2 Kode Program TCP Client

**File:** `tcpclient.py`

```python
from socket import *

serverName = 'localhost'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName, serverPort))

sentence = input('Input sesuatu: ')
clientSocket.send(sentence.encode())

modifiedSentence = clientSocket.recv(1024)
print('From Server:', modifiedSentence.decode())

clientSocket.close()
```

**Penjelasan:**
- Client membuat socket TCP dengan `SOCK_STREAM`
- `connect()` → memulai proses koneksi (3-way handshake: SYN, SYN-ACK, ACK)
- Kirim data menggunakan `send()` (tidak perlu specify alamat karena sudah terkoneksi)
- Menerima response dengan `recv()` lalu menampilkan dengan prefix `'From Server:'`
- `clientSocket.close()` menutup koneksi setelah selesai

---

### 4.3 Hasil Eksekusi TCP

**Langkah Testing:**
1. Buka terminal 1 → jalankan TCP server dengan `python tcpserver.py`
2. Buka terminal 2 → jalankan TCP client dengan `python tcpclient.py`
3. Input kalimat dan lihat hasilnya

**Terminal 1 - TCP Server:**
![TCP Server](Assets/Program/TCPServer.png)

Server siap menerima koneksi dan memproses pesan dari client.

**Terminal 2 - TCP Client:**
![TCP Client](Assets/Program/TCPClient.png)

Client terhubung ke server, mengirim pesan, dan menerima response uppercase.

**Hasil:**
- Input: `halo` → Output: `From Server: HALO`
- Koneksi TCP established sebelum transfer data berlangsung

---

## 5. Perbandingan UDP vs TCP (Hasil Praktikum)

### 5.1 Perbedaan Implementasi

| Aspek | UDP | TCP |
|-------|-----|-----|
| **Socket Type** | `SOCK_DGRAM` | `SOCK_STREAM` |
| **Koneksi** | Tidak perlu `connect()` | Perlu `connect()` |
| **Server Socket** | 1 socket untuk semua client | 2 socket (serverSocket + connectionSocket) |
| **Send/Receive** | `sendto()` / `recvfrom()` | `send()` / `recv()` |
| **Address** | Harus specify alamat tujuan | Otomatis (sudah ada koneksi) |

---

### 5.2 Perbedaan Hasil Eksekusi

| Karakteristik | UDP | TCP |
|--------------|-----|-----|
| **Kecepatan** | Lebih cepat (langsung kirim) | Ada delay handshake |
| **Server** | Handle multiple client simultan | Handle 1 client per waktu |
| **Client** | Bisa kirim berkali-kali | Kirim 1x, koneksi selesai |
| **Reliability** | Tidak ada jaminan | Data terjamin sampai |

---

## 6. Analisis Praktikum

### 6.1 UDP Socket

**Hasil Pengamatan:**
- Server bisa menerima pesan dari berbagai client
- Tidak ada proses koneksi yang terlihat sebelum pengiriman data
- Pesan langsung dikirim dan diterima tanpa handshake
- Tidak ada konfirmasi delivery dari server ke client

**Keunggulan UDP:**
- Implementasi sederhana dan ringkas
- Tidak ada overhead koneksi
- Cocok untuk aplikasi real-time seperti streaming dan gaming

**Keterbatasan:**
- Tidak ada jaminan pesan sampai ke tujuan
- Tidak ada urutan data yang terjamin
- Tidak ada mekanisme retransmisi jika paket hilang

---

### 6.2 TCP Socket

**Hasil Pengamatan:**
- Ada proses `connect()` sebelum data dapat dikirim
- Server membuat `connectionSocket` khusus untuk setiap client
- Data terjamin sampai dan berurutan
- Koneksi ditutup secara eksplisit setelah komunikasi selesai

**Keunggulan TCP:**
- Reliable delivery terjamin
- Data terurut sesuai urutan pengiriman
- Dilengkapi flow control dan congestion control

**Keterbatasan:**
- Overhead header lebih besar (20+ byte vs 8 byte UDP)
- Ada delay akibat proses 3-way handshake
- Implementasi lebih kompleks

---

## 7. Testing Tambahan

### 7.1 Multiple Clients (UDP)

**Test:** Jalankan beberapa client UDP secara bersamaan

**Hasil:**
- UDP server bisa handle multiple client karena hanya menggunakan satu socket
- Semua client menggunakan socket yang sama di sisi server
- Pesan diproses satu per satu dalam loop `while True`

---

### 7.2 Multiple Clients (TCP)

**Test:** Coba connect beberapa client TCP secara bersamaan

**Hasil:**
- TCP server handle client secara sequential (satu per satu)
- Client kedua harus menunggu client pertama selesai dan `connectionSocket` ditutup
- Setiap client mendapatkan `connectionSocket` terpisah saat giliran dilayani

**Catatan:** Untuk handle concurrent clients secara bersamaan, diperlukan implementasi **threading** atau **multiprocessing**.

---

## 8. Kesimpulan

| Aspek | UDP Socket | TCP Socket |
|-------|------------|------------|
| **Implementasi** | Lebih sederhana | Lebih kompleks tapi reliable |
| **Koneksi** | Connectionless (tidak perlu handshake) | Connection-oriented (perlu 3-way handshake) |
| **Delivery** | Tidak ada jaminan delivery | Data terjamin sampai |
| **Urutan Data** | Tidak dijamin | Terjamin berurutan |
| **Prioritas** | Mengutamakan kecepatan | Mengutamakan keandalan |
| **Metode Kirim** | `sendto()` | `send()` |
| **Metode Terima** | `recvfrom()` | `recv()` |
| **Socket Server** | 1 socket untuk semua client | 2 socket (`serverSocket` + `connectionSocket`) |
| **Fungsi Wajib** | Tidak perlu `connect()`/`listen()`/`accept()` | Perlu `connect()`/`listen()`/`accept()` |
| **Use Case** | DNS, Streaming, VoIP, Gaming | Web, Email, File Transfer |

Dari praktikum ini dapat disimpulkan bahwa socket programming memberikan kontrol penuh terhadap komunikasi jaringan di application layer. Pemilihan antara UDP dan TCP bergantung pada kebutuhan aplikasi: jika mengutamakan kecepatan dan efisiensi gunakan UDP, sedangkan jika mengutamakan keandalan dan integritas data gunakan TCP.

---