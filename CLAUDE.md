# CLAUDE.md

Panduan untuk Claude Code saat bekerja di repo ini. Pemilik repo berbahasa Indonesia; balas dalam Bahasa Indonesia.

## Apa ini

Teman Ngoding = ekstensi GNOME Shell (maskot piksel di desktop; karakter Claude/Gemini/Codex/OpenCode/DeepSeek)
+ beberapa script pendukung untuk membuka CLI AI sebagai root dan menampilkan info akun/limit. Tidak ada proses build;
semua file disalin apa adanya ke lokasi sistem.

## Cara memasang (kalau user minta "install")

1. **Cek kebutuhan dulu**, laporkan yang kurang sebelum memasang:
   ```bash
   gnome-shell --version          # harus cocok dengan "shell-version" di extension/teman-ngoding@yandi/metadata.json
   which meson ninja google-chrome python3 sudo gnome-extensions
   sudo test -x /root/.local/bin/claude && echo ok   # Claude Code untuk root
   sudo test -x /root/.local/bin/agy && echo ok      # Antigravity CLI untuk karakter Gemini (opsional)
   which codex opencode dsh                          # CLI lain (opsional, npm global; dsh = @deepseek-ai/dsh)
   ```
   - Versi GNOME beda → tambahkan versinya ke `shell-version` di `metadata.json` (tanya user dulu).
   - CLI root yang belum ada dipasang otomatis oleh `install.sh` (versi terbaru) lewat `bin/teman-ngoding-install-cli`
     (`SKIP_CLIS=1` untuk melewati, `UPDATE_CLIS=1` untuk menaikkan semua). Setelahnya login lewat `bin/teman-ngoding-switch <id>`.
   - Terminal dibangun dari `terminal/` oleh `install.sh` (butuh meson/ninja + header GTK4, libadwaita,
     VTE-GTK4, json-glib; di apt dipasang otomatis). Gagal build → `bin/teman-ngoding-terminal` jatuh ke `ptyxis` bawaan.

2. **Jalankan installer**:
   ```bash
   ./install.sh                          # sebagai user desktop, butuh sudo
   sudo TARGET_USER=yandi ./install.sh   # kalau sesi ini root
   ```
   Installer juga membersihkan instalasi Claude Buddy lama. Kalau sudo tidak bisa interaktif
   di sesi ini, minta user menjalankannya sendiri dengan mengetik `! ./install.sh`.

3. **Verifikasi**:
   ```bash
   gnome-extensions info teman-ngoding@yandi      # State: ACTIVE
   systemctl status teman-ngoding-status.timer    # active (waiting)
   sudo systemctl start teman-ngoding-status.service && cat /run/teman-ngoding/status.json
   ```
   Di Wayland, ekstensi baru sering belum dikenali sampai **logout/login**. Kalau `gnome-extensions enable`
   gagal setelah install pertama, minta user logout lalu login lagi, lalu jalankan
   `gnome-extensions enable teman-ngoding@yandi`.

## Hal penting

- `install.sh` mengganti nama user `yandi` (di `teman-ngoding-status`, `open-in-user-chrome`, aturan polkit)
  dengan `$USER` saat dipasang. File di repo tetap memakai `yandi`; jangan ubah pola itu tanpa menyesuaikan `sed` di `install.sh`.
- UUID ekstensi `teman-ngoding@yandi` dipakai di `metadata.json`, `buddy.js`, `bin/teman-ngoding-launch`,
  `install.sh`, dan nama folder. Kalau diganti, ganti di semua tempat.
- `buddy.js` membaca `accounts.root` / `accounts.yandi` dari `status.json` (Claude), jadi untuk user lain
  akun non-root tidak tampil sebagai cadangan. Ini bukan masalah selama Claude Code root sudah login.
- Karakter didefinisikan di `CHARACTERS` (`buddy.js`). Id karakter (`claude`, `gemini`, `codex`, `opencode`, `deepseek`)
  juga dipakai di `case` pada `bin/teman-ngoding-{run,launch,switch,install-cli}`, di `AGENTS` pada `teman-ngoding-status`,
  di kelas CSS `ngoding-theme-<id>` / `ngoding-chip-<id>`, dan di pemilih tab baru terminal
  (`teman_ngoding_clis` di `ptyxis-window.c` + menu `new_cli_menu` di `ptyxis-window.ui`). Tambah karakter = ubah semua tempat itu.
- Sisa limit (dan maskot tidur saat limit habis) hanya untuk Claude; karakter lain hanya menampilkan info akun.
- Codex dibuka dengan `--approve-for-me` (mode Auto-review) di `teman-ngoding-run`, supaya tidak minta Enter tiap izin.
- **Jangan pernah commit** `.credentials.json`, `.claude.json`, `oauth_creds.json`, `auth.json`, `.env`, atau
  `status.json`. Token OAuth / API key hanya boleh dibaca oleh `teman-ngoding-status` (berjalan sebagai root) dan tidak boleh ditulis ke output.

## Mengembangkan

Setelah mengubah file ekstensi, salin lalu muat ulang (tidak perlu logout, karena `extension.js`
meng-import salinan `buddy.js` dengan nama unik):
```bash
cp extension/teman-ngoding@yandi/* ~/.local/share/gnome-shell/extensions/teman-ngoding@yandi/
gnome-extensions disable teman-ngoding@yandi && gnome-extensions enable teman-ngoding@yandi
```
Log error: `journalctl --user -f -o cat /usr/bin/gnome-shell | grep -i ngoding`.
Perubahan di `metadata.json` atau `extension.js` tetap butuh logout/login.

Script di `bin/` / `lib/` / `systemd/` / `polkit/` → pasang ulang dengan `./install.sh`.

## Terminal (`terminal/`)

Ptyxis 50.1 dari https://gitlab.gnome.org/chergert/ptyxis, divendor apa adanya lalu diubah di satu commit
terpisah. Dibangun dengan app-id/gschema `io.github.yandyrohman.TemanNgoding.Terminal` ke
`/usr/local/lib/teman-ngoding/terminal` (bukan /usr/local/bin, supaya `ptyxis` bawaan tidak tertimpa).
`bin/teman-ngoding-terminal` mengatur `GSETTINGS_SCHEMA_DIR`/`XDG_DATA_DIRS` lalu menjalankannya.

- Tab baru = pilih CLI: tombol "+" (`new_terminal_button`) kini `GtkMenuButton` dengan menu `new_cli_menu`;
  `win.new-cli-tab(s)` membuat tab yang menjalankan `/usr/local/bin/teman-ngoding-run <id>` dengan judul
  = nama CLI (`title-prefix` + `ignore-osc-title`). Id kosong = Terminal Biasa (`win.new-terminal`).
  Shortcut new-tab & ☰ → Tab Baru memanggil `win.new-tab-picker` (membuka menu itu).
- Tema per tab: `palette-override` dan `style-override` (default/force-light/force-dark) di `PtyxisTab`.
  Aksi `win.session-palette(-dialog)` dan `win.interface-style` di `PtyxisWindow` bekerja pada tab aktif;
  state-nya disinkronkan saat pindah tab (`ptyxis_window_sync_tab_theme`). Terminal memakai
  `ptyxis_tab_get_dark()`, dressing/bingkai memakai `ptyxis_window_get_dark()` (= tab aktif). Binding palette
  profil di `ptyxis-tab.ui` sengaja dihapus; palette terminal diatur `ptyxis_tab_update_palette()`.
- Judul tab: klik dua kali tab → `win.set-title` (dialog `ptyxis-title-dialog`, sudah diterjemahkan).
- Palette di Preferences memakai aksi `session.palette` → tab aktif jendela `transient-for` Preferences
  (di-set ulang tiap kali Preferences dibuka), bukan profil default.
- `restore-session` default false (tiap klik = jendela bersih).
- Tema: hanya 21 palette bawaan di `terminal/src/palettes/` (didaftarkan di `ptyxis.gresource.xml`), semuanya
  `Primary=true`. Urutan daftar dari key `Order=` (bukan abjad; palette user tanpa `Order` di akhir).
  Default `ubuntu` (gschema + fallback di `ptyxis-profile.c`/`ptyxis-terminal.c`). Tema baru harus beda
  jelas warna latarnya dari yang sudah ada.
- `install.sh` hanya rebuild kalau hash isi `terminal/` berubah (`.source-hash` di prefix).
- Upgrade upstream: ganti isi `terminal/` dengan tag baru, lalu terapkan ulang `git diff ce6c5a7 -- terminal/`.
- Tes UI tanpa memasang: build ke prefix sementara (`meson configure --prefix=…` lalu `meson install`), jalankan
  dengan `GSETTINGS_SCHEMA_DIR=<prefix>/share/glib-2.0/schemas`.

## Copot pasang

```bash
gnome-extensions disable teman-ngoding@yandi
rm -rf ~/.local/share/gnome-shell/extensions/teman-ngoding@yandi \
       ~/.local/share/applications/io.github.yandyrohman.TemanNgoding.Terminal.desktop \
       ~/.local/share/icons/hicolor/scalable/apps/teman-ngoding.svg \
       ~/.config/teman-ngoding.json
sudo systemctl disable --now teman-ngoding-status.timer
sudo rm -rf /usr/local/bin/teman-ngoding-{launch,status,switch,run,terminal,install-cli} \
            /usr/local/lib/teman-ngoding /run/teman-ngoding \
            /etc/systemd/system/teman-ngoding-status.{service,timer} \
            /etc/polkit-1/rules.d/49-teman-ngoding-status.rules
sudo systemctl daemon-reload
```
Konfirmasi ke user sebelum menjalankan ini.
