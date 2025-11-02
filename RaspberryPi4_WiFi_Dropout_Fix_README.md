# 🛜 Raspberry Pi 4 Wi-Fi Dropout Fix for Klipper / Mainsail / Fluidd
**Applies to:** Raspberry Pi 4 (2 GB or higher) running MainsailOS, FluiddPi, or stock Raspberry Pi OS (Bullseye/Bookworm)  
**Tested on:** Voron Trident & Voron 2.4 with integrated Pis

---

## 📋 Background
Many users experience random Wi-Fi disconnects on Raspberry Pi 4 boards inside 3D printers.  
This happens even when signal strength looks fine. The most common cause is the Pi’s **Wi-Fi power-saving mode**, which puts the radio to sleep under light load.

---

## 🧠 Symptoms
- Mainsail / Fluidd UI becomes unreachable mid-print  
- `iwconfig` shows frequent reconnects or low bit rates  
- Network ping spikes or drops  
- Klipper logs show `MCU 'mcu' shutdown: Lost communication with host`

---

## 🔍 Example Diagnostic Output

```bash
iwconfig wlan0
```
**Before Fix:**
```
Bit Rate=19.5 Mb/s
Power Management=on
Tx excessive retries=4643
Signal level=-62 dBm
```

**After Fix:**
```
Bit Rate=14.4 Mb/s
Power Management=off
Tx excessive retries=50
Signal level=-65 dBm
```

---

## ⚙️ Solution: Disable Wi-Fi Power Management Permanently

### 1️⃣ Disable it immediately (session-only)
```bash
sudo iwconfig wlan0 power off
```

### 2️⃣ Create a persistent startup script
If `/etc/rc.local` does **not exist**, create it:

```bash
sudo nano /etc/rc.local
```

Paste the following:
```bash
#!/bin/bash
# rc.local – runs at boot
/sbin/iwconfig wlan0 power off
exit 0
```

Save (**Ctrl + O**, **Enter**, **Ctrl + X**)  
Make it executable:
```bash
sudo chmod +x /etc/rc.local
```

---

### 3️⃣ Enable rc.local as a systemd service
Create the service file:
```bash
sudo nano /etc/systemd/system/rc-local.service
```
Paste this block:
```bash
[Unit]
Description=/etc/rc.local compatibility
ConditionFileIsExecutable=/etc/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

Then enable and start:
```bash
sudo systemctl enable rc-local
sudo systemctl start rc-local
```

Reboot and verify:
```bash
iwconfig wlan0
```
✅ It should show:
```
Power Management:off
```

---

## ✅ Results
- **Connection Stability:** 100 % uptime even during long prints  
- **Retry Count:** Dropped from thousands to < 100  
- **Ping Consistency:** < 10 ms average  
- **Bit Rate:** 10–20 Mb/s steady (normal for 2.4 GHz in enclosure)

---

## 🧰 Optional Enhancements
- Lock 2.4 GHz channel to **1, 6, or 11** instead of “Auto”  
- Move Pi outside metal chamber or use short USB Wi-Fi adapter (Realtek 8812BU / 8811CU)  
- Use official 5 V 3 A PSU to prevent voltage sag and brown-outs  
- Run a live monitor:
  ```bash
  watch -n10 "iwconfig wlan0 | egrep 'Link|Signal|Bit|retry'"
  ```

---

## 🧩 Credits
Fix documented and verified on **Fathom Design Systems Trident** by *Kyle Nyberg* (Inertia Cube project).
