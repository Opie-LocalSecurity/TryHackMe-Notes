# Windows Command Line

## Basics & Help

- **PATH**: `set` → shows environment vars; look for `Path=` to know where commands can run.
- **OS version**: `ver` (e.g., `Microsoft Windows [Version 10.0.17763.1821]`).
- **Detailed system info**: `systeminfo | more` → hostname, OS edition/build, CPU, RAM, etc.
- **Help & screen control**: `command /?` (per-command help), `help`, `cls`.
- **Paging long output**: `... | more` (Space = page, Enter = line, `Ctrl+C` to exit).

## Networking – Configuration & Troubleshooting

- **Adapter config**:
    - `ipconfig` → IPv4/IPv6, mask, gateway.
    - `ipconfig /all` → DHCP/DNS, MAC, leases, suffixes.
- **Reachability**: `ping target` → ICMP echo + RTT stats (basic connectivity).
- **Path to host**: `tracert target` → hop-by-hop route (TTL-based).
- **DNS lookup**:
    - `nslookup host` (default resolver)
    - `nslookup host 1.1.1.1` (specific resolver)
- **Connections & listeners**:
    - `netstat` → established connections.
    - `netstat -a` (all), `b` (owning exe), `o` (PID), `n` (numeric).
    - Common combo: `netstat -abon`.

## File & Disk – Navigation & Manipulation

- **Where am I?** `cd` (prints current dir).
- **List contents**: `dir`
    - `dir /a` (include hidden/system)
    - `dir /s` (recurse)
- **Tree view**: `tree`
- **Change dir**: `cd target_dir`, `cd ..` (up), `cd \` (root).
- **Directories**: `mkdir name`, `rmdir name` (empty dir only).
- **View text**: `type file.txt` (dump), `more file.txt` (paged).
- **Copy/move/delete**: `copy src dest`, `move src dest`, `del file` / `erase file`.
- **Wildcards**:  and `?` (e.g., `copy *.md C:\Markdown`).

## Tasks & Processes

- **List processes**: `tasklist`
    - Filter: `tasklist /FI "imagename eq sshd.exe"` (more filters via `tasklist /?`).
- **Kill by PID**: `taskkill /PID 4567` (add `/F` to force if needed).
- **Cross-reference**: use `netstat -o` to grab PID for a connection, then `tasklist`/`taskkill`.

## Useful but “omitted” commands to remember

- `chkdsk` → scan/repair file system & bad sectors.
- `driverquery` → enumerate installed drivers (page with `| more`).
- `sfc /scannow` → verify/repair protected system files.

## Patterns & Shortcuts

- **Help everywhere**: `/?` works on nearly all commands.
- **Page long output**: either `... | more` or `more file.txt`.
- **Admin vs user**: some commands/flags require elevated Command Prompt.
- **Numeric vs names**: add `n` in network tools to avoid DNS latency.