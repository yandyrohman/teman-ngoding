# Teman Ngoding 🤝

Maskot piksel yang hidup di desktop GNOME. Dia mengambang, berkedip, ngobrol, main,
dan bisa diklik untuk membuka CLI AI (sebagai root) di terminal. Karakternya bisa
diganti lewat tombol kecil **⇄** di kartu menu:

| Karakter | Tampilan | Membuka |
|---|---|---|
| **Claude** (default) | Clawd oranye bermahkota 👑 | Claude Code |
| **Gemini** | Gradasi biru-ungu dengan bintang ✦ | Antigravity CLI (`agy`) |
| **Codex** | Putih dengan layar terminal `>_` di kepala | Codex CLI |
| **OpenCode** | Abu gelap dengan kurung kurawal `{ }` | OpenCode |
| **DeepSeek** | Paus biru dengan semburan air 🐋 | DeepSeek Harness (`dsh web`, Web UI di Chrome) |

Semua karakter memakai kerangka yang sama, jadi semua animasi, adegan, dan interaksi
berlaku untuk semuanya. Pilihan karakter disimpan di `~/.config/teman-ngoding.json`.
Kartu "Akun Saya" menampilkan akun CLI karakter aktif; sisa limit hanya tersedia untuk Claude Code.

## Terminal sendiri: pilih CLI per tab, tema & judul per tab

CLI dibuka di terminal milik Teman Ngoding sendiri (namanya juga "Teman Ngoding"), yaitu Ptyxis 50.1 (folder `terminal/`) yang dibangun
dengan app-id dan setting sendiri (`io.github.yandyrohman.TemanNgoding.Terminal`). Ptyxis bawaan
sistem tidak diubah sama sekali.

- **Tab baru = pilih CLI dulu.** Tombol **+** di kiri atas (atau `Ctrl+Shift+T`, atau ☰ → **Tab Baru…**)
  membuka pilihan: Claude Code, Antigravity CLI, Codex CLI, OpenCode, DeepSeek Harness, atau
  **Terminal Biasa** (shell user). Panah ▾ di sebelahnya tetap berisi profil/container bawaan Ptyxis.
- **Tema per tab.** ☰ → **Tema Tab Ini…** (atau palette di Preferences) dan tombol Sistem/Terang/Gelap
  di atas menu ☰ hanya mengubah tab yang sedang aktif. Tiap tab menyimpan temanya sendiri (ikut terbawa
  kalau tab dipindah ke jendela lain); bingkai jendela mengikuti tab yang aktif. Tab baru mulai dari tema
  default; **Ikuti Default** mengembalikan tab ke palette profil. Tema tidak disimpan setelah tab ditutup.
- **Judul tab bisa diganti.** Defaultnya nama CLI-nya (mis. "Claude Code"; judul yang dikirim CLI
  diabaikan). Ganti lewat **klik dua kali** pada tab, klik kanan tab → **Ganti Judul…**, ☰ →
  **Ganti Judul Tab…**, atau shortcut Set Title. Judul dikosongkan = kembali memakai judul dari program.

Perubahan terhadap upstream: `git diff ce6c5a7 -- terminal/` (commit vendor Ptyxis 50.1 yang masih asli).

## Isi repo

| Folder | Isi | Dipasang ke |
|---|---|---|
| `extension/teman-ngoding@yandi/` | Ekstensi GNOME Shell (`buddy.js` = maskot & karakter, `extension.js` = pemuat) | `~/.local/share/gnome-shell/extensions/` |
| `bin/teman-ngoding-launch` | Dipanggil ikon aplikasi: nyalakan maskot / buka CLI karakter aktif | `/usr/local/bin/` |
| `bin/teman-ngoding-run` | Jalankan `claude` / `agy` / `codex` / `opencode` / `dsh` sebagai root | `/usr/local/bin/` |
| `bin/teman-ngoding-terminal` | Buka terminal Teman Ngoding (fallback ke `ptyxis` bawaan) | `/usr/local/bin/` |
| `bin/teman-ngoding-install-cli` | Pasang semua CLI untuk root, versi terbaru (`--update` = naikkan versi, `--check` = cek saja) | `/usr/local/bin/` |
| `terminal/` | Source Ptyxis 50.1 + pemilih CLI di tab baru, tema & judul per tab; dibangun oleh `install.sh` | `/usr/local/lib/teman-ngoding/terminal/` |
| `bin/teman-ngoding-switch` | Ganti akun CLI (login via Chrome user, atau API key) | `/usr/local/bin/` |
| `bin/teman-ngoding-status` | Ambil info akun + limit → `/run/teman-ngoding/status.json` | `/usr/local/bin/` |
| `lib/open-in-user-chrome` | Buka URL login di Chrome milik user (bukan root) | `/usr/local/lib/teman-ngoding/` |
| `systemd/` | Service + timer status (tiap 3 menit) | `/etc/systemd/system/` |
| `polkit/` | Izinkan user menjalankan service status tanpa password | `/etc/polkit-1/rules.d/` |
| `desktop/`, `icons/` | Launcher aplikasi (klik kanan: pilih CLI) + ikon; nama file = app-id terminal supaya bisa di-pin di dock | `~/.local/share/` |

## Kebutuhan

- GNOME Shell 50 (lihat `metadata.json`)
- `google-chrome`, `python3`, `sudo`
- Untuk membangun terminal: `meson`, `ninja` dan header GTK4/libadwaita/VTE-GTK4/json-glib
  (di Ubuntu/Debian `install.sh` memasangnya sendiri lewat `apt-get`). Kalau build gagal,
  Teman Ngoding memakai `ptyxis` bawaan.
- CLI AI untuk root: **dipasang otomatis oleh `install.sh` (versi terbaru)** lewat
  `teman-ngoding-install-cli`, butuh `curl` dan `npm` (di apt dipasang sendiri). Yang sudah ada dilewati.
  CLI yang belum terpasang juga ditawarkan untuk dipasang saat pertama dibuka. Rinciannya:
  - [Claude Code](https://claude.com/claude-code) di `/root/.local/bin/claude`
    (`curl -fsSL https://claude.ai/install.sh | bash -s latest`)
  - Antigravity CLI (karakter Gemini) di `/root/.local/bin/agy`:
    `sudo bash -c 'curl -fsSL https://antigravity.google/cli/install.sh | bash'`
  - Codex CLI: `sudo npm install -g @openai/codex`
  - OpenCode: `sudo npm install -g opencode-ai`
  - DeepSeek Harness: `sudo npm install -g @deepseek-ai/dsh@latest`

## Pasang

```bash
./install.sh                       # sebagai user desktop (pakai sudo)
sudo TARGET_USER=yandi ./install.sh   # atau langsung sebagai root
UPDATE_CLIS=1 ./install.sh         # sekalian naikkan semua CLI ke versi terbaru
SKIP_CLIS=1 ./install.sh           # jangan pasang CLI apa pun
sudo teman-ngoding-install-cli --update   # nanti-nanti: update semua CLI saja
```

Script akan membersihkan Claude Buddy versi lama (kalau ada, posisi maskot dibawa),
menyalin file user, memasang bagian sistem, menyesuaikan nama user di script
status/polkit, lalu mengaktifkan ekstensi. Karena UUID ekstensi baru, di Wayland
biasanya perlu **logout/login sekali** sebelum maskot muncul.

## Login

Buka kartu **Akun Saya → Ganti Akun** pada karakter yang aktif:

- **Claude**: `claude auth login`
- **Gemini**: memakai Antigravity CLI (`agy`). Login akun Google, atau API key dari
  [AI Studio](https://aistudio.google.com/apikey) (disimpan di `/root/.gemini/.env`, plus
  `"modelProvider": "gemini"` di `/root/.gemini/antigravity-cli/settings.json`).
  Gemini CLI lama sudah ditolak Google untuk akun pribadi ("This client is no longer supported…").
- **Codex**: akun ChatGPT (`codex login`) atau OpenAI API key
- **OpenCode**: `opencode auth login` (pilih provider)
- **DeepSeek**: API key dari [platform.deepseek.com](https://platform.deepseek.com/api_keys), disimpan sebagai
  `DEEPSEEK_API_KEY` di `/root/.deepseek/.env`. DeepSeek Harness tidak punya TUI: klik maskot menyalakan
  `dsh web` di terminal (jangan ditutup selama dipakai) dan membuka Web UI `http://127.0.0.1:3080` di Chrome.
  Key yang disimpan lewat **Settings → Models** di Web UI lebih diutamakan daripada `.env`.

## Mengembangkan

Setelah mengubah `buddy.js`, salin ke folder ekstensi lalu muat ulang tanpa logout:

```bash
cp extension/teman-ngoding@yandi/* ~/.local/share/gnome-shell/extensions/teman-ngoding@yandi/
gnome-extensions disable teman-ngoding@yandi && gnome-extensions enable teman-ngoding@yandi
```

`extension.js` menyalin `buddy.js` ke nama unik di `$XDG_RUNTIME_DIR` sebelum di-import,
jadi versi baru langsung terpakai.

Karakter baru cukup ditambahkan ke `CHARACTERS` di `buddy.js` (warna badan, hiasan
kepala, kalimat khas), lalu ke `case` di `bin/teman-ngoding-run`,
`bin/teman-ngoding-launch`, dan `bin/teman-ngoding-switch`.

## Catatan keamanan

`teman-ngoding-status` membaca token OAuth / API key CLI untuk mengecek akun & limit,
tapi token **tidak** ditulis ke file output. Jangan pernah commit `.credentials.json`,
`.claude.json`, `oauth_creds.json`, `auth.json`, atau `.env` ke repo ini.
