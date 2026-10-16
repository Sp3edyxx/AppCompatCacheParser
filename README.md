# AppCompatCacheParser

AppCompatCache (ShimCache) viewer with Sigcheck-style signature verification. Native Rust + egui, same look as SrumParser. One ~4.2 MB exe with no runtime (the old WPF build was 59 MB; its sources are in `legacy-wpf\`).

Reading the live value needs Administrator (`HKLM\...\AppCompatCache` is admin-only); a "Restart as Administrator" button appears if access is denied.

## Using it

- Toolbar left: search (all columns, case-insensitive), reload live registry, and the filters In Instance (files modified since boot), Show Untrusted (unsigned / tampered / invalid / expired / revoked only) and Show Not Found (files gone from disk only).
- Toolbar right: Live Registry, Open Hive, Export CSV. The space in between shows the row count and red warnings (a bare ⚠ when narrow); hover them for details.
- Columns: Last Modified, Name, Path (with the file's icon), Signature, Generics. Default order is the cache's own (most recent insertion first); click any header to sort, drag header edges to resize. Hover Path for location notes, Signature for the signer, a tag for its reason.
- Click / Ctrl / Shift select rows, arrows / PgUp / PgDn / Home / End move, right-click for Copy, Copy Path, Copy Signer, Open File Location, Filter by This Exe.

| Keys | |
|---|---|
| F5 · Ctrl+O | Live registry · open offline SYSTEM hive (or drop it on the window) |
| Ctrl+F · Esc | Search · clear search |
| Ctrl+I | In Instance |
| Ctrl+C · Ctrl+A | Copy selected rows (tab-separated) · select all |
| Ctrl+E | Filter by selected exe |
| Ctrl+S | Export current view to CSV |

## Reading

- Live: `HKLM\SYSTEM\ControlSet00N\Control\Session Manager\AppCompatCache`, N from `Select\Current` (64-bit view).
- Offline: built-in read-only `regf` reader (key lists lf/lh/li/ri, inline and big-data values). A hive with unreplayed `.LOG` data shows "⚠ Hive dirty".
- Formats: XP/2003, 7/2008 R2 (x86 and x64), 8/2012, 8.1/2012 R2, 10, 10 1703+/11. Vista layouts are rejected with a message. **Only the 10/11 layouts have been checked against real data;** the older parsers follow the documented layouts.
- Only entries that are real paths are listed (UWP package records and binary rows are dropped); `\??\` and `\\?\` prefixes are stripped. `SIGN.MEDIA=` entries are kept.
- Times: FILETIME stored UTC, shown in local time, sorted on UTC.

## Signatures

Same method as SrumParser / Sysinternals Sigcheck: embedded Authenticode via `WinVerifyTrust` (`WINTRUST_ACTION_GENERIC_VERIFY_V2`), otherwise Windows catalogs (`CryptCATAdmin*`). The PE Authenticode digest is also recomputed independently, so a patched file or a copied signature shows **Tampered** even when the PE SIP registry entry is hijacked (that hijack is detected and shown as a red warning). Offline: cached CRLs only, nothing uploaded. Results are cached per path + size + modified time in `%LOCALAPPDATA%\AppCompatCacheParser\sigcache.tsv`.

Files are not looked up for offline hives (they describe another disk) or on network drives.

## Generics tags

| Tag | Flags |
|---|---|
| UNT | Untrusted binary (unsigned / tampered / invalid / expired / revoked) outside `<drive>:\Windows`, `\Program Files`, `\Program Files (x86)` (`\Windows\Temp` counts as outside) |
| USB | Run from removable media: `SIGN.MEDIA=` entry, or a removable / CD drive |

## Why it is small and fast

Same as SrumParser: egui on OpenGL (glow), only Poppins Regular + Medium embedded (SIL Open Font License), Win32 bound with `raw-dylib`, static CRT, column-wise rows with only visible rows laid out, signatures and icons resolved once per distinct path on background threads, `opt-level = "s"` + fat LTO + `panic = "abort"`.
