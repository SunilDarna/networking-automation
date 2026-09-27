# Module 03: Automation Mechanics & Low-Level CLI Quirks

Unlike general-purpose Linux servers, network devices operate under strict CLI terminal constraints designed for human eyes, not automated scripts. Failing to account for these quirks causes intermittent pipeline failures and timeout crashes.

---

## 1. The Pagination Trap (The Infinite Hang)

### The Problem
Network switches paginate CLI output by default (pausing after 24 lines and printing `--More--` or `[Press space to continue]`).
When an automated script (like Ansible or Python Netmiko) executes `show running-config`, the switch pauses at line 24 waiting for a keyboard spacebar press. Because Ansible is non-interactive, the job hangs indefinitely until the SSH timeout threshold is exceeded (usually 300 seconds).

### The Solution
Automation frameworks must disable terminal pagination upon establishing the shell session:
- **Cisco IOS-XE / NX-OS:** `terminal length 0` and `terminal width 512`
- **Arista EOS:** `terminal length 0`
- **F5 tmsh:** `modify cli preference pager disabled`
- **Palo Alto PAN-OS:** `set cli pager off`

---

## 2. Prompt Regex Detection Failures

### The Problem
Ansible's `network_cli` connection plugin sends a command and then continually reads the incoming byte buffer until it matches a regular expression representing the prompt (e.g. `hostname#`).
If an enterprise MOTD login banner contains a `#` or `>` symbol (e.g., `### AUTHORIZED ACCESS ONLY ###`), Ansible prematurely assumes the shell prompt is ready, sends commands into the login banner, and fails.

### The Solution
In `ansible.cfg` or inventory group variables, enforce strict multiline anchors and explicit regex patterns:
```ini
[persistent_connection]
command_timeout = 60
connect_timeout = 30

[paramiko_connection]
record_host_keys = False
```

```yaml
ansible_terminal_initial_prompt_newline: true
ansible_terminal_prompt_regex: '^[a-zA-Z0-9_\-\.\:\/]+[#>] *$'
```

---

## 3. Direct Execution vs. Two-Phase Commits

| Feature | Direct CLI (Cisco IOS) | Transactional (Arista, PAN-OS, Cisco XR) |
| :--- | :--- | :--- |
| **Commit Timing** | Instant on Enter key | Explicit commit command |
| **Syntax Errors** | Partial execution risk | Zero production impact on error |
| **Blast Radius** | High | Low |
| **Rollback Support**| Manual inverse commands | Automated (`rollback` or `commit confirmed`) |

### Production Best Practice
Whenever possible, utilize **`commit confirmed <minutes>`** in automation pipelines. If the automation script loses reachability to the device after applying a routing change, the device automatically reverts to its baseline state after the timer expires!
