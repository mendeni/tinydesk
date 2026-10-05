# TinyDesk developer preview

A mouse-operated desktop running on an ESP32, displayed in your terminal.
Drag and resize windows, browse files, edit and save text, and use the
embedded shell. MQTT and Modbus apps connect the desktop to real projects.

This release is a developer preview: expect rough edges, and please report
what breaks.

## New in 0.1.4

* **The first update you can install from Software Update:** boards
  running 0.1.3 (ESP32-C6, ESP32 with PSRAM) find 0.1.4 with *Check for
  official updates*, or tell you about it within a day.
* **Your pins survive an official update.** If you built the firmware
  with your own `board.conf`, its pins are built in, and an official
  update (built without them) would start without Ethernet, SD card or
  RS-485. Software Update now asks first: *Save and install* copies them
  to `/etc/board.conf` (as `board save` does in the shell); `ota install`
  refuses until they are saved.
* **Non-ASCII names line up and stay whole:** Wi-Fi networks, files and
  window titles with characters such as é, ☕ or 中 no longer push the
  columns after them out of line (the signal bars overlapped the name),
  and desktop icon labels, titles and dialogs never cut a character in
  half.
* **Long command lines** in the Terminal (and over SSH and serial)
  wrap over several rows and stay editable; before, text was left behind
  or doubled.
* **Editor:** a file keeps its name when saved (on Linux the second Ctrl+S
  asked for a name). **`mqtt status`** names the TLS handshake. Both are
  contributions, with new tests for the widgets and the Editor.
* **For contributors and people who build from source:** the ESP code
  shared by the boards is in `ports/esp_idf`; the C sources are formatted
  with clang-format 16, with a pre-commit hook (`pip install pre-commit`,
  then `pre-commit install`); see `CONTRIBUTING.md`.

Also since 0.1.0: official updates in Software Update and fixes for
boards without PSRAM (0.1.3), the SD card at `/sd` and in Files (0.1.2),
`ping -c`, MQTT over TLS in the PC programs, and `tdsh.exe` for Windows.

## Install

* **ESP boards, from the browser:** <https://schikani.github.io/tinydesk-docs/install/>
  (Chrome or Edge on a computer). Pick the edition and your exact board.
* **Open the board in the browser:** <https://schikani.github.io/tinydesk-docs/console/>
  (a real terminal with mouse support; the installer's own Logs & Console
  cannot show the desktop).
* **Without the browser:** each `*-factory.bin` below is flashed at offset 0
  with esptool; see the installer page for the commands.
* **PC programs:** `tinydesk-desktop-windows-x64.zip` and
  `tinydesk-shell-windows-x64.zip` (Windows 10 or later, run them in Windows
  Terminal), `tinydesk-desktop-linux-x86_64.tar.gz` and
  `tinydesk-shell-linux-x86_64.tar.gz` (x86_64, built on Ubuntu 22.04).

Check the files against `SHA256SUMS.txt`. The firmware is not signed.
Factory flashing can erase user data: back up an existing board first.

## Before connecting

Use a UTF-8 terminal with ANSI/VT cursor control and xterm mouse reporting,
or the web terminal above. ESP32-C6 uses its built-in USB port (any speed);
the classic ESP32 Desktop edition uses 921600 baud, the Shell edition
115200. On the local console, run `passwd` as root (initial password:
`TinyDesk`). Remote access is blocked until that password changes. Telnet
starts disabled; enable it explicitly in Network only on a trusted network.
It is unencrypted and root-only because it takes over the existing desktop.
SSH provides a separate encrypted shell, not the windowed desktop (the 4 MB
ESP32 Desktop build has no SSH server).

Physical password recovery is unavailable after a Telnet takeover until the
board reboots. Remote takeover is refused while physical recovery is active.

## Choose the exact board

| Desktop target | Required hardware | Screen | Limits |
| --- | --- | --- | --- |
| ESP32-C6 | 8 MB flash, built-in USB Serial/JTAG | up to 80×25 | |
| ESP32 with PSRAM | 16 MB flash plus PSRAM (e.g. WROVER-IE N16R8) | up to 256×96 | |
| ESP32 4 MB | 4 MB flash, no PSRAM needed (e.g. WROOM-32) | up to 80×25 | no SSH server, no over-the-air updates |
| Windows, Linux | x86_64 PC | up to 400×150 | no Wi-Fi, OTA or users (the PC's network is used) |

Measured on our boards with this firmware (About → Free RAM, idle desktop
after start-up, Wi-Fi connected unless noted):

| Board | Firmware size (app slot) | Free RAM |
| --- | --- | --- |
| ESP32-C6 | 1.83 MB (2.5 MB slot, 27 % free) | about 167 KB |
| ESP32 with PSRAM | 1.73 MB (3 MB slot, 42 % free) | about 155 KB internal + 3.2 MB PSRAM |
| ESP32 4 MB | 1.66 MB (2.5 MB slot, 34 % free) | about 96 KB, Wi-Fi not connected |

## What is in this preview

- Desktop: overlapping windows (move, resize, minimise, maximise, full
  screen), taskbar, start menu, desktop icons, drag and drop, copy and paste
  with the PC; Dark theme by default; the screen follows the terminal's size.
- Apps: Terminal (TinyDesk Shell), Files, Editor, Network, MQTT (TLS),
  Modbus TCP/RTU with a TCP server, System Monitor, Task Manager, Log Viewer,
  Settings, Software Update (OTA with rollback), About.
- Shell scripts (`.tdsh`): variables, `if`/`while`/`for`, functions, pipes
  and redirection; right-click → Run on the desktop.
- Two editions: TinyDesk Desktop, and TinyDesk Shell on its own.
- Security: physical recovery uses a transport trust policy, not just shell
  task identity; Telnet is opt-in; Telnet, SSH and FTP reject the factory
  password.
- PC programs: the Windows desktop follows Windows Terminal when it is
  resized or maximised, and its shell has `ifconfig`, `ping`, `date`, `cal`
  and `tz`.
- Board pins come from a configuration file (`board` command), not from
  the code.
- Tests gate release builds; every release starts as a draft pre-release.
