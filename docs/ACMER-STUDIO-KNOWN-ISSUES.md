# ACMER Studio — Known Issues

> Reference material for bugs already investigated. Not part of the
> installation recipe — see `ACMER-STUDIO-BOTTLES-MANUAL.md` for the
> step-by-step.

---

## 1. Keyboard in G-code Export Dialog (resolved)

**Symptom**: when opening the G-code export dialog, you can't type the file
name; the keyboard doesn't work and the main window "loses all functionality"
(click, but nothing responds).

### 1.1. Log Facts (run 2026-08-09)

| Finding | Reading |
|---|---|
| App is **Electron + Node** (errors `network_change_notifier_win.cc`, JS renderer logs, IPC `msg { type: 'init', ... }`) | Keyboard input goes through Chromium; native dialogs go to Wine's comdlg32 |
| Corrupted Chinese text (mojibake GBK) in logs | ACMER app in Chinese; Chromium IME/keyboard is sensitive to this |
| `DXVK: v3.0.2` + `AMD Radeon Graphics (RADV RENOIR)` | AMD iGPU (Ryzen 4000 APU); render via Vulkan/DXVK |
| Second DXVK init at end of log (`03b4:... Game: ACMER Studio.exe`) | Likely Chromium GPU process restart at dialog time |
| `opencv_*3415.dll`, `ceres.dll`, `ortools.dll`, `libprotobuf.dll`, `tinyxml2.dll`, `onnxruntime.dll`, `libsodium.dll` not found | **Calibrate**, **PathOpt**, and **BatchDup (smart autofill)** tools don't load — not the keyboard problem |
| `libvkd3d-utils-1.dll` / `wined3d.dll` not found | DXCore fallback noise; render is already via DXVK. Cosmetic |
| `WSALookupServiceBegin failed: 8` | App trying network; harmless |

### 1.2. Diagnosis

**Evidence in the app (app.asar, 2026-08-09)**: G-code export and
save/save-as use Electron's `dialog.showSaveDialog` → on Windows this opens
the **native Win32 dialog** (comdlg32 → `GetSaveFileNameW`), a separate modal
window, owner of the main window.

The symptom (main window disabled + name field without keyboard) is classic
Wine behavior in **unmanaged mode** ("Allow the window manager to control the
windows" OFF — required for the app to start): Wine handles X11 focus alone
and **modal dialogs don't receive keyboard focus** — keystrokes don't reach
the field; the owner window gets disabled by modality (that's why "click but
nothing works"). Documented Wine bug for years, with the SAME config combo
(decorate OFF + control OFF + no virtual desktop). GE-Proton makes it worse:
it carries Valve's focus patch that "may break modal dialogs" (commit
`d30ce49`).

It's not a Chromium crash — it's the modal waiting for input that never
arrives (confirm with Esc: if it closes, it's focus, not hang).

### 1.2.1. Fixes (prioritized)

**✅ Resolved on 2026-08-09: virtual desktop.** `winecfg` → Graphics →
**"Emulate a virtual desktop"** (1920×1080) — tested and working: the app
starts, the G-code export dialog receives keyboard, and the file name is
editable. Wine becomes the owner of all windows and controls focus
internally; the WM never touches the windows — fixes both sides (app starts
without WM interference; modal receives keyboard).

Other tested/manual options (kept as reference, not used):

1. **Click the name field before typing** (zero cost): Electron/Chromium modal
   dialogs sometimes only receive keyboard after a click
   (electron/electron#42948).
2. **sys-wine runner** (without Proton focus patches): swap GE-Proton11-3 for
   sys-wine/kron4ek (already installed in Bottles) and revalidate app +
   dialog.
3. **`ELECTRON_DISABLE_GPU=1`** in the bottle: only if the freeze seems like
   a renderer hang after the above tests.
