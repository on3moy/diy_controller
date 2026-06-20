# DIY Controller

Map any USB or Bluetooth gamepad to keyboard shortcuts and mouse actions. Configure
buttons, sticks, bumpers, and the D-pad in a single JSON file — no code changes needed.

## Use case: lecture remote

Pair with a GameSir Cyclone 2 (or any gamepad) to control slides hands-free:
navigate slides, move the mouse as a pointer, and trigger hotkeys from across the room.
Works with PowerPoint, Google Slides, Keynote-on-web, and PDFs — anything driven by arrow keys.

## Prerequisites

- **Python 3.9+**
- **uv** — install via winget or see [docs.astral.sh/uv](https://docs.astral.sh/uv/getting-started/installation/):

  ```powershell
  winget install astral-sh.uv
  ```

## Install

```powershell
uv sync
```

## Run

1. Connect the controller (USB or Bluetooth). Windows should detect it as a gamepad.
2. Start:

   ```powershell
   uv run controller_remote.py
   ```

3. Press **Ctrl+C** in the terminal to quit.

## Default mapping

| Action            | Controller input          | Sends      |
|-------------------|---------------------------|------------|
| Previous slide    | D-pad Left, Left bumper   | `Left`     |
| Next slide        | D-pad Right, Right bumper | `Right`    |
| Move mouse        | Left stick                | mouse move |
| Left click        | A button                  | click      |
| Start slideshow   | Y button                  | `F5`       |
| Exit presentation | B button                  | `Esc`      |

## Remapping

All mappings live in `config.json` — no code changes needed.

Button and axis numbers vary by controller model. To find the right numbers for yours:

```powershell
uv run controller_remote.py --diagnose
```

Press each button / move each stick; it prints the index. Update `config.json` and re-run.
You can also use the `explore_controller.ipynb` notebook for an interactive live watcher.

### Valid action strings

`left`, `right`, `up`, `down`, `pageup`, `pagedown`, `f5`, `esc`, `space`, `home`,
`end`, `b` (black screen), `w` (white screen), and `click` (left mouse click).

### config.json fields

- `deadzone` — ignore stick drift below this threshold (0–1).
- `mouse_speed` — cursor movement multiplier.
- `poll_hz` — how often to read the controller per second.
- `buttons` / `bumpers` — `{ "<button index>": "<action>" }`.
- `hat` — D-pad directions → actions. Only applies if your controller exposes a hat input (`Hats: 1+` in diagnose output). Many modern gamepads report the D-pad as regular buttons instead — map those indices under `buttons`.
- `axes` — which axis numbers drive mouse X/Y movement.
