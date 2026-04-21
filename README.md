# Panasonic Let's Note — Circular Scroll Driver

A custom Linux driver for the circular trackpad found on Panasonic Let's Note laptops. Turns the outer ring of the touchpad into a real scroll wheel — clockwise to scroll down, counter-clockwise to scroll up.

No extra hardware, no GUI, no DE integration required. Runs as a silent background daemon.

---

## Quick Install

```bash
# 1. Install dependencies
sudo pip3 install evdev python-xlib --break-system-packages

# 2. Copy the driver to your PATH
sudo cp scrolldriver /usr/local/bin/scrolldriver
sudo chmod +x /usr/local/bin/scrolldriver

# 3. Start it
sudo scrolldriver
```

That's it. Run `sudo scrolldriver` again to stop it.

> **Autostart:** Toggling the driver on also registers it as a **systemd service** so it starts automatically on every boot. Toggling it off removes the service. No manual systemd setup needed.

---

## How It Works

Draw a circle with one finger in the **outer ring** of your touchpad:

- **Clockwise** → Scroll Down
- **Counter-clockwise** → Scroll Up

### Scroll zone

The touchpad is treated as a 525×525 unit grid. The active scroll zone is an annular ring:

- **Inner edge:** radius 170 units from centre (configurable)
- **Outer edge:** radius 262 units (the physical edge of the pad)

Touches inside the inner radius are ignored entirely, so normal cursor movement and clicking are unaffected.

### Angle tracking

The driver reads raw multitouch events from the kernel (`/dev/input/eventX`) via `evdev`. On each sync event it:

1. Checks that exactly one finger is in the scroll zone.
2. Computes the current angle with `atan2`.
3. Accumulates the angular delta since the last event.
4. Fires an X11 scroll event (button 4 / button 5) each time the accumulation exceeds `scroll_step_deg`.

Scroll events are injected via **Xlib** when an X session is detected, falling back to a **UInput** virtual device otherwise. The X display is discovered automatically by scanning `/proc` for running sessions, so it works even when the daemon is started as root by systemd before your desktop session is fully up.

### Settle delay

A configurable `settle_ms` delay prevents accidental scroll triggers when your finger first enters the outer ring while moving the cursor. The angle is tracked silently during the settle window but no events are emitted.

---

## Usage

```bash
sudo scrolldriver            # toggle on / off  (also enables/disables autostart)
sudo scrolldriver start      # start the driver in the background
sudo scrolldriver stop       # stop the driver
sudo scrolldriver status     # show running state and current settings
sudo scrolldriver config     # list all configurable settings
```

### Autostart behaviour

| Action | Effect |
|---|---|
| Toggle **on** | Starts daemon + creates & enables `scrolldriver.service` |
| Toggle **off** | Stops daemon + disables & removes `scrolldriver.service` |
| `start` / `stop` | Starts or stops the daemon only — does not touch autostart |

The systemd unit file is written to `/etc/systemd/system/scrolldriver.service` and targets `multi-user.target`, so it runs early in boot before any graphical session.

---

## Configuration

Settings live in `~/.config/circular-scroll/config.json` and take effect immediately on the next finger touch — no restart required.

```bash
# Show all current settings
sudo scrolldriver config

# Change a single value
sudo scrolldriver config inner_r=150
sudo scrolldriver config scroll_step_deg=12
sudo scrolldriver config settle_ms=100

# Change multiple values at once
sudo scrolldriver config inner_r=160 scroll_step_deg=15 settle_ms=80
```

| Setting | Default | Range | Description |
|---|---|---|---|
| `inner_r` | `170` | 60–240 | Inner radius of the scroll zone in raw touchpad units. Increase to shrink the ring; decrease to widen it. |
| `scroll_step_deg` | `18` | 5–60 | Degrees of arc your finger must travel to produce one scroll tick. Lower values are more sensitive. |
| `settle_ms` | `150` | 0–600 | Milliseconds your finger must stay in the zone before scrolling activates. Prevents accidental triggers when sliding to the edge. |

---

## Files

| File | Purpose |
|---|---|
| `scrolldriver` | Main driver script — installed as a system command |
| `requirements.txt` | Python package dependencies |

### Runtime paths

| Path | Purpose |
|---|---|
| `/tmp/scrolldriver.pid` | PID file written by the daemon while running |
| `~/.config/circular-scroll/config.json` | User configuration |
| `/etc/systemd/system/scrolldriver.service` | Systemd unit (created on toggle-on, removed on toggle-off) |

---

## Device Detection

The driver auto-detects the touchpad by matching `Synaptics TM3562-003` against the names reported by all input devices. The kernel device node (`/dev/input/eventX`) is resolved fresh on every start, so the driver survives reboots even if the kernel assigns a different event number.

To use this with a different circular trackpad, change the `TOUCHPAD_NAME` constant near the top of the `scrolldriver` script.

## Supported Devices

| Device | Status |
|---|---|
| Panasonic CF-SV1 | ✅ Tested |
| Any laptop with `Synaptics TM3562-003` | ✅ Should work |

---

## License

MIT — free to use and modify.
