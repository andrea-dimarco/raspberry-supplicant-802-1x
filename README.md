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

### 2.1. Config
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


### 2.2. Enable SSH

To enable SSH by default use the following commands on the console:

```
sudo update-rc.d ssh defaults
sudo update-rc.d ssh enable
```


### 2.3. Enable Wi-Fi interface

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


## 3. 802-1X Credentials

To configure our computer for using WPA-Supplicant, two configuration files need to be edited.


### 3.1. Supplicant Config file

First, create a new text file in `/etc` with your favourite editor or, if you are logged in to a graphical environment, by typing in a terminal:

```
sudo gedit /etc/wpa_supplicant.conf
```

Add this configuration in the file:

```
# Where is the control interface located? This is the default path:
ctrl_interface=/var/run/wpa_supplicant

# Who can use the WPA frontend? Replace "0" with a group name if you
#   want other users besides root to control it.
# There should be no need to chance this value for a basic configuration:
ctrl_interface_group=0

# IEEE 802.1X works with EAPOL version 2, but the version is defaults 
#   to 1 because of compatibility problems with a number of wireless
#   access points. So we explicitly set it to version 2:
eapol_version=2

# When configuring WPA-Supplicant for use on a wired network, we don’t need to
#   scan for wireless access points. See the wpa-supplicant documentation if
#   you are authenticating through 802.1x on a wireless network:
ap_scan=0
```

#### 3.1.1. Authentication Info

Add your personal information in the file:

```
network={
 key_mgmt=IEEE8021X
 eap=PEAP
 identity="USERNAME"
 anonymous_identity="USERNAME"
 password="PASS"
 phase1="auth=MD5"
 phase2="auth=CHAP password=PASS"
 eapol_flags=0
}
```

To test the authentication process, we can call WPA-Supplicant directly with our new configuration file.
For example, in case of a wired network, execute the following command:

```
sudo wpa_supplicant -c /etc/wpa_supplicant.conf -D wired -i eth0
```


### 3.2. Change Default Behaviour

If WPA-Supplicant can authenticate our computer to the network, we can add it to the global network configuration. By doing this WPA-Supplicant is automatically run when we boot up the computer or restart its network.

Open the network interface configuration file:

```
sudo gedit /etc/network/interfaces
```

and

```
# The loopback interface, this is the default configuration:
auto lo
iface lo inet loopback

# The first network interface.
# In this case we want to receive an IP-address through DHCP:
auto eth0
iface eth0 inet dhcp

# In this case we have a wired network:
wpa-driver wired

# Tell the system we want to use WPA-Supplicant with our configuration file:
wpa-conf /etc/wpa_supplicant.conf
```

To test our new configuration, we stop the network on our system before saving the above configuration file:

```
sudo /etc/init.d/networking stop
```

After saving this file, we should be able to start the network with 802.1x authentication enabled:

```
sudo /etc/init.d/networking start
```

For most cases, this is all that is required to automatically authenticate our computer to the network using the IEEE 802.1x protocol. 


## 4. MAINTENANCE

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