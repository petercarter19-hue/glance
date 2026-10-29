# Glance

**Your PC's memory use, right by the clock.** Hover it to see which apps are using the most, and end a greedy one in two clicks, without opening Task Manager.

![The Glance number on the taskbar with its card open: memory and CPU meters and the top ten apps with End buttons](docs/glance.png)

- **A number by the clock** shows how full your memory is. Green under 60%, amber to 85%, red above. When memory crosses 90%, the tile pulses red once. That's all the nagging it does.
- **Hover the number** and a card slides up: memory and CPU meters, the app that grew the most in the last 10 minutes, and the top ten apps. Each app is one row, the way Task Manager groups them, so your browser is one line, not forty.
- **End takes two clicks.** The first click turns the button into a red "End?"; the second ends the app. Move away or wait a few seconds and it cancels. Windows' own parts show a lock instead of a button, and Windows Explorer (your taskbar and desktop) gets "Restart" instead of "End".
- **Free, small, private.** One file, no install, no account. It reads your PC's own numbers and never sends anything anywhere.

## Get it

1. Download **Glance.exe** from the [latest release](https://github.com/petercarter19-hue/glance/releases/latest).
2. Double-click it. A notice says Glance is running, and the number appears by the clock.
   - If Windows says **"Windows protected your PC"**, click **More info**, then **Run anyway**. Windows shows this for apps from small developers that aren't code-signed. The code is all here to read.
   - If your browser says the file "isn't commonly downloaded", choose **Keep**.
3. **Don't see the number?** Windows tucks new icons under the little **^** arrow by the clock. Click the **^** and drag Glance onto the taskbar. It stays there after that.

Glance starts with Windows, and it's in your Start menu as **Glance**.

**To restart it,** open it again, from the Start menu or by double-clicking Glance.exe. The running copy closes and a fresh one takes its place.

## Using it

- **Hover** the number to open the card; move away to close it.
- **Click** the number to pin the card open; press Esc or click anywhere else to close it.
- **Right-click** the number for the menu:
  - **Show CPU tile:** adds a second number for CPU use. Hovering the CPU tile sorts the list by CPU.
  - **Start with Windows:** on by default.
  - **Open Task Manager**, **About Glance**.
  - **Remove Glance from this PC:** see below.
  - **Quit Glance.**
- **If Windows won't end an app** (one running as administrator), the button offers **As admin**. Windows asks your permission first. If it still refuses, the card says so.

End works like Task Manager's "End task": anything unsaved in that app is lost. Save your work first, and only end apps you recognize.

## Remove it

Right-click the number and choose **Remove Glance from this PC**. That turns off Start with Windows, removes it from the Start menu, deletes its settings, and closes it. Then delete Glance.exe.

## Requirements

Windows 11 or Windows 10, 64-bit. Nothing else to install. The file is about 49 MB, because it carries everything it needs inside it.

## Privacy

Glance only reads numbers Windows already has (the same ones Task Manager shows). It makes no network connections and collects nothing. Its settings and an error log, if anything goes wrong, live in `%LocalAppData%\Glance`.

## Build it yourself

Needs the [.NET 10 SDK](https://dotnet.microsoft.com/download).

```
dotnet publish Glance.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -p:EnableCompressionInSingleFile=true -o dist
```

`tools\make-icon.ps1` redraws `glance.ico`.

### Tests

- `tools\smoke-test.ps1` runs the app against a test program that holds 3 GB of memory (`tools\Hog`; publish it first with the same command into `tools\Hog\dist`), posts two End clicks to the card, and checks the test program is gone.
- `tools\message-test.ps1` posts the shell's own hover and click messages to the tray window and checks the card opens, pins and closes.
- Both drop the app's own pictures of itself into `tools\shots\`.

Debug flags: `--preview` writes the card's state and pictures (`preview.txt`, `card.png`, `tiles.png`) to `%LocalAppData%\Glance` every second; `--pinned` opens the card pinned at startup; `--quit` closes the running copy.

## How it works

C# / .NET 10 WinForms, no main window.

- **`TrayIcon`** talks to `Shell_NotifyIcon` directly at version 4, so the shell sends `NIN_POPUPOPEN` on hover. That is what lets the card replace the stock tooltip.
- **`ProcessScanner`** snapshots processes with Toolhelp, reads each one's private working set (the same number Task Manager shows) and CPU time, and groups them by executable.
- **`ProcessKiller`** ends a group plus the processes it started:
  - It stays inside the user's own session.
  - A process only counts as a child if it started after its parent, because Windows reuses process numbers.
  - Each process is checked by its start time right before it's ended.
  - Windows' protected processes and Explorer are never swept in.
  - **As admin** runs a second copy of Glance elevated with `--end`, which makes the same checks.
- **`HoverCard`** is a borderless, no-activate, custom-painted window that follows the Windows light or dark setting.

Opening Glance while it's running signals the running copy to quit (a named event) and takes over the single-instance lock, which is how "open it again to restart" works.

## License

[MIT](LICENSE)
