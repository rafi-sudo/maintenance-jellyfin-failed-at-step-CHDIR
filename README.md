# maintenance-jellyfin-failed-at-step-CHDIR
# Case
jellyfin.service: Changing to the requested working directory failed:01:42 armbian-nis systemd[2749]: jellyfin.service: Failed at step CHDIR spawning /usr/bin/jellyfin: No such directory saat liat ```journalctl -u jellyfin atau systemctl status jellyfin```
# Explanation
Pesan kesalahan tersebut menunjukkan bahwa sistem gagal menemukan direktori kerja yang diminta untuk layanan jellyfin. Ini bisa terjadi karena beberapa alasan, seperti direktori yang hilang atau kesalahan konfigurasi.

Untuk mengatasi masalah ini, coba beberapa langkah berikut:

Periksa apakah direktori /usr/bin/jellyfin ada: Cek apakah file executable Jellyfin ada di direktori tersebut dengan menjalankan perintah:
<pre>
<code id="code-block">
cat /usr/bin/jellyfin
</code>
</pre>

Jika file tersebut tidak ada, mungkin instalasi Jellyfin tidak lengkap atau terjadi kesalahan saat pemasangan.
**Perbaiki jalur direktori kerja di unit systemd**: Pastikan bahwa file unit jellyfin.service mengarah ke jalur yang benar. Buka file unit service Jellyfin menggunakan editor teks, misalnya nano:
<pre>
<code id="code-block">
sudo nano /etc/systemd/system/jellyfin.service
</code>
</pre>

# Case : Saya malah gini direkori nya /etc/systemd/system/jellyfin.service.d/jellyfin.service.conf
Kalau direktori konfigurasi unit jellyfin.service berada di ```/etc/systemd/system/jellyfin.service.d/jellyfin.service.conf```, itu berarti ada override file yang digunakan untuk menyesuaikan konfigurasi default layanan Jellyfin.

Untuk memperbaiki masalah, kamu bisa cek dan edit file tersebut dengan langkah-langkah berikut:
1. Periksa isi file override
Gunakan editor teks untuk melihat isi file konfigurasi tersebut:
```sudo nano /etc/systemd/system/jellyfin.service.d/jellyfin.service.conf```
Cari bagian konfigurasi seperti WorkingDirectory. Contohnya:
<pre>
<code id="code-block">
[Service]
WorkingDirectory=/path/yang/salah
ExecStart=/path/yang/salah/jellyfin
</code>
</pre>

# Case : Isinya gini :
<pre>
<code id="code-block">
them causes "restart.sh" functionality to break with the following error:
#   sudo: effective uid is not 0, is /usr/bin/sudo on a file system with the                                            #   'nosuid' option set or an NFS file system without root privileges?                                                  # See issue #7503 for details on the troubleshooting that went into this.
# Since these were added for NixOS specifically and are above and beyond
# what 99% of systemd units do, they have been moved here as optional
# additional flags to set for maximum system security and can be enabled at  # the administrator's or package maintainer's discretion.                  # Uncomment these only if you know what you're doing, and doing so may cause                                            
# bugs with in-server Restart and potentially other functionality as well.   #NoNewPrivileges=true
#SystemCallArchitectures=native
#RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6 AF_NETLINK                #RestrictNamespaces=false                                                   #RestrictRealtime=true                                                      #RestrictSUIDSGID=true                                                      #ProtectControlGroups=false                                                 #ProtectHostname=true                                                        #ProtectKernelLogs=false                                                    #ProtectKernelModules=false                                                 #ProtectKernelTunables=false                  
#LockPersonality=true         
#PrivateTmp=false                                                           
#PrivateDevices=false                        
#PrivateUsers=true
#RemoveIPC=true
#SystemCallFilter=~@clock
#SystemCallFilter=~@aio                                                     #SystemCallFilter=~@chown                                                   #SystemCallFilter=~@cpu-emulation
#SystemCallFilter=~@debug
#SystemCallFilter=~@keyring
#SystemCallFilter=~@memlock                                                 #SystemCallFilter=~@module                                                  #SystemCallFilter=~@mount
#SystemCallFilter=~@obsolete
#SystemCallFilter=~@privileged                                              #SystemCallFilter=~@raw-io                                                  #SystemCallFilter=~@reboot
#SystemCallFilter=~@setuid
#SystemCallFilter=~@swap                                                     SystemCallErrorNumber=EPERM
</code>
</pre>

Dari isi file override tersebut, terlihat bahwa file ini hanya berisi komentar (dimulai dengan #). Tidak ada konfigurasi aktif yang akan diterapkan oleh systemd dari file ini. Beberapa hal yang perlu diperhatikan dan dilakukan:
 1. Periksa Konfigurasi Default Jellyfin
Karena file ini hanya berisi komentar, maka konfigurasi default layanan jellyfin kemungkinan masih berasal dari file unit utama jellyfin.service. Kamu bisa memeriksanya dengan:
```cat /lib/systemd/system/jellyfin.service```
Cek apakah ada bagian WorkingDirectory dan ExecStart yang salah.
# Case isinya gini :
<pre>
<code id="code-block">
[unit] description = Jellyfin Media Server
After = network-online.target

[Service]
Type = simple
EnvironmentFile = /etc/default/jellyfin
User = jellyfin
Group = jellyfin
WorkingDirectory = /var/lib/jellyfin
ExecStart = /usr/bin/jellyfin $JELLYFIN_WEB_OPT $JELLYFIN_RESTART_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT $JELLYFIN_NOWEBAPP_OPT $JELLYFIN_ADDITIONAL_OPTS
Restart = on-failure
TimeoutSec = 15
SuccessExitStatus=0 143

[Install]
WantedBy = multi-user.target
</code>
</pre>

Dari isi file tersebut, file unit service jellyfin.service terlihat cukup standar dan mengarah ke konfigurasi yang seharusnya. Berikut poin-poin penting yang perlu dicek dan langkah penyelesaian masalah:
 1. Cek Eksistensi File dan Direktori
Beberapa jalur yang tercantum perlu diverifikasi apakah benar ada di sistemmu:
***Cek apakah file ```/etc/default/jellyfin``` ada:***
```cat /etc/default/jellyfin```
***Jika file tersebut tidak ada, buat file kosongnya terlebih dahulu:***
```sudo touch /etc/default/jellyfin```
***Cek apakah direktori ```/var/lib/jellyfin``` ada:***
```cat /var/lib/jellyfin```
***Jika tidak ada, buat direktori tersebut:***
<pre>
<code id="code-block">
sudo mkdir -p /var/lib/jellyfin
sudo chown jellyfin:jellyfin /var/lib/jellyfin
</code>
</pre>

***Cek apakah file binary Jellyfin ada di ```/usr/bin/jellyfin:```***
```cat /usr/bin/jellyfin```
# Case di yang ```etc/default/jellyfin``` isinya gini, gimana? 
ExecStart = /usr/bin/jellyfin $JELLYFIN_WEB_OPT $JELLYFIN_RESTART_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT $JELLYFIN_NOWEBAPP_OPT $JELLYFIN_ADDITIONAL_OPTS
Baris ExecStart di konfigurasi tersebut menggunakan beberapa variabel lingkungan seperti ```$JELLYFIN_WEB_OPT, $JELLYFIN_RESTART_OPT```, dan lainnya. Variabel-variabel ini biasanya didefinisikan di file EnvironmentFile (yaitu ```/etc/default/jellyfin``` dalam kasus ini). Jika variabel-variabel tersebut **tidak ada atau salah konfigurasi**, layanan Jellyfin bisa gagal berjalan.

Berikut langkah-langkah untuk memastikan semuanya benar:
1. Cek apakah file /etc/default/jellyfin ada dan memiliki konfigurasi yang benar:
```cat /etc/default/jellyfin```
2. Isi file ini biasanya mendefinisikan variabel seperti:
   <pre>
<code id="code-block">
JELLYFIN_WEB_OPT=""
JELLYFIN_RESTART_OPT=""
JELLYFIN_FFMPEG_OPT=""
JELLYFIN_SERVICE_OPT=""
JELLYFIN_NOWEBAPP_OPT=""
JELLYFIN_ADDITIONAL_OPTS=""
</code>
</pre>

3. Jika file ini tidak ada atau tidak lengkap, buat file baru:
5. Lalu tambahkan code diatas dengan ```sudo nano```
# Case : isi ```cat /etc/default/jellyfin```
Jellyfin default configuration options
#This is a POSIX shell fragment

#Use this file to override the default configurations; add additional
#options with JELLYFIN_ADD_OPTS.

#Under systemd, use
#/etc/systemd/system/jellyfin.service.d/jellyfin.service.conf
#to override the user or this config file's location.

#General options

#Program directories
JELLYFIN_DATA_DIR="/var/lib/jellyfin"
JELLYFIN_CONFIG_DIR="/etc/jellyfin"
JELLYFIN_LOG_DIR="/var/log/jellyfin"
JELLYFIN_CACHE_DIR="/var/cache/jellyfin"

#web client path, installed by the jellyfin-web package
JELLYFIN_WEB_OPT="--webdir=/usr/share/jellyfin/web"

#Restart script for in-app server control
JELLYFIN_RESTART_OPT="--restartpath=/usr/lib/jellyfin/restart.sh"

#ffmpeg binary paths, overriding the system values
JELLYFIN_FFMPEG_OPT="--ffmpeg=/usr/lib/jellyfin-ffmpeg/ffmpeg"

#Disable glibc dynamic heap adjustment
MALLOC_TRIM_THRESHOLD_=131072

#[OPTIONAL] run Jellyfin as a headless service
#JELLYFIN_SERVICE_OPT="--service"

#[OPTIONAL] run Jellyfin without the web app
#JELLYFIN_NOWEBAPP_OPT="--nowebclient"

#Space to add additional command line options to jellyfin (for help see ~$ jellyfin --help)
JELLYFIN_ADDITIONAL_OPTS=""

#[OPTIONAL] run Jellyfin with ASP.NET Server Garbage Collection (uses more RAM and less CPU than Workstation GC)
#0 = Workstation
#1 = Server
#COMPlus_gcServer=1
#SysV init/Upstart options
#Note: These options are ignored by systemd; use /etc/systemd/system/jellyfin.d overrides instead.
#Application username
JELLYFIN_USER="jellyfin"
#Full application command
JELLYFIN_ARGS="$JELLYFIN_WEB_OPT $JELLYFIN_RESTART_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT $JELLYFIN_NOWEBAPP_OPT $JELLFIN_ADDITIONAL_OPTS"

Di akhir file terdapat typo pada variabel JELLFIN_ADDITIONAL_OPTS
<pre>
<code id="code-block">
RAM and less CPU than Workstation GC)
#0 = Workstation
#1 = Server
#COMPlus_gcServer=1
#SysV init/Upstart options
#Note: These options are ignored by systemd; use /etc/systemd/system/jellyfin.d overrides instead.
#Application username
JELLYFIN_USER="jellyfin"
#Full application command
JELLYFIN_ARGS="$JELLYFIN_WEB_OPT $JELLYFIN_RESTART_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT $JELLYFIN_NOWEBAPP_OPT --->$JELLFIN_ADDITIONAL_OPTS"    <------- Typo
</code>
</pre>
, yang seharusnya JELLYFIN_ADDITIONAL_OPTS. Ubah baris terakhir menjadi:
```JELLYFIN_ARGS="$JELLYFIN_WEB_OPT $JELLYFIN_RESTART_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT $JELLYFIN_NOWEBAPP_OPT $JELLYFIN_ADDITIONAL_OPTS"```
setelah itu
 1. Periksa Jalur yang Didefinisikan
Beberapa jalur penting yang disebutkan di file ini harus diperiksa keberadaannya: seperti
<pre>
<code id="code-block">
ls /var/lib/jellyfin
ls /etc/jellyfin
ls /var/log/jellyfin
ls /var/cache/jellyfin
</code>
</pre>
bisa pakai ```ls``` atau ftp/sftp ssh terminal.

# Case :
1. /var/lib/jellyfin ini ada tapi berupa jellyfin.bak
2. /etc/jellyfin ada tapi berupa jellyfin.bak
3. /var/log/jellyfin tidak ada
4. /var/cache/jellyfin ada

Solusi :
1. Direktori /var/lib/jellyfin dan /etc/jellyfin berbentuk jellyfin.bak
Ini menunjukkan bahwa direktori asli mungkin telah di-backup sebelumnya. Untuk menjalankan Jellyfin dengan benar, direktori yang sesuai harus dikembalikan atau dibuat ulang.
Jalankan perintah
<pre>
<code id="code-block">
sudo mv /var/lib/jellyfin.bak /var/lib/jellyfin
sudo mv /etc/jellyfin.bak /etc/jellyfin
sudo rm -rf /var/lib/jellyfin.bak /etc/jellyfin.bak
sudo mkdir -p /var/lib/jellyfin /etc/jellyfin
sudo chown -R jellyfin:jellyfin /var/lib/jellyfin /etc/jellyfin
</code>
</pre>

2. Direktori /var/log/jellyfin tidak ada
Jellyfin membutuhkan direktori ini untuk menyimpan log. Direktori ini harus dibuat secara manual.
jalankan perintah:
<pre>
<code id="code-block">
sudo mkdir -p /var/log/jellyfin
sudo chown -R jellyfin:jellyfin /var/log/jellyfin
</code>
</pre>

3. Pastikan kepemilikan direktori /var/cache/jellyfin sudah benar:
   ```sudo chown -R jellyfin:jellyfin /var/cache/jellyfin```
4. Setelah semua direktori disiapkan, reload dan restart layanan Jellyfin:
```sudo systemctl daemon-reload```
```sudo systemctl restart jellyfin```
  Periksa status layanan:
```sudo systemctl status jellyfin```
# Finish Case
Kenapa variable ini 
```JELLYFIN_ARGS="$JELLYFIN_WEB_OPT $JELLYFIN_RESTART_OPT $JELLYFIN_FFMPEG_OPT $JELLYFIN_SERVICE_OPT $JELLYFIN_NOWEBAPP_OPT $JELLYFIN_ADDITIONAL_OPTS"```
Sangatlah penting.
Variabel **JELLYFIN_ARGS** sangat penting dan sensitif terhadap penulisan case karena sistem Unix/Linux membedakan huruf kapital dan kecil. Kesalahan penulisan pada variabel seperti **JELLYFIN_WEB_OPT** atau **JELLYFIN_FFMPEG_OPT** dapat menyebabkan Jellyfin gagal mengenali konfigurasi atau file yang dibutuhkan, mengakibatkan aplikasi tidak berjalan dengan benar. Oleh karena itu, penulisan variabel harus konsisten dengan case yang benar agar Jellyfin berfungsi seperti yang diharapkan.

Variabel JELLYFIN_ARGS digunakan untuk ***menggabungkan berbagai*** ***opsi konfigurasi yang diperlukan untuk menjalankan Jellyfin***. Ini mencakup pengaturan seperti ***direktori web, path FFmpeg, dan opsi restart***. Penulisan variabel yang tepat dan konsisten (termasuk case-sensitive) sangat penting agar Jellyfin dapat menemukan dan menggunakan konfigurasi yang benar, sehingga server berjalan dengan normal.

