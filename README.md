# 🛠️ Integrated Network Infrastructure: Multi-VLAN, VLSM Subnetting, NAT, and Secured Extended ACL
### 🏢 Enterprise Network Blueprint Design for PT Global Tekno Nusantara (PT GTN)

---

## 📌 Project Overview & Client Requirements

Dokumentasi ini menyajikan arsitektur jaringan terintegrasi yang dirancang khusus untuk memenuhi seluruh spesifikasi teknis dan kebutuhan operasional dari **Sarah Lockhart** selaku perwakilan manajemen **PT Global Tekno Nusantara (PT GTN)**. 

<img width="1193" height="558" alt="TOPOLOGY 3 FLOOR  drawio (1)" src="https://github.com/user-attachments/assets/9b1e6a57-737a-4b53-8b2d-0f1f0fa183b4" />
<br><br>



Proyek ini mengintegrasikan pembagian wilayah logis korporat 3 lantai menggunakan metode **VLSM (Variable Length Subnet Mask)**, penyebaran parameter **DHCP Pool** terpusat dari Switch Core yang berelasi dengan IP Gateway Router Utama, integrasi **Trunking Lintas Lantai**, implementasi translasi alamat **NAT (Inside/Outside)** via jalur Serial ISP, konfigurasi **Wireless WPA2-PSK** menggunakan perangkat **Smartphone**, serta pengamanan ketat **Extended Access Control List (ACL)** untuk mengisolasi divisi finansial tanpa memutus akses internet.

### 🏢 Pemenuhan Kebutuhan Klien (Sarah Lockhart - PT GTN):
1. **Efisiensi Ruang Alamat IP (VLSM):** Menggunakan teknik perhitungan VLSM agar alokasi IP di setiap lantai pas dengan jumlah perangkat riil karyawan, sehingga mengeliminasi pemborosan IP address.
2. **Isolasi Mutlak Data Finansial:** Memastikan Divisi Finance di Lantai 2 tidak dapat diintip atau diakses oleh divisi internal kantor lainnya guna menjaga kerahasiaan data perusahaan.
3. **Keamanan Akses Nirkabel (Wireless Security):** Menyediakan koneksi Wi-Fi terpisah dengan enkripsi kuat WPA2-PSK AES menggunakan perangkat **Smartphone** untuk mobilitas tinggi.
4. **Transparansi & Keamanan Akses Publik (NAT):** Menyembunyikan seluruh IP private lokal korporat di belakang satu IP publik legal pada saat menembus internet/ISP, sehingga server publik di luar tidak dapat melacak struktur IP internal kantor.

---

## 📐 Jaringan Cetak Biru & Relasi Topologi Lintas Lantai

Sistem ini memisahkan peran perangkat jaringan secara tegas guna menjamin performa distribusi IP lokal terpusat dan jalur perutean yang efisien menuju internet.

### 📊 Skema Alokasi Subnetting VLSM Jaringan Klien
| VLAN ID | Nama Divisi / VLAN | Alokasi Lantai | Network Address (VLSM) | Subnet Mask | Wildcard Mask | Default Router / Gateway IP (di Router Utama) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **VLAN 40** | Staff_Wireless | Lantai 3 | `192.168.10.0/26` | `255.255.255.192` | `0.0.0.63` | `192.168.10.1` |
| **VLAN 50** | Guest | Lantai 1 | `192.168.10.64/27` | `255.255.255.224` | `0.0.0.31` | `192.168.10.65` |
| **VLAN 30** | Finance | Lantai 2 | `192.168.10.96/28` | `255.255.255.240` | `0.0.0.15` | `192.168.10.97` |
| **VLAN 20** | IT_Support | Lantai 3 | `192.168.10.112/28` | `255.255.255.240` | `0.0.0.15` | `192.168.10.113` |
| **VLAN 10** | Management | Lantai 3 | `192.168.10.128/29` | `255.255.255.248` | `0.0.0.7` | `192.168.10.129` |

### 🔗 Alur Kerja Perangkat & Peran Infrastruktur:
* **Switch Akses Lantai (Lantai 1, 2, & 3):** Ditempatkan di masing-masing lantai untuk memetakan port fisik komputer dan Access Point (AP) karyawan ke VLAN-ID yang sesuai. Perangkat ini bekerja murni pada Layer 2—**tidak mengelola IP address maupun DHCP pool**.
* **Switch Core Utama (Pusat DHCP Pool & Trunk VLAN):** Bertindak sebagai agregator utama di Layer 2. Perangkat ini bertugas murni untuk **mengatur lalu lintas Trunk VLAN** (IEEE 802.1Q) dari seluruh switch akses lantai menuju ke atas. Di samping itu, Switch Core memegang database **DHCP Pool** terpusat untuk membagikan alamat IP secara otomatis ke komputer dan smartphone klien. **DHCP Pool di Switch Core murni membagikan IP dan parameter network, serta tidak mengatur IP route sama sekali.**
* **Router Utama (Pusat Gateway, NAT, IP Route & Pemancar Internet):** Router Utama adalah pusat kecerdasan Layer 3 di jaringan ini. Router Utama memegang seluruh sub-interface virtual sebagai alamat IP Gateway riil bagi masing-masing VLAN (`192.168.10.1` hingga `192.168.10.129`). Perangkat ini bertanggung jawab melakukan routing antar-VLAN, mentranslasikan IP lewat NAT, dan mengeksekusi kendali routing tabel (*IP Route*) tunggal menuju internet/WAN.

---

## 🚀 Fitur Utama & Kebijakan Logika Jaringan

### A. Otomasi Distribusi IP Berbasis Hubungan DHCP Pool & Gateway
Saat perangkat end-user (*PC atau Smartphone*) di tiap lantai mengirimkan permintaan alamat IP (*DHCP Request*), **Switch Core Utama** menangkap pesan tersebut melalui port VLAN terkait dan menyuplai konfigurasi IP. 

Karena DHCP Pool di Switch Core tidak mengelola rute, ia menanamkan informasi IP Gateway milik **Router Utama** sebagai *Default Gateway* pada klien, sehingga paket data memiliki target lompatan (*next-hop*) langsung ke arah router pemancar internet.

#### 📸 Bukti Database DHCP Pool pada Switch Core Utama
<img width="332" height="297" alt="ip dhcp pool" src="https://github.com/user-attachments/assets/d72ee879-3a48-412d-b786-7ac79bc70326" />
<br><br>

* **Keterangan Visual:** Menampilkan status database pool IP dinamis terpusat (`POOL_STAFF`, `POOL_FINANCE`, `POOL_ITSUPPORT`, `POOL_MANAGEMENT`) di Switch Core, memperlihatkan rentang subnet mask VLSM serta parameter `default-router` yang mengarah tepat ke IP sub-interface milik Router Utama tanpa adanya konfigurasi routing lokal.

#### 📸 Bukti Aktivitas Interface & Trunking pada Switch Core

<img width="287" height="335" alt="IP INTERFCE SWITCH CORE" src="https://github.com/user-attachments/assets/3c1e7cd2-94e3-4562-8b2f-63dc1df82082" />
<br><br>

* **Keterangan Visual:** Menampilkan konfigurasi interface di Switch Core yang bertindak sebagai pengatur interkoneksi L2, termasuk penugasan inter-switch trunk native vlan dan pembagian ke port akses lantai.

#### 📸 Bukti Aktivitas Sub-Interface Virtual pada Router Utama
<img width="660" height="257" alt="ip interface routerutama" src="https://github.com/user-attachments/assets/97cd9d78-e046-460a-b899-1147274389c8" />
<br><br>

* **Keterangan Visual:** Menampilkan status seluruh sub-interface virtual terkonfigurasi (`FastEthernet` / `GigabitEthernet`) berstatus *up/up* pada Router Utama yang bertindak sebagai pemegang IP gateway utama bagi traffic dari DHCP klien.

---

### B. Konfigurasi Translasi Alamat Jaringan (IP NAT Inside & Outside)
Agar perangkat klien dengan alamat IP Private lokal dapat berkomunikasi di jaringan internet global, **Router Utama** mengaktifkan fitur **Network Address Translation (NAT/PAT)** dengan pembagian zona interkoneksi yang ketat:

1. **`ip nat inside` (Jaringan Dalam):** Ditetapkan dan diaktifkan secara menyeluruh di setiap sub-interface virtual VLAN korporat. Ini menandakan bahwa seluruh paket data yang datang dari komputer lantai diidentifikasi sebagai paket lokal internal yang berhak ditranslasikan.
2. **`ip nat outside` (Jaringan Luar):** Ditetapkan secara eksklusif pada port WAN fisik (Interface Serial) di Router Utama yang terhubung langsung menggunakan kabel serial menuju ke arah Router ISP (Internet Service Provider).

#### 📸 Bukti Penetapan Zona NAT pada Router Utama
<img width="347" height="377" alt="IP NAT INSIDE ROUTERUTAMA" src="https://github.com/user-attachments/assets/c8c973d6-a965-48b2-af8c-179c0af4e301" />
<br><br>

* **Keterangan Visual:** Menampilkan konfigurasi baris perintah sistem di mana status `ip nat inside` melekat erat pada tiap sub-interface virtual divisi lokal korporat.

---

### C. Mekanisme IP Route Terpusat & Topologi Routing ISP
Seluruh kendali perutean dipusatkan mutlak di perimeter luar (Router Utama), sedangkan Router ISP hanya mengenali IP publik link serial.

#### 📸 Peta Jalur Routing Table Terpusat di Router Utama & ISP
<img width="655" height="337" alt="ip route ROUTERUTAM" src="https://github.com/user-attachments/assets/05785ef4-2d14-441e-84a9-c955bf7dfe6e" />
<br><br>

* **Keterangan Visual:** Menampilkan status tabel routing statis dan dinamis pada Router Utama, membuktikan konfigurasi target lompatan rute default (`0.0.0.0/0`) mengarah sukses ke network luar via gateway ISP.

---

### D. Fitur Perimeter Keamanan Nirkabel & Isolasi Data (Extended ACL)

#### 1. Keamanan Nirkabel (Wireless Security WPA2-PSK AES via Smartphone)
Jaringan Wireless didistribusikan melalui dua Access Point (AP) terpisah yang bertindak sebagai Layer 2 Bridge. Keamanan dikunci menggunakan enkripsi **WPA2-PSK AES** dengan end-user menggunakan perangkat **Smartphone**.

* **AP STAFF (Lantai 3):** Terhubung ke port switch access VLAN 40.
* **AP GUEST (Lantai 1):** Terhubung ke port switch access VLAN 50.

#### 2. Dinding Pengaman Wilayah Finansial (SECURE_FINANCE Extended ACL)
Sesuai instruksi Sarah Lockhart mengenai privasi data keuangan PT GTN, divisi **Finance (VLAN 30)** yang berada di Lantai 2 dipasangi aturan dinding pengaman **Extended Access Control List (ACL)** bernama `SECURE_FINANCE`. Aturan ini diterapkan langsung pada sub-interface gateway VLAN 30 di Router Utama dengan ketentuan logika:
* Mencegat dan memblokir seketika (*deny*) segala paket data yang keluar dari subnet Finance apabila tujuannya mengarah ke area internal kantor lainnya (Management, IT Support, Staff, Guest) demi keamanan data internal.
* Tetap memberikan izin penuh (*permit*) paket data dari Finance jika tujuannya mengarah ke alamat global/luar (Internet / Web Server Publik).

#### 📸 Struktur Aturan Aturan Extended ACL
<img width="467" height="111" alt="SECURE_FINANCE " src="https://github.com/user-attachments/assets/9cead9f9-037d-4d3c-9f50-03b13e1eca4e" />
<br><br>

* **Keterangan Visual:** Dokumentasi isi aturan ACL `SECURE_FINANCE` yang dipasang pada sub-interface dengan perintah pemblokiran IP private antar-divisi.

---

## 🔍 Lembar Validasi Keberhasilan Proyek (UAT Testing)

Bagian ini menyajikan bukti pengujian validasi menyeluruh untuk memastikan otomasi DHCP Pool per VLAN berhasil didapatkan oleh end-user (*termasuk Smartphone*), sistem keamanan ACL berjalan, dan konektivitas web server eksternal tembus internet.

### 📊 Uji 1: Alokasi Sukses DHCP Pool Terdistribusi per VLAN & AP
Pengujian ini membuktikan bahwa **masing-masing VLAN lintas lantai telah berhasil 100% mendapatkan alamat IP secara otomatis** yang valid dan presisi sesuai dengan jangkauan pool subnet VLSM masing-masing menggunakan PC dan Smartphone.

*   📸 **AP GUEST & AP STAFF Interface Configuration**
    
    <img width="947" height="392" alt="AP GUEST " src="https://github.com/user-attachments/assets/f51d92a1-62c2-41bb-a444-2544137a7c18" />
    <br><br>

<img width="962" height="397" alt="AP STAFF" src="https://github.com/user-attachments/assets/f8e8156d-863d-4955-bb4c-a5bcff81e66a" />
<br><br>
    * **Keterangan Visual:** Antarmuka wireless interface konfigurasi pada Access Point yang sukses mengunci SSID dan enkripsi WPA2-PSK.

*   📸 **Bukti Penerimaan IP Valid - Smartphone Staff Wireless (VLAN 40)**
   
    <img width="957" height="562" alt="DHCP  SUKSES STAFF" src="https://github.com/user-attachments/assets/ff55d866-1241-40d7-ad9a-c6887f3a36de" />
<br><br>
    * **Keterangan Visual:** Menampilkan layar **Smartphone Staff** sukses terasosiasi ke SSID Staff dan mendapatkan IP dinamis valid dari segmen DHCP pool VLAN 40.
*   📸 **Bukti Penerimaan IP Valid - PC IT Support (VLAN 20)**
  
    <img width="952" height="332" alt="DHCP  SUKSES VLAN 20 " src="https://github.com/user-attachments/assets/f8314f40-41a5-438f-befd-73a6f4cc2cd6" />
<br><br>

    * **Keterangan Visual:** Menampilkan PC IT Support sukses mendapatkan IP dinamis valid dari segmen DHCP pool VLAN 20.
*   📸 **Bukti Penerimaan IP Valid - PC Finance (VLAN 30)**

    <img width="955" height="322" alt="DHCP  SUKSES VLAN 30 FINANCE" src="https://github.com/user-attachments/assets/e8f8af26-1355-4d67-918e-6fddaa1407d6" />
<br><br>

    * **Keterangan Visual:** Menampilkan PC Finance sukses mendapatkan IP dinamis valid dari segmen DHCP pool VLAN 30.
*   📸 **Bukti Penerimaan IP Valid - Smartphone Guest Wireless (VLAN 50)**
   
    * **Keterangan Visual:** Menampilkan layar **Smartphone Guest** sukses terasosiasi ke SSID Guest dan mendapatkan IP dinamis valid dari segmen DHCP pool VLAN 50.
*   📸 **Bukti Penerimaan IP Valid - PC Management (VLAN 10)**
    
    <img width="960" height="330" alt="DHCP SUKSES VLAN 10" src="https://github.com/user-attachments/assets/738ce309-31bd-49d5-ae80-86c787ee3ece" />
<br><br>
    * **Keterangan Visual:** Menampilkan PC Management sukses mendapatkan IP dinamis valid dari segmen DHCP pool VLAN 10.

---

### 📊 Uji 2: Pembuktian Translasi Alamat Real-Time (`show ip nat translations`)
Memastikan bahwa fungsi `ip nat inside` pada sub-interface vlan dan `ip nat outside` pada serial interface benar-benar melakukan translasi paket secara riil saat perangkat melewati batas luar kantor.

#### 📸 Tabel Translasi IP NAT Aktif

<img width="627" height="136" alt="IP NAT TRANSLATION" src="https://github.com/user-attachments/assets/f54aaf6e-80e2-4b13-8311-6f59b23a712b" />
<br><br>
* **Keterangan Visual:** Menampilkan log pemetaan dinamis (`show ip nat translations`) yang memperlihatkan IP asal private local korporat diubah secara instan menjadi IP publik global legal saat melintasi jalur WAN.

---

### 📊 Uji 3: Kegagalan Ping Akibat Proteksi Isolasi Divisi (Finance ❌ Internal Kantor)
Membuktikan secara nyata dari sisi pengguna (*end-user*) PC Finance bahwa komunikasi data ke jaringan internal kantor lainnya telah dibatasi total oleh sistem keamanan Extended ACL.

#### 📸 Terminal Konsol Uji Blokir Internal

<img width="545" height="370" alt="TEST SECURE FINANCE HOST UNREACHABLE" src="https://github.com/user-attachments/assets/f223462c-0331-4e01-a5c8-451014344b46" />
<br><br>
* **Keterangan Visual:** Konsol terminal pada PC Finance menampilkan respon **`Reply from 192.168.10.97: Destination host unreachable`**, membuktikan paket data sukses dihadang di tingkat gateway Router Utama berkat aturan ACL `SECURE_FINANCE`.

---

### 📊 Uji 4: Keberhasilan Akses Layanan Web Publik via Browser (Tembus Internet 🌐)
Membuktikan fungsionalitas riil bahwa jaringan berhasil menembus internet luar yang dipancarkan oleh Router Utama melewati ISP menuju ke server web tujuan yang dihosting secara global.

#### 📸 Validasi Visual Web Browser PC & Smartphone - Sukses Akses Halaman Web Publik
<img width="947" height="427" alt="web sukses" src="https://github.com/user-attachments/assets/32f8f497-6489-4acc-b543-1b580923ea55" />
<br><br>
* **Keterangan Visual:** Tampilan antarmuka grafis (GUI) Web Browser yang **sukses memuat halaman HTML internal/public web server** milik PT GLOBAL TEKNO NUSANTARA secara utuh. Ini menjadi bukti mutlak bahwa seluruh alur interkoneksi IP Route, distribusi DHCP Pool, pemancaran NAT, konektivitas nirkabel Smartphone, dan pembatasan ACL telah sukses berjalan sempurna dan siap dideploy.
