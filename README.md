# Raspberry Pi Stratum 1 NTP Server (GPS + PPS)

This Ansible playbook transforms a fresh Raspberry Pi OS installation into a **Stratum 1 Time Server**. It uses a GPS module with Pulse Per Second (PPS) output to achieve microsecond-level clock accuracy, independent of the internet.

## 🚀 Features

* **Automated Provisioning:** Sets up `gpsd`, `chrony`, and `pps-tools`.
* **Kernel-Level Precision:** Enables `pps-gpio` overlays for nanosecond hardware timestamps.
* **Optimized Chrony Config:** Uses the `KPPS` (Kernel PPS) driver locked to NMEA for high stability.
* **Hardware Conflict Resolution:** Automatically disables the serial login console to free up `/dev/ttyS0`.

## 🛠 Prerequisites

1. **Hardware:** - Raspberry Pi (Any model with GPIO).
* GPS Module with PPS output (e.g., Adafruit Ultimate GPS, WaveShare GPS Hat).
* PPS pin connected to **GPIO 18** (default).


2. **Software:** - Fresh install of Raspberry Pi OS (Lite is recommended).
* Ansible installed on your control machine.



---

## 📦 Installation & Usage

### 1. Clone the repository

```bash
git clone https://github.com/albal/gps-playbook.git
cd gps-playbook

```

### 2. Configure your Inventory

Edit the `hosts.ini` file to match your Raspberry Pi's IP address and credentials:

```ini
[gps_servers]
192.168.1.100 ansible_user=pi

```

### 3. Run the Playbook

```bash
ansible-playbook -i hosts.ini gps_ntp_setup.yml

```

*Note: The Pi will reboot automatically if hardware overlays need to be initialized.*

---

## 🔍 Verification

Once the playbook finishes, log into your Pi and run these checks:

### Check Hardware Pulse

Ensure the kernel is receiving the 1Hz pulse from the GPS:

```bash
sudo ppstest /dev/pps0

```

*You should see "assert" lines appearing every second.*

### Check Sync Status

View the Chrony sources to confirm it has locked onto the GPS:

```bash
chronyc sources -v

```

**Look for:**

* `#* KPPS`: The `*` indicates this is the current primary time source.
* `Reach 377`: Indicates a perfect 8/8 successful recent polls.
* `Last sample`: Should show an offset within +/- 1000ns to 5000ns.

---

## ⚙️ Configuration Details

| Component | Path | Description |
| --- | --- | --- |
| **GPSD** | `/etc/default/gpsd` | Configured with `-n` to start polling at boot. |
| **Chrony** | `/etc/chrony/chrony.conf` | Uses SHM for NMEA (seconds) and PPS for sub-microsecond precision. |
| **Kernel** | `/boot/firmware/config.txt` | Enables the `pps-gpio` overlay. |

---

## 🤝 Troubleshooting

* **No Data in `gpsmon`?** Check if the serial console is truly disabled in `/boot/firmware/cmdline.txt`.
* **Reach stays at 0?** Ensure your GPS has a 3D fix (usually indicated by a blinking LED on the GPS module).
* **Time is wildly off?** Run `sudo chronyc makestep` to force an immediate clock jump.

Would you like me to help you add a section for monitoring this server via Prometheus or Grafana?
