### Gunakan Firewall (iptables) untuk Membatasi Akses
Firewall adalah alat pertama yang dapat digunakan untuk mengontrol lalu lintas jaringan yang masuk dan keluar dari sistem Anda.

Membatasi Akses Berdasarkan IP
Batasi akses ke port tertentu hanya untuk IP yang dapat dipercaya. Dengan ini, Anda hanya memperbolehkan IP yang dikenal untuk mengakses layanan tertentu.
Misalnya, jika hanya ada satu IP yang diizinkan untuk mengakses SSH, Anda bisa menambahkan aturan seperti ini:

`sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.100 -j ACCEPT`
Ini akan membatasi akses SSH hanya untuk alamat IP 192.168.1.100.

Menangkal Port Scanning dengan Rate Limiting
Anda bisa menggunakan iptables untuk membatasi laju permintaan ke server agar port scanning tidak bisa dilakukan secara bebas.

Contoh, batasi koneksi masuk pada port tertentu (misalnya, port 22 untuk SSH):
`sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set`
`sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j REJECT`
Aturan ini akan menolak koneksi lebih dari 5 kali dalam 60 detik, yang bisa melindungi dari pemindaian port yang sangat cepat.

Drop Paket dengan TTL (Time-To-Live) Rendah
Port scanner sering kali mengirimkan paket dengan nilai TTL yang rendah untuk mencoba menganalisis apakah host yang dipindai berada di belakang router atau firewall. Anda bisa membuat firewall menanggapi paket dengan TTL rendah agar pemindai kesulitan mendeteksi host Anda.

Untuk melakukannya, Anda dapat menambahkan aturan berikut di iptables:
`sudo iptables -A INPUT -p tcp -m ttl --ttl-lt 64 -j DROP'
Aturan ini akan menolak paket dengan TTL lebih rendah dari 64.

### Nonaktifkan Layanan yang Tidak Diperlukan
Matikan dan hapus layanan atau daemon yang tidak digunakan, karena setiap layanan yang aktif bisa menjadi titik kelemahan.

Untuk melihat layanan yang sedang berjalan:

`systemctl list-units --type=service`

Untuk menghentikan dan menonaktifkan layanan yang tidak diperlukan, seperti telnet, ftp, dsb.:

`sudo systemctl stop <nama_layanan>`
`sudo systemctl disable <nama_layanan>`

### Batasi Pengguna dan Izin Akses
Batasi siapa saja yang dapat mengakses sistem Anda, dan pastikan hanya pengguna yang berhak yang dapat masuk.

Hapus atau nonaktifkan akun yang tidak digunakan.

Gunakan sudo untuk memberikan hak akses terbatas kepada pengguna biasa, daripada menggunakan akses root secara langsung.
Pastikan file konfigurasi seperti /etc/sudoers sudah benar dan menghindari penggunaan akses root sembarangan.

### Amankan Akses SSH
Jika server Anda menggunakan SSH, pastikan Anda mengonfigurasi akses SSH dengan baik untuk menghindari potensi ancaman:

Nonaktifkan login root melalui SSH dengan menambahkan atau mengedit baris berikut di /etc/ssh/sshd_config:
PermitRootLogin no

Gunakan otentikasi berbasis kunci dan nonaktifkan otentikasi berbasis kata sandi:
PasswordAuthentication no

Gunakan port non-standar untuk SSH (port 22)
Port 22

Restart SSH:
`sudo systemctl restart ssh`

### Gunakan SELinux atau AppArmor
Meskipun SELinux tidak diaktifkan secara default di Debian, Anda bisa mengaktifkan AppArmor, yang berfungsi untuk membatasi akses program pada sumber daya sistem.

Untuk mengaktifkan AppArmor:

`sudo apt install apparmor apparmor-utils`
`sudo systemctl enable apparmor`
`sudo systemctl start apparmor`

### Konfigurasi File System
Menambahkan proteksi pada sistem file bisa membantu menghindari potensi eksploitasi.

Gunakan ext4 dengan opsi noexec, nosuid, dan nodev untuk membatasi jenis akses di partisi tertentu.
Untuk mengonfigurasi ini, edit /etc/fstab dan tambahkan opsi pada partisi yang relevan, misalnya:
UUID=xxxx-xxxx /tmp ext4 defaults,noexec,nosuid,nodev 0 2

### Pasang dan Gunakan Auditd
Auditd adalah alat untuk mencatat aktivitas sistem dan membantu dalam pelacakan.

Pasang dengan:
`sudo apt install auditd`
Setelah diinstal, Anda bisa menambahkan aturan audit untuk mencatat aktivitas penting, seperti login, perintah sudo, atau akses file tertentu.

### Gunakan PAM (Pluggable Authentication Module) untuk menetapkan kebijakan password yang kuat, seperti panjang minimal, kompleksitas, dan kedaluwarsa kata sandi.
Atur kebijakan PAM di /etc/pam.d/common-password.
Contoh kebijakan kata sandi kuat:

password requisite pam_pwquality.so retry=3 minlen=12

### Nonaktifkan IPv6 (Jika Tidak Diperlukan)
Jika Anda tidak menggunakan IPv6, Anda bisa menonaktifkannya untuk mengurangi permukaan serangan.

Untuk menonaktifkan IPv6, edit file /etc/sysctl.conf dan tambahkan baris berikut:
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

Kemudian terapkan perubahan:
`sudo sysctl -p`

### Amankan Paket dan Repositori
Pastikan hanya repositori yang terpercaya yang digunakan untuk menginstal perangkat lunak.

Gunakan APT pinning untuk mengunci versi paket tertentu atau hanya memperbolehkan paket dari repositori yang terpercaya.
Untuk mengunci versi paket:

`echo "package-name hold" | sudo dpkg --set-selections`

### Gunakan Fail2Ban untuk Memblokir IP yang Mencurigakan
Fail2Ban adalah alat yang dapat digunakan untuk melindungi server dari brute-force attacks dan port scanning dengan memblokir IP yang mencoba mengakses port secara berulang kali dalam waktu singkat.

Untuk menginstal dan mengonfigurasi Fail2Ban di Debian:

Install Fail2Ban:
`sudo apt install fail2ban`

Konfigurasi Fail2Ban untuk melindungi layanan SSH:
Edit file konfigurasi utama:
`sudo nano /etc/fail2ban/jail.local`

Tambahkan aturan berikut untuk mengaktifkan perlindungan SSH:
[sshd]
enabled  = true
port     = ssh
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 600

Penjelasan:
enabled: Mengaktifkan perlindungan untuk SSH.
port: Port yang digunakan oleh SSH.
logpath: Lokasi file log yang digunakan untuk melacak upaya login.
maxretry: Jumlah percakapan gagal sebelum IP diblokir.
bantime: Durasi waktu IP diblokir (dalam detik).

Restart Fail2Ban untuk menerapkan perubahan:
`sudo systemctl restart fail2ban`

### Audit dan Monitor Log
Pastikan Anda memantau log sistem secara teratur untuk mendeteksi anomali dan potensi serangan.

Gunakan logwatch atau alat lainnya untuk menganalisis dan melaporkan log:

`sudo apt install logwatch`
`sudo logwatch --detail high --mailto your-email@example.com --service all --range today`
