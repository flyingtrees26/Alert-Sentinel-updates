[README.md](https://github.com/user-attachments/files/34198800/README.md)
# Alert Sentinel

A desktop emergency-alert console for macOS. It watches the official alert feed for
your area and, when a warning comes in, sounds a siren, reads the alert aloud, takes
over the screen, and can push the alert to Google Home / Nest speakers, Chromecast
TVs, and phones on your Wi-Fi.

**Countries:** United States (National Weather Service), Canada (Environment Canada),
United Kingdom (Met Office), Australia (Bureau of Meteorology), and 37 European
countries (MeteoAlarm).

> Alert Sentinel is a personal tool, not an official alerting system. It can miss or
> delay alerts. Always keep Wireless Emergency Alerts on your phone, NOAA Weather
> Radio, and local emergency guidance as your main sources.

---

## Install

### Option 1: double-click (easiest)

1. Unzip **Alert-Sentinel.zip** and move the **Alert-Sentinel** folder to your
   **home folder** (Finder → ⌘⇧H). Keeping it out of Downloads, Desktop, and
   Documents avoids problems with macOS privacy settings and antivirus tools like
   Bitdefender Safe Files.
2. Right-click **Install Alert Sentinel.command** → **Open** → **Open**.
   *(Right-click is only needed the first time; macOS blocks a normal double-click
   on downloaded scripts.)*
3. Wait for it to finish. The first run downloads a few hundred MB. It builds
   **Alert Sentinel.app** in your personal Applications folder and opens it.

You can run **Install Alert Sentinel.command** with a double-click to do everything at
once. **If it asks for a password you don't have, run the commands below separately
in Terminal instead.** None of them need an administrator password.

### Option 2: run the commands yourself

Open **Terminal** and paste these one block at a time. They assume the folder is at
`~/Alert-Sentinel`; change the first line if yours is somewhere else.

**1. Go to the app folder**
```bash
cd ~/Alert-Sentinel/alert-sentinel-electron
```

**2. Get Node.js** *(skip if `node -v` already prints a version)*. This downloads a
private copy into this folder, checks that it's genuine, and needs no password:
```bash
ARCH=$( [ "$(uname -m)" = arm64 ] && echo arm64 || echo x64 )
BASE=https://nodejs.org/dist/latest-v22.x
LINE=$(curl -fsSL $BASE/SHASUMS256.txt | grep "darwin-$ARCH.tar.gz")
FILE=$(echo "$LINE" | awk '{print $2}'); SUM=$(echo "$LINE" | awk '{print $1}')
curl -fL $BASE/$FILE -o /tmp/$FILE
[ "$(shasum -a 256 /tmp/$FILE | awk '{print $1}')" = "$SUM" ] && echo "Checksum OK" || echo "CHECKSUM FAILED - stop here"
mkdir -p .node && tar -xzf /tmp/$FILE -C .node --strip-components 1
```

**3. Use that Node in this Terminal window**
```bash
export PATH="$PWD/.node/bin:$PATH"
node -v
```

**4. Install the app's packages**
```bash
npm install
```

**5. Try it**
```bash
npm start
```
The Alert Sentinel window should open. If it does, quit it (⌘Q) and continue to make
the real app with its icon.

**6. Build Alert Sentinel.app**
```bash
mkdir -p ~/Applications
APP=~/Applications/"Alert Sentinel.app"
rm -rf "$APP"
ditto node_modules/electron/dist/Electron.app "$APP"
mkdir -p "$APP/Contents/Resources/app"
echo '{ "name": "alert-sentinel", "productName": "Alert Sentinel", "version": "1.0.0", "main": "start.js" }' > "$APP/Contents/Resources/app/package.json"
echo "require('$PWD/main.js');" > "$APP/Contents/Resources/app/start.js"
plutil -replace CFBundleName -string "Alert Sentinel" "$APP/Contents/Info.plist"
plutil -replace CFBundleDisplayName -string "Alert Sentinel" "$APP/Contents/Info.plist"
plutil -replace CFBundleIdentifier -string "com.flyingtrees26.alertsentinel" "$APP/Contents/Info.plist"
```

**7. Give it its icon**
```bash
mkdir -p /tmp/icon.iconset
for s in 16 32 128 256 512; do
  sips -z $s $s build/icon.png --out /tmp/icon.iconset/icon_${s}x${s}.png >/dev/null
  sips -z $((s*2)) $((s*2)) build/icon.png --out /tmp/icon.iconset/icon_${s}x${s}@2x.png >/dev/null
done
iconutil -c icns /tmp/icon.iconset -o "$APP/Contents/Resources/electron.icns"
rm -rf /tmp/icon.iconset
```

**8. Seal it and open it**
```bash
xattr -cr "$APP"
codesign --force --deep --sign - "$APP"
open "$APP"
```

From now on, open it with **Spotlight** (⌘Space → *Alert Sentinel*), or right-click
its Dock icon → **Options → Keep in Dock**.

### First launch

macOS will ask a few things. Click **Allow** on each:
- **Incoming network connections.** Phones and Google speakers download alerts from this Mac.
- **Notifications.**
- **Access to Downloads,** the first time you use **Install Update**.

---

## Using it

1. Pick your **Country**, then enter your location: a ZIP code (US), a city (Canada),
   a region or state from the list (UK, Australia), or an optional region name (Europe).
2. Click **Connect**. Your location is remembered, and the app reconnects by itself
   every time it starts.
3. Under **Send Test Alert**, try a test to hear and see what a real alert does.

Settings worth knowing (under Send Test Alert):
- **Alert length:** 4 to 30 repeats.
- **Take over screen during alerts (kiosk mode).**
- **Keep Mac awake for alerts.** The Mac won't sleep while the app is open (the
  screen still can), so alerts come in overnight. The screen wakes when one arrives.

**Google Home / Nest / Chromecast:** found automatically on your Wi-Fi. Check the ones
that should sound alerts. TVs and Nest Hubs also show the alert on screen. **Turn On**
wakes a TV through its Chromecast (turn on HDMI-CEC / Anynet+ on the TV).

**Phones and other computers:** open the address shown under **Network Access** on any
device on the same Wi-Fi to get alerts there too.

---

## Updating

1. In the app, click **Get updates**. It opens the
   [releases page](https://github.com/flyingtrees26/alert-sentinel-updates/releases).
2. Under the newest release's **Assets**, download the `Alert-Sentinel-update-X.Y.Z.zip`.
   Leave it in Downloads; don't unzip it.
   *(Don't use the green "Code → Download ZIP" button. That isn't an update.)*
3. In the app, click **⬇ Install Update** → **Install & Restart**.

The app backs up the files it replaces to `alert-sentinel-electron/.update-backups/`.
If the button says it isn't allowed to look in Downloads, use **Choose update file…**
and pick the zip, or allow Alert Sentinel under System Settings → Privacy & Security →
Files & Folders.

To update by hand instead, unzip the update and copy its files into
`alert-sentinel-electron` at the same paths, then quit and reopen the app.

---

## Other ways to run it

**From Terminal** (no app needed):
```bash
cd ~/Alert-Sentinel/alert-sentinel-electron
export PATH="$PWD/.node/bin:$PATH"
npm start
```

**The browser demo:** `alert-sentinel-demo.html` is a standalone page that shows how
alerts look and sound. Double-click it to open it in your browser. For casting to a
Chromecast and remembering speaker names, it has to be opened from a web address:
double-click **Start Demo Server.command** (or run `node demo-server.js` in the app
folder) and it opens `http://localhost:4877/demo.html` in Chrome.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| **"Electron failed to install correctly"**, **ENOENT**, **EACCES**, or **"disk full"** | Almost always a full disk. Run `df -h ~`. You need at least 1 GB free (aim for 10). Empty the Trash, clear `~/Library/Caches`, then run the installer again. |
| **Bitdefender: "Safe Files blocked …"** | Choose **Block**, then move the folder with **Finder** instead of Terminal, and keep it in your home folder (not Downloads, Desktop, or Documents). |
| **A "wants to use your confidential information … keychain" password box** | Click **Deny**. The app doesn't use the keychain; this is fixed in current versions. |
| **The window shows the plain Electron welcome screen** | You opened Electron.app directly. Open **Alert Sentinel** from Spotlight instead. |
| **The app looks unstyled (plain black-and-white page)** | `renderer/style.css` is missing. Reinstall the latest update. |
| **Buttons do nothing after an update** | The app was still running in the background. Quit fully with **⌘Q** and reopen. |
| **"Can't find the Alert Sentinel folder"** | You moved the folder. Run **Install Alert Sentinel.command** again from its new spot. |
| **CONNECTION ERROR** | The reason shows under "Area clear." Usually it's no internet, or the alert service is briefly down; the app retries every 2 minutes. |

---

## What's in the folder

```
Alert-Sentinel/
├── README.md                       ← this file
├── Install Alert Sentinel.command  ← one-click install / reinstall
├── Start Demo Server.command       ← opens the browser demo at localhost
├── alert-sentinel-demo.html        ← standalone browser demo
└── alert-sentinel-electron/        ← the app itself
    ├── main.js          app startup, alert checks, casting
    ├── regions.js       countries and their alert feeds
    ├── cast.js          Google Home / Nest / Chromecast
    ├── alert-audio.js   siren + voice audio for speakers
    ├── alert-card.js    the alert picture shown on TVs
    ├── updater.js       Install Update button
    ├── server.js        local server for phones / speakers
    ├── renderer/        the app's window
    ├── public/          pages for phones (receiver, offline page, demo)
    └── build/icon.png   the app icon
```

## Uninstall

Delete **Alert Sentinel** from `~/Applications` and delete the **Alert-Sentinel** folder.
Its saved settings live in `~/Library/Application Support/Alert Sentinel`.
