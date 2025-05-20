# Carbonio Service Monitor

This Python script is designed to monitor the status of Carbonio services on a Ubuntu-based server. It regularly checks the status of core Carbonio modules using the `zmcontrol status` command and logs the results. Additionally, it attempts to restart critical services if they are found to be stopped.

## Features

- Runs `zmcontrol status` and parses the output in real-time
- Logs service status with timestamps
- Automatically restarts critical services if they are not running:
  - `mailbox` → restarted with `zmmailboxd start`
  - `config service` → restarted with `zmconfid start`

## Prerequisites

- Python 3
- Carbonio mail server installed and properly configured
- Script must be run as a user with permission to execute zmcontrol and related commands (e.g., `zextras`, `carbonio`, or `root`)
- Optional: passwordless sudo access for specific Carbonio commands

## Installation

1. Copy the script `carbonio_monitor.py` to a directory of your choice (e.g., `/usr/local/bin`).

2. Make the script executable:

```bash
chmod +x /usr/local/bin/carbonio_monitor.py
```

3. Adjust logging path in the script if needed:
```python
LOG_FILE = "/home/youruser/carbonio_services.log"
```

4. (Optional) Configure sudoers to allow running Carbonio commands without a password. Run `sudo visudo` and add:

```bash
youruser ALL=(carbonio) NOPASSWD: /opt/zextras/bin/zmcontrol, /opt/zextras/bin/zmmailboxd, /opt/zextras/bin/zmconfid
```

(replace `carbonio` or `zextras` with the actual system user who owns the Carbonio commands)

## Usage

You can run the script manually:

```bash
python3 /path/to/carbonio_monitor.py
```

Or set up a scheduled job via crontab to monitor every 5 minutes:

```bash
crontab -e
```

Add this line:

```cron
*/5 * * * * /usr/bin/python3 /path/to/carbonio_monitor.py
```

## Log Output

Service statuses and restart attempts are logged with timestamps. Example:

```
[2025-05-20 02:05:18] Service 'mailbox' status: Running
[2025-05-20 02:05:18] Service 'config service' status: NOT running
[2025-05-20 02:05:18] Attempting to restart 'config service'...
```

## Troubleshooting

- If the script logs "unknown user" or fails to execute `zmcontrol`, verify that the target user (e.g., `zextras`) exists and has permission to run Carbonio management commands.
- Ensure the path to `zmcontrol`, `zmmailboxd`, and `zmconfid` is correct.


