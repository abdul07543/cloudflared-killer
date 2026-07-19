# 🔪 The Killer — Mobile Hosting Server

Turn an Android phone into a real, publicly reachable web server using **Termux + PHP + Cloudflare Tunnel** — no router port-forwarding, no static IP, no paid hosting.

Renamed from the original "Skyhostr / sky" project — same underlying mechanism (PHP + `cloudflared`), new branding, redesigned dashboard, a couple of small quality-of-life features, and improved security.

---

## What's different from the original

| Aspect | Old | New |
|---|---|---|
| **Command** | `sky` command | `killer` |
| **Stop Command** | `stop` command | `killer-stop` |
| **Dashboard** | Plain HTML bio card | Dark-themed live status dashboard (uptime, visit counter, QR code) |
| **Config** | Real tunnel ID committed in `config.yml` | `config.yml.example` placeholder only — real config stays local |
| **File Manager** | `tinyfilemanager.php` with default demo password | `filemanager.php` — same tool, **you must set your own password before going live** |

---

## Features

- ✅ PHP web server on your phone
- ✅ Cloudflare Tunnel (no port forwarding, works on WiFi or mobile data)
- ✅ Custom domain + HTTPS
- ✅ One command start (`killer`), one command stop (`killer-stop`)
- ✅ Auto-reconnecting tunnel (survives WiFi ↔ mobile data switches)
- ✅ Live dashboard: server clock, PHP version, visit counter, QR code to share the link
- ✅ File manager for editing your site from the browser
- ✅ Backup & restore for moving to a new phone

---

## Requirements

- **Android phone** (Android 7.0+)
- **[Termux](https://f-droid.org/packages/com.termux/)** - Install from **F-Droid ONLY**, not Play Store (Play Store build is outdated)
- **Cloudflare account** (free tier works) - Sign up at [https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)
- **A domain** added to your Cloudflare account - [Learn how](https://developers.cloudflare.com/fundamentals/setup/manage-domains/add-site/)
- **PHP 7.4+** and **Git**

---

## Installation Guide

### Step 1: Install Termux packages

Open Termux and run:

```bash
pkg update -y && pkg upgrade -y
pkg install php git nano curl wget cloudflared zip unzip -y
```

### Step 2: Get the project onto your phone

**Option A: Clone from your private repo**
```bash
cd ~
git clone https://github.com/abdul07543/cloudflared-killer.git
cd killer-mobile-server
```

**Option B: Download and extract manually**
- Download `killer-mobile-server.zip` from this repo
- Transfer to phone via Google Drive, USB, or cloud storage
- Extract in Termux:
```bash
unzip killer-mobile-server.zip
cd killer-mobile-server
```

### Step 3: Run the installer

```bash
chmod +x install.sh
./install.sh
```

This script:
- ✅ Installs all required packages
- ✅ Sets executable permissions on `killer` and `killer-stop`
- ✅ Creates the `~/public_html` directory for your website

### Step 4: Cloudflare login & create tunnel

```bash
cloudflared tunnel login
```
This opens a browser to authenticate. After login, return to Termux.

```bash
cloudflared tunnel create killer
```

**Copy the Tunnel ID** that appears in the output (looks like: `abcd1234-ef56-7890-ghij-klmnopqrstuv`)

**Reference:** [Cloudflare Tunnel Documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/)

### Step 5: Configure the tunnel

```bash
mkdir -p ~/.cloudflared
cp config.yml.example ~/.cloudflared/config.yml
nano ~/.cloudflared/config.yml
```

Edit the file and replace:
- `YOUR-TUNNEL-ID` → Your Tunnel ID from Step 4
- `yourdomain.com` → Your actual domain name

**Save:** Press `CTRL+O`, then `Enter`, then `CTRL+X`

Example config:
```yaml
tunnel: abcd1234-ef56-7890-ghij-klmnopqrstuv
credentials-file: /root/.cloudflared/abcd1234-ef56-7890-ghij-klmnopqrstuv.json

ingress:
  - hostname: yourdomain.com
    service: http://localhost:8000
  - hostname: www.yourdomain.com
    service: http://localhost:8000
  - service: http_status:404
```

### Step 6: Connect your domain to Cloudflare

```bash
cloudflared tunnel route dns killer yourdomain.com
cloudflared tunnel route dns killer www.yourdomain.com
```

This creates DNS records in Cloudflare pointing to your tunnel.

**Verify:** Log in to [https://dash.cloudflare.com](https://dash.cloudflare.com), select your domain, go to DNS Records, and confirm entries exist for `yourdomain.com` and `www.yourdomain.com`.

### Step 7: Secure the file manager — ⚠️ DO THIS BEFORE GOING LIVE

Open `~/public_html/filemanager.php` in a text editor:

```bash
nano ~/public_html/filemanager.php
```

Find this section (around line 50-60):

```php
$auth_users = array(
    'admin' => '$2y$10$/K.hjNr84lLNDt8fTXjoI.DBp6PpeyoJ.mGwrrLuCZfAwfSAGqhOW', //admin@123
    'user'  => '$2y$10$Fg6Dz8oH9fPoZ2jJan5tZuv6Z4Kp7avtQ9bDfrdRntXtPeiMAZyGO' //12345
);
```

**⚠️ WARNING:** These are stock TinyFileManager demo credentials — **PUBLICLY KNOWN**. Generate a new password hash:

```bash
php -r "echo password_hash('YOUR-NEW-PASSWORD', PASSWORD_DEFAULT);"
```

Replace the admin hash with your new one:

```php
$auth_users = array(
    'admin' => 'YOUR-NEW-HASH-HERE',
);
```

Remove the `'user'` line if you don't need it. Save with `CTRL+O`, `Enter`, `CTRL+X`.

### Step 8: Start the server

```bash
./killer
```

This:
1. Starts the PHP built-in web server on `localhost:8000`
2. Starts the Cloudflare tunnel
3. Monitors and auto-reconnects if the tunnel drops

**Your site is now LIVE at:** `https://yourdomain.com`

### Step 9: Stop the server

```bash
./killer-stop
```

---

## Usage

### Access your website

```
https://yourdomain.com
```

### Access the file manager

```
https://yourdomain.com/filemanager.php
```

Login with credentials you set in **Step 7**.

**File Manager Features:**
- 📁 Upload/download files
- ✏️ Edit files in the browser
- 📄 Preview documents
- 🗑️ Delete/rename files
- 📦 Create ZIP archives

### Monitor logs

```bash
# View tunnel logs
tail -f ~/.killer/killer.log

# View PHP error logs
tail -f ~/.killer/php.log
```

### Backup your server

Before switching phones or making major changes, backup everything:

```bash
cd ~
zip -r killer-backup.zip killer-mobile-server .cloudflared
```

Move the backup file to Google Drive, Dropbox, or cloud storage. **Do NOT commit it to a public repo** — it contains your tunnel credentials.

**Download link pattern:** Your sync service URL for `killer-backup.zip`

### Restore on a new phone

1. Install Termux on the new phone
2. Transfer `killer-backup.zip` to the new phone
3. In Termux:

```bash
termux-setup-storage
cp /sdcard/Download/killer-backup.zip ~/
unzip killer-backup.zip
chmod +x ~/killer-mobile-server/killer ~/killer-mobile-server/killer-stop
cd ~/killer-mobile-server
./killer
```

Your server is running on the new phone with the same domain and tunnel.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **Domain not resolving** | 1. Run `cloudflared tunnel list` to confirm tunnel is active<br>2. Check [https://dash.cloudflare.com](https://dash.cloudflare.com) → DNS Records → verify entries exist<br>3. Wait 5-10 minutes for DNS propagation |
| **Tunnel keeps dropping** | 1. Check logs: `tail -f ~/.killer/killer.log`<br>2. Disable battery optimization for Termux: Settings → Battery → Battery optimization → find Termux → disable<br>3. Keep phone screen on or increase inactivity timeout |
| **PHP errors** | Check logs: `tail -f ~/.killer/php.log` and fix the PHP code |
| **File manager won't load** | 1. Check if server is running: `./killer` or `./killer-stop && ./killer`<br>2. Verify domain is accessible: try main domain first<br>3. Check browser cache: hard refresh (Ctrl+Shift+R) |
| **Can't login to file manager** | Verify password hash was set correctly in `filemanager.php` |
| **Tunnel credentials lost** | Run `cloudflared tunnel login` and `cloudflared tunnel create killer` again (will change Tunnel ID) |
| **Phone sleeps and kills Termux** | Android Settings → Battery → Battery optimization → find Termux → Disable |

---

## Important Security Notes

⚠️ **CRITICAL:**

1. **Change file manager password BEFORE going live** — Default credentials are publicly known
2. **Never commit `config.yml` to a public repo** — Contains your tunnel credentials
3. **Never commit `.cloudflared/` directory** — Contains tunnel authentication files
4. **Never commit backup zips** — They contain all your secrets
5. The included `.gitignore` blocks all of the above by default
6. Use HTTPS only (all Cloudflare Tunnel connections are automatically HTTPS)
7. Disable public listing in your web server for sensitive directories

---

## Project Files

| File | Purpose |
|------|---------|
| `killer` | Start script for the web server |
| `killer-stop` | Stop script to cleanly shut down |
| `install.sh` | Installation script |
| `config.yml.example` | Template for Cloudflare tunnel config |
| `public_html/` | Your website files go here |
| `public_html/filemanager.php` | Browser-based file manager (password-protected) |
| `.gitignore` | Prevents committing sensitive files |

---

## Useful Links

- **Termux F-Droid:** https://f-droid.org/packages/com.termux/
- **Cloudflare Dashboard:** https://dash.cloudflare.com
- **Cloudflare Tunnels Docs:** https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/
- **PHP Built-in Server:** https://www.php.net/manual/en/features.commandline.webserver.php
- **Cloudflared CLI Docs:** https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/
- **TinyFileManager Docs:** https://github.com/prasathmani/tinyfilemanager
- **Android Battery Optimization Fix:** https://developer.android.com/training/monitoring-device-state/doze-standby

---

## License

This project is based on the original **Skyhostr** project. See `LICENSE` file for details.

---

## Support & Issues

- 🐛 Found a bug? Report it: [Create an Issue](https://github.com/abdul07543/cloudflared-killer/issues)
- 💡 Have a feature request? [Create an Issue](https://github.com/abdul07543/cloudflared-killer/issues)
- 📖 Need help? Check [Discussions](https://github.com/abdul07543/cloudflared-killer/discussions)
- 🔗 Contribute: [Fork & create a PR](https://github.com/abdul07543/cloudflared-killer/pulls)

---

## Repository Links

- **Main Repo:** https://github.com/abdul07543/cloudflared-killer
- **Issues:** https://github.com/abdul07543/cloudflared-killer/issues
- **Pull Requests:** https://github.com/abdul07543/cloudflared-killer/pulls
- **Discussions:** https://github.com/abdul07543/cloudflared-killer/discussions
- **Project Board:** https://github.com/abdul07543/cloudflared-killer/projects
- **Wiki:** https://github.com/abdul07543/cloudflared-killer/wiki
- **Security Policy:** https://github.com/abdul07543/cloudflared-killer/security
- **Actions/Workflows:** https://github.com/abdul07543/cloudflared-killer/actions

---

**The Killer** — Mobile Hosting Server (PHP + Cloudflare Tunnel)  
*Turn your Android phone into a production-ready web server*
