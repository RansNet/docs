# Device Bootstrapping

RansNet devices will auto "call home" to mfusion for central monitoring and management. But they must know how to reach to mfusion.

Device bootstrapping enables RansNet devices to connect with mfusion for monitoring and management.

!!! info
    If you are using RansNet's cloud-hosted mfusion, devices automatically register to `portal.ransnet.com` once online. Manual bootstrapping is only required for **on-premise/private mfusion deployments** or when using a **static WAN connection**.

---

## Command Line Interface

RansNet Command Line Interface (CLI) provides an intuitive way to bootstrap RansNet devices. You may also use the CLI to configure advanced features for complex deployment scenarios or perform in-depth troubleshooting.

CLI can only be accessed via **serial console** or **SSH**.

RansNet CLI operates in 4 modes:

| Mode | Description |
|------|-------------|
| User Mode | Basic monitoring commands (`>` prompt) |
| Privileged Mode | Full monitoring and system commands (`#` prompt) |
| Config Mode | Device-wide configuration (`(config)#` prompt) |
| Context Mode | Per-interface or specific feature configuration (`(config-if-*)#` prompt) |

The CLI supports a number of keyboard shortcuts to help you navigate and edit commands efficiently:

| Shortcut | Description |
|---|---|
| `Ctrl + A` | Move to the beginning of the line |
| `Ctrl + E` | Move to the end of the line |
| `Ctrl + C` | Clear the current line |
| `Ctrl + D` | Delete the character to the right of the cursor |
| `Ctrl + K` | Delete everything to the right of the cursor |
| `Ctrl + U` | Delete everything to the left of the cursor |
| `Ctrl + W` | Delete the word to the left of the cursor |
| `Ctrl + L` | Clear the screen |
| `?` | Show the list of available commands at the current context |
| `no` | Negate or remove an existing configuration command |
| `Tab` / `Space` | Auto-complete the current command (type enough characters to make it unique) |

---

## Getting Console Access

All RansNet devices come with a console port for offline access. Different hardware series use different baud rates. Use an RJ45 serial cable with the settings below:

| Series | Baud Rate |
|--------|-----------|
| CMG / HSG | 19200 |
| HSA / UA / XE / UAP | 115200 |

---

## Getting SSH Access

If you don't have a serial console cable, RansNet devices allow SSHv2 access from LAN by default.

**CMG/HSG series:** Connect your PC to `eth2-Port3` (Management Port) using an Ethernet cable. Your PC will receive a DHCP IP in the `10.10.10.x/24` range. SSH to `10.10.10.1`.

**HSA/UA/XE series:** Connect your PC to `LAN1`. After receiving a DHCP IP, SSH to `192.168.8.1`.

!!! warning
    It is highly recommended to remove the default SSH firewall rules (disabling SSH access) after the device is onboarded to mfusion.

---

## Connecting to mfusion

### Static WAN Connection

If using a static WAN connection, follow below CLI to configure the WAN interface IP and default gateway (replace addresses with your actual values):

```
mbox> enable
Enter enable password:
mbox# configure
mbox(config-if-eth0)#interface eth0
mbox(config-if-eth0)#enable
mbox(config-if-eth0)#ip address 200.13.198.166/30
mbox(config-if-eth0)#exit
mbox(config)#ip name-server 8.8.8.8 8.8.4.4
mbox(config)#ip default-gateway 200.13.198.165
mbox(config)#end
mbox# write memory
```

### On-Premise mfusion

Use the `ip host` command to point `portal.ransnet.com` to your own mfusion IP, then verify connectivity with a ping:

```
mbox> enable
Enter enable password:
mbox# configure
mbox(config)# ip host portal.ransnet.com 192.168.1.1
mbox(config)# do ping portal.ransnet.com
PING portal.ransnet.com (192.168.1.1): 56 data bytes
64 bytes from 192.168.1.1: seq=0 ttl=58 time=2.865 ms
64 bytes from 192.168.1.1: seq=1 ttl=58 time=2.936 ms
64 bytes from 192.168.1.1: seq=2 ttl=58 time=2.790 ms
64 bytes from 192.168.1.1: seq=3 ttl=58 time=2.640 ms
64 bytes from 192.168.1.1: seq=4 ttl=58 time=2.788 ms

--- 192.168.1.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.640/2.803/2.936 ms
mbox(config)# end
mbox# write memory
```

---

## Factory Reset

RansNet devices have a built-in self-protection mechanism. Each time a configuration is pushed from mfusion, the device backs up the current working config before applying the change. If a wrong configuration causes the device to lose remote connectivity, the device will **reboot within 15 minutes** and restore the last working config automatically.

In rare situations (e.g. device hang due to over-utilization or power issue) where remote management is completely lost, on-site staff can perform a factory reset to recover the device.

### Reset HSA / UA / XE / UAP

1. Press and hold the **Reset** button for more than **10 seconds**, then release.
   *(The device will re-flash to default firmware and reboot.)*
2. Wait **3 minutes** for the device to boot up with default configuration and automatically register back to `portal.ransnet.com`.
3. On mfusion, verify the device is back online, then **resync your configuration**.
   *(The device will reboot again and come up with the new working configuration.)*

### Reset CMG / HSG Database

CMG/HSG series do not have a reset button — a power cycle usually recovers the device. In cases of database corruption, follow these steps.

!!! warning
    Ensure you have downloaded your database backup before proceeding.

1. Console or SSH into the device.
2. Log in to privileged mode using the `enable` credentials.
3. Run `write erase` and follow the prompts:

```
RansNet# write erase
Do you want to erase current CLI config "y" or "n": y
[info] resetting start-up config to default…
[note] Please restart mbox to apply the default config.
Remove local captive portal contents. Remove all "y" or "n": y
Remove mbox portal user files (e.g. Historical Reports). Remove all "y" or "n": y
Do you want to reset all databases "y" or "n": y
[info] ...
Do you want to erase local config backup files "y" or "n": y
Do you want to erase MAP statistics "y" or "n": y
RansNet# reboot
```
