# Ultimate Windows & Browser Shortcuts Handbook (2026 Edition)

*A complete, beginner-to-expert reference for keyboard shortcuts across Windows 10/11, File Explorer, browsers (Chrome, Edge, Firefox, Brave, Opera), VS Code, Git, terminals, Microsoft Office, Google Workspace, and everyday web tools.*

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [How Keyboard Shortcuts Work](#2-how-keyboard-shortcuts-work)
3. [Modifier Keys Explained](#3-modifier-keys-explained)
4. [Keyboard Layout](#4-keyboard-layout)
5. [Browser Shortcuts](#5-browser-shortcuts)
6. [Windows Shortcuts](#6-windows-shortcuts)
7. [File Explorer](#7-file-explorer)
8. [Text Editing](#8-text-editing)
9. [Mouse Shortcuts](#9-mouse-shortcuts)
10. [Clipboard](#10-clipboard)
11. [Screenshots](#11-screenshots)
12. [Virtual Desktops](#12-virtual-desktops)
13. [Taskbar](#13-taskbar)
14. [Notifications](#14-notifications)
15. [Search](#15-search)
16. [Settings](#16-settings)
17. [Accessibility](#17-accessibility)
18. [Developer Shortcuts](#18-developer-shortcuts)
19. [Microsoft Office](#19-microsoft-office)
20. [Google Docs, Sheets, Slides & Drive](#20-google-docs-sheets-slides--drive)
21. [Gmail](#21-gmail)
22. [YouTube](#22-youtube)
23. [Google Search Tricks](#23-google-search-tricks)
24. [PDF Viewers](#24-pdf-viewers)
25. [Terminal & PowerShell](#25-terminal--powershell)
26. [Command Prompt](#26-command-prompt)
27. [Windows Terminal](#27-windows-terminal)
28. [GitHub](#28-github)
29. [VS Code](#29-vs-code)
30. [Git](#30-git-command-line)
31. [Chrome DevTools](#31-chrome-devtools)
32. [Edge DevTools](#32-edge-devtools)
33. [Firefox Developer Tools](#33-firefox-developer-tools)
34. [React Developer Workflow](#34-react-developer-workflow)
35. [Node.js Workflow](#35-nodejs-workflow)
36. [Daily Productivity Workflow](#36-daily-productivity-workflow)
37. [Shortcut Cheat Sheets](#37-shortcut-cheat-sheets)
38. [Memory Techniques](#38-memory-techniques)
39. [Practice Exercises](#39-practice-exercises)
40. [Printable Reference & Index](#40-printable-reference--index)

> **A note on scope:** This handbook is intentionally dense — tables over paragraphs, examples over theory. Read it top to bottom once, then keep it as a reference you dip into. Every shortcut lists **what it does**, **where it works**, and **why it's worth learning**.

---

# 1. Introduction

Keyboard shortcuts are the single highest-leverage productivity skill on a computer. A single shortcut might save 2–3 seconds. That sounds trivial — until you realize the average knowledge worker performs the same handful of actions (switch window, copy, paste, save, close tab, find) **hundreds of times per day**.

**The math that matters:**

| Action | Mouse time | Keyboard time | Saved per use | Uses/day | Saved/day |
|---|---|---|---|---|---|
| Switch app | ~2.5s | ~0.4s (Alt+Tab) | 2.1s | 40 | 84s |
| Copy/Paste | ~3s | ~0.6s (Ctrl+C/V) | 2.4s | 60 | 144s |
| Close tab | ~1.5s | ~0.3s (Ctrl+W) | 1.2s | 30 | 36s |
| Save file | ~2s | ~0.3s (Ctrl+S) | 1.7s | 25 | 42.5s |
| New tab | ~1.5s | ~0.3s (Ctrl+T) | 1.2s | 25 | 30s |

That's roughly **6 minutes a day** from five shortcuts alone. Multiply by the 100+ shortcuts in this book and power users routinely reclaim **30–60 minutes per day** — nearly a full workday every two weeks — without moving faster, just moving *less*.

This handbook is organized so you can:

- **Learn systematically** (sections 2–17: foundations)
- **Go deep on developer tools** (sections 18, 25–35)
- **Master office & web apps** (sections 19–24)
- **Build daily habits** (section 36)
- **Practice deliberately** (sections 38–39)
- **Reference quickly** (sections 37, 40)

### Who this is for

- **Beginners** who've never used a shortcut beyond Ctrl+C/Ctrl+V
- **Office professionals** living in Word, Excel, Outlook, and Gmail
- **Power users** who want fewer trips to the mouse
- **Developers** working across VS Code, terminals, Git, and browser DevTools
- **Designers and researchers** who multitask across dozens of tabs and windows
- **System administrators** managing Windows at scale

### How to use this book

1. Skim your relevant sections first (e.g., a developer jumps to 18, 25–35).
2. Pick **5 new shortcuts a week** — no more. Muscle memory beats memorization.
3. Use the **Practice Exercises** (Section 39) to drill them.
4. Print the **Cheat Sheets** (Section 37) and keep one taped near your monitor for the first month.

---

# 2. How Keyboard Shortcuts Work

A keyboard shortcut is a combination of one or more **modifier keys** (Ctrl, Alt, Shift, Win) held down together with a **regular key** (a letter, number, function key, or symbol) that triggers an action instantly, bypassing menus.

### The anatomy of a shortcut

```
Ctrl        +        Shift        +        T
(modifier)      (modifier)              (action key)
```

- **Order of pressing:** Hold modifiers first, then tap the action key, then release everything.
- **Chords vs. combos:** Most Windows/browser shortcuts are simultaneous combos (Ctrl+C). Some apps (like VS Code) use **chords** — sequential key groups, e.g., `Ctrl+K` then `Ctrl+S` (press, release, press again).
- **Case sensitivity:** Shortcuts are almost never case-sensitive by letter, but Shift is often literally part of the combo (Ctrl+Shift+T ≠ Ctrl+T).

### Why shortcuts feel hard at first — and stop being hard

Your brain builds a shortcut as a habit only after roughly **20–40 correct repetitions**. The discomfort in week one is normal; by week three, common shortcuts become faster than *thinking about* using the mouse.

### Common mistakes beginners make

| Mistake | Fix |
|---|---|
| Pressing the letter before the modifier | Always press and hold the modifier(s) first |
| Trying to learn 20 shortcuts at once | Learn 5 per week, drill them daily |
| Giving up after one failed attempt | Shortcuts often need muscle-memory repetition; try 10 times |
| Not knowing which app has focus | Shortcuts apply to the **focused window** — click or Alt+Tab first |
| Confusing OS shortcuts with app shortcuts | Win+key = Windows OS; Ctrl+key is usually app-specific |

---

# 3. Modifier Keys Explained

| Key | Location | Primary Role | Memory Trick |
|---|---|---|---|
| **Ctrl** (Control) | Bottom-left/right of keyboard | Application-level commands (copy, save, find) | "**C**trl **c**ontrols the app" |
| **Alt** (Alternate) | Beside spacebar | Menu access, window switching, "alternate" actions | "**Alt**ernate view/menu" |
| **Shift** | Above Ctrl | Capitalization, extending selections, reversing direction | "**Shift** the selection/extend it" |
| **Win** (Windows key) | Between Ctrl and Alt | Operating-system-level features | "**Win**dows OS commands" |
| **Fn** (Function) | Laptops only | Toggles function-key behavior (brightness, volume vs F1–F12) | "**F**unction toggle" |

### How they combine

- **Ctrl** = "do this to the document/app" → Ctrl+S (save), Ctrl+Z (undo)
- **Alt** = "switch/alternate context" → Alt+Tab (switch app), Alt+F4 (close app)
- **Shift** = "extend/reverse/capitalize" → Shift+Click (select range), Ctrl+Shift+T (reverse of Ctrl+T)
- **Win** = "control Windows itself" → Win+D (desktop), Win+L (lock)

### Layered combos (three modifiers)

Some power shortcuts stack three keys, e.g. **Ctrl+Shift+Esc** (Task Manager directly) or **Ctrl+Shift+N** (new Incognito/InPrivate window). The rule of thumb: **more modifiers = more powerful/destructive/rare action**, so Windows intentionally makes dangerous or advanced actions require more keys.

---

# 4. Keyboard Layout

Understanding physical zones speeds up learning.

```
Esc  F1 F2 F3 F4  F5 F6 F7 F8  F9 F10 F11 F12         [PrtScn][ScrLk][Pause]
`  1 2 3 4 5 6 7 8 9 0 - =  Backspace        [Ins][Home][PgUp]
Tab  Q W E R T Y U I O P [ ] \                [Del][End][PgDn]
Caps  A S D F G H J K L ; '  Enter
Shift  Z X C V B N M , . /  Shift                  [ ↑ ]
Ctrl  Win  Alt      Space      Alt  Win  Menu  Ctrl  [←][↓][→]
```

### Key zones to memorize

- **Function row (F1–F12):** App-specific actions (F2 rename, F5 refresh, F11 fullscreen, F12 DevTools)
- **Navigation cluster (Insert/Home/PgUp/Delete/End/PgDn):** Cursor and page movement
- **Arrow cluster:** Directional movement, often combined with Ctrl/Shift for word/paragraph jumps
- **Number row symbols:** Many shortcuts use Shift + number (e.g., Ctrl+Shift+1 for Heading 1 in Word/Docs)

### Finger placement tip

Keep your left pinky near **Ctrl/Shift**, thumb near **Space/Alt**, and right hand free for the letter or arrow key. This lets you fire most shortcuts without looking down.

---

# 5. Browser Shortcuts

Applies to **Chrome, Edge, Brave, and Opera** (all Chromium-based, so ~95% identical) with **Firefox** differences called out. Works on **Windows 10/11**.

> **Chromium note:** Chrome, Edge, Brave, and Opera share the Chromium engine, so nearly every shortcut below is identical across all four. Where Firefox differs, it's noted in its own column.

## 5.1 Tab Management

| Shortcut | Description | Chrome/Edge/Brave/Opera | Firefox | Example use |
|---|---|---|---|---|
| Ctrl+T | Open new tab | ✅ | ✅ | Start a new research thread |
| Ctrl+Shift+T | Reopen last closed tab | ✅ | ✅ (multi-press cycles history) | Undo an accidental close |
| Ctrl+W or Ctrl+F4 | Close current tab | ✅ | ✅ | Clean up clutter |
| Ctrl+Tab | Next tab | ✅ | ✅ | Cycle right through tabs |
| Ctrl+Shift+Tab | Previous tab | ✅ | ✅ | Cycle left through tabs |
| Ctrl+1 to Ctrl+8 | Jump to tab # 1–8 | ✅ | ✅ | Instantly jump to tab 3 |
| Ctrl+9 | Jump to **last** tab | ✅ | ✅ | Always the last, not tab 9 |
| Ctrl+Shift+A | Search open tabs | ✅ | — | Find a tab among 40 open |
| Alt+Shift+T (Brave) | Toggle tab search | Brave only | — | Brave-specific enhancement |

**Memory trick:** T = **T**ab, W = "**W**ave goodbye" to a tab, Ctrl+Shift+T = "bring the **T**ab back."

**Time saved:** Using Ctrl+1–8 instead of clicking saves ~1.5s per switch; at 50 switches/day that's ~75s daily, ~6+ minutes a week.

## 5.2 Window Management

| Shortcut | Description | Works in |
|---|---|---|
| Ctrl+N | New window | All Chromium + Firefox |
| Ctrl+Shift+N | New Incognito/InPrivate window (Chrome/Edge) | Chrome, Edge, Brave |
| Ctrl+Shift+P | New Private window (Firefox) / New Incognito (Opera uses Ctrl+Shift+N) | Firefox, Opera |
| Alt+F4 | Close entire browser window | Windows-wide |
| Win+Up / Win+Down | Maximize / restore window | Windows 10/11 |
| Win+Left / Win+Right | Snap window to half screen | Windows 10/11 |
| F11 | Toggle fullscreen | All browsers |

**Common mistake:** Confusing Ctrl+W (close tab) with Ctrl+Shift+W or Alt+F4 (close whole window) — losing every open tab by accident. If this happens, immediately use **Ctrl+Shift+T** to restore the session.

## 5.3 Address Bar & Navigation

| Shortcut | Description | Notes |
|---|---|---|
| Ctrl+L or Alt+D or F6 | Focus address bar | Type immediately after |
| Ctrl+Enter | Auto-complete `.com` around typed text | "example" → "www.example.com" |
| Alt+Enter | Open typed address/search in a **new tab** | Keeps current page open |
| Alt+Left / Alt+Right | Back / Forward | Also works with mouse side buttons |
| Backspace (Firefox only) | Back | Disabled by default in Chrome |
| Ctrl+R or F5 | Reload page | Standard refresh |
| Ctrl+Shift+R or Shift+F5 | Hard refresh (bypass cache) | Use when a page shows stale content |
| Esc | Stop page load | Cancels a hanging load |

**Example:** Debugging a CSS change that "isn't showing"? Nine times out of ten it's a cache issue — **Ctrl+Shift+R** fixes it before you touch DevTools.

## 5.4 Downloads, History & Bookmarks

| Shortcut | Description | Chromium | Firefox |
|---|---|---|---|
| Ctrl+J | Open Downloads page | ✅ | Opens Library > Downloads |
| Ctrl+H | Open History | ✅ | ✅ |
| Ctrl+D | Bookmark current page | ✅ | ✅ |
| Ctrl+Shift+D | Bookmark **all** open tabs | ✅ | ✅ |
| Ctrl+Shift+O | Open Bookmark Manager | ✅ | ✅ (Library) |
| Ctrl+Shift+B | Toggle bookmarks bar | ✅ | ✅ |

## 5.5 Find, Save, Print, Zoom

| Shortcut | Description |
|---|---|
| Ctrl+F | Find on page |
| Ctrl+G / Enter | Find next match |
| Ctrl+Shift+G / Shift+Enter | Find previous match |
| Ctrl+S | Save page |
| Ctrl+P | Print (or "Save as PDF" via the print dialog) |
| Ctrl+Plus / Ctrl+Minus | Zoom in / out |
| Ctrl+0 | Reset zoom to 100% |

**Real-world example:** On a long legal or technical PDF-style webpage, press **Ctrl+F**, type a keyword, then **Enter** repeatedly to jump through every match instead of scrolling.

## 5.6 Reading Mode / Reader View

| Browser | Shortcut | Notes |
|---|---|---|
| Edge | Ctrl+Shift+U or click the book icon | "Immersive Reader" strips ads/clutter |
| Firefox | F9 | Classic Reader View |
| Chrome | No native shortcut (Reading List: Ctrl+Alt+M some builds) | Use `chrome://flags` reading mode or an extension |
| Brave | Same as Chrome | Brave Speedreader available via icon |

## 5.7 Password Manager, Profiles & Incognito

| Shortcut | Description |
|---|---|
| Ctrl+Shift+N | Incognito (Chrome/Brave) / new window (some Opera builds) |
| Ctrl+Shift+P | Private Browsing (Firefox) |
| Alt+F, then Y | Open Passwords manager menu path (varies by version — safer to use `chrome://settings/passwords`) |

**Tip:** Type `chrome://settings/passwords`, `edge://settings/passwords`, or `about:logins` (Firefox) directly into the address bar for instant access — faster than menu diving.

## 5.8 Developer Tools (browser-level access)

| Shortcut | Description | Browser |
|---|---|---|
| F12 | Open DevTools | All Chromium + Firefox |
| Ctrl+Shift+I | Open DevTools (alt) | All Chromium + Firefox |
| Ctrl+Shift+J | Open DevTools Console directly | Chromium |
| Ctrl+Shift+K | Open Console directly | Firefox |
| Ctrl+Shift+C | Inspect element (picker mode) | All |
| Ctrl+Shift+M | Toggle device/responsive mode | Chromium |

(Full DevTools deep-dive in Sections 31–33.)

## 5.9 PDF Viewer (in-browser)

| Shortcut | Description |
|---|---|
| Ctrl+F | Search within PDF |
| Ctrl+Plus/Minus | Zoom |
| Ctrl+S | Download/save PDF |
| Ctrl+P | Print PDF |
| Arrow keys / Space | Scroll page by page |

## 5.10 Media Controls

| Shortcut | Description |
|---|---|
| Space (with video focused) | Play/Pause |
| K | Play/Pause (YouTube-style players) |
| M | Mute/unmute |
| F | Fullscreen video |
| ← / → | Rewind/forward ~5–10s |
| ↑ / ↓ | Volume up/down |

## 5.11 Extensions & Sidebar

| Shortcut | Description | Browser |
|---|---|---|
| Ctrl+Shift+E (varies) | Focus extensions/toolbar | Depends on extension |
| Alt+Shift+B (Edge) | Toggle Edge sidebar | Edge |
| Ctrl+Shift+Y | Extensions/keyboard shortcuts manager | Chrome (`chrome://extensions/shortcuts`) |

## 5.12 Split View / Vertical Tabs (Edge-specific)

| Shortcut | Description |
|---|---|
| Ctrl+Shift+, (comma) | Toggle vertical tabs (Edge) |
| Split-screen icon in address bar | No universal shortcut; click icon to split a tab into two panes |

## 5.13 Tab Groups & Pinning

| Action | How |
|---|---|
| Pin a tab | Right-click tab → Pin (no default keyboard shortcut; assign one in `chrome://extensions/shortcuts` if desired) |
| Mute a tab | Right-click tab → Mute site, or click the speaker icon that appears on noisy tabs |
| Group tabs | Right-click → "Add tab to new group" |
| Duplicate tab | Ctrl+drag tab downward, or right-click → Duplicate |

### Browser Shortcuts — Master Table (Quick Reference)

| Category | Top 3 shortcuts to learn first |
|---|---|
| Tabs | Ctrl+T, Ctrl+W, Ctrl+Shift+T |
| Navigation | Alt+Left/Right, Ctrl+L |
| Search | Ctrl+F, Ctrl+G |
| Windows | Ctrl+N, Ctrl+Shift+N |
| Refresh | F5, Ctrl+Shift+R |

---

# 6. Windows Shortcuts

Covers **Windows 11** (primary) with **Windows 10** differences noted.

## 6.1 Core Win-Key Shortcuts

| Shortcut | Action | Win 11 | Win 10 | Memory trick |
|---|---|---|---|---|
| Win | Open Start menu | ✅ | ✅ | — |
| Win+D | Show/hide desktop | ✅ | ✅ | **D**esktop |
| Win+E | Open File Explorer | ✅ | ✅ | **E**xplorer |
| Win+I | Open Settings | ✅ | ✅ | **I** = settIngs |
| Win+L | Lock PC | ✅ | ✅ | **L**ock |
| Win+M | Minimize all windows | ✅ | ✅ | **M**inimize |
| Win+Shift+M | Restore minimized windows | ✅ | ✅ | Reverse of Win+M |
| Win+R | Open Run dialog | ✅ | ✅ | **R**un |
| Win+Tab | Open Task View | ✅ | ✅ | Tab = switch |
| Win+A | Open Quick Settings (Action Center) | ✅ | Opens Action Center | **A**ction |
| Win+N | Open Notification Center | ✅ (Win 11 22H2+) | — | **N**otifications |
| Win+X | Open Power User (Quick Link) menu | ✅ | ✅ | E**x**pert menu |
| Win+. or Win+; | Open Emoji panel | ✅ | ✅ | Period = punctuation-like symbol picker |
| Win+H | Start Voice Typing | ✅ | ✅ (dictation) | **H**ear my voice |
| Win+P | Project/display mode | ✅ | ✅ | **P**rojector |
| Win+G | Open Xbox Game Bar | ✅ | ✅ | **G**aming |
| Win+K | Connect to wireless display/audio | ✅ | ✅ | Casting |
| Win+V | Open Clipboard History | ✅ | ✅ | **V** = pas**V**te key |
| Win+S | Open Search | ✅ | ✅ | **S**earch |
| Win+Pause/Break | Open System info | ✅ | ✅ | Legacy but works |
| Win+Comma | Peek at desktop temporarily | ✅ | ✅ | Hold to peek |
| Win+U | Open Accessibility settings | ✅ | ✅ | **U**sability |

## 6.2 Window & Snap Management

| Shortcut | Action |
|---|---|
| Win+Left / Right | Snap window to left/right half |
| Win+Up | Maximize window |
| Win+Down | Restore/minimize window |
| Win+Up then Win+Left/Right | Snap to a screen quadrant |
| Win+Shift+Left / Right | Move window to the other monitor |
| Alt+Tab | Switch between open apps |
| Alt+Shift+Tab | Switch backwards through apps |
| Alt+Esc | Cycle windows in open order (no preview) |
| Alt+F4 | Close active window/app |
| Win+Z | Open Snap Layouts flyout |
| Ctrl+Alt+Tab | Open a "sticky" Alt+Tab that stays open |

**Real-world example:** Writing a report while referencing a spreadsheet? **Win+Left** the doc, **Win+Right** the spreadsheet — a perfect 50/50 split in under a second, no dragging.

## 6.3 Virtual Desktops & Task View

| Shortcut | Action |
|---|---|
| Win+Tab | Open Task View (see all desktops/windows) |
| Win+Ctrl+D | Create a new virtual desktop |
| Win+Ctrl+F4 | Close current virtual desktop |
| Win+Ctrl+Left / Right | Switch between virtual desktops |

**Workflow tip:** Dedicate Desktop 1 to communication (email, Slack), Desktop 2 to deep work (code/docs), Desktop 3 to browser research. Switch contexts instead of hunting through 40 mixed windows.

## 6.4 Task Manager, Clipboard History, Emoji, Voice Typing

| Shortcut | Action |
|---|---|
| Ctrl+Shift+Esc | Open Task Manager directly |
| Ctrl+Alt+Delete | Open security screen (Task Manager, Lock, Sign out) |
| Win+V | Clipboard History (multi-item paste) |
| Win+. | Emoji, GIFs, kaomoji, symbols panel |
| Win+H | Voice Typing (dictate anywhere there's a text field) |

## 6.5 Snipping Tool & Screenshots

(See full detail in Section 11.)

| Shortcut | Action |
|---|---|
| Win+Shift+S | Open Snipping Tool region capture |
| PrtScn | Copy full screen to clipboard |
| Win+PrtScn | Save full screen screenshot to Pictures\Screenshots |
| Alt+PrtScn | Copy active window only |

## 6.6 Quick Settings & Notification Center

| Shortcut | Action |
|---|---|
| Win+A | Quick Settings (Wi-Fi, Bluetooth, volume, brightness) |
| Win+N | Notification Center + Calendar |

## 6.7 Search, Run, Registry, Device Manager, Settings

| Shortcut | Action |
|---|---|
| Win+S or Win+Q | Open Windows Search |
| Win+R | Run dialog (type `regedit`, `devmgmt.msc`, `services.msc`, etc.) |
| Win+I | Settings app |
| Win+Pause | System properties/info |

**Handy Run commands** (type after Win+R):

| Command | Opens |
|---|---|
| `regedit` | Registry Editor |
| `devmgmt.msc` | Device Manager |
| `control` | Classic Control Panel |
| `appwiz.cpl` | Programs & Features |
| `ncpa.cpl` | Network Connections |
| `msconfig` | System Configuration |
| `cmd` | Command Prompt |
| `powershell` | PowerShell |
| `%temp%` | Temp folder |
| `services.msc` | Services manager |

> ⚠️ **Registry caution:** Only edit the registry if you know exactly what you're changing, and back it up first (`File > Export` in Registry Editor). Incorrect edits can break Windows.

## 6.8 Display, Lock, Shutdown, Restart, Sleep

| Shortcut | Action |
|---|---|
| Win+L | Lock |
| Win+Ctrl+Shift+B | Restart graphics driver (fixes a frozen/black screen) |
| Alt+F4 (on desktop) | Opens Shutdown dialog |
| Win+X, then U, then U/R/S | Shutdown / Restart / Sleep via Power User menu |

## 6.9 Widgets & Phone Link

| Shortcut | Action |
|---|---|
| Win+W | Open Widgets panel |
| No default shortcut | Phone Link — open via Start menu search |

## 6.10 Function Keys (general Windows behavior)

| Key | Typical Action |
|---|---|
| F1 | Help |
| F2 | Rename selected item |
| F3 | Search (in many apps) |
| F4 | Show address bar dropdown (File Explorer) |
| F5 | Refresh |
| F6 | Cycle through screen elements/panes |
| F10 | Activate menu bar |
| F11 | Fullscreen toggle |
| F12 | DevTools (browser) / Save As (Word, legacy) |

---

# 7. File Explorer

Windows 10/11 File Explorer shortcuts.

| Shortcut | Action | Notes |
|---|---|---|
| F2 | Rename selected item | Fastest rename method |
| Ctrl+C | Copy | Standard |
| Ctrl+V | Paste | Standard |
| Ctrl+X | Cut | For move operations |
| Delete | Send to Recycle Bin | Recoverable |
| Shift+Delete | **Permanently** delete | ⚠️ Not recoverable — use carefully |
| Ctrl+A | Select all items | In current folder |
| Ctrl+Click | Select multiple individual items | Toggle each item |
| Shift+Click | Select a range | From last-clicked to new click |
| Ctrl+Shift+N | Create new folder | **N** = **N**ew folder |
| Alt+D or Ctrl+L | Focus the address bar | Type a path directly |
| Alt+P | Toggle Preview Pane | View files without opening them |
| Alt+Shift+P | Toggle Details Pane | See metadata |
| Ctrl+Shift+. | Show hidden files (varies by build; also via View menu/Ctrl+H legacy) | Use View > Show > Hidden items |
| Alt+Enter | Open Properties of selected item | Fast metadata access |
| Backspace | Go up one folder level (back) | Also Alt+Left |
| Alt+Up | Go to parent folder | One level up |
| Ctrl+Shift+E | Show all folders (expand tree) | Navigation pane |
| Ctrl+Scroll wheel | Change icon size / view | Zoom in/out on icons |
| Ctrl+E or Ctrl+F | Focus search box | Search current folder |
| Ctrl+N | Open a new File Explorer window | |
| Ctrl+W | Close current window | |
| Windows key + E | Open a fresh File Explorer window | |

### Selection tricks

| Action | Shortcut |
|---|---|
| Invert selection | Select none, then Edit menu / Ctrl+A then Ctrl+Click to deselect specific items |
| Select from current to end | Shift+End (in Details view, after clicking first item) |
| Deselect one item from a selection | Ctrl+Click on it again |

### Sort & Group

| Action | How |
|---|---|
| Sort by column | Click the column header (Name, Date, Size, Type) |
| Reverse sort order | Click same header again |
| Group items | Right-click empty space → Group by → choose criteria |
| Change view | Ctrl+Shift+1 (extra-large icons) through Ctrl+Shift+8 (content view), varies slightly by build |

### File extensions & hidden files (settings, not shortcuts, but essential)

Go to **View > Show > File name extensions** and **View > Show > Hidden items** to toggle visibility — critical for developers who need to see `.env`, `.gitignore`, or actual file types.

**Common mistake:** Pressing **Delete** expecting permanent removal, then wondering why disk space didn't free up — remember Delete = Recycle Bin, **Shift+Delete** = gone for good.

---

# 8. Text Editing

These work in **almost every Windows app**: Notepad, Word, browsers, VS Code, email clients.

## 8.1 Universal Editing

| Shortcut | Action |
|---|---|
| Ctrl+C | Copy |
| Ctrl+X | Cut |
| Ctrl+V | Paste |
| Ctrl+Z | Undo |
| Ctrl+Y or Ctrl+Shift+Z | Redo |
| Ctrl+A | Select all |
| Ctrl+B / I / U | Bold / Italic / Underline (rich text apps) |

## 8.2 Cursor Movement & Selection

| Shortcut | Action |
|---|---|
| Home | Move cursor to **start of line** |
| End | Move cursor to **end of line** |
| Ctrl+Home | Move cursor to **start of document** |
| Ctrl+End | Move cursor to **end of document** |
| Ctrl+Left / Right | Jump one **word** left/right |
| Ctrl+Up / Down | Jump one **paragraph** up/down |
| Shift+Home | Select from cursor to start of line |
| Shift+End | Select from cursor to end of line |
| Ctrl+Shift+Left/Right | Select one word at a time |
| Ctrl+Shift+Home/End | Select to start/end of document |
| Ctrl+Backspace | Delete previous word |
| Ctrl+Delete | Delete next word |

### Home vs. End vs. Ctrl+Home vs. Ctrl+End — the key distinction

| Key | Scope | Direction |
|---|---|---|
| **Home** | Current **line only** | Beginning of that line |
| **End** | Current **line only** | End of that line |
| **Ctrl+Home** | **Entire document** | Very top |
| **Ctrl+End** | **Entire document** | Very bottom |

Add **Shift** to any of these and it becomes a *selection* instead of just cursor movement — e.g., **Ctrl+Shift+End** selects everything from the cursor to the end of the document (great for deleting a huge trailing block of text).

## 8.3 Multi-Cursor Editing (VS Code & modern editors)

| Shortcut | Action |
|---|---|
| Alt+Click | Add a new cursor at click point |
| Ctrl+Alt+Down/Up | Add cursor below/above (VS Code) |
| Ctrl+D | Select next occurrence of current word (VS Code) |
| Ctrl+Shift+L | Select **all** occurrences of current word (VS Code) |

(Full VS Code shortcuts in Section 29.)

## 8.4 Clipboard History & Emoji Picker (also see Sections 6, 10)

| Shortcut | Action |
|---|---|
| Win+V | Clipboard history (paste from last 25 copied items) |
| Win+. | Emoji, symbols, GIFs |

**Memory trick:** Home/End = literal "beginning/end of the row you're standing in." Add Ctrl to teleport to "home base" (top) or "the end" (bottom) of the whole document.

---

# 9. Mouse Shortcuts

Keyboard shortcuts get most of the attention, but modifier+mouse combos are equally powerful.

| Combo | Action | Works in |
|---|---|---|
| Ctrl+Click (link) | Open link in new **background** tab | All browsers |
| Ctrl+Shift+Click (link) | Open link in new **foreground** tab | All browsers |
| Shift+Click (link) | Open link in a new window | All browsers |
| Middle-click (link) | Open link in new tab | All browsers |
| Middle-click (tab) | Close that tab | All browsers |
| Ctrl+Scroll | Zoom in/out | Browsers, many apps |
| Alt+Click | Add cursor / select non-contiguous text (editor-dependent) | VS Code, some editors |
| Shift+Click (files) | Select a range | File Explorer |
| Ctrl+Click (files) | Select multiple individual files | File Explorer |
| Right-click+drag | Move file and get a context menu (Copy/Move/Shortcut) | File Explorer |
| Double-click title bar | Maximize/restore window | Windows-wide |
| Drag window to screen edge | Snap window | Windows-wide (Aero Snap) |
| Shake window (drag rapidly) | Minimize all other windows | Windows-wide |

### Browser gestures (trackpad/precision touchpad, Windows 11)

| Gesture | Action |
|---|---|
| Three-finger swipe left/right | Switch between open apps |
| Three-finger swipe up | Task View |
| Three-finger swipe down | Show desktop |
| Two-finger swipe left/right (in browser) | Back/Forward navigation |

---

# 10. Clipboard

| Shortcut | Action |
|---|---|
| Ctrl+C | Copy selection |
| Ctrl+X | Cut selection |
| Ctrl+V | Paste |
| Ctrl+Shift+V | Paste **without formatting** (plain text) — most browsers, Docs, Slack |
| Win+V | Open Clipboard History panel (up to 25 recent items, pin favorites) |

### Enabling Clipboard History

Clipboard History is off by default on some setups. Enable it: **Settings > System > Clipboard > toggle "Clipboard history" on.** Once on, **Win+V** shows a scrollable list — click any past item to paste it, or pin frequently used snippets (like your email signature) so they never age out.

**Real-world example:** Copy five different product names from a spreadsheet, switch to an email, and press **Win+V** repeatedly to paste each one in order — no re-copying between each paste.

**Common mistake:** Assuming clipboard only holds one item — most users don't know Win+V exists and manually re-copy each time.

---

# 11. Screenshots

| Shortcut | Action | Where it saves |
|---|---|---|
| PrtScn | Copy entire screen to clipboard | Clipboard only |
| Win+PrtScn | Screenshot entire screen | Auto-saved to `Pictures\Screenshots` |
| Alt+PrtScn | Screenshot **active window only** | Clipboard only |
| Win+Shift+S | Open Snipping Tool overlay (rectangular, freeform, window, or fullscreen snip) | Clipboard (+ notification to save/annotate) |
| Win+G, then Win+Alt+PrtScn | Screenshot via Game Bar | Videos\Captures |

### Snipping Tool modes (Win+Shift+S)

| Mode | Use case |
|---|---|
| Rectangular snip | Most common — drag a box |
| Freeform snip | Capture an irregular shape |
| Window snip | Capture one specific open window |
| Fullscreen snip | Capture everything |

**Tip:** After **Win+Shift+S**, a thumbnail notification appears in the corner — click it to annotate (highlight, crop, draw arrows) before saving or pasting.

**Common mistake:** Using PrtScn and then not knowing where the image went — remember plain PrtScn only copies to clipboard; you must paste it (Ctrl+V) somewhere.

---

# 12. Virtual Desktops

(Expanded from Section 6.3)

| Shortcut | Action |
|---|---|
| Win+Tab | Task View — see/manage all desktops |
| Win+Ctrl+D | New virtual desktop |
| Win+Ctrl+F4 | Close current desktop |
| Win+Ctrl+Left/Right | Move between desktops |

### Workflow example: The "3-Desktop System"

1. **Desktop 1 – Communication:** Outlook, Teams/Slack, Calendar
2. **Desktop 2 – Deep Work:** VS Code, terminal, documentation
3. **Desktop 3 – Research:** Browser with 10+ reference tabs

Switch with **Win+Ctrl+Left/Right** instead of Alt-Tabbing through 25 mixed windows. This alone can cut context-switching time by half.

---

# 13. Taskbar

| Shortcut | Action |
|---|---|
| Win+1 through Win+9 | Launch/switch to the app pinned in that taskbar position |
| Win+T | Cycle focus through taskbar apps (press Enter to open) |
| Shift+Click taskbar icon | Open a **new instance** of that app |
| Ctrl+Click taskbar icon | Cycle through open windows of that app |
| Middle-click taskbar icon | Open a new instance (for many apps) |
| Win+B | Focus the system tray (notification area icons) |

**Example:** If VS Code is pinned as your 2nd taskbar icon, **Win+2** always jumps straight to it — or launches it if closed.

---

# 14. Notifications

| Shortcut | Action |
|---|---|
| Win+N | Open Notification Center & Calendar |
| Win+A | Open Quick Settings (separate from notifications in Win 11) |
| Click "Clear all" or use Settings > Notifications | Bulk-dismiss (no dedicated key combo) |

**Tip:** Focus Assist (Settings > System > Notifications > Focus Assist) can be scheduled to auto-silence notifications during deep-work blocks — pair this with your "Desktop 2" deep-work virtual desktop from Section 12.

---

# 15. Search

| Shortcut | Action |
|---|---|
| Win+S or Win+Q | Open Windows Search |
| Win, then start typing | Fastest way to launch any app — Start menu doubles as a search box |
| Ctrl+F (in-app) | Find within the current document/page |
| Win+E, then Ctrl+F | Search inside File Explorer |

**Power tip:** Windows Search understands more than app names — try typing a math expression (`24*7`), a unit conversion (`10 miles to km`), or a settings term (`network`) directly into the Start search box.

---

# 16. Settings

| Shortcut | Action |
|---|---|
| Win+I | Open Settings app |
| Win+A | Quick Settings flyout (Wi-Fi, Bluetooth, brightness, volume, airplane mode) |
| Win+U | Accessibility settings directly |

### Fast settings deep-links (type in Run or Search)

| Command | Opens |
|---|---|
| `ms-settings:display` | Display settings |
| `ms-settings:network` | Network settings |
| `ms-settings:personalization` | Personalization |
| `ms-settings:privacy` | Privacy settings |
| `ms-settings:windowsupdate` | Windows Update |

---

# 17. Accessibility

| Shortcut | Action |
|---|---|
| Win+U | Accessibility settings |
| Win+Ctrl+Enter | Toggle Narrator (screen reader) |
| Win+Plus (+) | Zoom in with Magnifier |
| Win+Minus (-) | Zoom out with Magnifier |
| Win+Esc | Exit Magnifier |
| Right Shift (hold 8 sec) | Toggle Filter Keys |
| Shift (5x) | Toggle Sticky Keys (press modifiers one at a time instead of together) |
| Alt+Left Shift+Num Lock | Toggle Mouse Keys (control cursor with numpad) |
| Win+H | Voice Typing / dictation |

**Sticky Keys** is especially valuable for anyone who finds holding multiple keys simultaneously difficult — after enabling, you can press **Ctrl**, release, then press **C**, and Windows treats it as Ctrl+C.

---

# 18. Developer Shortcuts

A cross-cutting summary; full depth in Sections 25–35.

| Category | Key shortcuts | Section |
|---|---|---|
| Browser DevTools | F12, Ctrl+Shift+I/J/C | 31–33 |
| VS Code | Ctrl+P, Ctrl+Shift+P, F12, Ctrl+D | 29 |
| Git CLI | `git status`, `git add -p`, `git commit`, `git log --oneline` | 30 |
| Terminal | Ctrl+C (kill), Ctrl+R (search history), Tab (autocomplete) | 25–27 |
| Windows Terminal | Ctrl+Shift+T (new tab), Alt+Shift+D (split pane) | 27 |

---

# 19. Microsoft Office

## 19.1 Word

| Shortcut | Action |
|---|---|
| Ctrl+B / I / U | Bold / Italic / Underline |
| Ctrl+Shift+C then Ctrl+Shift+V | Copy formatting / paste formatting (Format Painter) |
| Ctrl+1 / 2 / 5 | Single / Double / 1.5 line spacing |
| Ctrl+Shift+1 through +9 | Apply Heading style 1–9 (varies; commonly Ctrl+Alt+1/2/3 for H1/H2/H3) |
| Ctrl+E / L / R / J | Center / Left / Right / Justify align |
| Ctrl+K | Insert hyperlink |
| Ctrl+Enter | Insert page break |
| F7 | Spell check |
| Shift+F3 | Cycle text case (lower/UPPER/Title) |
| Ctrl+Shift+> / < | Increase/decrease font size |

## 19.2 Excel

| Shortcut | Action |
|---|---|
| Ctrl+Arrow key | Jump to edge of data region |
| Ctrl+Shift+Arrow | Select to edge of data region |
| Ctrl+Space | Select entire column |
| Shift+Space | Select entire row |
| Ctrl+; | Insert today's date |
| Ctrl+Shift+; | Insert current time |
| F2 | Edit active cell |
| F4 | Repeat last action / toggle absolute reference ($) while editing formula |
| Alt+= | AutoSum |
| Ctrl+1 | Format Cells dialog |
| Ctrl+Shift+L | Toggle filters |
| Ctrl+PageUp/PageDown | Switch between sheet tabs |

## 19.3 PowerPoint

| Shortcut | Action |
|---|---|
| F5 | Start slideshow from beginning |
| Shift+F5 | Start from current slide |
| Ctrl+M | New slide |
| Ctrl+D | Duplicate slide/object |
| B | Black out screen during presentation |
| W | White out screen during presentation |
| Ctrl+Shift+> / < | Increase/decrease font size |

## 19.4 Outlook

| Shortcut | Action |
|---|---|
| Ctrl+N | New email (or new item in current view) |
| Ctrl+R | Reply |
| Ctrl+Shift+R | Reply All |
| Ctrl+F | Forward |
| Ctrl+Enter | Send email |
| Ctrl+Shift+M | New message (from anywhere in Outlook) |
| Ctrl+1 / 2 / 3 | Switch to Mail / Calendar / Contacts |

## 19.5 OneNote

| Shortcut | Action |
|---|---|
| Win+N (some versions) or Ctrl+N | New quick note |
| Ctrl+Alt+1/2/3 | Apply Heading style |
| Tab / Shift+Tab | Indent / outdent bullet |
| Ctrl+Shift+M | Insert new page |

---

# 20. Google Docs, Sheets, Slides & Drive

## 20.1 Google Docs

| Shortcut | Action |
|---|---|
| Ctrl+Alt+M | Insert comment |
| Ctrl+K | Insert link |
| Ctrl+Shift+C | Word count |
| Ctrl+/ | Show all keyboard shortcuts (in-app cheat sheet!) |
| Ctrl+Alt+1/2/3 | Apply Heading 1/2/3 |
| Ctrl+Shift+V | Paste without formatting |

## 20.2 Google Sheets

| Shortcut | Action |
|---|---|
| Ctrl+Shift+V | Paste values only |
| Alt+Shift+= (varies) | Insert sum/AutoSum |
| Ctrl+; | Insert today's date |
| Ctrl+Alt+M | Insert comment |
| Ctrl+Shift+1 | Apply date format |

## 20.3 Google Slides

| Shortcut | Action |
|---|---|
| Ctrl+M | New slide |
| Ctrl+Enter | Start presentation |

## 20.4 Google Drive

| Shortcut | Action |
|---|---|
| Shift+F | New folder |
| / (forward slash) | Focus search bar |
| Ctrl+A | Select all files in view |
| G then D | Go to "My Drive" |

## 20.5 Google Calendar

| Shortcut | Action |
|---|---|
| C | Create new event |
| D / W / M | Switch to Day / Week / Month view |
| T | Jump to Today |

**Universal Google Workspace tip:** In nearly any Google app, pressing **`?`** or **Ctrl+/** opens a built-in shortcut cheat sheet specific to that app.

---

# 21. Gmail

| Shortcut | Action |
|---|---|
| C | Compose new email |
| R | Reply |
| A | Reply All |
| F | Forward |
| E | Archive |
| # | Delete |
| Shift+I | Mark as read |
| Shift+U | Mark as unread |
| J / K | Move to next/previous email (in list) |
| G then I | Go to Inbox |
| G then S | Go to Starred |
| / | Focus search bar |
| S | Star an email |
| Ctrl+Enter (or Cmd+Enter Mac) | Send email |

> **Note:** Gmail keyboard shortcuts must first be enabled: **Settings (gear icon) > See all settings > General > Keyboard shortcuts > On**.

---

# 22. YouTube

| Shortcut | Action |
|---|---|
| K or Space | Play/Pause |
| J | Rewind 10 seconds |
| L | Fast-forward 10 seconds |
| ← / → | Rewind/forward 5 seconds |
| M | Mute |
| F | Fullscreen |
| T | Toggle theater mode |
| I | Toggle miniplayer |
| C | Toggle captions |
| Shift+N / Shift+P | Next / previous video |
| 0–9 | Jump to 0%–90% of video |
| Shift+, / Shift+. | Decrease/increase playback speed |
| Home / End | Jump to start/end of video |

**Memory trick:** J-K-L mirrors classic video-editing software layout: **J**og back, **K**eep (pause/play), **L**eap forward.

---

# 23. Google Search Tricks

| Trick | Example | Effect |
|---|---|---|
| `"exact phrase"` | `"climate policy 2026"` | Matches exact wording only |
| `site:` | `site:wikipedia.org shortcuts` | Search within one site |
| `-word` | `jaguar -car` | Excludes a term |
| `filetype:` | `filetype:pdf resume template` | Finds specific file types |
| `OR` | `python OR javascript tutorial` | Either term |
| `define:` | `define:ergonomic` | Instant dictionary definition |
| `*` (wildcard) | `"the * is the limit"` | Fills in unknown word |
| `related:` | `related:nytimes.com` | Similar sites |
| Numeric range | `laptop $500..$900` | Price/number ranges |

---

# 24. PDF Viewers

Applies to Chrome/Edge built-in viewer and Adobe Acrobat Reader.

| Shortcut | Action | Works in |
|---|---|---|
| Ctrl+F | Search | All PDF viewers |
| Ctrl+Plus/Minus | Zoom | All |
| Ctrl+0 | Reset zoom | Most |
| Space / Shift+Space | Scroll down / up a page | All |
| Ctrl+P | Print | All |
| Ctrl+S | Save | All |
| Home / End | First / last page | Most |
| Ctrl+Shift+P (Acrobat) | Print with options | Acrobat |
| Rotate view | Ctrl+Shift+Plus/Minus (Acrobat) | Acrobat |

---

# 25. Terminal & PowerShell

| Shortcut | Action |
|---|---|
| Tab | Autocomplete file/folder/command names |
| Ctrl+C | Kill/interrupt running command |
| Ctrl+L or `cls`/`clear` | Clear screen |
| Up / Down arrow | Cycle through command history |
| Ctrl+R | Reverse-search command history |
| Ctrl+A / Ctrl+E | Jump to start/end of line (PowerShell 7+/PSReadLine) |
| Ctrl+U | Clear line before cursor |
| Ctrl+W | Delete previous word |

### Common PowerShell commands

| Command | Purpose |
|---|---|
| `Get-Process` | List running processes |
| `Get-ChildItem` (alias `ls`/`dir`) | List directory contents |
| `Set-Location` (alias `cd`) | Change directory |
| `Get-Help <cmd>` | Built-in documentation |
| `Get-Content` (alias `cat`) | Print file contents |

---

# 26. Command Prompt

| Shortcut | Action |
|---|---|
| Tab | Autocomplete path |
| Up / Down arrow | Command history |
| F7 | Show command history as a selectable list |
| Ctrl+C | Cancel current command |
| Esc | Clear current line |
| Right-click | Paste (in classic Command Prompt) |

### Navigation commands

| Command | Action |
|---|---|
| `cd ..` | Move up one directory |
| `cd \` | Go to drive root |
| `dir` | List files |
| `cls` | Clear screen |
| `cd /d D:\Projects` | Change drive **and** directory in one step |

---

# 27. Windows Terminal

| Shortcut | Action |
|---|---|
| Ctrl+Shift+T | New tab |
| Ctrl+Shift+W | Close tab |
| Ctrl+Tab / Ctrl+Shift+Tab | Next/previous tab |
| Alt+Shift+D | Split pane (auto-direction) |
| Alt+Shift+Plus / Minus | Split pane horizontally/vertically |
| Alt+Arrow keys | Move focus between panes |
| Ctrl+Shift+F | Search terminal output |
| Ctrl+Plus/Minus | Zoom text size |
| Ctrl+Shift+1/2/3... | Open a new tab with a specific profile (PowerShell, WSL, CMD) |
| Ctrl+Shift+C | Copy selection |
| Ctrl+Shift+V | Paste |

**Real-world workflow:** Split a pane vertically (**Alt+Shift+Plus**) to run your dev server on the left and `git status`/logs on the right — no window switching needed.

---

# 28. GitHub

| Shortcut | Action | Where |
|---|---|---|
| `t` | Activate file finder in a repo | Repo view |
| `g` then `c` | Go to Code tab | Repo view |
| `g` then `i` | Go to Issues tab | Repo view |
| `g` then `p` | Go to Pull requests tab | Repo view |
| `/` | Focus site search bar | Any GitHub page |
| `.` | Open repo in github.dev (web VS Code) | Repo view |
| `c` | Create new issue/PR comment shortcut context | Issue/PR view |
| `Ctrl+Enter` | Submit comment | Issue/PR view |
| `y` | Get a permalink to current view | File view |

---

# 29. VS Code

## 29.1 Navigation

| Shortcut | Action |
|---|---|
| Ctrl+P | Quick Open (jump to any file by name) |
| Ctrl+Shift+P | Command Palette (run any command) |
| Ctrl+Tab | Switch between open editor tabs |
| Ctrl+G | Go to line number |
| Ctrl+Shift+O | Go to symbol in file |
| Ctrl+T | Go to symbol in workspace |
| Alt+Left / Right | Navigate back/forward through cursor history |

## 29.2 Search & Replace

| Shortcut | Action |
|---|---|
| Ctrl+F | Find in file |
| Ctrl+H | Replace in file |
| Ctrl+Shift+F | Find across entire project |
| Ctrl+Shift+H | Replace across entire project |
| Alt+Enter | Select all occurrences of Find match |

## 29.3 Multi-Cursor & Selection

| Shortcut | Action |
|---|---|
| Ctrl+D | Select next occurrence of current selection/word |
| Ctrl+Shift+L | Select all occurrences |
| Ctrl+Alt+Up/Down | Add cursor above/below |
| Alt+Click | Add a cursor at click point |
| Shift+Alt+Right | Expand selection (smart) |
| Shift+Alt+Left | Shrink selection |

## 29.4 Refactoring

| Shortcut | Action |
|---|---|
| F2 | Rename symbol (renames everywhere it's used) |
| Ctrl+. | Quick Fix / Refactor suggestions |
| Shift+Alt+F | Format document |
| Ctrl+K, Ctrl+F | Format selection |

## 29.5 Go To Definition / Peek

| Shortcut | Action |
|---|---|
| F12 | Go to Definition |
| Alt+F12 | Peek Definition (inline popup) |
| Shift+F12 | Find All References |
| Ctrl+K, F12 | Open Definition to the side |

## 29.6 Git Integration (built-in)

| Shortcut | Action |
|---|---|
| Ctrl+Shift+G, G | Open Source Control panel |
| Ctrl+Enter (in commit box) | Commit staged changes |

## 29.7 Terminal & Split Editor

| Shortcut | Action |
|---|---|
| Ctrl+` (backtick) | Toggle integrated terminal |
| Ctrl+Shift+\` | New terminal |
| Ctrl+\ | Split editor |
| Ctrl+1 / 2 / 3 | Focus editor group 1/2/3 |

## 29.8 Zen Mode & Focus

| Shortcut | Action |
|---|---|
| Ctrl+K, Z | Enter Zen Mode (distraction-free, full screen) |
| Esc, Esc | Exit Zen Mode |

## 29.9 Debugging

| Shortcut | Action |
|---|---|
| F5 | Start/Continue debugging |
| Shift+F5 | Stop debugging |
| F9 | Toggle breakpoint |
| F10 | Step over |
| F11 | Step into |
| Shift+F11 | Step out |

## 29.10 Command Palette power moves

Press **Ctrl+Shift+P**, then type any of these fuzzy-matched phrases:

| Type | Runs |
|---|---|
| `>reload` | Reload window |
| `>format` | Format document |
| `>theme` | Change color theme |
| `>shortcuts` | Open Keyboard Shortcuts editor |

---

# 30. Git (Command Line)

Git itself isn't keyboard-shortcut-based — it's command-driven — but these habits are the terminal equivalent of shortcuts: short, memorized, muscle-memory commands.

| Command | Purpose |
|---|---|
| `git status` | See what's changed |
| `git add .` | Stage all changes |
| `git add -p` | Stage changes interactively, hunk by hunk |
| `git commit -m "message"` | Commit staged changes |
| `git commit --amend` | Edit the last commit |
| `git log --oneline --graph` | Compact visual history |
| `git diff` | See unstaged changes |
| `git diff --staged` | See staged changes |
| `git stash` / `git stash pop` | Temporarily shelve changes |
| `git checkout -b feature-x` | Create and switch to new branch |
| `git switch -c feature-x` | Modern equivalent of the above |
| `git rebase -i HEAD~3` | Interactively edit last 3 commits |
| `git push -u origin main` | Push and set upstream tracking |
| `git pull --rebase` | Pull and replay local commits on top |
| `git reset --soft HEAD~1` | Undo last commit, keep changes staged |

### Shell productivity tips

- Use **Tab** to autocomplete branch names and file paths.
- Alias common commands in `.gitconfig` or your shell profile: `alias gs='git status'`, `alias gc='git commit'`.
- **Ctrl+R** in Bash/Zsh (or PowerShell with PSReadLine) reverse-searches your command history — type a few letters of a past `git` command to instantly recall it.

---

# 31. Chrome DevTools

| Shortcut | Action |
|---|---|
| F12 or Ctrl+Shift+I | Open/close DevTools |
| Ctrl+Shift+J | Jump straight to Console |
| Ctrl+Shift+C | Inspect element (element picker) |
| Ctrl+Shift+M | Toggle Device Toolbar (responsive/mobile emulation) |
| Ctrl+Shift+P (inside DevTools) | Command Menu (run any DevTools action) |
| Ctrl+F (inside Elements panel) | Search DOM |
| Ctrl+Shift+F (inside Sources) | Search across all files |
| Ctrl+P (inside Sources) | Quickly open any file |
| Esc (inside DevTools) | Toggle the Console drawer at the bottom |
| Ctrl+] / [ | Indent/outdent in Sources editor |

### Panels quick reference

| Panel | Purpose |
|---|---|
| Elements | Inspect/edit live DOM & CSS |
| Console | Run JS, view logs/errors |
| Sources | Debug JS, set breakpoints, "Pretty Print" minified code |
| Network | Inspect requests, timing, payloads, throttle connection speed |
| Performance | Record and profile page rendering/JS execution |
| Lighthouse | Automated audits (performance, SEO, accessibility) |
| Application | Inspect storage — cookies, localStorage, Service Workers |

**Pretty Print:** In the Sources panel, click the `{}` icon at the bottom to un-minify compressed JS/CSS for readable debugging.

---

# 32. Edge DevTools

Edge is Chromium-based, so **all Chrome DevTools shortcuts (Section 31) work identically**. Edge-specific additions:

| Shortcut | Action |
|---|---|
| Ctrl+Shift+I | Open DevTools |
| Ctrl+Shift+M | Device emulation |
| Ctrl+Shift+U | 3D View (Edge-exclusive DOM visualization, in newer builds) |
| Ctrl+Shift+X | Accessibility Insights tab (Edge-exclusive) |

---

# 33. Firefox Developer Tools

| Shortcut | Action |
|---|---|
| F12 or Ctrl+Shift+I | Open/close DevTools |
| Ctrl+Shift+K | Jump straight to Console |
| Ctrl+Shift+C | Inspect element |
| Ctrl+Shift+M | Responsive Design Mode |
| Ctrl+Shift+E | Network panel directly |
| Ctrl+Shift+S | Style Editor directly |
| Ctrl+Shift+Z | Toggle Debugger panel |

**Firefox-only strength:** The **Grid panel** and **Flexbox inspector** in Firefox DevTools are considered best-in-class for visually debugging CSS Grid/Flexbox layouts — worth opening Firefox specifically for CSS layout work even if you develop primarily in Chrome.

---

# 34. React Developer Workflow

Combining shortcuts across VS Code, browser, and terminal for a React project:

1. **Ctrl+`** (VS Code) — open integrated terminal, run `npm run dev`
2. **Ctrl+P** (VS Code) — jump to component file by name
3. **F2** — rename a component/prop across the whole codebase safely
4. **Ctrl+Shift+F** — project-wide search for a prop or hook usage
5. Switch to browser, **F12** — open DevTools, install/open **React Developer Tools** panel
6. **Ctrl+Shift+M** — test responsive breakpoints
7. **Ctrl+Shift+R** — hard refresh after a build/cache issue
8. Back in VS Code, **Ctrl+K Z** — Zen Mode for focused component writing

**Tip:** Use **Ctrl+D** repeatedly in VS Code to multi-select every instance of a prop name you're renaming in a single component before applying **F2** for the project-wide rename.

---

# 35. Node.js Workflow

| Shortcut/Command | Action |
|---|---|
| Ctrl+C | Stop a running Node process/server |
| Ctrl+` then `npm run dev` | Start dev server without leaving VS Code |
| `rs` + Enter (in nodemon) | Manually restart a nodemon-managed server |
| Ctrl+R (shell) | Recall a previous `npm` command from history |
| Tab | Autocomplete `npm run <script-name>` |

**Debugging tip:** In VS Code, set a breakpoint (**F9**) inside a Node file, then press **F5** to launch the built-in Node debugger — no separate tool needed.

---

# 36. Daily Productivity Workflow

A shortcut-first walkthrough of a full professional day.

### Morning

- **Win+L** to unlock, log in
- **Win**, type app name, **Enter** to launch email/calendar
- **Win+Ctrl+D** ×3 to set up your three virtual desktops for the day

### Coding

- **Ctrl+`** to open terminal, start dev environment
- **Ctrl+P** to jump between files
- **Ctrl+D / F2** for renames and multi-edit
- **F5** to run/debug

### Research

- **Ctrl+T** to open tabs for each source
- **Ctrl+Shift+A** to search among open tabs when it gets crowded
- **Ctrl+D** to bookmark key references
- **Ctrl+F** inside long articles/PDFs to jump to relevant sections

### Meetings

- **Win+N** to check calendar/notifications before joining
- **Alt+Tab** to quickly reference notes or screens during the call
- **Win+Shift+S** to snip and share a screenshot in chat

### Email

- **C** (Gmail) or **Ctrl+N** (Outlook) to compose
- **R / Ctrl+R** to reply, **E** to archive (Gmail)
- **J/K** to move through the inbox without touching the mouse

### Debugging

- **F12** to open DevTools, **Ctrl+Shift+R** to hard-refresh after each fix
- **Ctrl+Shift+F** (VS Code/DevTools) to search across all files for an error string

### Documentation

- **Ctrl+Alt+1/2/3** for consistent heading structure (Docs/Word)
- **Ctrl+K** to insert reference links quickly

### Testing

- **F5** to re-run in VS Code debugger
- **Ctrl+Shift+R** in browser between test passes to avoid cache false-positives

### Deployment

- **Ctrl+`** → run deploy script
- **Ctrl+Shift+G, G** to review Git changes before pushing

### Browsing

- **Ctrl+1–8** to jump straight to a specific tab
- **Ctrl+Shift+T** to recover an accidentally closed research tab

### Reading PDFs

- **Ctrl+F** to search, **Space** to page down, **Ctrl+Plus/Minus** to zoom

### File Management

- **Win+E**, **F2** to rename, **Ctrl+Shift+N** to organize into new folders

### Multitasking

- **Win+Left/Right** to snap-compare two windows
- **Win+Tab** to switch entire desktop contexts

### End of Day

- **Win+L** to lock before stepping away
- Close out tabs deliberately with **Ctrl+W** rather than leaving 60 tabs for tomorrow

---

# 37. Shortcut Cheat Sheets

## 37.1 Browser Cheat Sheet

| Action | Shortcut |
|---|---|
| New tab | Ctrl+T |
| Close tab | Ctrl+W |
| Reopen tab | Ctrl+Shift+T |
| Switch tab | Ctrl+Tab / Ctrl+1-8 |
| Address bar | Ctrl+L |
| Find | Ctrl+F |
| Hard refresh | Ctrl+Shift+R |
| Bookmark | Ctrl+D |
| Incognito | Ctrl+Shift+N |
| DevTools | F12 |

## 37.2 Windows Cheat Sheet

| Action | Shortcut |
|---|---|
| Desktop | Win+D |
| Explorer | Win+E |
| Lock | Win+L |
| Settings | Win+I |
| Snap left/right | Win+Left/Right |
| Task View | Win+Tab |
| Clipboard history | Win+V |
| Screenshot tool | Win+Shift+S |
| Emoji | Win+. |
| Task Manager | Ctrl+Shift+Esc |

## 37.3 VS Code Cheat Sheet

| Action | Shortcut |
|---|---|
| Quick Open | Ctrl+P |
| Command Palette | Ctrl+Shift+P |
| Terminal | Ctrl+\` |
| Go to Definition | F12 |
| Rename Symbol | F2 |
| Format Document | Shift+Alt+F |
| Multi-select | Ctrl+D |
| Find in files | Ctrl+Shift+F |

## 37.4 Git Cheat Sheet

| Action | Command |
|---|---|
| Status | `git status` |
| Stage all | `git add .` |
| Commit | `git commit -m "msg"` |
| Push | `git push` |
| Branch | `git switch -c name` |
| History | `git log --oneline --graph` |

## 37.5 File Explorer Cheat Sheet

| Action | Shortcut |
|---|---|
| Rename | F2 |
| New folder | Ctrl+Shift+N |
| Permanent delete | Shift+Delete |
| Properties | Alt+Enter |
| Address bar | Ctrl+L |

## 37.6 Text Editing Cheat Sheet

| Action | Shortcut |
|---|---|
| Start of doc | Ctrl+Home |
| End of doc | Ctrl+End |
| Select word | Ctrl+Shift+Right |
| Delete word back | Ctrl+Backspace |

## 37.7 Office Cheat Sheet

| Action | Shortcut |
|---|---|
| Bold | Ctrl+B |
| Hyperlink | Ctrl+K |
| AutoSum (Excel) | Alt+= |
| New slide (PPT) | Ctrl+M |

## 37.8 Google Workspace Cheat Sheet

| Action | Shortcut |
|---|---|
| Show shortcuts | Ctrl+/ or ? |
| Compose (Gmail) | C |
| Archive (Gmail) | E |
| Insert comment (Docs) | Ctrl+Alt+M |

---

# 38. Memory Techniques

Every shortcut is easier to remember when tied to a **letter meaning** or a **short story**. Use these techniques throughout the book:

| Technique | How it works | Example |
|---|---|---|
| **Letter-to-word mapping** | The action key's letter matches the action's name | Ctrl+**T** = **T**ab, Ctrl+**W** = close **W**indow/tab, Ctrl+**N** = **N**ew |
| **Opposite-pairing** | Add Shift to reverse an action | Ctrl+T (new tab) ↔ Ctrl+Shift+T (undo close) |
| **Physical story** | Turn the combo into a tiny mental scene | Ctrl+Shift+T = "Shift back in time to bring the Tab back" |
| **Spatial grouping** | Learn keys by keyboard zone | F-keys = "function row = special powers"; arrow cluster = navigation |
| **Escalation logic** | More modifiers = more powerful/rarer action | Ctrl+W (close tab) → Alt+F4 (close app) |

### Worked examples

- **Ctrl+T** → T = **T**ab (open one)
- **Ctrl+W** → W = "**W**ave goodbye" (close tab)
- **Ctrl+Shift+T** → "**Shift** back in time, bring the **T**ab back"
- **Win+D** → D = **D**esktop
- **Win+E** → E = **E**xplorer
- **Win+L** → L = **L**ock
- **F2** → "**2** seconds to re-**N**ame" (rename is the 2nd most common file action after opening)
- **Ctrl+Shift+R** → "**R**efresh, but for **R**eal this time" (hard refresh)

---

# 39. Practice Exercises

## 39.1 Beginner — "No Mouse for 5 Minutes"

Without touching the mouse:

1. Open your browser (Win, type browser name, Enter)
2. Open 5 tabs (Ctrl+T ×5)
3. Switch between tabs 1 through 5 using Ctrl+1 through Ctrl+5
4. Bookmark the current page (Ctrl+D)
5. Duplicate the current tab (Ctrl+L, Alt+Enter, or right-click alternative — try Ctrl+T then retyping URL for practice)
6. Close a tab (Ctrl+W)
7. Restore it (Ctrl+Shift+T)

## 39.2 Beginner — File Explorer Basics

1. Open File Explorer (Win+E)
2. Create a new folder (Ctrl+Shift+N)
3. Rename it (F2)
4. Select all items in the folder (Ctrl+A)
5. Open Properties on one file (Alt+Enter)

## 39.3 Intermediate — Window Management Sprint

1. Open two apps (e.g., Notepad and a browser)
2. Snap Notepad to the left (Win+Left)
3. Snap the browser to the right (Win+Right)
4. Switch focus using Alt+Tab
5. Minimize both (Win+M)
6. Restore them (Win+Shift+M)
7. Open Task View and create a second virtual desktop (Win+Tab, then Win+Ctrl+D)

## 39.4 Intermediate — Text Editing Drill

In any text editor:

1. Type three paragraphs
2. Jump to the very start (Ctrl+Home)
3. Jump to the very end (Ctrl+End)
4. Select the last paragraph only (Ctrl+Shift+Home from the end, or click + Shift+Ctrl+Up twice)
5. Delete a word backward (Ctrl+Backspace)
6. Undo everything (Ctrl+Z repeatedly)

## 39.5 Advanced — VS Code Refactor Challenge

1. Open a JS/TS file with a repeated variable name
2. Place cursor on the variable, press Ctrl+D repeatedly to select each occurrence
3. Type a new name to replace all selected instances at once
4. Alternatively, use F2 to rename the symbol project-wide
5. Format the document (Shift+Alt+F)
6. Open integrated terminal (Ctrl+\`) and run the file/tests

## 39.6 Advanced — Full Workflow Simulation

Simulate a real 10-minute work sprint using **only the keyboard**:

1. Open email, compose a message, send it (C, type, Ctrl+Enter)
2. Switch to browser, research a topic across 4 tabs
3. Screenshot a finding (Win+Shift+S)
4. Switch to VS Code, open a file (Ctrl+P), make an edit, save (Ctrl+S)
5. Open terminal, run `git status` and `git add .` and commit
6. Lock your PC when done (Win+L)

**Scoring yourself:** Time each exercise the first time with the mouse, then again keyboard-only. Track the seconds saved — this is the fastest way to *believe* in shortcuts, not just know them.

---

# 40. Printable Reference & Index

## 40.1 Top 100 Must-Know Shortcuts

| # | Shortcut | Action | # | Shortcut | Action |
|---|---|---|---|---|---|
| 1 | Ctrl+C | Copy | 26 | Win+L | Lock PC |
| 2 | Ctrl+V | Paste | 27 | Win+E | File Explorer |
| 3 | Ctrl+X | Cut | 28 | Win+I | Settings |
| 4 | Ctrl+Z | Undo | 29 | Win+D | Show desktop |
| 5 | Ctrl+Y | Redo | 30 | Win+Tab | Task View |
| 6 | Ctrl+A | Select all | 31 | Win+V | Clipboard history |
| 7 | Ctrl+S | Save | 32 | Win+Shift+S | Screenshot tool |
| 8 | Ctrl+P | Print | 33 | Win+. | Emoji panel |
| 9 | Ctrl+F | Find | 34 | Win+R | Run dialog |
| 10 | Ctrl+N | New | 35 | Alt+Tab | Switch apps |
| 11 | Ctrl+T | New browser tab | 36 | Alt+F4 | Close app |
| 12 | Ctrl+W | Close tab | 37 | F2 | Rename |
| 13 | Ctrl+Shift+T | Reopen closed tab | 38 | F5 | Refresh |
| 14 | Ctrl+Tab | Next tab | 39 | F11 | Fullscreen |
| 15 | Ctrl+L | Address bar | 40 | F12 | DevTools |
| 16 | Ctrl+Shift+N | Incognito window | 41 | Home | Line start |
| 17 | Ctrl+H | History | 42 | End | Line end |
| 18 | Ctrl+D | Bookmark | 43 | Ctrl+Home | Doc start |
| 19 | Ctrl+J | Downloads | 44 | Ctrl+End | Doc end |
| 20 | Ctrl+Shift+R | Hard refresh | 45 | Ctrl+Backspace | Delete word |
| 21 | Ctrl+Plus/Minus | Zoom | 46 | Shift+Delete | Permanent delete |
| 22 | Ctrl+0 | Reset zoom | 47 | Ctrl+Shift+Esc | Task Manager |
| 23 | Ctrl+B/I/U | Bold/Italic/Underline | 48 | Win+X | Power menu |
| 24 | Ctrl+K | Insert link | 49 | Win+Up/Down | Maximize/restore |
| 25 | Ctrl+Enter | Send/confirm | 50 | Win+Left/Right | Snap window |
| 51 | Ctrl+G | Go to line/find next | 76 | Ctrl+P (VS Code) | Quick Open |
| 52 | Ctrl+Shift+P | Command Palette (VS Code) | 77 | Ctrl+Shift+F | Search project-wide |
| 53 | Ctrl+D (VS Code) | Select next occurrence | 78 | Ctrl+\` | Toggle terminal |
| 54 | F2 (VS Code) | Rename symbol | 79 | F12 (VS Code) | Go to definition |
| 55 | Shift+Alt+F | Format document | 80 | Ctrl+/ | Toggle comment |
| 56 | Ctrl+Shift+C | Inspect element | 81 | Ctrl+Shift+J | Console |
| 57 | Ctrl+Shift+M | Device toolbar | 82 | Ctrl+Alt+M (Docs) | Insert comment |
| 58 | Ctrl+Alt+1/2/3 | Heading styles | 83 | Alt+= (Excel) | AutoSum |
| 59 | Ctrl+; (Excel) | Insert date | 84 | Ctrl+1 (Excel) | Format cells |
| 60 | Ctrl+Shift+L (Excel) | Toggle filters | 85 | F5 (PowerPoint) | Start slideshow |
| 61 | Ctrl+M (PowerPoint) | New slide | 86 | C (Gmail) | Compose |
| 62 | E (Gmail) | Archive | 87 | # (Gmail) | Delete |
| 63 | J/K (Gmail) | Navigate emails | 88 | K (YouTube) | Play/pause |
| 64 | F (YouTube) | Fullscreen | 89 | T (YouTube) | Theater mode |
| 65 | Ctrl+Shift+A | Search tabs | 90 | Ctrl+9 | Jump to last tab |
| 66 | Ctrl+1-8 | Jump to tab N | 91 | Middle-click | Close tab (mouse) |
| 67 | Alt+Left/Right | Browser back/forward | 92 | Win+Ctrl+D | New virtual desktop |
| 68 | Win+Ctrl+Left/Right | Switch virtual desktop | 93 | Win+N | Notification Center |
| 69 | Win+A | Quick Settings | 94 | Win+G | Game Bar |
| 70 | Win+H | Voice typing | 95 | Win+U | Accessibility settings |
| 71 | Ctrl+Shift+V | Paste without formatting | 96 | Ctrl+Shift+Z | Redo (alt) |
| 72 | Shift+F10 | Right-click menu (keyboard) | 97 | Ctrl+Alt+Delete | Security screen |
| 73 | Alt+Enter (Explorer) | Properties | 98 | Ctrl+Shift+Escape | Task Manager (direct) |
| 74 | Ctrl+Shift+N (Explorer) | New folder | 99 | PrtScn | Copy screen |
| 75 | Backspace (Explorer) | Go up a folder | 100 | Win+PrtScn | Save screenshot |

## 40.2 Top 50 Developer Shortcuts

See Sections 18, 25, 27, 29, 30, 31, 33 for full context. Highest-value picks: Ctrl+P, Ctrl+Shift+P, F2, F12, Ctrl+D, Ctrl+Shift+F, Ctrl+\`, Shift+Alt+F, F5/F9/F10/F11 (debugging), Ctrl+Shift+I, Ctrl+Shift+C, Ctrl+Shift+M, Ctrl+R (shell history), Tab (autocomplete), `git status`, `git add -p`, `git commit -m`, `git log --oneline --graph`, Ctrl+Shift+T (Windows Terminal new tab), Alt+Shift+D (split pane).

## 40.3 Top 50 Office Shortcuts

See Section 19 in full. Highest-value picks: Ctrl+B/I/U, Ctrl+K, Ctrl+1/2/5 (spacing), Ctrl+Enter (page break/send), Ctrl+; (date), Alt+= (AutoSum), F4 (repeat/toggle reference), Ctrl+Shift+L (filters), F5 (slideshow), Ctrl+M (new slide), Ctrl+R (reply), Ctrl+Shift+R (reply all).

## 40.4 Top 50 Browser Shortcuts

See Section 5 in full and Cheat Sheet 37.1.

## 40.5 Top 50 Windows Shortcuts

See Section 6 in full and Cheat Sheet 37.2.

## 40.6 30-Day Shortcut Learning Plan

| Days | Focus | Goal |
|---|---|---|
| 1–5 | Core browser (Ctrl+T/W/Tab/L/F) | Never touch the mouse for tabs |
| 6–10 | Core Windows (Win+D/E/L/I, snapping) | Manage windows entirely by keyboard |
| 11–15 | File Explorer + text editing | Fast renames, navigation, selection |
| 16–20 | Clipboard, screenshots, search | Win+V, Win+Shift+S, Win+S mastery |
| 21–25 | Developer tools (VS Code or DevTools) | Ctrl+P, F12, F2, Ctrl+D fluency |
| 26–30 | Office/Google Workspace + full workflow | Combine everything into your daily routine |

**Rule for each week:** Learn 5 new shortcuts, drill them with the matching exercise in Section 39, and consciously avoid the mouse for those specific actions until they're automatic.

## 40.7 Alphabetical Index (Selected Core Shortcuts)

| Shortcut | Section |
|---|---|
| Alt+F4 | 6.2 |
| Alt+Tab | 6.2 |
| Ctrl+0 | 5.5 |
| Ctrl+1–8 (tabs) | 5.1 |
| Ctrl+A | 8.1 |
| Ctrl+B/I/U | 19.1 |
| Ctrl+C/X/V | 8.1, 10 |
| Ctrl+D (bookmark) | 5.4 |
| Ctrl+D (VS Code) | 29.3 |
| Ctrl+End/Home | 8.2 |
| Ctrl+F | 5.5 |
| Ctrl+H | 5.4 |
| Ctrl+L | 5.3 |
| Ctrl+N | 5.2 |
| Ctrl+P (VS Code) | 29.1 |
| Ctrl+R (shell) | 25 |
| Ctrl+S | 5.5 |
| Ctrl+Shift+Esc | 6.4 |
| Ctrl+Shift+N | 5.2 |
| Ctrl+Shift+R | 5.3 |
| Ctrl+Shift+T | 5.1 |
| Ctrl+T | 5.1 |
| Ctrl+W | 5.1 |
| Ctrl+Z/Y | 8.1 |
| F2 | 7, 29.4 |
| F5 | 5.3, 29.9, 19.3 |
| F11 | 5.2 |
| F12 | 5.8, 29.5, 31 |
| Shift+Delete | 7 |
| Win+. | 6.1 |
| Win+A | 6.6 |
| Win+D | 6.1 |
| Win+E | 6.1 |
| Win+I | 6.1 |
| Win+L | 6.1 |
| Win+N | 6.1 |
| Win+PrtScn | 11 |
| Win+R | 6.1 |
| Win+S | 6.1 |
| Win+Shift+S | 11 |
| Win+Tab | 6.3 |
| Win+V | 10 |
| Win+X | 6.1 |

---

## Closing Notes

Mastery isn't memorizing every entry in this handbook — it's building **automatic muscle memory** for the 30–40 shortcuts you personally use daily. Start with the **Top 100** list (40.1), drill with the **Practice Exercises** (Section 39), and revisit the **Cheat Sheets** (Section 37) whenever you pick up a new tool.

Every second saved compounds. Good luck — and enjoy leaving the mouse behind.

**— End of Handbook —**
