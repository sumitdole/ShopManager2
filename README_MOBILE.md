# Shop Manager — Android App (Phase 1)

A standalone Android version of the shop app, built with Kivy/KivyMD.
It has its own local SQLite database on the phone (does **not** share
data with the Windows desktop app yet — see "About syncing" below).

## Important: I could not test-run this one

Unlike the desktop app, I wasn't able to install Kivy in the sandbox I
build in, so this code is **syntax-checked but not runtime-tested**. It
follows the standard, well-documented KivyMD 1.1.x patterns carefully,
but you should expect to hit at least a small bug or two on first run —
that's normal for a UI framework I couldn't actually execute. Report
back exactly what breaks (a screenshot + the error from `buildozer
android debug deploy run logcat` is ideal) and I'll fix it fast.

## What's included (Phase 1 — same scope as desktop Phase 1)

- **Dashboard** — today's sales/profit, low stock, pending credit, quick actions
- **Billing** — search & add products, adjust quantity, pick a customer,
  Cash/UPI/Card/Credit payment, saves the bill and shows a text receipt
  you can **Share** via WhatsApp/SMS/etc.
- **Products** — search, add, edit, delete
- **Customers** — search, add, view purchase history & credit balance,
  record credit payments
- **Inventory** — stock list (low-stock items sorted to the top), tap to
  make a manual adjustment

**Not included yet:** Suppliers, Purchases, Expenses, Returns, Reports,
Analytics, Bill History, login/roles, and PDF invoices (this phone
version shares a plain-text receipt instead — see the note at the top
of `screens/billing.py` for why).

## About syncing with the desktop app

You said sync can wait, so this phone app is fully standalone right
now — bills made here don't appear on the desktop app and vice versa.
That said, I built the database schema to intentionally match the
desktop app's tables (down to column names), and added a couple of
sync-friendly fields (`bills.synced`, a `device_info` table) so that
when you're ready, the sync feature can be added without redoing the
data model on either side. Mobile invoice numbers are prefixed `MINV-`
so they're visually distinguishable from desktop `INV-` numbers until
sync unifies them.

## Step 1 — Try it on your PC first (fast feedback loop)

Before doing a full Android build (which takes 20–60+ minutes the first
time), run it as a regular desktop window to catch obvious bugs in
seconds:

```
pip install -r requirements.txt
python main.py
```

This opens a resizable window running the exact same code that will go
into the APK. Click around — Products, Billing, Customers, Inventory —
and tell me what breaks. This step alone will catch most issues far
faster than a full Android build/deploy cycle.

## Step 2 — Set up Android building (one-time, needs Linux)

Buildozer (the tool that turns this into an APK) **only runs on
Linux**. On Windows, use WSL2:

1. Open PowerShell **as Administrator** and run:
   ```
   wsl --install -d Ubuntu
   ```
   Restart if prompted, then open the "Ubuntu" app from your Start menu
   and finish the one-time Linux username/password setup.

2. Inside that Ubuntu/WSL terminal, install build dependencies:
   ```
   sudo apt update
   sudo apt install -y python3-pip build-essential git python3-venv \
       ffmpeg libsdl2-dev libsdl2-image-dev libsdl2-mixer-dev libsdl2-ttf-dev \
       libportmidi-dev libswscale-dev libavformat-dev libavcodec-dev zlib1g-dev \
       openjdk-17-jdk unzip
   pip3 install --user buildozer cython
   ```

3. Copy this `shop_mobile` folder into your WSL filesystem (not your
   Windows `C:\` drive — WSL builds are much faster and more reliable
   on its own filesystem). From the WSL terminal:
   ```
   cp -r /mnt/c/Users/<you>/Downloads/shop_mobile ~/shop_mobile
   cd ~/shop_mobile
   ```

## Step 3 — Build the APK

```
~/.local/bin/buildozer android debug
```

The **first** build downloads the Android SDK/NDK (a few GB) and will
take a while — go make tea. Subsequent builds are much faster. When it
finishes, your APK is at:
```
bin/shopmanager-1.0-arm64-v8a_armeabi-v7a-debug.apk
```

## Step 4 — Install it on your phone

With your phone connected via USB (USB debugging enabled in Developer
Options) and WSL's USB passthrough set up, or simply by copying the
`.apk` file to your phone and opening it (allow "install from unknown
sources" when prompted):

```
~/.local/bin/buildozer android debug deploy run
```

or manually copy the `.apk` to the phone and tap it to install.

## If the build fails

Buildozer error output is usually specific about which native
dependency failed. Copy the last ~30 lines of the error and send them
to me — Android build failures are almost always a missing system
package (step 2) or a version mismatch, both fixable.

## Data safety

The phone's database lives in the app's private storage
(`/data/data/org.shopmanager.shopmanager/files/app/shop_data.db`
roughly) — uninstalling the app deletes it. There's no backup/export
screen on the phone yet (that's Phase 2, along with the rest of the
missing modules above). Until then, treat the desktop app as your
source of truth and use the phone app for quick on-the-go billing only.
