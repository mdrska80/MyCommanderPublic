# MyCommander

**MyCommander** is a fast two-pane terminal file manager written in Rust.
It is inspired by Total Commander, M602, Norton Commander, and Midnight Commander, but aims for a modern TUI workflow: keyboard-first navigation, optional mouse support, remote panels, archives as folders, built-in viewers/editors, and developer tools directly inside the file manager.

![MyCommander](mycommander.png)

## Core Idea

MyCommander is a commander-style application for people who want to manage files quickly without constantly switching between windows. The foundation is two independent panels, classic function-key shortcuts from `F3` to `F10`, fast search, archive browsing, and a consistent control model for both local and remote filesystems.

>[!info] NERD fonts
> For best experience please use **Nerd icons** in your terminal: https://www.nerdfonts.com/font-downloads

The project is built on:

- **Rust stable** for performance and safety.
- **Ratatui + Crossterm** for terminal UI, keyboard input, and mouse handling.
- **Tokio** for asynchronous operations, copying, networking, and background workers.
- **TOML configuration** for themes, columns, filters, tools, and associations.
- A modular architecture where each major feature area lives in its own module.

## Quick Shortcuts

| Shortcut | Action |
|---|---|
| `Tab` | Switch active panel |
| `F3` | Viewer |
| `F4` | Editor |
| `F5` | Copy |
| `F6` | Move / rename |
| `F7` | New directory |
| `F8` / `Del` | Delete |
| `F10` / `Alt+X` | Quit |
| `Ctrl+P` | Quick Open |
| `Ctrl+Shift+P` / `Ctrl+Alt+P` | Command Palette |
| `Ctrl+Q` | Quick View in the opposite panel |
| `Alt+F1` / `Alt+F2` | Select drive or connection for the left/right panel |
| `Alt+F7` | Find files |
| `Ctrl+Down` / `~` | Command line |
| `Ctrl+O` | Toggle log/terminal overlay |
| `Ctrl+C` / `Ctrl+X` / `Ctrl+V` | System file clipboard |
| `Num+` / `Num-` / `Num*` | Select, unselect, invert selection |

## Feature Categories

### Two-Pane Interface

- Two independent side-by-side panels with their own path, cursor, history, and selection.
- Switch panels with `Tab` or a mouse click.
- Per-panel path history via `Alt+Left` and `Alt+Right`.
- Optional panel synchronization to show the same path in the opposite panel.
- Panel header with the current path and free disk space.
- Columns for name, extension, size, modified date, and permissions/attributes.
- Configurable column order and widths.
- Sorting by column header click or with `Ctrl+F3` through `Ctrl+F6`.
- Panel tabs for working in multiple paths at once.
- Tab overview for cases where more tabs are open than fit in the panel.
- Branch/flat view via `Ctrl+B`, showing recursive content as a single list.

### File Operations

- Copy with `F5`, including confirmation, progress, speed, ETA, and cancellation.
![Copy progress dialog](<CopyDialog.png>)

- Move and rename with `F6`.
- Delete with `F8` or `Del`, including confirmation.
- Create a directory with `F7`.
- Create a new text file with `Shift+F4` and open it immediately in the editor.
- Inline rename with `Shift+F6`, preselecting the filename without the extension.
- Rename through a dialog with `F2`.
- Calculate directory size asynchronously with `Space`, using a cache.
- File or directory properties dialog via `Alt+Enter`.
![Properties dialog](<Properties.png>)
- Change attributes where supported by the active backend.
- Open files with an external program according to configured associations.

### Where Am I?
- Insert the filename under the cursor into the command line with `Ctrl+Enter`.
- Insert the full path into the command line with `Ctrl+Shift+Enter`.
- Result can be selected and copied out or you can click icon to copy it.

### Background Workers

- Copying can be moved to the background from the copy dialog with `Ctrl+J`.
- Background copy jobs are minimized into the bottom area of the UI.
![Background copy jobs in bottom bar](<Jobs.png>)
- Pressing `Ctrl+J` again opens the **Operation Queue**, where active jobs are visible.
![Operation Queue](<JobsDialog.png>)

Running jobs can be controlled with:

- `P` - pause/resume
- `C` - cancel
- `D` - discard/remove
- `R` - retry after failure

### Tabs
![[Tabs.png]]
- List of files supports tabs
- List of terminals/consoles support tabs
- Document workspace support tabs
- `CTRL+T` to add new tab
- by clicking at order/total tabs you get list of tabs to quick select
![[TabsQuickList.png]]
- by clicking + you can also add new tab

### Selection, Filters, and Colors

- Select items with `Space`, `Ins`, `Ctrl+A`, `Ctrl+click`, and `Shift+click`.
- Range selection with `Shift+Up/Down/PgUp/PgDn/Home/End`.
- `Num+` opens a dialog for selection by mask or saved filter.
- `Num-` unselects items by mask or filter.
- `Num*` inverts the current selection.
- Saved filters can match by wildcard, regex, extension, type, size, date, age, attributes, and logical combinations.
- Filter manager via `Ctrl+Shift+F` for creating, editing, and deleting filters.
- Filters can have their own colors and act as visual categorization directly in the panel.
- Color rules in configuration can match by file type, extension, regex, size, age, and attributes.
- File type detection uses magic bytes, extensions, and a text/binary fallback.

### Search and Fast Navigation

#### Inline Quick Search

- Inline quick search starts when typing a regular character in a panel.
![Inline quick search](<Filter.png>)
The active query is shown at the bottom. Matching items are highlighted in the file list. You can move between matches with the arrow keys. The typed text does not have to match only the beginning of the filename.

- Jump mode in the style of Total Commander: typing moves the cursor to the matching item.
- Matched parts of the filename are highlighted directly in the panel.
- Quick search has independent state per panel and also works inside archives and remote panels.

#### Quick Open

![Quick Open](<QuickOpen.png>)

- Quick Open via `Ctrl+P` quickly finds an item in the active panel and its subdirectories.
- Quick Open supports multi-token search, recent items, recursive local scanning, and `Enter`, `F3`, `F4` actions.

#### Command Palette

![Command Palette](<CommandPalette.png>)

- Command Palette via `Ctrl+Shift+P` or `Ctrl+Alt+P`.
- The Command Palette is context-aware: it offers different actions for files, directories, selections, archives, remote panels, the viewer, the editor, the REST client, or the terminal overlay.
- The main idea is simple: if you know what you want, you should not have to remember where it lives in the menu.

#### Standard File Search

![Find Files dialog](<FindFiles.png>)

- `Alt+F7` searches by name, partial name, file contents, and date.
- Search runs asynchronously, streams results as they are found, can be cancelled, and can feed results into a panel or save them as a list.

### Viewer and Editor

- Built-in viewer with `F3`, without leaving the application.
- Text viewer with scrolling, line numbers, tab expansion, word wrap, and syntax highlighting.
- Encoding picker for UTF-8, UTF-16, Windows-1250/1251/1252, ISO-8859, Shift-JIS, GBK, and more.
- Per-path preferred encoding storage.
- Text selection in the viewer by mouse or keyboard, copied with `Ctrl+C`.
- Image viewer with terminal graphics protocol support and fallback rendering.
- Hex viewer for binary files.
- Built-in editor with `F4`.
- Editor supports cursor movement, text selection, clipboard, undo/redo, search, replace, save, and a status line.
- Hex editor for binary edits with explicit save confirmation.
- External viewer/editor support based on configurable rules for extensions and file types.

### Document Workspace

![Document Workspace](<DocumentWorkspace.png>)

- A workspace showing all currently open documents, both viewers and editors.
- Documents are stored in separate tabs.
- Clicking the open-tab count opens a quick tab switcher and closer.
![Document Workspace tab switcher](<DocumentWorkspaceTabs.png>)
- Available via `F12`.

### Quick preview
- reachable via `CTRL+Q`
- in one pane list of files is displayed, in other pane is preview
- for text files you can see text. Syntax highlighted if possible
- for binary files you can see hex preview.
- for images you can see image
- for pal files you can see palette

### Archives

- Archives behave like virtual folders.
- Pressing `Enter` on a supported archive opens its contents in the panel without manual extraction.
- The virtual path in the panel shows the current location inside the archive.
- Navigation inside archives uses the same controls as normal directories.
- `F3` can view a file inside an archive without extracting it manually.
- `F4` can edit a file inside an archive without extracting it manually. It is automatically updated after save.
- `F5` copies files out of an archive as extraction.

#### Supported formats
- .zip, .jar, .war, .ear, .apk, .epub, .docx, .xlsx, .pptx, .odt, .ods, .odp, .cbz
- .tar, .tar.gz, .tgz, .tar.bz2, .tbz, .tbz2, .tar.xz, .txz

### Remote and Network Panels

- FTP panel with active and passive modes.
- FTPS via explicit TLS.
- FTP upload/download uses the same progress dialog as local copy.
- Copying between two FTP panels through a local buffer.
- SSH/SFTP login with password or private key, including passphrase support.
- SFTP panels use the same panel behavior as the local filesystem.
- SMB/Samba panel with NTLM login, domain support, saved connections, navigation, copying, and basic operations.
- Drive menu via `Alt+F1` and `Alt+F2` shows local drives, mount points, and open FTP/SSH/SMB connections.
- Open network connections can be closed from the drive menu.
- Refresh a remote panel with `Ctrl+R`.
- `F3` can view a file inside an archive without extracting it manually.
- `F4` can edit a file inside an archive without extracting it manually. It is automatically updated after save.

#### Command Line `cd` Behavior

The inline terminal can use these commands for quick connections:

- `cd ftp://user@host` for a quick FTP connection
- `cd sftp://user@host` for a quick SFTP connection
- `cd ssh://user@host` or `ssh user@host` for a quick SSH connection
- `cd smb://user:password@host/share` for a quick SMB connection
- `cd \\host\share` for a quick SMB connection; credentials are requested when needed

### IP Scanner

- IP scanner is a special panel mode, not a fake filesystem listing.
- Start it from the menu and enter a range such as `192.168.16.*`, `192.168.16`, `192.168.16.`, or `192.168.16.1-255`.
- Scans at most 255 addresses in the selected range.
- Results appear progressively without blocking the UI.
- A worker pool limits parallelism so the app does not spawn 255 threads or processes at once.
- Ping detects host availability.
- TCP probes check ports 20, 21, and 22.
- The panel shows IP, hostname, status, latency, FTP, SSH, and open ports.
- Reverse DNS fills in hostnames for reachable devices.
- `Refresh` repeats the same scan.
- `Backspace` or `..` exits the IP scan panel.
- SSH terminal can be opened with the host prefilled from the current IP row.
- Selected IP addresses can be exported or copied.
### SSH Terminal

- Interactive SSH terminal opens through `Net -> SSH Terminal...` or `Ctrl+Alt+T`.
- It uses the same dialog and saved connection data as SFTP.
- SSH terminal runs inside the terminal overlay, shown and hidden with `Ctrl+O`.
- The remote shell receives an `xterm-256color` PTY, so colors and fullscreen programs such as `vim`, `htop`, or `tmux` work.
- PTY size changes are forwarded to the remote session when the UI is resized.
- The terminal overlay supports multiple tabs, including a mix of local shells and SSH sessions.
- `Ctrl+PageUp` / `Ctrl+PageDown` switches terminal tabs.
- `Alt+1..9` jumps to a specific terminal tab.
- `Ctrl+F4` closes the active terminal tab.
- `Ctrl+Shift+T` opens a new local terminal tab.
- `Ctrl+F` opens search in terminal scrollback.
- `F7` searches the next match, `Shift+F7` the previous one.
- `Ctrl+C` copies selected text if a selection exists; otherwise it sends interrupt to the shell.
- `Ctrl+Shift+C` always sends interrupt to the SSH session.
- A disconnected SSH terminal tab stays in the tab list and can be reconnected with `Ctrl+R`.

#### Bookmarks and Quick Connections

- The SSH dialog has an integrated list of saved connections.
- `Insert` or `Ctrl+S` saves the current form as a bookmark.
- `Delete` removes the selected bookmark.
- `Ctrl+N` clears the form for a new connection.
- `Tab` switches between the form and the bookmark list.
- `Up` / `Down` moves between fields or bookmarks depending on the current focus.
- The command line accepts `cd sftp://user@host` for an SFTP panel.
- The command line accepts `cd ssh://user@host` or `ssh user@host` for SSH terminal.
- IP scanner can open SSH terminal with the host prefilled from the selected IP row.
- Saved connections are stored in the application configuration; passwords and passphrases should be handled with the same care as other local configuration files.

### REST/TCP Client

- REST/TCP client opens in the opposite panel with `Ctrl+Alt+R`.
- The primary panel selects the input file; the secondary panel shows the client.
- The request body is the content of the file under the cursor.
- Request configuration is stored next to the input file as a sidecar `*.toml`.
- Response is stored next to the input file as `*.response`.
- HTTP/HTTPS transport supports method, endpoint, headers, timeout, and body from file.
- TCP transport supports host, port, timeout, and an optional delimiter.
- Requests run in the background and can be cancelled.
- UI shows request state, HTTP status, and timing.
- Response preview uses syntax highlighting for JSON and XML.
- The form supports multi-line headers, clipboard, text selection, mouse input, and scrolling.
- When the file under the cursor changes, the REST client switches to the new context without breaking focus.

### Command Line and Terminal Overlay

- Inline command line via `Ctrl+Down` or `~`.
- Commands run in the current directory of the active panel.
- `cd <path>` is handled directly by the application and changes the active panel path.
- Supported variables: `%f`, `%d`, `%s`, `%F`, `%D`.
- Command history and file autocomplete with `Tab`.
- Mouse click can focus the command line.
- `Ctrl+O` hides the panels and shows the log/terminal overlay behind them.
- Terminal overlay is useful for command output, connection logs, long-running processes, and checking background work.

### Clipboard and OS Integration

- `Ctrl+C` copies selected files to the system clipboard.
- `Ctrl+X` marks selected files as cut.
- `Ctrl+V` pastes files into the current panel.
- Integration uses the native OS format, such as CF_HDROP on Windows.
- Clipboard works with Explorer, Finder, or Linux file managers.
- When pasting into the same directory, name collisions are resolved automatically with a unique name.
- Drag and drop uses the same copy/move behavior as clipboard operations.

### Mouse Support

- Clicking a panel changes focus.
- Clicking an item moves the cursor.
- Mouse wheel scrolls the list.
- Clicking a column header sorts the panel.
- Clicking function keys in the bottom bar runs the corresponding action.
- Drag and drop moves selected or dragged items.
- `Shift+click` selects a range.
- `Ctrl+click` toggles an individual item.
- Mouse support is optional; all important operations are available from the keyboard.

### Menu, Configuration, and Appearance

- Top menu bar in the style of commander applications.
- Menu groups: Files, Mark, Commands, Net, Show, Configuration, and Help.
- Menu can be controlled with `F9`, `Alt+<letter>`, arrow keys, Enter, Esc, or mouse.
- Each menu item shows its shortcut when one exists.
- Configuration dialogs for general settings, columns, colors, filters, associations, syntax highlighting, and external tools.
- Configuration is stored in `config.toml`.
- Color themes are based on `theme.rs` and TOML files in the `themes` directory.
- Unicode frames and Nerd Font icons.
- ASCII icon fallback for terminals without Nerd Font support.
- Color picker for convenient theme tuning.
- Toggle hidden files with `Ctrl+H`.

### Diff, Synchronization, and Utilities

- Compare by content for two files.
- Side-by-side diff viewer using the Myers diff algorithm.
- Synchronize directories for visual comparison of two panels.
- Job queue operations: progress, cancel, pause, retry, and discard depending on job type.
- Log overlay helps track output and diagnostics without leaving the application.

### Performance and Reliability

- UI stays responsive thanks to background workers and asynchronous operations.
- Long copies and network transfers share a unified progress model.
- Large directories use virtual scrolling.
- File type detection is cached in `FileEntry`.
- Expensive local operations run outside the main render loop.
- Release profile uses optimization, LTO, and strip.

## Installation and Running

```bash
git clone <repo-url>
cd MyCommander
cargo run --release
```

For development:

```bash
cargo check
cargo test
cargo clippy -- -D warnings
cargo fmt --check
```

## Configuration

On Windows, user configuration is typically located at:

```text
%APPDATA%\mycommander\config.toml
```

Configuration includes:

- general panel behavior,
- color theme,
- columns,
- file color rules,
- saved filters,
- keybindings,
- external viewers and editors,
- syntax highlighting,
- command line and tool settings.

## License

MIT
