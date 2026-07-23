# DuckyGrab

Windows 11/10 browser data exfiltration via USB Rubber Ducky.
Copies raw database files from Chrome, Edge, Brave, Firefox to DUCKY storage.
Uses CMD batch to bypass AMSI.

## Files

```
DUCKY:/
├── inject.bin   ← compiled from payload.txt via PayloadStudio
├── s.dat        ← batch script 
```

## Setup

1. Open [PayloadStudio](https://payloadstudio.hak5.org), paste `payload.txt`, compile, download `inject.bin`
2. Copy `inject.bin` and `s.dat` to DUCKY root
3. Unplug DUCKY, plug into target, wait ~30 seconds, unplug

## Flow

```
Plug in → ATTACKMODE HID STORAGE → Win+R → cmd /c ...
→ Find drive containing s.dat (loop A-Z)
→ Copy s.dat to %tmp%\r.bat → start /min
→ r.bat copies browser files to DUCKY:\loot\
→ r.bat self-deletes
```

## Loot

```
DUCKY:\loot\<COMPUTERNAME>-<USERNAME>\
├── Chrome\        Local State, Default\{Login Data, Cookies, History, Bookmarks, Web Data}
├── Edge\          (same)
├── Brave\         (same)
├── Firefox\       {logins.json, key4.db, cookies.sqlite, places.sqlite, cert9.db}
└── sysinfo.txt
```

Multi-profile (Profile 1, Profile 2, ...) collected automatically.

## Config

Edit `payload.txt` before compiling:

| Define | Default | Purpose |
|--------|---------|---------|
| `#BOOT_DELAY` | `3000` | Delay before keystroke injection (ms) |

## Decrypt

Chromium: requires `Login Data` + `Local State` (DPAPI + AES-256-GCM). Decrypt with [HackBrowserData](https://github.com/moond4rk/HackBrowserData) or `sqlite3` + `pycryptodome`. Must run on the same user account or with the DPAPI master key.
Firefox: requires `logins.json` + `key4.db`. Decrypt with [firefox_decrypt](https://github.com/unode/firefox_decrypt/).

## Notes
- No admin/UAC required
- Browsers can stay open
- No PowerShell = no AMSI detection
- CMD window flash <1 second, batch runs minimized
- `s.dat` neutral extension, not flagged by Defender as a script
- `ATTACKMODE HID STORAGE` keeps the drive accessible — replace inject.bin without removing MicroSD
