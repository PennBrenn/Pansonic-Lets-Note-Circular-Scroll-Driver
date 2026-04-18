# Panasonic Let's Note — Circular Scroll Driver

A custom Linux driver for the circular trackpad found on Panasonic Let's Note laptops. Turns the outer ring of the touchpad into a real scroll wheel — clockwise to scroll down, counter-clockwise to scroll up.

No extra hardware, no GUI, no DE integration required. Runs as a silent background daemon.

## How It Works

Draw a circle with one finger in the **outer ring** of your touchpad:

- **Clockwise** → Scroll Down
- **Counter-clockwise** → Scroll Up

The scroll zone is an annular ring from radius 170 to the edge (262 units). The driver tracks the angle of your finger in real time and converts accumulated rotation into X11 button 4/5 events — identical to a real scroll wheel.

## Installation

### 1. Dependencies

```bash
sudo pip3 install evdev python-xlib --break-system-packages
```

### 2. Install the `scrolldriver` command

```bash
sudo cp scrolldriver /usr/local/bin/scrolldriver
sudo chmod +x /usr/local/bin/scrolldriver
```

### 3. (Optional) Start on login

Add this line to your sudoers file (`sudo visudo`) to allow passwordless start:

```
penn ALL=(ALL) NOPASSWD: /usr/local/bin/scrolldriver
```

Then add `sudo scrolldriver start` to your session autostart.

## Usage

```bash
sudo scrolldriver            # toggle on / off
sudo scrolldriver start      # start the driver in the background
sudo scrolldriver stop       # stop the driver
sudo scrolldriver status     # show running state and current settings
sudo scrolldriver config     # list all configurable settings
```

## Configuration

Settings are stored in `~/.config/circular-scroll/config.json` and take effect immediately (no restart needed).

```bash
# Show current settings
sudo scrolldriver config

# Change individual values
sudo scrolldriver config inner_r=150
sudo scrolldriver config scroll_step_deg=12
sudo scrolldriver config settle_ms=100

# Multiple at once
sudo scrolldriver config inner_r=160 scroll_step_deg=15 settle_ms=80
```

| Setting | Default | Description |
|---|---|---|
| `inner_r` | `170` | Inner radius of the scroll zone (raw units, 60–240) |
| `scroll_step_deg` | `18` | Degrees of arc required per scroll tick — lower = more sensitive |
| `settle_ms` | `150` | Milliseconds your finger must stay in the zone before scrolling activates — prevents accidental triggers when moving to the edge |

## Files

| File | Purpose |
|---|---|
| `scrolldriver` | Main driver — installed as system command |
| `requirements.txt` | Python dependencies |

## Device

Auto-detects `Synaptics TM3562-003` by name — the device node (`/dev/input/eventX`) is resolved automatically at each start, so it survives reboots even if the kernel assigns a different event number.

## Supported Devices

| Device | Status |
|---|---|
| Panasonic CF-SV1 | ✅ Tested |
| Panasonic CF-SZ6 / CF-SZ5 | ✅ Should work |
| Panasonic CF-RZ6 / CF-RZ5 | ✅ Should work |
| Panasonic CF-LX6 / CF-MX5 | ✅ Should work |
| Any laptop with `Synaptics TM3562-003` | ✅ Should work |

The driver auto-detects the touchpad by name. Any device reporting as `Synaptics TM3562-003` will work. Other circular trackpads can be supported by changing `TOUCHPAD_NAME` in the `scrolldriver` script.

## License

MIT — free to use and modify.
