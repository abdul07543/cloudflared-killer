# 🔪 The Killer — Mobile Hosting Server

Turn an Android phone into a real, publicly reachable web server using **Termux + PHP + Cloudflare Tunnel** — no router port-forwarding, no static IP, no paid hosting.

Renamed from the original "Skyhostr / sky" project — same underlying mechanism (PHP + `cloudflared`), new branding, redesigned dashboard, a couple of small quality-of-life features, and the security gaps closed.                                     
## What's different from the original

| Old | New |
|---|---|                                                     | `sky` command | `killer` |
| `stop` command | `killer-stop` |                            | Plain HTML bio card | Dark-themed live status dashboard (uptime, visit counter, QR code) |
| Real tunnel ID committed in `config.yml` | `config.yml.example` placeholder only — real config stays local |
| `tinyfilemanager.php` with default demo password | `filemanager.php` — same tool, **you must set your own password before going live** |

## Features

- ✅ PHP web server on your phone
- ✅ Cloudflare Tunnel (no port forwarding, works on WiFi or mobile data)
- ✅ Custom domain + HTTPS
- ✅ One command start (`killer`), one command stop (`killer-stop`)
- ✅ Auto-reconnecting tunnel (survives WiFi ↔ mobile data switches)
- ✅ Live dashboard: server clock, PHP version, visit counter, QR code to share the link
- ✅ File manager for editing your site from the browser
- ✅ Backup & restore for moving to a new phone

## Requirements

- Android phone
- [Termux](https://f-droid.org/packages/com.termux/) (install from F-Droid, **not** Play Store — the Play Store build is outdated)
- A Cloudflare account (free) + a domain added to it

---

## 1. Install Termux packages

```bash
pkg update -y && pkg upgrade -y
pkg install php git nano curl wget cloudflared zip unzip -y
```

## 2. Get the project onto your phone

If you're setting this up from this chat, transfer the `killer-mobile-server` folder to your phone (e.g. via Google Drive, then `termux-setup-storage` + `cp`), or push it to your **own private** Git repo and `git clone` it.

```bash
cd ~
# either clone your own repo:
git clone YOUR_PRIVATE_REPO_URL killer-mobile-server
cd killer-mobile-server
```

## 3. Run the installer

```bash
chmod +x install.sh
./install.sh
```

This installs packages, sets executable permissions on `killer` / `killer-stop`, and creates `~/public_html`.

## 4. Cloudflare login & tunnel

```bash
cloudflared tunnel login
cloudflared tunnel create killer
```

Copy the Tunnel ID it prints out.

## 5. Configure the tunnel

```bash
mkdir -p ~/.cloudflared
cp config.yml.example ~/.cloudflared/config.yml
nano ~/.cloudflared/config.yml
```

Replace `YOUR-TUNNEL-ID` and `yourdomain.com` with your real values. Save with `CTRL+O`, `Enter`, `CTRL+X`.

## 6. Connect your domain

```bash
cloudflared tunnel route dns killer yourdomain.com
cloudflared tunnel route dns killer www.yourdomain.com
```

## 7. Secure the file manager — do this before starting the server

Open `~/public_html/filemanager.php`, find:

```php
$auth_users = array(
    'admin' => '$2y$10$/K.hjNr84lLNDt8fTXjoI.DBp6PpeyoJ.mGwrrLuCZfAwfSAGqhOW', //admin@123
    'user'  => '$2y$10$Fg6Dz8oH9fPoZ2jJan5tZuv6Z4Kp7avtQ9bDfrdRntXtPeiMAZyGO' //12345
);
```

These are the **stock TinyFileManager demo credentials** — publicly known. Generate your own password hash:

```bash
php -r "echo password_hash('YOUR-NEW-PASSWORD', PASSWORD_DEFAULT);"
```

Paste the output in place of the existing hash, and remove the `user` account if you don't need it. Skipping this step means anyone who finds `/filemanager.php` on your domain can log in with the default password and read/write every file on your phone.

## 8. Start the server

```bash
./killer
```

This starts PHP, starts the tunnel, and auto-reconnects the tunnel if it drops. Your site is now live at:

```
https://yourdomain.com
```

## 9. Stop the server

```bash
./killer-stop
```

## File manager access

```
https://yourdomain.com/filemanager.php
```

## Backup

```bash
cd ~
zip -r killer-backup.zip killer-mobile-server .cloudflared
```

Move `killer-backup.zip` to Google Drive or similar. **Do not** put it in a public repo — it contains your tunnel credentials.

## Restore on a new phone

```bash
termux-setup-storage
cp /sdcard/killer-backup.zip ~/
unzip killer-backup.zip
chmod +x ~/killer-mobile-server/killer ~/killer-mobile-server/killer-stop
cd ~/killer-mobile-server
./killer
```

## Troubleshooting

| Problem | Fix |
|---|---|
| Domain not resolving | `cloudflared tunnel list` — confirm tunnel is active |
| Tunnel keeps dropping | Check `~/.killer/killer.log` |
| PHP errors | Check `~/.killer/php.log` |
| Phone sleeps and kills Termux | Disable battery optimization for Termux in Android settings |

## Important notes

- Closing Termux stops the server — this is a phone acting as a server, not a managed host.
- Never commit `config.yml`, `.cloudflared/`, or backup zips to a public GitHub repo — they contain your tunnel credentials and real domain. The included `.gitignore` blocks this by default.
- Change the file manager password before pointing a real domain at this.

---

**The Killer** — Mobile Hosting Server (PHP + Cloudflare Tunnel)
