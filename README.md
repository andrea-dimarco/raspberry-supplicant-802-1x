# Raspberry Pi Supplicant for 802.1x (2024) (WIP)

Please follow the instruction below.

## 1. INSTALLATION

Open a **terminal** and execute:

```
sh setup.sh
```

Click **Choose device** and select your Raspberry Pi model from the list.

Next, click **Choose OS** and select the operating system at the top of the list.

Next, plug a microSD card. click **Choose storage** and select your storage device.

Finally, click on the **cog** icon, toggle **Enable SSH** and **Set username and password**, now define a custom username and password.

DO NOT turn on **Configure wireless LAN** since the Raspberry will be connected via *LAN cable*.

*Optional*: compile the **Set local settings option**

Click on **WRITE** to install the OS.

## 2. SETUP

### 2.1 Config
Connect the Raspberry Pi to a monitor and on the console type:

```
sudo raspi-config
```

then
- Go to Localisation Options > WLAN Country.
- Select your Country in the list.

Once you set the country, you can start raspi-config again to set up your Wi-Fi connection:

- Go to System Options.
- Choose Wireless LAN.
- Enter your network SSID (its name).
- There is no list, so make sure to type the exact name.
- Enter your passphrase (password).
- Finish.


### 2.2 Enable SSH

To enable SSH by default use the following commands on the console:

```
sudo update-rc.d ssh defaults
sudo update-rc.d ssh enable
```


### 2.3 Enable Wi-Fi interface

Use:

```
sudo nmcli con add con-name hotspot ifname wlan0 type wifi ssid "RAISE-WiFi"
```

Now set the access point security and password

```
sudo nmcli con modify hotspot wifi-sec.key-mgmt wpa-psk
sudo nmcli con modify hotspot wifi-sec.psk "logic-rules"
```

Now configure Network Manager to run in access point mode:

```
sudo nmcli con modify hotspot 802-11-wireless.mode ap 802-11-wireless.band bg ipv4.method shared
```


## 3 MAINTENANCE

To edit the network's settings use:

```
sudo nmtui
```

If you experience any issue, check the logs with:

```
journalctl
```

You can use grep to filter the output:

```
journalctl | grep hotspot
journalctl | grep wifi
```