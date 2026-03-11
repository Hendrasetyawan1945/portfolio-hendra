# 🦞 DOKUMENTASI KONFIGURASI OPENCLAW

OpenClaw 2026.3.2 — Server: VM-12-126-opencloudos

## 1. Menonaktifkan Sandbox (Full Host Mode)

Sandbox dinonaktifkan agar agent dapat mengakses sistem host secara langsung tanpa isolasi.

### Lokasi File
`~/.openclaw/openclaw.json`

### Konfigurasi yang Ditambahkan
```json
"agents": {
  "defaults": {
    "sandbox": {
      "mode": "off"
    }
  }
}
```

### Verifikasi
```bash
openclaw sandbox explain
```

Output yang benar:
```
mode: off  scope: agent  runtime: direct
```

## 2. Mengaktifkan Tools Full Execution

Mengubah tools profile dari 'messaging' ke 'full' agar agent bisa menjalankan perintah exec, process, read, write, dan edit.

### Konfigurasi
```json
"tools": {
  "profile": "full",
  "sandbox": {
    "tools": {
      "allow": ["exec", "process", "read", "write", "edit"]
    }
  }
}
```

### Status Tools Setelah Perubahan
✅ ALLOW (global): exec, process, read, write, edit, image
❌ DENY (default): browser, canvas, gateway, telegram, whatsapp, discord

## 3. Konfigurasi Telegram Bot

Bot Telegram dikonfigurasi untuk menerima pesan dari semua grup (groupPolicy: open).

### Konfigurasi Channel Telegram
```json
"channels": {
  "telegram": {
    "enabled": true,
    "dmPolicy": "pairing",
    "botToken": "8768856097:AAH_AnhQbfS0pT3me_ykneDdxZLwm7GZLzk",
    "groupPolicy": "open",
    "streaming": "partial"
  }
}
```

⚠️  Sebelumnya `groupPolicy = 'allowlist'` tanpa `allowFrom` → semua pesan grup di-drop diam-diam.

## 4. System Prompt — Verifikasi Sebelum Eksekusi

Bot dikonfigurasi untuk selalu menanyakan klarifikasi, menampilkan rencana, dan meminta konfirmasi YA/BATAL sebelum menjalankan perintah apapun.

### Cara Mengatur via Telegram
```
/system [isi prompt sistem di sini]
```

### Alur Kerja Bot
1. **TAHAP 1 — Gali Informasi**: Tanya klarifikasi satu per satu
2. **TAHAP 2 — Tampilkan Rencana**: Format `RENCANA EKSEKUSI` dengan tujuan, target, langkah, risiko
3. **TAHAP 3 — Tunggu Konfirmasi**: Tunggu jawaban YA atau BATAL
4. **TAHAP 4 — Laporan Hasil**: Format `HASIL EKSEKUSI` dengan status, waktu, output, detail

### Pengecualian (Tidak Perlu Konfirmasi)
Perintah read-only seperti: `ls`, `cat`, `ps`, `df`, `uptime`, cek status service, lihat log — langsung dieksekusi tanpa konfirmasi.

## 5. Proyek: Dashboard Monitor Harian

Script monitor otomatis yang mengirim laporan CPU, RAM, status service, dan website ke Telegram setiap pagi.

### Lokasi Script
`/root/scripts/monitor.sh`

### Yang Dipantau
- 💻 CPU & RAM dengan grafik ASCII bar
- 🔧 Status Service: nginx, pm2
- 🌐 Status Website/Domain dengan HTTP status code

### Jadwal Cron (Setiap Pagi Jam 07:00)
```bash
crontab -e
0 7 * * * /root/scripts/monitor.sh >> /root/scripts/monitor.log 2>&1
```

### Cara Mendapatkan Chat ID
```bash
curl "https://api.telegram.org/bot<TOKEN>/getUpdates" | grep -o '"id":[0-9]*' | head -1
```

### Contoh Output Laporan
```
🖥️ LAPORAN SERVER HARIAN
📅 Rabu, 11 Maret 2026 — 07:00 WIB
💻 CPU: [███░░░░░░░░░░░░░░░░░] 15%
🧠 RAM: [█████████░░░░░░░░░░░] 45%
✅ nginx  ✅ pm2
✅ https://domainmu.com [200]
```

## 6. Ringkasan Perubahan Konfigurasi

| Konfigurasi               | Sebelum               | Sesudah                          |
| :------------------------ | :-------------------- | :------------------------------- |
| Sandbox Mode              | tidak eksplisit       | mode: off (permanen) ✅          |
| Tools Profile             | messaging             | full ✅                          |
| Telegram groupPolicy      | allowlist (broken)    | open ✅                          |
| Gateway                   | perlu restart manual  | running ✅                       |
| System Prompt             | tidak ada             | verifikasi + konfirmasi ✅       |
| Monitor Harian            | tidak ada             | cron 07:00 setiap hari ✅        |
