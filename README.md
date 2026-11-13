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

## Screenshots

![The Alert Sentinel console](https://github.com/user-attachments/assets/249fa237-f53d-43c4-b9aa-ef72caebb166)

| Critical alert | Silent mode (law-enforcement alerts) |
|---|---|
| ![Tornado warning takeover](https://github.com/user-attachments/assets/89fb8012-debc-4100-af10-9fb84c6b7f60) | ![Alert switched to silent mode](https://github.com/user-attachments/assets/9bd1ee88-ff49-45cf-a79a-7ca798c21c42) |

| On your TV (Chromecast / Nest Hub) | On your phone |
|---|---|
| ![Alert card shown on a TV](https://github.com/user-attachments/assets/df45935b-b337-4e80-b74e-becfc68ab1b1) | ![Phone receiver page](https://github.com/user-attachments/assets/764ece8d-0373-4daa-8886-002b91adf2e6) |

Works outside the US too:

![Country picker](https://github.com/user-attachments/assets/a442d5d0-a6e9-49dd-9566-8cb95c87c098)

*Screenshots use test alerts and a sample location.*

---

## Install

### Option 1: double-click (easiest)

1. Download **Alert-Sentinel-X.Y.Z.zip** (the newest one) from the
   [releases page](https://github.com/flyingtrees26/alert-sentinel-updates/releases),
   double-click it to unzip it, and move the **Alert-Sentinel** folder to your
   **home folder** (Finder → ⌘⇧H). *(Safari unzips downloads for you, so you may
   already have the folder.)* Keeping it out of Downloads, Desktop, and Documents
   avoids problems with macOS privacy settings and antivirus tools like
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
cd ~/Alert-Sentinel
```
*(Older copies keep the app in an `alert-sentinel-electron` folder inside. If yours
does, `cd` into that folder instead, here and below.)*

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
- **Access to Downloads,** the first time you use **Install Update**.

The very first time the app opens, it sends a test notification that says
**Notifications are enabled** (which also turns notifications on), then a **Welcome to
Alert Sentinel** notification 5 seconds later. To check or change
them later: System Settings → Notifications → Alert Sentinel.

The app keeps a small `launch-history.txt` in `~/Library/Application Support/Alert Sentinel`.
Updates never replace it; it just gets a new line each time a new version runs.
To see the first-launch notifications again, quit with **⌘Q**, delete the whole
`launch-history.txt` file (not just lines inside it), and reopen the app.

---

## Using it

1. Pick your **Country**, then enter your location: a ZIP code (US), a city (Canada),
   a region or state from the list (UK, Australia), or an optional region name (Europe).
2. Click **Connect**. Your location is remembered, and the app reconnects by itself
   every time it starts.
3. Under **Send Test Alert**, try a test to hear and see what a real alert does.

During an alert, press **Esc** (or click **Silence Now**) to stop the sound right away.
Esc works even when another app is in front.

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
2. Under the newest release's **Assets**, download `Alert-Sentinel-X.Y.Z.zip`. It's
   the same file new users install from. Leave it in Downloads. You don't need to
   unzip it, but it's fine if you do, or if Safari already did: the button finds
   the zip or the unzipped **Alert-Sentinel** folder either way.
   *(Don't use the green "Code → Download ZIP" button. That isn't an update.)*
3. In the app, click **⬇ Install Update** → **Install & Restart**.

The app backs up the files it replaces to a `.update-backups` folder inside the app folder.
If the button says it isn't allowed to look in Downloads, use **Choose update file…**
and pick the zip, or allow Alert Sentinel under System Settings → Privacy & Security →
Files & Folders.

Each download is complete, so installing the newest one brings any version 1.1.0 or
later fully up to date. To update by hand instead, unzip it and run
**Install Alert Sentinel.command** from the new folder.

### Emergency updates

Every 2 minutes the app checks the releases page. If the newest release has
**EMERGENCY** in its title, the app downloads it, installs it, and restarts by itself,
no clicks needed. If an alert is playing, it waits until the alert is done.
A notification tells you when an emergency update arrives, and again once it's installed. You can
turn this off with **Install emergency updates automatically** under Updates.

**To post an emergency release:** make a new release like normal (newer version
number, zip attached) and put **EMERGENCY** in the title, for example
*Alert Sentinel 1.2.4 EMERGENCY*. Normal releases never install by themselves.

---

## Other ways to run it

**From Terminal** (no app needed):
```bash
cd ~/Alert-Sentinel
export PATH="$PWD/.node/bin:$PATH"
npm start
```

**The browser demo:** `public/demo.html` is a standalone page that shows how alerts
look and sound. Double-click it to open it in your browser. For casting to a
Chromecast and remembering speaker names, it has to be opened from a web address:
double-click **Start Demo Server.command** (or run `node demo-server.js` in the
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
├── sentinel-update.json            ← lets Install Update recognize this download
├── main.js          app startup, alert checks, casting
├── regions.js       countries and their alert feeds
├── cast.js          Google Home / Nest / Chromecast
├── alert-audio.js   siren + voice audio for speakers
├── alert-card.js    the alert picture shown on TVs
├── updater.js       Install Update button
├── emergency.js     emergency updates (checks every 2 minutes)
├── server.js        local server for phones / speakers
├── renderer/        the app's window
├── public/          pages for phones (receiver, offline page, demo)
└── build/icon.png   the app icon
```

## Uninstall

Delete **Alert Sentinel** from `~/Applications` and delete the **Alert-Sentinel** folder.
Its saved settings live in `~/Library/Application Support/Alert Sentinel`.
