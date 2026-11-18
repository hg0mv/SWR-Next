# SWR Next

**SWR curve meter for the Yaesu FT-891** – Linux and Windows. Optional Hamlib support for other radios (Linux).

SWR Next steps across a band, sends a short low-power carrier at every point, reads the radio's SWR meter over CAT and draws the SWR curve of your antenna. When it is done it restores the radio's frequency, mode and power.

> **Beta release.** Please read [Safety](#safety) before the first sweep.

![Graph view](docs/screenshot-graph.png)

## Features

- SWR curve for one band, or **all bands with one click** with a summary table
- Best SWR and frequency, 2:1 and 1.5:1 bandwidth, and the **trend** (is the resonance above or below the band?)
- The previous sweep stays on the graph in grey: handy while trimming an antenna
- **Calibration wizard** with known loads (50, 25, 100, 150 … ohm)
- Stepped curve with a shaded band showing the resolution of the radio's meter
- Instrument-style display: big 7-segment readout and an analogue meter
- CSV export
- English and Hungarian user interface
- Optional **Hamlib** backend for other radios that report SWR (Linux build)

![Summary view](docs/screenshot-summary.png)

## Download and run

### Windows

1. Unzip `swrnext-0.5-beta-windows-x64.zip` anywhere.
2. Connect the FT-891 by USB. In **Device Manager → Ports (COM & LPT)** you will see two *Silicon Labs CP210x* ports. The **Enhanced COM Port** is the CAT port; note its number (for example `COM3`).
3. Start `swrnext.exe` once and close it. This creates `%APPDATA%\swrnext\swrnext.conf`.
4. Edit the config: set `port=COM3` (your number) and `baud=` to the radio's CAT RATE.
5. Start the program again.

If Windows does not show the ports, install the Silicon Labs CP210x driver from Yaesu or Silicon Labs.

### Linux

```
tar xzf swrnext-0.5-beta-linux-x86_64.tar.gz
cd swrnext-0.5-beta-linux-x86_64
./swrnext --demo        # try it without a radio
./swrnext
```

The settings are in `~/.config/swrnext/swrnext.conf`. The FT-891's Enhanced port is usually `/dev/ttyUSB0`. Your user must be in the `dialout` group: `sudo usermod -aG dialout $USER`, then log in again.

## Radio setup (FT-891)

| Menu | Setting |
|---|---|
| 05-06 CAT RATE | same as `baud=` in the config (default 38400) |
| 05-07 CAT TOT | 100 msec or longer |
| 05-08 CAT RTS | ENABLE |

Turn off any antenna tuner before you measure, otherwise you measure the tuner, not the antenna. An external tuner controlled by CAT (FC-50) is switched off automatically.

## Using it

- Pick a band on the left, then **Measure** (Space key). **All bands** (A key) measures every band and opens the summary.
- Move the mouse over the graph to read a point.
- **Summary** shows all bands; click a row to open its graph.
- **Save CSV** saves the current band, or all bands from the summary view (Linux: home folder, Windows: Documents).
- **Esc** stops a sweep, **Q** quits.

## Safety

- The program transmits a short carrier on every point (default **5 W, FM**). Check that the frequency is clear first.
- **In FM and AM mode the microphone is live.** Keep quiet during a sweep or unplug the microphone; CAT TX works without it.
- The power is set *after* each band change and read back. If the radio does not confirm the power, the sweep stops.
- One carrier lasts at most 2 seconds; then TX is switched off whatever happens.
- If the output power suddenly jumps, the sweep stops.
- At the end – and after Stop, closing the window or an error – the original frequency, mode, power, repeater shift and tuner state of every band that was touched are restored.

## Calibration

The FT-891 reports its SWR meter as a raw value from 0 to 255, in steps of about 13. The default conversion table is an **estimate**. For correct values, open **Calibrate**, connect known loads and click their buttons:

| Load | SWR |
|---|---|
| 50 ohm dummy load | 1.0 |
| 25 ohm (two 50 ohm loads in parallel) or 100 ohm | 2.0 |
| 16.7 ohm (three 50 ohm in parallel) or 150 ohm | 3.0 |

Use non-inductive resistors rated for at least 5 W. **Save calibration** writes the table to the config file, and all existing curves are recalculated.

The resolution of the radio's meter cannot be improved by software. The shaded band around the curve shows it. For precise values use an antenna analyser.

## Settings

| Key | Default | Meaning |
|---|---|---|
| `port` | `/dev/ttyUSB0` / `COM3` | CAT port |
| `baud` | 38400 | CAT speed |
| `power` | 5 | measuring power in watts (5–100) |
| `mode` | FM | carrier mode: FM, AM, RTTY, CW |
| `points` | 41 | points per band |
| `settle_ms` | 200 | wait after TX on, before reading |
| `off_ms` | 250 | pause between points |
| `lang` | en | `en` or `hu` |
| `backend` | native | `native` (FT-891 CAT) or `hamlib` |
| `rig_model` | 1036 | Hamlib model number (`rigctl -l`) |
| `display` | auto | `auto`, `steps` or `line` |
| `meter_step` | 13 | meter step size in raw units |
| `band=` | | custom band, e.g. `band=40,7000000,7200000` |
| `cal=` | | calibration point `raw,swr` (written by the wizard) |

Command line: `-p PORT`, `-b BAUD`, `-w WATT`, `-n POINTS`, `--demo`.

## Building from source

### Linux

```
sudo apt install build-essential libx11-dev
make
```

With Hamlib (other radios):

```
sudo apt install libhamlib-dev
make clean && make HAMLIB=1
```

Then set `backend=hamlib` and `rig_model=` in the config. With `rig_model=2` and `port=localhost:4532` it talks to a running `rigctld`.

### Windows (cross-compiled on Ubuntu)

```
sudo apt install mingw-w64 zip
make windows
```

This builds a single static `swrnext.exe` with icon and version info. You can test it under Wine; map the radio port first:

```
ln -s /dev/ttyUSB0 ~/.wine/dosdevices/com3
wine swrnext.exe
```

### Release packages

```
make dist
```

This creates the Linux and Windows packages, the source package and `SHA256SUMS.txt` in `dist/`.

## Source layout

| File | Contents |
|---|---|
| `swrnext.c` | measuring, radio control, user interface (platform independent) |
| `platform.h` | platform layer interface |
| `plat_x11.c` | Linux: X11 window, termios serial port |
| `plat_win32.c` | Windows: Win32 GDI window, COM port |

## Known limitations (beta)

- The default SWR table is an estimate until you calibrate.
- The FT-891 meter has about 20 steps, so the curve is stepped.
- Hamlib support is only available in the Linux build.
- The Windows version is new in this release; please report problems.

## License

SWR Next is free software, licensed under the **GNU General Public License v3.0 or later** (GPL-3.0-or-later). See [LICENSE](LICENSE).

This program comes with ABSOLUTELY NO WARRANTY. You transmit with your own radio and are responsible for its use.
