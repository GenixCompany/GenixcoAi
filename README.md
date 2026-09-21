<div align="center">

# Genixco AI

### The AI assistant that actually talks to Windows.

**by Genix**

![version](https://img.shields.io/badge/version-1.0.0-5B8CFF?style=for-the-badge)
![python](https://img.shields.io/badge/python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PySide6](https://img.shields.io/badge/UI-PySide6-41CD52?style=for-the-badge&logo=qt&logoColor=white)
![OpenRouter](https://img.shields.io/badge/provider-OpenRouter-8A2BE2?style=for-the-badge)
![platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%2011-0078D4?style=for-the-badge&logo=windows&logoColor=white)
![single file](https://img.shields.io/badge/architecture-single%20file-FF8A00?style=for-the-badge)

*A production-oriented desktop AI assistant — chat, reason, and execute real Windows actions through a permission-guarded tool layer. Everything in one `main.py`.*

</div>

---

```
╭────────╮
│   AI   │   ← always-on-top floating button. drag it. click it. own your desktop.
╰────────╯
```

---

## Table of Contents

- [Why Genixco AI](#why-genixco-ai)
- [Feature Highlights](#feature-highlights)
- [Screenshots / UI Map](#screenshots--ui-map)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Architecture](#architecture)
- [Tool Catalogue](#tool-catalogue)
- [Tool Protocol](#tool-protocol)
- [Security Model](#security-model)
- [Settings Reference](#settings-reference)
- [Data & File Locations](#data--file-locations)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)
- [Roadmap](#roadmap)
- [FAQ](#faq)
- [راهنمای فارسی](#راهنمای-فارسی)
- [License](#license)

---

## Why Genixco AI

Most "AI desktop apps" are a chat box with a wrapper around an API. Genixco AI is different in three ways:

| | Typical AI wrapper | **Genixco AI** |
|---|---|---|
| Actions | Tells you *how* to open Chrome | **Actually launches Chrome** and reports the real result |
| Honesty | Claims success because the model said so | Reports only what the **tool** returned — no fabricated results |
| Safety | Pipes model output into `os.system()` | Validated **tool layer** + permissions + confirmation dialogs |
| Secrets | API key in a config file or hardcoded | **Windows Credential Manager** / env var, never in source, logs or exports |
| Footprint | Dozens of modules | **One file.** `python main.py` |

> **Design rule:** the AI never executes anything directly. It *requests* a tool; Python validates, checks permission, asks the user if required, executes, and feeds the **real** result back to the model.

---

## Feature Highlights

### Conversational AI
- Full **OpenRouter** integration (`/chat/completions`) with any model ID you type.
- **Token-by-token streaming** — the answer appears as it is generated.
- Retries on transient failures (429 / 5xx), configurable timeout, clean session handling.
- Never blocks the UI: every network call runs on a `QThread`.

### Windows Automation
- **16 real tools**: launch apps, open URLs and folders, read/write files, list directories, delete paths, clipboard I/O, screenshots, system info, process list, shell commands.
- Standardised tool results — success and failure are both explicit and structured.
- Master automation switch + per-category toggles + tray-level **Pause Automation**.

### The Floating Button
- Frameless, rounded, translucent, always-on-top **AI** button.
- Drag anywhere; position is saved and restored across restarts.
- Multi-monitor aware — auto-clamps back on screen if your resolution or monitor layout changes.
- Adjustable size, opacity, corner radius; hide/show from the tray; right-click opens the tray menu.

### Modern UI
- Dark & Light themes, custom **accent color**, adjustable font size and UI scale.
- Chat bubbles, live connection state (`● Online / Offline / Processing`), live task state (`Thinking… / Calling Tool… / Waiting for Confirmation… / Completed / Error`).
- Tray icon with a full menu; icon and button are drawn in code — **zero external assets**.

### Security & Privacy
- API key stored in Windows Credential Manager (via `keyring`) or read from `OPENROUTER_API_KEY`.
- Every log line passes a **redaction filter** — keys, tokens, passwords and secrets never hit disk.
- Restricted-path engine, allow-listed applications, destructive-command blocker, confirmation dialogs.
- **Prompt-injection aware**: file/command output is labelled as untrusted DATA and can never grant permissions.

### Operations
- Rotating log files, first-run setup wizard, built-in **Diagnostics** page.
- Chat history with export to Markdown; settings export/import (secrets excluded); one-click reset.
- Graceful shutdown: threads stopped, sessions closed, settings flushed, temp files removed.

---

## Screenshots / UI Map

```
┌──────────────────────────────────────────────────────────┐
│  Genixco AI                        ● Online   New   ⚙    │
│  Genix                                                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ╭──────────────────────────────────────╮                │
│  │ Genixco AI                           │                │
│  │ Hello! I'm Genixco AI by Genix.      │                │
│  ╰──────────────────────────────────────╯                │
│                                                          │
│                     ╭──────────────────────────────────╮ │
│                     │ You                              │ │
│                     │ Open Chrome                      │ │
│                     ╰──────────────────────────────────╯ │
│                                                          │
│  ╭──────────────────────────────────────╮                │
│  │ System                               │                │
│  │ ✓ launch_application: Chrome launched│                │
│  ╰──────────────────────────────────────╯                │
│                                                          │
│  ╭──────────────────────────────────────╮                │
│  │ Genixco AI                           │                │
│  │ Chrome has been opened successfully. │                │
│  ╰──────────────────────────────────────╯                │
│                                                          │
├──────────────────────────────────────────────────────────┤
│ Ask Genixco AI anything...                               │
│ Ready                       [Clear] [Stop] [   Send   ]  │
└──────────────────────────────────────────────────────────┘
```

**Confirmation dialog**

```
┌──────────────────────────────────────────────┐
│ Genixco AI wants to perform this action:     │
│                                              │
│ Action:  delete_path                         │
│ Delete a file or folder (always confirmed).  │
│                                              │
│ Parameters:                                  │
│ {                                            │
│   "path": "C:\\Users\\User\\Documents\\a.txt"│
│ }                                            │
│                                              │
│                    [ Cancel ]  [ Confirm ]   │
└──────────────────────────────────────────────┘
```

---

## Quick Start

### 1. Requirements

| | Packages | Purpose |
|---|---|---|
| **Python** | 3.10 or newer (tested on 3.13) | — |
| **Required** | `PySide6`, `requests` | UI + HTTP client |
| **Recommended** | `keyring` | API key in Windows Credential Manager |
| | `psutil` | process list, RAM/disk stats |
| **Optional** | `pywin32` | deeper Windows integration |

### 2. Install

```bash
pip install PySide6 requests
pip install keyring psutil        # strongly recommended
```

### 3. Run

```bash
python main.py
```

### 4. First launch

The **Setup Wizard** appears:

```
Welcome to Genixco AI

Provider:  [ OpenRouter ▼ ]
API Key:   [ ••••••••••••••••••••••• ]
Model ID:  [ openai/gpt-4o-mini ]

[ Test Connection ]        [ Skip ]  [ Save & Continue ]
```

`Test Connection` sends a **real** request to OpenRouter. You can `Skip` and use the app without AI — settings, tray and the floating button all still work.

### 5. Try it

```
You:  Open my Downloads folder
You:  Create a folder named Projects on my Desktop
You:  What's my CPU and RAM usage?
You:  Open youtube.com
You:  Read C:\Users\Me\notes.txt and summarise it
You:  Copy this to my clipboard: meeting at 14:30
```

---

## Configuration

### API key resolution order

```
1. Environment variable   OPENROUTER_API_KEY     ← highest priority
2. Secure storage         Windows Credential Manager (keyring)
3. Fallback file          %APPDATA%\GenixcoAI\secret.dat   (obfuscated)
```

Set it as an environment variable (recommended for power users):

```powershell
setx OPENROUTER_API_KEY "sk-or-v1-your-key-here"
```

> The key is **never** written to `settings.json`, **never** hardcoded, **never** logged, **never** included in exports, and **never** rendered anywhere in the UI except the masked input you typed it into.

### Provider settings

| Setting | Default | Notes |
|---|---|---|
| Provider | `OpenRouter` | abstraction ready for more providers |
| Model ID | `openai/gpt-4o-mini` | any valid OpenRouter model ID |
| Base URL | `https://openrouter.ai/api/v1` | fully editable (proxies, gateways) |
| Temperature | `0.7` | 0.0 – 2.0 |
| Max tokens | `4096` | 64 – 200000 |
| Timeout | `60s` | 5 – 600 |
| Streaming | `On` | SSE token streaming |

### Model suggestions

```text
openai/gpt-4o-mini              fast + cheap, great default
anthropic/claude-sonnet-4       strongest tool reasoning
google/gemini-2.5-flash         very fast, long context
meta-llama/llama-4-maverick     open-weight option
```

> Tool calling uses a plain-JSON protocol, so it works with **any** chat model — no native function-calling support required. Stronger models simply follow it more reliably.

---

## Architecture

```
          User
            │
            ▼
  ┌────────────────────┐
  │  Genixco AI UI     │  FloatingButton · ChatWindow · SettingsWindow · Tray
  └─────────┬──────────┘
            │  QThread / Signals-Slots   (UI never blocks)
            ▼
  ┌────────────────────┐
  │   AIProvider       │  OpenRouterProvider → /chat/completions (stream)
  └─────────┬──────────┘
            │  model reply
            ▼
  ┌────────────────────┐
  │   Tool Parser      │  extract  {"tool": ..., "arguments": {...}}
  └─────────┬──────────┘
            ▼
  ┌────────────────────┐
  │   Validation       │  known tool? argument types? value sanity?
  └─────────┬──────────┘
            ▼
  ┌────────────────────┐
  │ PermissionManager  │  master switch · category flags · path rules · allow-list
  └─────────┬──────────┘
            ▼
  ┌────────────────────┐
  │   Confirmation     │  modal dialog for sensitive actions
  └─────────┬──────────┘
            ▼
  ┌────────────────────┐
  │   WindowsTools     │  real OS execution
  └─────────┬──────────┘
            ▼
  ┌────────────────────┐
  │   Tool Result      │  {success, tool, message, data|error}
  └─────────┬──────────┘
            ▼
       back to the AI  →  natural-language answer grounded in reality
```

### Internal layout of `main.py`

```
main.py
│
├── Imports & Constants          brand, defaults, known apps, danger patterns
├── AppConfig                    resolves %APPDATA%\GenixcoAI (never hardcoded)
├── RedactingFormatter / Logger  rotating logs with secret scrubbing
├── SettingsManager              atomic JSON persistence, export/import, reset
├── SecureStorage                keyring → env var → obfuscated fallback
├── PermissionManager            allow/deny + confirmation policy + path security
├── CommandExecutor              validated shell execution with timeout
├── WindowsTools                 the 16 real implementations
├── ToolRegistry                 specs, catalogue rendering, dispatch
├── AIProvider (interface)       send_message / stream_message / test_connection
├── OpenRouterProvider           HTTP client, retries, SSE streaming
├── ChatManager                  conversation state + history files + export
├── ThemeManager                 dark/light stylesheet generator
├── ChatWorker (QThread)         the reason→tool→result loop
├── TestConnectionWorker         non-blocking connection test
├── ConfirmationDialog           sensitive-action approval
├── FloatingButton               draggable always-on-top AI button
├── MessageBubble / ChatInput    chat primitives
├── ChatWindow                   main interaction window
├── SettingsWindow               9 tabs incl. Diagnostics
├── SetupDialog                  first-run wizard
├── SystemTray                   tray icon + full menu
├── MainApplication              wiring, lifecycle, cleanup
└── main()                       entry point
```

---

## Tool Catalogue

| Tool | Category | Sensitive | Description |
|---|---|:--:|---|
| `get_system_info` | system | | OS, CPU, RAM, disks |
| `get_current_time` | system | | local date & time |
| `get_running_processes` | system | | top processes by memory *(needs psutil)* |
| `launch_application` | app | ● | launch a known/resolvable application |
| `open_url` | browser | | open an http/https URL in the default browser |
| `open_folder` | file | | open a folder in File Explorer |
| `open_file` | file | ● | open a file with its default program |
| `read_text_file` | file | | read UTF-8 text (5 MB cap) |
| `write_text_file` | file | ● | create/overwrite a text file |
| `create_folder` | file | ● | create a folder |
| `list_directory` | file | | list folder contents |
| `delete_path` | file | ● | delete file/folder — **always** confirmed |
| `clipboard_get` | clipboard | | read clipboard text |
| `clipboard_set` | clipboard | | write clipboard text |
| `take_screenshot` | screenshot | ● | capture screen to PNG |
| `execute_command` | shell | ● | run a shell command — **disabled by default** |

Each tool returns exactly one shape:

```jsonc
// success
{ "success": true,  "tool": "launch_application", "message": "Chrome launched.", "data": { "path": "..." } }

// failure
{ "success": false, "tool": "launch_application", "message": "Could not find 'foo' on this system. It was not launched.", "error": "application not found" }
```

---

## Tool Protocol

The model is told to reply with **only** this when it wants an action:

```json
{
  "tool": "launch_application",
  "arguments": { "application": "notepad" }
}
```

The parser tolerates fenced code blocks, then the pipeline runs. Raw JSON is **never** streamed into the chat — the user sees intent, a ✓/✗ tool line, and the final natural-language answer.

Only tools that are currently **enabled** are advertised to the model, so disabling a capability in Settings removes it from the AI's vocabulary entirely.

---

## Security Model

### 1. Layered permission
```
enable_automation (master)  →  category flag  →  allow-list / path rules  →  confirmation  →  execute
```
A failure at any layer returns a structured error to the AI — it is never silently skipped, and never faked as success.

### 2. Protected paths
Blocked by default (editable in Settings → Security):
```
C:\Windows        C:\Program Files        C:\Program Files (x86)
C:\ProgramData\Microsoft                  /etc  /bin  /usr  /boot
```
Paths are expanded and fully resolved before matching, so `..` traversal cannot escape the rule.

### 3. Shell hardening
- `Allow Shell Commands` is **OFF** by default.
- Destructive patterns are blocked outright: `format`, `del /s`, `rd /s`, `rm -rf`, `shutdown`, `reg delete`, `diskpart`, `vssadmin`, `bcdedit`, `cipher /w`.
- Hard timeout, captured `stdout`/`stderr`, non-zero exit reported as failure.
- The model can never run arbitrary Python — no `eval`, no `exec`, no free-form code path.

### 4. Application safety
Unknown applications are **not** launched blindly — they must be a known app or resolvable on `PATH`. An optional allow-list restricts launching to named apps only.

### 5. Prompt-injection resistance
File contents, command output and web text are returned as **DATA** with an explicit warning, and the permission engine is completely independent of anything the model says. A malicious `readme.txt` cannot grant itself shell access.

### 6. Secret hygiene
```
source code   ✗ never          logs        ✗ redacted
settings.json ✗ excluded       exports     ✗ excluded
chat history  ✗ redacted       exceptions  ✗ sanitised
```

---

## Settings Reference

<details>
<summary><b>General</b></summary>

Start with Windows · Minimize to tray · Always on top · Language · Theme · Animations · Notifications · Global hotkey
</details>

<details>
<summary><b>Floating Button</b></summary>

Enable · Always on top · Show on startup · Opacity · Size · Corner radius · Saved position · Reset position
</details>

<details>
<summary><b>AI Provider</b></summary>

Provider · API Key (masked, show/hide, key-source indicator) · Model ID · Base URL · Temperature · Max tokens · Timeout · Streaming · **Test Connection**
</details>

<details>
<summary><b>AI Behavior</b></summary>

Editable System Prompt (+ restore default) · Maximum tool calls per turn · Ask before sensitive actions · Confirm file operations · Confirm application launch · Confirm command execution
</details>

<details>
<summary><b>Windows Integration</b></summary>

Master automation switch · Application launching · File operations · System information · Clipboard · Browser actions · Screenshots
</details>

<details>
<summary><b>Security</b></summary>

Allow shell commands · Require confirmation · Log tool actions · Allowed applications list · Restricted paths list
</details>

<details>
<summary><b>Appearance</b></summary>

Theme (Dark / Light) · Accent color (hex) · Window opacity · Font size · UI scale
</details>

<details>
<summary><b>Storage</b></summary>

Save chat history · Clear chat history · Export conversation (Markdown) · Open logs folder · Export settings (no secrets) · Import settings · Reset all settings
</details>

<details>
<summary><b>Diagnostics</b></summary>

```
Genixco AI Diagnostics

Version:          1.0.0
Python:           3.13.x
OS:               Windows 11
Provider:         OpenRouter
API Key:          Configured
Key storage:      Windows Credential Manager
Model:            openai/gpt-4o-mini
Streaming:        On
AI:               Enabled
Automation:       Active
Tools enabled:    15
Shell commands:   Blocked
Floating button:  Enabled
```
*The real API key is never displayed here.*
</details>

---

## Data & File Locations

```
%APPDATA%\GenixcoAI\
├── settings.json          all settings — contains no secrets
├── secret.dat             fallback key store (only if keyring is unavailable)
├── logs\
│   └── genixco.log        rotating, 1 MB × 3, fully redacted
└── history\
    └── chat_<id>.json     one file per conversation
```

On non-Windows systems the root falls back to `$XDG_CONFIG_HOME/GenixcoAI` or `~/.config/GenixcoAI`. No user path is ever hardcoded.

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Enter` | Send message *(configurable)* |
| `Shift + Enter` | New line |
| `Ctrl + Enter` | Always send |
| `Ctrl + Shift + Space` | Open the assistant *(configurable)* |
| `Tab` | Keyboard navigation throughout the UI |
| Left-click the **AI** button | Open assistant |
| Right-click the **AI** button | Tray menu |
| Drag the **AI** button | Reposition (auto-saved) |

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Could not connect to OpenRouter` | Check internet, proxy, and the Base URL in Settings |
| `Invalid or unauthorized API key` | Re-enter the key; if `OPENROUTER_API_KEY` is set it overrides the UI value |
| `Model not found` | Verify the exact model ID on openrouter.ai |
| `Insufficient credits` | Top up your OpenRouter account |
| Request times out | Raise **Timeout**, or disable streaming for slow networks |
| Tool says "capability is disabled" | Settings → Windows Integration / Security |
| Shell commands blocked | Settings → Security → Allow shell commands (understand the risk) |
| Floating button invisible | Tray → *Show / Hide AI Button*, or Settings → *Reset position* |
| `get_running_processes` fails | `pip install psutil` |
| Key not in Credential Manager | `pip install keyring` |
| Nothing at all happens | Check `%APPDATA%\GenixcoAI\logs\genixco.log` — no exception ever crashes the app |

---

## Limitations

Stated honestly, because faking capability is against this project's design rules.

- **Global hotkey** is registered as a Qt application shortcut: it fires while a Genixco window has focus. A true system-wide hook needs `RegisterHotKey` + a native message loop, deliberately omitted for stability.
- **Secure key storage** requires `keyring`. Without it, the fallback file is base64 obfuscation — not strong encryption.
- **`get_running_processes`** requires `psutil` and returns an explicit error otherwise (no silent fake data).
- **Registry access** is limited to the `HKCU\...\Run` startup entry. No general registry tool is exposed to the AI, by design.
- **Tool calling** uses a text-JSON protocol rather than native function calling, for universal model compatibility. Weak models may occasionally emit invalid JSON — the reply is then shown as ordinary text.
- **Windows-specific actions** (launching `notepad`, `chrome`, File Explorer) only make sense on Windows. The app still runs on Linux/macOS, but Windows apps will not resolve.
- **UI scale** changes require a restart, since Qt reads the scale factor before the application object exists.

---

## Roadmap

- [ ] Native global hotkey via `RegisterHotKey`
- [ ] Additional providers: OpenAI · Anthropic · Ollama · Local LLM · Custom API *(the `AIProvider` interface is already in place)*
- [ ] Conversation browser with search
- [ ] Voice input / output
- [ ] Per-tool granular permission profiles
- [ ] Packaged `.exe` build via PyInstaller

---

## FAQ

**Is it really a single file?**
Yes — `main.py`, ~2,800 lines, no external assets. The icon and the floating button are painted in code.

**Does the AI run code on my machine?**
No. It can only request one of the predefined tools. Everything else is impossible by construction.

**Can it lie about completing an action?**
The system prompt forbids it, and structurally it can't help: the UI prints the actual tool result (`✓`/`✗`) before the model's sentence, and the model only sees the real result object.

**Does anything leave my computer?**
Only your chat messages, to OpenRouter, using the model you chose. Files are read locally and only included if a tool you permitted returned them.

**Can I use a local model?**
Point **Base URL** at any OpenAI-compatible endpoint (e.g. an Ollama or LM Studio gateway) and set the matching model ID.

**How do I fully uninstall?**
Delete `main.py`, remove `%APPDATA%\GenixcoAI\`, turn off *Start with Windows*, and delete the `GenixcoAI` credential from Windows Credential Manager.

---

## راهنمای فارسی

**Genixco AI** یک دستیار هوشمند دسکتاپ برای ویندوز است که توسط **Genix** ساخته شده و کل آن در یک فایل `main.py` قرار دارد.

**نصب و اجرا**

```bash
pip install PySide6 requests keyring psutil
python main.py
```

**ویژگی‌های کلیدی**

- **دکمه شناور AI**: گرد، شفاف، همیشه روی سایر پنجره‌ها، قابل جابه‌جایی با ماوس، با ذخیره موقعیت و پشتیبانی از چند مانیتور.
- **چت با استریم زنده**: پاسخ مدل کلمه‌به‌کلمه نمایش داده می‌شود و رابط کاربری هرگز فریز نمی‌شود.
- **۱۶ ابزار واقعی ویندوز**: اجرای برنامه، باز کردن URL و پوشه، خواندن و نوشتن فایل، ساخت و حذف پوشه، کلیپ‌بورد، اسکرین‌شات، اطلاعات سیستم و اجرای دستور شل.
- **امنیت چندلایه**: کلید سوئیچ اصلی → مجوز دسته‌ای → مسیرهای محدودشده → تأیید کاربر → اجرا.
- **کلید API امن**: از طریق `OPENROUTER_API_KEY` یا Windows Credential Manager؛ هرگز در سورس، لاگ، تنظیمات یا خروجی Export ذخیره نمی‌شود.
- **بدون ادعای دروغ**: هوش مصنوعی فقط زمانی موفقیت را اعلام می‌کند که ابزار واقعاً موفق شده باشد.

**تنظیم کلید API**

```powershell
setx OPENROUTER_API_KEY "sk-or-v1-..."
```

یا از مسیر: `Settings → AI Provider → API Key → Test Connection`

**مسیر فایل‌ها**

```
%APPDATA%\GenixcoAI\settings.json      تنظیمات (بدون هیچ کلیدی)
%APPDATA%\GenixcoAI\logs\              لاگ‌های پاک‌سازی‌شده
%APPDATA%\GenixcoAI\history\           تاریخچه گفتگوها
```

**نکته امنیتی**: اجرای دستورات شل به‌صورت پیش‌فرض **خاموش** است و دستورات مخرب حتی پس از فعال‌سازی نیز مسدود می‌مانند.

---

## License

Proprietary — © Genix. All rights reserved.
Use of the OpenRouter API is subject to OpenRouter's own terms and pricing.

<div align="center">

---

**Genixco AI** · built by **Genix**
*Real actions. Real results. No fabrication.*

</div>
