<b>Cara Install Driver GPU Nvidia di Kali Linux</b>.<br/> 
Di tutorial kali ini saya akan sharing sedikit mengenai bagaimana cara install driver Nvidia proprietary di Kali Linux. Jika hardware kalian terpasang VGA Nvidia sebaiknya install juga drivernya agar kinerja hardware bisa lebih maksimal.<br/>
<a name='more'></a><br/>
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-BE1P5aLnw88/XiCVuvh8PTI/AAAAAAAAL7Q/xvpfpCzfuIoFCDXrFmdmsTG9-BqMFID2ACLcBGAsYHQ/s1600/nvidia%2Bsetting.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="536" data-original-width="838" height="408" src="https://1.bp.blogspot.com/-BE1P5aLnw88/XiCVuvh8PTI/AAAAAAAAL7Q/xvpfpCzfuIoFCDXrFmdmsTG9-BqMFID2ACLcBGAsYHQ/s640/nvidia%2Bsetting.png" width="640" /></a></div>
<br />
Sebelumnya saya harus beritahu kalian bahwa tutorial yang ada di web Kali Linux mengenai <b><a href="https://www.kali.org/docs/general-use/install-nvidia-drivers-on-kali-linux/">cara install driver Nvidia</a></b> tidak bisa diterapkan di laptop. Artinya, tutorial tersebut hanya bisa diterapkan jika kalian menggunakan komputer dengan GPU Nvidia saja. Sementara saat ini rata-rata laptop memiliki dual GPU yakni Intel dan Nvidia.<br />
<br />
Sebagai referensi tambahan, saya memasang Kali Linux di laptop MSI. Spesifikasi lengkapnya:<br />
<ul>
<li>OS: Kali Linux 2019.4</li>
<li>Desktop: XFCE</li>
<li>Kernel:&nbsp;5.3.0-kali2-amd64</li>
<li>CPU: Intel i7-7700HQ (8) @ 3.800GHz</li>
<li>GPU: NVIDIA GeForce GTX 1060 Mobile</li>
<li>GPU: Intel HD Graphics 630</li>
</ul>
<span style="font-size: large;"><b><u>Persiapan Instalasi</u></b></span><br />
Di beberapa kasus termasuk yang saya alami, driver open source bawaan yakni nouveau memberikan masalah yang cukup serius dimana sering tiba-tiba restart saat menjalankan perintah lspci. Solusinya, block modul tersebut agar tidak diload saat booting, dan jangan lupa restart setelah nouveau di block.<br />
Buat file blacklist dengan perintah<br />
<blockquote class="tr_bq">
sudo nano&nbsp;/etc/modprobe.d/blacklist-nouveau.conf</blockquote>
Lalu isikan:<br />
<blockquote class="tr_bq">
blacklist nouveau<br />
blacklist lbm-nouveau<br />
options nouveau modeset=0<br />
alias nouveau off<br />
alias lbm-nouveau off</blockquote>
<div>
Update initramfs dan reboot dengan perintah</div>
<blockquote class="tr_bq">
sudo&nbsp;update-initramfs -u &amp;&amp; sudo shutdown -r now</blockquote>
<div>
Oke, sekarang nouveau yang "mengganggu" sudah diblokir. Sekarang jalankan perintah berikut:</div>
<blockquote class="tr_bq">
lspci -v | grep VGA</blockquote>
<div>
Pastikan output atau hasil dari perintah tersebut menampilkan Nvidia.</div>
<div>
Jika sudah oke, sekarang masuk ke proses instalasi&nbsp;</div>
<br />
<span style="font-size: large;"><b><u>Proses Instalasi</u></b></span><br />
Sekarang kita install driver Nvidia dan juga Cuda.<br />
<blockquote class="tr_bq">
sudo apt update &amp;&amp; sudo apt install -y nvidia-driver nvidia-xconfig nvidia-settings&nbsp;<b>ocl-icd-libopencl1 nvidia-cuda-toolkit</b></blockquote>
Perhatikan dua paket terakhir yang saya cetak tebal. Paket tersebut dibutuhkan oleh GPU Nvidia yang sudah support <b><a href="https://www.geforce.com/hardware/technology/cuda">CUDA</a></b>. Jika belum, jangan diinstall.<br />
<br />
Selanjutnya adalah mengkonfigurasi agar proses desktop dijalankan oleh Nvidia. Jalankan perintah berikut untuk melihat BUS ID dari GPU Nvidia.<br />
<blockquote class="tr_bq">
nvidia-xconfig --query-gpu-info | grep 'BusID : ' | cut -d ' ' -f6</blockquote>
Contoh output<br />
<blockquote class="tr_bq">
PCI:1:0:0</blockquote>
Oke, langkah selanjutnya adalah membuat file config&nbsp;/etc/X11/xorg.conf<br />
<blockquote class="tr_bq">
sudo nano&nbsp;/etc/X11/xorg.conf</blockquote>
Isinya:<br />
<pre>Section "ServerLayout"
    Identifier "layout"
    Screen 0 "nvidia"
    Inactive "intel"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "<b>PCI:1:0:0</b>"
EndSection

Section "Screen"
    Identifier "nvidia"
    Device "nvidia"
    Option "AllowEmptyInitialConfiguration"
EndSection

Section "Device"
    Identifier "intel"
    Driver "modesetting"
EndSection

Section "Screen"
    Identifier "intel"
    Device "intel"
EndSection
</pre>
<br />
Sesuaikan sendiri BUS ID nya dengan output di langkah sebelumnya.<br />
<br />
Seperti yang saya tulis diawal, disini saya menggunakan Kali Linux dengan desktop GNOME yang menggunakan display manager GNOME. Sekarang buat file di direktori lightdm untuk mengeksekusi setup script.<br />
<blockquote class="tr_bq">
sudo nano /etc/gnome/display_setup.sh</blockquote>
Isinya<br />
<blockquote class="tr_bq">
#!/bin/sh<br />
xrandr --setprovideroutputsource modesetting NVIDIA-0<br />
xrandr --auto</blockquote>
Beri hak eksekusi file tersebut<br />
<blockquote class="tr_bq">
sudo&nbsp;chmod +x /etc/gnome/display_setup.sh</blockquote>
Selanjutnya edit file konfigurasi LightDM<br />
<blockquote class="tr_bq">
sudo nano /etc/gnome/gnome.conf</blockquote>
Pada section&nbsp;<b>[Seat:*]</b> tambahkan<br />
<blockquote class="tr_bq">
display-setup-script=/etc/gnome/display_setup.sh</blockquote>
Lihat screenshot berikut<br />
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-j7ijLeXuQ4Q/XiCR6pq9dgI/AAAAAAAAL6Y/ZrMg1hkWokIoxXXejAdMfzVBavlFbalbQCLcBGAsYHQ/s1600/setup%2Bscript.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="173" data-original-width="733" height="150" src="https://1.bp.blogspot.com/-j7ijLeXuQ4Q/XiCR6pq9dgI/AAAAAAAAL6Y/ZrMg1hkWokIoxXXejAdMfzVBavlFbalbQCLcBGAsYHQ/s640/setup%2Bscript.png" width="640" /></a></div>
<br />
Selanjutnya reboot sistem dan setelah login kembali, jalankan perintah berikut untuk memastikan bahwa driver Nvidia sudah terpasang.<br />
<blockquote class="tr_bq">
nvidia-smi</blockquote>
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-6Ovk78ogdaE/XiCSXKCzrUI/AAAAAAAAL6g/WzF8LCURUqoiU4JgXP8OXphF1YeD1mOPgCLcBGAsYHQ/s1600/nvidiainit.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="416" data-original-width="984" height="270" src="https://1.bp.blogspot.com/-6Ovk78ogdaE/XiCSXKCzrUI/AAAAAAAAL6g/WzF8LCURUqoiU4JgXP8OXphF1YeD1mOPgCLcBGAsYHQ/s640/nvidiainit.png" width="640" /></a></div>
<br />
<span style="font-size: large;"><b><u>Benchmark</u></b></span><br />
Untuk benchmark, kita bisa gunakan hashcat. Jalankan perintah<br />
<blockquote class="tr_bq">
hashcat -b</blockquote>
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-p1hAdFF_IXs/XiCSziusKaI/AAAAAAAAL6o/UrE76bCH5UMRQMBB2KpkHBeHdvfF9ZN-QCLcBGAsYHQ/s1600/hashcat%2Bbenchmark.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="384" data-original-width="911" height="268" src="https://1.bp.blogspot.com/-p1hAdFF_IXs/XiCSziusKaI/AAAAAAAAL6o/UrE76bCH5UMRQMBB2KpkHBeHdvfF9ZN-QCLcBGAsYHQ/s640/hashcat%2Bbenchmark.png" width="640" /></a></div>
<br />
Dan ini output di nvidia-smi<br />
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-Ru2QW9HE-lY/XiCTEX3_ZhI/AAAAAAAAL6w/--RxgB1YT6YYEhQGwz-B36zgs9bi9ry-wCLcBGAsYHQ/s1600/smi.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="380" data-original-width="795" height="304" src="https://1.bp.blogspot.com/-Ru2QW9HE-lY/XiCTEX3_ZhI/AAAAAAAAL6w/--RxgB1YT6YYEhQGwz-B36zgs9bi9ry-wCLcBGAsYHQ/s640/smi.png" width="640" /></a></div>
<div class="separator" style="clear: both; text-align: center;">
<br /></div>
Terakhir adalah memeriksa apakah direct rendering sudah aktif.<br />
<blockquote class="tr_bq">
glxinfo | grep -i "direct rendering"</blockquote>
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-68Ezi_J8cLw/XiCTff21-uI/AAAAAAAAL64/BAGfQNd2QyYmN_9Rg2QBRYd6VW7xFr1LwCLcBGAsYHQ/s1600/rendering.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="59" data-original-width="776" height="48" src="https://1.bp.blogspot.com/-68Ezi_J8cLw/XiCTff21-uI/AAAAAAAAL64/BAGfQNd2QyYmN_9Rg2QBRYd6VW7xFr1LwCLcBGAsYHQ/s640/rendering.png" width="640" /></a></div>
<br />
<span style="font-size: large;"><b><u>Mengatasi Screen Tearing</u></b></span><br />
Di beberapa kasus setelah menginstall driver Nvidia terjadi screen tearing saat memutar video. Kita bisa mengatasinya dengan mengaktifkan PRIME sync. Jalankan perintah<br />
<blockquote class="tr_bq">
xrandr --verbose | grep PRIME</blockquote>
Dan pastikan outputnya adalah<br />
<blockquote class="tr_bq">
PRIME Synchronization: 0</blockquote>
0 artinya PRIME sync belum aktif.<br />
Edit file&nbsp;&nbsp;<b>/etc/default/grub,</b> lalu tambahkan value&nbsp;<b>nvidia-drm.modeset=1 </b>di&nbsp;<b>GRUB_CMDLINE_LINUX_DEFAULT</b>. Lihat screenshot:<br />
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-7AwsURRDfcc/XiCUlKVoPvI/AAAAAAAAL7E/-oS5uzvttuA_3WVyv6BgFBcGhpDWcDtNACLcBGAsYHQ/s1600/grub.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="174" data-original-width="703" height="158" src="https://1.bp.blogspot.com/-7AwsURRDfcc/XiCUlKVoPvI/AAAAAAAAL7E/-oS5uzvttuA_3WVyv6BgFBcGhpDWcDtNACLcBGAsYHQ/s640/grub.png" width="640" /></a></div>
<br />
Update dan reboot<br />
<blockquote class="tr_bq">
sudo&nbsp;update-grub &amp;&amp; sudo reboot</blockquote>
Oke mungkin sekian tutorial kali ini, jika ada yang ingin ditanyakan silahkan tinggalkan komentar.<br />
<br />
<b>Referensi:</b><br />
<ul>
<li>https://forums.kali.org/showthread.php?35748-TUTORIAL-Installing-official-NVIDIA-driver-in-Optimus-laptop</li>
<li>https://wiki.archlinux.org/index.php/NVIDIA_Optimus#Display_managers</li>
</ul>
</div>
