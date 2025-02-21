# Nonaktifkan Layanan yang Tidak Diperlukan
Matikan dan hapus layanan atau daemon yang tidak digunakan, karena setiap layanan yang aktif bisa menjadi titik kelemahan.

Untuk melihat layanan yang sedang berjalan:

systemctl list-units --type=service

Untuk menghentikan dan menonaktifkan layanan yang tidak diperlukan, seperti telnet, ftp, dsb.:

sudo systemctl stop <nama_layanan>
sudo systemctl disable <nama_layanan>

# Batasi Pengguna dan Izin Akses
Batasi siapa saja yang dapat mengakses sistem Anda, dan pastikan hanya pengguna yang berhak yang dapat masuk.

Hapus atau nonaktifkan akun yang tidak digunakan.

Gunakan sudo untuk memberikan hak akses terbatas kepada pengguna biasa, daripada menggunakan akses root secara langsung.
Pastikan file konfigurasi seperti /etc/sudoers sudah benar dan menghindari penggunaan akses root sembarangan.

# Amankan Akses SSH
Jika server Anda menggunakan SSH, pastikan Anda mengonfigurasi akses SSH dengan baik untuk menghindari potensi ancaman:

Nonaktifkan login root melalui SSH dengan menambahkan atau mengedit baris berikut di /etc/ssh/sshd_config:
PermitRootLogin no

Gunakan otentikasi berbasis kunci dan nonaktifkan otentikasi berbasis kata sandi:
PasswordAuthentication no

Gunakan port non-standar untuk SSH (port 22)
Port 22

Restart SSH:
sudo systemctl restart ssh

# Gunakan SELinux atau AppArmor
Meskipun SELinux tidak diaktifkan secara default di Debian, Anda bisa mengaktifkan AppArmor, yang berfungsi untuk membatasi akses program pada sumber daya sistem.

Untuk mengaktifkan AppArmor:

sudo apt install apparmor apparmor-utils
sudo systemctl enable apparmor
sudo systemctl start apparmor

# Konfigurasi File System
Menambahkan proteksi pada sistem file bisa membantu menghindari potensi eksploitasi.

Gunakan ext4 dengan opsi noexec, nosuid, dan nodev untuk membatasi jenis akses di partisi tertentu.
Untuk mengonfigurasi ini, edit /etc/fstab dan tambahkan opsi pada partisi yang relevan, misalnya:

bash
Copy
UUID=xxxx-xxxx /tmp ext4 defaults,noexec,nosuid,nodev 0 2

# Pasang dan Gunakan Auditd
Auditd adalah alat untuk mencatat aktivitas sistem dan membantu dalam pelacakan.

Pasang dengan:
sudo apt install auditd
Setelah diinstal, Anda bisa menambahkan aturan audit untuk mencatat aktivitas penting, seperti login, perintah sudo, atau akses file tertentu.

# Gunakan PAM (Pluggable Authentication Module) untuk menetapkan kebijakan password yang kuat, seperti panjang minimal, kompleksitas, dan kedaluwarsa kata sandi.
Atur kebijakan PAM di /etc/pam.d/common-password.
Contoh kebijakan kata sandi kuat:

password requisite pam_pwquality.so retry=3 minlen=12

# Nonaktifkan IPv6 (Jika Tidak Diperlukan)
Jika Anda tidak menggunakan IPv6, Anda bisa menonaktifkannya untuk mengurangi permukaan serangan.

Untuk menonaktifkan IPv6, edit file /etc/sysctl.conf dan tambahkan baris berikut:
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

Kemudian terapkan perubahan:
sudo sysctl -p

# Amankan Paket dan Repositori
Pastikan hanya repositori yang terpercaya yang digunakan untuk menginstal perangkat lunak.

Gunakan APT pinning untuk mengunci versi paket tertentu atau hanya memperbolehkan paket dari repositori yang terpercaya.
Untuk mengunci versi paket:

bash
Copy
echo "package-name hold" | sudo dpkg --set-selections

# Menggunakan Fail2ban 
Walaupun Anda tidak menggunakan firewall, Anda bisa menggunakan Fail2ban untuk memblokir IP yang mencoba melakukan brute-force.

sudo apt install fail2ban

# Audit dan Monitor Log
Pastikan Anda memantau log sistem secara teratur untuk mendeteksi anomali dan potensi serangan.

Gunakan logwatch atau alat lainnya untuk menganalisis dan melaporkan log:

sudo apt install logwatch
sudo logwatch --detail high --mailto your-email@example.com --service all --range today
