The [99-leap-hand.rules](./99-leap-hand.rules) rule creates a persistent symlink `/dev/ttyLEAP` and sets the FTDI latency timer to 1 ms for that device.

You can copy the rule [99-leap-hand.rules](./99-leap-hand.rules) in `/etc/udev/rules.d`, then reload and trigger udev to make it effective:

```
sudo udevadm control --reload
sudo udevadm trigger
```