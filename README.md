# 🎮 DIY Controller

Map any USB or Bluetooth gamepad to keyboard shortcuts and mouse actions. Configure
buttons, sticks, bumpers, and the D-pad in a single JSON file — no code changes needed.

Built on **[pygame](https://www.pygame.org/)** for gamepad input and **[pynput](https://pypi.org/project/pynput/)** for keyboard/mouse control. Cross-platform (Windows, macOS, Linux).

## 🎤 Use case: lecture remote

Pair with a GameSir Cyclone 2 (or any gamepad) to control slides hands-free:
navigate slides, move the mouse as a pointer, and trigger hotkeys from across the room.
Works with PowerPoint, Google Slides, Keynote-on-web, and PDFs — anything driven by arrow keys.

## 📋 Prerequisites

- **Python 3.9+**
- **uv** — install via winget or see [docs.astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/):

  ```powershell
  winget install astral-sh.uv
  ```

## ⚡ Install

```powershell
uv sync
```

## ▶️ Run

1. 🔌 Connect the controller (USB or Bluetooth). Windows should detect it as a gamepad.
2. ▶️ Start:

   ```powershell
   uv run controller_remote.py
   ```

3. ⏹️ Press **Ctrl+C** in the terminal to quit.

## 🗺️ Default mapping

| Action            | Controller input          | Sends      |
|-------------------|---------------------------|------------|
| ⬅️ Previous slide    | D-pad Left, Left bumper   | `Left`     |
| ➡️ Next slide        | D-pad Right, Right bumper | `Right`    |
| 🖱️ Move mouse        | Left stick                | mouse move |
| 🖱️ Left click        | A button                  | click      |
| ▶️ Start slideshow   | Y button                  | `F5`       |
| ❌ Exit presentation | B button                  | `Esc`      |

## 🔧 Remapping

All mappings live in `config.json` — no code changes needed.

Button and axis numbers vary by controller model. To find the right numbers for yours, use either method:

### Method 1: Diagnostic mode (command-line) 🖥️

```powershell
uv run controller_remote.py --diagnose
```

Press each button and move each stick. The script prints the button/axis/hat index for every input.
Update `config.json` with the indices and re-run the main script.

### Method 2: Interactive notebook 📓

```powershell
jupyter notebook explore_controller.ipynb
```

Open `explore_controller.ipynb` in Jupyter. Cell 4 provides a live watcher that continuously prints controller input changes as you interact with it. Great for discovering your gamepad's exact layout without leaving the notebook.

## 📝 Valid action strings

Keyboard keys (one-shot on button press):
- **Navigation:** `left`, `right`, `up`, `down`, `pageup`, `pagedown`, `home`, `end`
- **Shortcuts:** `f5`, `esc`, `space`
- **Presentation modes:** `b` (black screen), `w` (white screen)
- **Mouse:** `click` (left mouse click)

Unknown action strings are logged as warnings during startup.

## ⚙️ config.json reference

All fields are optional and have sensible defaults. Reload config by restarting the script.

### Performance & response settings

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| `poll_hz` | int | 60 | Polling frequency in Hz. Higher = lower latency but more CPU usage. [pygame.time.Clock docs](https://www.pygame.org/docs/ref/time.html#pygame.time.Clock.tick) |
| `deadzone` | float | 0.15 | Ignore stick drift below this threshold (0–1 range). Reduces noise from analog sticks at rest. [pynput mouse movement](https://pynput.readthedocs.io/en/latest/mouse.html) |
| `mouse_speed` | int | 18 | Multiplier for left-stick cursor movement. Measured in pixels per frame. Increase for faster mouse, decrease for precision. |

### Input mapping

| Field | Type | Purpose | Notes |
|-------|------|---------|-------|
| `buttons` | dict | Map button indices to actions | Keys are button indices (as int or string, e.g., `"0"`, `"1"`). Values are action strings from the list above. Each press fires once. |
| `bumpers` | dict | Map shoulder buttons to actions | LB = button index 4, RB = button index 5 (may vary by controller). Semantically separate from `buttons` but works identically. |
| `hat` | dict | D-pad directions to actions | Only applies if your controller exposes a hat input (check diagnose output for `Hats: 1+`). Many modern gamepads report D-pad as regular buttons instead — map those under `buttons`. Keys: `"left"`, `"right"`, `"up"`, `"down"`. |
| `axes` | dict | Configure stick-to-mouse mapping | Contains `"move_x"` and `"move_y"` (int indices). Analog stick values are continuously scaled by `mouse_speed` and sent to the mouse. [pynput mouse.move](https://pynput.readthedocs.io/en/latest/mouse.html#pynput.mouse.Controller.move) |

### Example config (GameSir Cyclone 2)

```json
{
  "deadzone": 0.15,
  "mouse_speed": 18,
  "poll_hz": 60,
  "buttons": {
    "0": "click",
    "1": "esc",
    "3": "f5",
    "13": "left",
    "14": "right"
  },
  "bumpers": {
    "4": "left",
    "5": "right"
  },
  "axes": {
    "move_x": 0,
    "move_y": 1
  }
}
```

## 🔍 Discovering your controller's layout

Different controllers expose buttons, sticks, and D-pads differently. Use the diagnostic mode to map your hardware:

1. **List available inputs:** Run `--diagnose` and note the output header:
   ```
   Joystick name: GameSir Cyclone 2
   Buttons: 16  Axes: 6  Hats: 1
   ```

2. **Test each input:** Press a button, move a stick, or press the D-pad. The diagnostic output will show:
   - `Button 0 pressed` / `Button 0 released` → button index
   - `Axis 0 value: 0.95` → axis index and analog value (-1 to 1)
   - `Hat state: (-1, 0)` → D-pad directions (left=-1 or right=1 for X; up=1 or down=-1 for Y)

3. **Update config.json:** Map the discovered indices to action strings and restart.

## 📚 External documentation

For advanced configurations, refer to:
- **[pygame.joystick](https://www.pygame.org/docs/ref/joystick.html)** — gamepad input API and state polling
- **[pynput.keyboard](https://pynput.readthedocs.io/en/latest/keyboard.html)** — keyboard control (what `tap()` uses internally)
- **[pynput.mouse](https://pynput.readthedocs.io/en/latest/mouse.html)** — mouse control and movement
- **[uv documentation](https://docs.astral.sh/uv/)** — package manager and Python version management

## 🐛 Troubleshooting

**Controller not detected?**
- Ensure the gamepad is connected and recognized by Windows (check Device Manager → Human Interface Devices).
- Try a different USB port.
- Run `--diagnose` to see if pygame detects it.

**Incorrect button/axis mapping?**
- Use `--diagnose` or `explore_controller.ipynb` to discover the exact indices for your model.
- Some controllers report D-pad as buttons (indices 13–14) instead of a hat input; check which one your device uses.

**Sluggish mouse movement?**
- Increase `mouse_speed` in config.json (default 18).
- Increase `poll_hz` (default 60 Hz) for lower latency, but watch CPU usage.

**Actions not firing?**
- Verify action strings are in the valid list (typos are logged as warnings).
- Check that button/axis indices match your controller's layout using diagnostic mode.
