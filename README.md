# Automatic Bluetooth device Connection Before Login on Linux/GNOME

## 1️⃣ Enable the Bluetooth service at startup

Open a terminal and run:

```bash
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
```

* `enable` → the service will start automatically at boot.
* `start` → starts the service immediately without rebooting.

---

## 2️⃣ Configure the Bluetooth daemon for auto-enabling devices

Edit the main configuration file:

```bash
sudo nano /etc/bluetooth/main.conf
```

Add or modify the following:

```ini
[Policy]
AutoEnable=true
```

This ensures Bluetooth automatically powers on if there are trusted devices.

---

## 3️⃣ Pair and trust your device at the system level

Open the Bluetooth control tool:

```bash
bluetoothctl
```

Then execute the following commands, replacing `AA:BB:CC:DD:EE:FF` with your device's MAC:

```bash
power on
agent on
default-agent
scan on       # Press a key on your device to detect it
pair AA:BB:CC:DD:EE:FF
trust AA:BB:CC:DD:EE:FF
connect AA:BB:CC:DD:EE:FF
scan off
```

* `trust` → allows the device to connect automatically without user intervention.
* This creates a global configuration so the login manager (GDM) can use the device.

---

## 4️⃣ Create a systemd service to auto-connect the device

Create a new service file:

```bash
sudo nano /etc/systemd/system/bt-device.service
```

Add the following content:

```ini
[Unit]
Description=Connect Bluetooth Device at Startup
After=bluetooth.target

[Service]
Type=oneshot
ExecStartPre=/bin/sleep 5
ExecStart=/usr/bin/bluetoothctl connect AA:BB:CC:DD:EE:FF
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

* `ExecStartPre=/bin/sleep 5` → gives the Bluetooth daemon a few seconds to fully initialize.
* Replace `AA:BB:CC:DD:EE:FF` with your device’s MAC.

Enable and start the service:

```bash
sudo systemctl enable bt-device.service
sudo systemctl start bt-device.service
```

---

## 5️⃣ Reboot and test

After rebooting:

1. Bluetooth service starts automatically.
2. Your device connects before login.
3. You should be able to type your password at the GDM login screen immediately.

---

## ⚡ Optional: Retry loop for unreliable devices

If your device sometimes fails to connect on boot, create a small script:

```bash
sudo nano /usr/local/bin/bt-connect-device.sh
```

```bash
#!/bin/bash
MAC="AA:BB:CC:DD:EE:FF"
for i in {1..5}; do
    bluetoothctl connect $MAC && exit 0
    sleep 2
done
exit 1
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/bt-connect-device.sh
```

Update the service to use this script:

```ini
ExecStart=/usr/local/bin/bt-connect-device.sh
```

This will try connecting the device multiple times until successful.
