# YouTube Dual Stream Setup (Vertical & Horizontal) using FFmpeg and Tmux on Ubuntu

This guide will help you set up **two simultaneous live streams to YouTube** on a VPS:
- One stream for **vertical video (9:16)** from a looping folder
- One stream for **horizontal video (16:9, 1080p)** from a single video file

---

### ✅ 1. Install Paket yang Diperlukan

```bash
sudo apt update
sudo apt install tmux -y
sudo apt install ffmpeg -y
sudo apt install python3 python3-pip -y
pip install gdown
```

---

### ✅ 2. (Opsional) Download File dari Google Drive dengan `gdown`

```bash
gdown https://drive.google.com/uc?id=your_file_id
```

> Ganti `your_file_id` dengan ID file Google Drive kamu

ATAU
> Final: Struktur Perintah Lengkap
```bash
pip install gdown  # kalau belum install
gdown --id your_file_id -O /root/v1.zip
unzip /root/v1.zip -d /root/
```

### ✅ 3. Persiapan Folder Video

```bash
mkdir -p /root/video_vertical
mkdir -p /root/video_normal
```
Copy File bisa pakai perintah
```bash
sudo mv /home/ubuntu/v6.mp4 /root/video_vertical/
```
---

### ✅ 4. Buat File `stream_vertical.sh`

```bash
nano /root/stream_vertical.sh
```

Isi dengan:

```bash
#!/bin/bash
while true; do
  for file in /root/video_vertical/*.mp4; do
    ffmpeg -re -i "$file" \
    -vf "fps=30,scale=720:1280" \
    -c:v libx264 -preset veryfast -b:v 4000k -maxrate 5000k -bufsize 6000k \
    -g 60 -keyint_min 60 -sc_threshold 0 \
    -c:a aac -b:a 160k -ar 44100 \
    -f flv rtmp://a.rtmp.youtube.com/live2/YOUR_VERTICAL_KEY
  done
done
```

> Ganti `YOUR_VERTICAL_KEY` dengan stream key vertikal YouTube kamu

---

### ✅ 5. Buat File `stream_horizontal.sh`

```bash
nano /root/stream_horizontal.sh
```

Isi dengan:

```bash
#!/bin/bash
ffmpeg -stream_loop -1 -re -i /root/video_normal/vid2.mp4 \
-vf "fps=30,scale=-2:1080" \
-c:v libx264 -preset veryfast -b:v 6000k -maxrate 7000k -bufsize 8000k \
-g 60 -keyint_min 60 -sc_threshold 0 \
-c:a aac -b:a 160k -ar 44100 \
-f flv rtmp://a.rtmp.youtube.com/live2/YOUR_HORIZONTAL_KEY
```

> Ganti `YOUR_HORIZONTAL_KEY` dengan stream key horizontal YouTube kamu

---

### ✅ 6. Ubah Jadi Bisa Dieksekusi

```bash
chmod +x /root/stream_vertical.sh
chmod +x /root/stream_horizontal.sh
```

---

### ✅ 7. Jalankan di `tmux`

#### ▶ Vertikal (9:16)
```bash
tmux new -s vertical
bash /root/stream_vertical.sh
```

> Tekan `Ctrl + B`, lalu `D` untuk keluar (detach)

#### ▶ Horizontal (16:9)
```bash
tmux new -s horizontal
bash /root/stream_horizontal.sh
```

> Tekan `Ctrl + B`, lalu `D` untuk keluar (detach)

---

Kalau mau lihat kembali sesi `tmux`:

```bash
tmux ls
```

Untuk masuk kembali ke sesi:

```bash
tmux attach -t vertical
# atau
tmux attach -t horizontal
```

Untuk Hapus sesi temux:

```bash
tmux kill-session -t vertical
# atau
tmux kill-session -t horizontal
```

Msih perlu butuh script auto-start saat VPS reboot
