# UWB Fall Monitor

Track a tag in 3D with four UWB anchors and show it live in a browser. The viewer draws a top view, a side view and a height chart, and raises an alert when the tag's height drops quickly.

Built with ESP32 boards carrying DW3000 UWB chips (Makerfabs ESP32 UWB DW3000).

## How it works

1. The tag sends a poll to each anchor in turn (single-sided two-way ranging).
2. Each anchor replies with its timestamps, and the tag turns them into a distance.
3. The tag streams the distances over Wi-Fi as lines like `R,2,1.853` (anchor 2, 1.853 m).
4. The viewer (`visualizer3d.html`) combines the four distances into an x, y, height position.

## What you need

- 5 ESP32 + DW3000 boards: 4 anchors and 1 tag
- USB power for each board (power banks work)
- A computer with Chrome or Edge
- A tape measure

## Files

| File | Goes on | Purpose |
|---|---|---|
| `anchor/anchor.ino` | the 4 anchors | Answers polls from the tag. Same file for all, only `ANCHOR_ID` changes. |
| `tag/tag.ino` | the tag | Ranges to the anchors and sends the results over Wi-Fi. |
| `visualizer3d.html` | your computer | Live 3D view and fall alert. Open it by double-clicking. |

## Setup

### 1. Install the libraries (Arduino IDE)

- **Dw3000** library for the Makerfabs board: copy the `dw3000` folder from
  `github.com/Makerfabs/Makerfabs-ESP32-UWB-DW3000` into your Arduino `libraries` folder.
- **WebSockets** by Markus Sattler: install it from Library Manager (tag only).
- Board: ESP32 Dev Module.

### 2. Flash the anchors

Open `anchor.ino`, set the ID line, and upload. Repeat for each board:

```cpp
#define ANCHOR_ID 1    // 1, 2, 3 or 4, never the same on two boards
```

Label each board with its ID.

### 3. Flash the tag

Open `tag.ino` and upload. Keep `#define USE_AP true` so the tag makes its own Wi-Fi network (`UWB-TAG`, password `uwb12345`).

Antenna delay values (`TX_ANT_DLY`, `RX_ANT_DLY`) must be identical in all five sketches.

### 4. Mount the anchors

Height accuracy depends on the layout. Put two anchors **high** (about 2.2 m or more) and two **low** (about 0.3 m), on opposite corners:

| Anchor | Corner | Height |
|---|---|---|
| 1 | front-left | high |
| 2 | front-right | low |
| 3 | back-right | high |
| 4 | back-left | low |

- Point each antenna into the room, with no metal right behind it.
- Keep a clear line of sight to where the tag will be.
- If all four are at similar heights, a tag on the floor and one near the ceiling look the same to the system, and the height will be wrong.

### 5. Run the viewer

1. Power the anchors and the tag.
2. Join the Wi-Fi network **UWB-TAG** on your computer.
3. Open `visualizer3d.html` by double-clicking it (Chrome or Edge).
4. Enter your anchors in the table: x and y on the floor, and antenna height, all in metres from one floor corner.
5. Set **Max tag height** to about 1.8.
6. Leave the address at `192.168.4.1` and click **Connect WiFi**.

Click **Demo** at any time to see the viewer with a simulated tag and no hardware.

The tag appears once all four anchor rows show distances.

## Calibration

Each board adds a small error to its distances. To correct it:

1. Hold the tag so its antenna is exactly 0.50 m from one anchor's antenna.
2. Read that anchor's distance in the right-hand panel.
3. Type the difference into that anchor's **offset** box (for example `0.12` if it reads 0.62).
4. Repeat for all four anchors.

Then check:

- Tag on the floor reads about 0 to 0.2 m high.
- Tag held at 1 m reads about 1 m.
- **Fit error** stays under about 10 cm while the tag is still.

## Troubleshooting

| Problem | Likely cause |
|---|---|
| Can't connect to the tag | PC isn't on the `UWB-TAG` network, or `USE_AP` is `false`. Test `http://192.168.4.1:81` in the browser. |
| Page is blank or buttons do nothing | Press F12 and read the Console. Open the file you edited, not an older copy. |
| Tag never appears | One of the four anchors isn't answering. Check power, ID and line of sight. |
| Many missed readings | Remove debug `Serial.print` lines from the anchors, check for blocked line of sight, and avoid power banks that switch off at low current. |
| Height wrong or tag shown near the ceiling | Anchors at similar heights. Move two high and two low, and lower the max tag height. |
| Position jitters | Normal UWB noise. The viewer smooths it, and the tag still reads steadier when it is not moving. |

## Limits

- UWB height is less accurate than floor position, often 20 to 50 cm off. Fall detection relies on a fast drop and a low final height, not an exact number.
- The viewer needs all four anchors. With fewer, the position is paused.
- For more reliable fall detection, add an accelerometer or a barometer to the tag.

## Settings in the viewer

| Setting | Meaning |
|---|---|
| Anchor x, y, height | Where each anchor is, in metres. |
| Offset | How much too long that anchor reads. |
| Fall alert: height below | Height at which a low tag counts as fallen. |
| Max tag height | Highest height the solver may report. It rules out mirrored answers. |