# Panasonic Let's Note — Circular Scroll Driver

- **Clockwise** → Scroll Down
- **Counter-clockwise** → Scroll Up

---

## Instillation

```bash
# 1. Install dependencies
sudo pip3 install evdev python-xlib --break-system-packages

# 2. Install the driver (no repo clone needed)
sudo curl -fsSL https://raw.githubusercontent.com/PennBrenn/Pansonic-Lets-Note-Circular-Scroll-Driver/main/scrolldriver \
  -o /usr/local/bin/scrolldriver
sudo chmod +x /usr/local/bin/scrolldriver

# 3. Start it
sudo scrolldriver start
```
---

## Supported Devices

| Device | Status |
|---|---|
| Panasonic CF-SV1 | Tested |
| Any laptop with `Synaptics TM3562-003` | Should work |

The driver auto-detects the touchpad by name. To support a different circular trackpad, change `TOUCHPAD_NAME` in the `scrolldriver` script.

## How It Works

The driver watches raw touchpad events and tracks your finger's angle inside an annular scroll zone (radius 170 → 262 units from centre). Every time you complete enough arc, it fires an X11 scroll event — identical to a real scroll wheel.

A short settle delay (150 ms by default) prevents accidental scroll triggers when you move your finger to the edge of the pad.

## Usage

```bash
sudo scrolldriver            # toggle on / off
sudo scrolldriver start      # start the driver in the background
sudo scrolldriver stop       # stop the driver
sudo scrolldriver status     # show running state and current settings
sudo scrolldriver config     # list all configurable settings
```

## Configuration

Settings live in `~/.config/circular-scroll/config.json` and take effect immediately — no restart needed.

```bash
# Show current settings
sudo scrolldriver config

# Change a value
sudo scrolldriver config inner_r=150
sudo scrolldriver config scroll_step_deg=12
sudo scrolldriver config settle_ms=100

# Change multiple at once
sudo scrolldriver config inner_r=160 scroll_step_deg=15 settle_ms=80
```

| Setting | Default | Description |
|---|---|---|
| `inner_r` | `170` | Inner radius of the scroll zone (raw units, 60–240) |
| `scroll_step_deg` | `18` | Degrees of arc per scroll tick — lower = more sensitive |
| `settle_ms` | `150` | Delay before scrolling activates after entering the zone (ms) |

## Autostart on Login

Add this to your sudoers file (`sudo visudo`) to allow passwordless start:

```
YOUR_USERNAME ALL=(ALL) NOPASSWD: /usr/local/bin/scrolldriver
```

Then add `sudo scrolldriver start` to your session autostart (e.g. autostart apps in your DE, or `~/.profile`).

## License

MIT — free to use and modify.
